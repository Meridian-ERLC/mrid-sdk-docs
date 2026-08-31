# Events and logging

Signals support `event:Connect(callback)`, `event:Once(callback)`, and `event:Wait()`.

## Event catalog

| Area | Events |
| --- | --- |
| SDK | `Ready`, `ApiUnavailable`, `ApiRecovered` |
| Players | `PlayerReady`, `PlayerRemoved` |
| Moderation | `BanDetected`, `BanEnforced`, `ServerEvent` |
| Anti-Cheat | `AntiCheatDetection`, `AntiCheatIncident`, `AntiCheatAction`, `RiskChanged`, `PlayerLimited`, `PlayerKicked`, `PlayerBanned`, `FalsePositive` |

## Examples

```lua
Meridian.Events.Ready:Connect(function(status)
    print("Meridian ready:", status)
end)

Meridian.Events.ApiUnavailable:Connect(function(reason)
    warn("Meridian API unavailable:", reason)
end)

Meridian.Events.ApiRecovered:Connect(function()
    print("Meridian API recovered")
end)

Meridian.Events.PlayerReady:Connect(function(meridianPlayer)
    print(meridianPlayer.Username, "is ready")
end)

Meridian.Events.PlayerRemoved:Connect(function(meridianPlayer)
    print(meridianPlayer.Username, "left")
end)

Meridian.Events.BanDetected:Connect(function(player, ban)
    print(player.Name, ban.reason)
end)

Meridian.Events.BanEnforced:Connect(function(player, ban)
    print("Enforced:", player.Name, ban.banId)
end)

Meridian.Events.ServerEvent:Connect(function(event)
    print(event.type)
end)

Meridian.Events.AntiCheatDetection:Connect(function(detection)
    print(detection.Type, detection.UserId, detection.Confidence)
end)

Meridian.Events.RiskChanged:Connect(function(player, newRisk, oldRisk)
    print(player.Name, oldRisk.Score, newRisk.Score)
end)
```

## Logging

```lua
Meridian.Log:Info(...)
Meridian.Log:Warn(...)
Meridian.Log:Error(...)
Meridian.Log:Debug(...)

Meridian.Log:Info("Purchase completed", player.UserId, itemId)
```

Debug logs require `Debug = true`. The SDK intentionally avoids logging installation keys, authorization headers, session tokens, and full API response bodies.

