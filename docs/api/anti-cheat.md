# Anti-Cheat API

`Meridian.AntiCheat` provides detection reporting, risk state, movement contexts and grants, account exceptions, enforcement actions, configuration status, and upload control.

## Status and configuration

```lua
if Meridian.AntiCheat:IsEnabled() then
    print("Anti-Cheat enabled")
end

local config = Meridian.AntiCheat:GetConfig() -- read-only
local status = Meridian.AntiCheat:GetStatus()
```

Runtime dashboard configuration may override the local enabled value. Status includes `Enabled`, `Profile`, `Detectors`, `ActivePlayers`, `ApiConnected`, `DetectionQueue`, `DroppedDetections`, and `LastUpload`.

## Report detections

`Flag` is a simplified wrapper around `Detect`:

```lua
local detection = Meridian.AntiCheat:Flag(player, "economy.invalid_purchase", {
    ItemId = itemId,
    Balance = balance,
    Price = price,
})

local detection, err = Meridian.AntiCheat:Detect(player, {
    Type = "combat.impossible_damage",
    Severity = "critical",
    Confidence = 0.99,
    Evidence = { Damage = 500, MaximumDamage = 75 },
    Tags = { "combat", "damage" },
    Action = "flag",
})
```

Severities are `info`, `low`, `medium`, `high`, and `critical`. Confidence is clamped to 0–1. Detection names follow `<system>.<detection>`, such as `movement.speed`, `movement.teleport`, `combat.impossible_damage`, or `inventory.duplicate_item`.

A detection contains `Id`, `Type`, `UserId`, `Severity`, `Confidence`, `RiskAdded`, `RiskScore`, `Evidence`, `Tags`, `CreatedAt`, `RuleVersion`, `SDKVersion`, and `ServerId`.

Evidence is sanitized: strings are truncated, non-finite numbers removed, table depth/items bounded, instances reduced to class and name, and unsupported values removed. Never include secrets.

## Risk and incidents

```lua
local risk = Meridian.AntiCheat:GetRisk(player)
local incidents = Meridian.AntiCheat:GetIncidents(123456789)
```

Risk returns `Score`, `Level`, `RecentDetections`, and `LastDetectionAt`.

| Score | Level |
| ---: | --- |
| 0–19 | normal |
| 20–39 | suspicious |
| 40–69 | high |
| 70–89 | critical |
| 90–100 | severe |

Local risk can decay using `DecayInterval` and `DecayAmount`. The backend should calculate authoritative cross-server risk rather than trust the SDK-supplied total. Each player retains up to 50 recent local detections.

## Security contexts

Contexts suppress built-in movement analysis during legitimate unusual movement:

```lua
Meridian.AntiCheat:BeginContext(player, "Teleport")
character:PivotTo(destination)
Meridian.AntiCheat:EndContext(player, "Teleport")

Meridian.AntiCheat:SetContext(player, "Dash", { Duration = 2 })
```

Suggested names include `Teleport`, `Dash`, `Vehicle`, `Knockback`, `Launch`, `Respawn`, `Cutscene`, `Swimming`, `Flying`, `Admin`, `Physics`, and `Ability`. Custom names work; durations are bounded.

## Movement grants

```lua
local granted, err = Meridian.AntiCheat:GrantMovement(player, {
    MaxSpeed = 100,
    MaxAcceleration = 200,
    MaxVerticalSpeed = 150,
    Duration = 3,
    Reason = "DashAbility",
})

Meridian.AntiCheat:GrantTeleport(player, {
    Destination = destination,
    Radius = 10,
    Duration = 3,
})
character:PivotTo(CFrame.new(destination))
```

Current checks use `MaxSpeed` and `MaxVerticalSpeed`; `MaxAcceleration` is retained for forward compatibility. Teleport destinations must be `Vector3` values.

## Ignore accounts

```lua
Meridian.AntiCheat:Ignore(player, { Duration = 600, Reason = "Developer testing" })
Meridian.AntiCheat:Ignore(player) -- remainder of this server
Meridian.AntiCheat:Unignore(player)
```

Ignore is server-local and should be used sparingly.

## Actions

```lua
local kicked, err = Meridian.AntiCheat:Kick(player, {
    Reason = "Suspicious activity detected.",
})

local banned, ban = Meridian.AntiCheat:Ban(player, {
    Reason = "Exploiting",
    DisplayReason = "Exploiting",
    Duration = -1,
    Evidence = { DetectionId = detection.Id },
})

local limited, action = Meridian.AntiCheat:Limit(player, {
    Feature = "Trading",
    Duration = 300,
})
```

Ban creates active and historical DataStore records and enforces the configured mode. It is permanent when duration is absent or `-1`, temporary when positive. Because `AutoBan` defaults to `false`, automatic bans require explicit activation.

`Limit` only emits `PlayerLimited`; the game must enforce the restriction:

```lua
Meridian.Events.PlayerLimited:Connect(function(player, action)
    if action.Feature == "Trading" then
        TradingService:SetRestricted(player, true, action.Duration)
    end
end)
```

## Automatic policy

```lua
AntiCheat = {
    Actions = {
        ["movement.speed"] = { MinimumConfidence = 0.95, Action = "kick" },
        ["remote.unauthorized"] = { MinimumConfidence = 1, Action = "ban" },
    },
    AutoKick = true,
    AutoBan = false,
    Risk = { KickScore = 90, BanScore = 100 },
}
```

Automatic bans additionally require `AutoBan = true`, critical severity, confidence of at least `0.98`, and risk at or above `BanScore`.

## Movement detection

Built-in checks are `movement.speed`, `movement.teleport`, and `movement.vertical`. Detection is server-side, samples at most every 0.25 seconds by default, requires repeated speed/vertical anomalies, resets on character/root changes, ignores invalid time gaps, respects contexts/grants, bounds per-player state, and does not trust client telemetry.

Profiles are `lenient`, `balanced`, `strict`, and `custom`. Custom movement settings include `MaximumUnexpectedDistance` for teleport and `MaximumSpeed` for vertical movement.

## Detection queue

Defaults are a 5-second flush interval, batch size 50, and queue limit 500.

```lua
local uploaded, err = Meridian.AntiCheat:Flush()
```

Flush sends up to `BatchSize`, retains the batch during API failure, removes it after success, and drops the oldest item at the hard queue limit. Dropped count appears in status.

```text
POST /v1/sdk/anticheat/detections/batch
Authorization: Bearer <SDK session token>
```

