# Moderation API

`Meridian.Moderation` exposes `GetBan`, `IsBanned`, `CheckAccess`, `RefreshBan`, `EnforceBan`, `Unban`, and `Reconcile`. Methods accept a Roblox `Player`, Meridian player object, or user ID where applicable.

## Ban lookup and access

```lua
local ban, err = Meridian.Moderation:GetBan(player)
local isBanned, activeBan, checkErr = Meridian.Moderation:IsBanned(player)
local allowed, accessBan, accessErr = Meridian.Moderation:CheckAccess(player)
```

`GetBan` returns the active DataStore ban or `nil`.

```lua
{
    active = true,
    banId = "ban_xxxxx",
    reason = "Exploiting",
    displayReason = "You have been banned.",
    createdAt = 1788180000,
    expiresAt = nil,
    revision = 6,
    enforcementMode = "Kick",
    userId = 123456789,
}
```

DataStore failures follow `FailOpen`. With `FailOpen = true`, a failed read does not cause an uncertain kick.

## Refresh and enforcement

```lua
local allowed, ban, err = Meridian.Moderation:RefreshBan(player)
Meridian.Moderation:EnforceBan(player)
```

`RefreshBan` reads current DataStore state and enforces an active ban against a tracked player. `EnforceBan` currently delegates to `RefreshBan`; developers generally do not need to call it.

## Unban

```lua
local removed, err = Meridian.Moderation:Unban(123456789)
```

This reads the active ban, calls Roblox `UnbanAsync` when native enforcement applies, removes `active:<userId>`, marks the tracked player unbanned, and retains historical `ban:<banId>` data.

## Reconciliation

```lua
local decisions, err = Meridian.Moderation:Reconcile({
    123456789,
    987654321,
})
```

Decisions are indexed by user ID:

```lua
{
    [123456789] = { allowed = true, ban = nil, revision = 7 },
    [987654321] = {
        allowed = false,
        ban = { active = true, banId = "ban_xxxxx", reason = "Exploiting" },
        revision = 4,
    },
}
```

Reconciliation deduplicates and validates IDs, sends batches of up to 100, uses a two-second cache, serializes overlapping calls, writes confirmed bans, enforces them online, and clears stale bans only when the API explicitly returns `allowed = true`. API failures or omitted users never remove bans.

It runs at startup, player join, after `player.banned` events, while processing the unban queue, every 90 seconds for connected players, and every 15 minutes for stored active bans.

## Ban storage

The `Meridian_Bans_v1` DataStore uses:

```text
active:<userId>
ban:<banId>
```

The active key is the enforcement source; historical records remain. Older revisions cannot overwrite newer bans. Expired bans are ignored and cleaned asynchronously.

## Enforcement modes

- `Kick` calls `player:Kick(ban.reason)` and shows only the stored reason.
- `RobloxBan` uses `Players:BanAsync()` in production. Studio falls back to kick.

```lua
RobloxBans = {
    Enabled = true,
    ApplyToUniverse = true,
    ExcludeAltAccounts = false,
    ApplyDeviceBlock = false,
}
```

## Unban queue

`Meridian_UnbanQueue_v1` stores `pending:<universeId>`. A grouped heartbeat event can contain up to 50 unique IDs:

```json
{ "type": "player.unbanned", "user_ids": [123456789, 987654321] }
```

The SDK persists the event, reconciles each user, removes confirmed stale bans, invokes `UnbanAsync` when required, acknowledges successes, and retains failures. If no game server exists, the SDK cannot invoke `UnbanAsync`; native bans should also be removed through backend Roblox Open Cloud integration.

