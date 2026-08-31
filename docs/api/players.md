# Players API

Access tracked players through `Meridian.Players`.

```lua
local meridianPlayer = Meridian.Players:Get(player)
local byId = Meridian.Players:GetByUserId(123456789)

for _, trackedPlayer in Meridian.Players:GetAll() do
    print(trackedPlayer.Username)
end

local count = Meridian.Players:Count()
```

## Meridian player object

```lua
{
    Player = player,
    UserId = 123456789,
    Username = "Player",
    JoinedAt = 1788180000,
    Banned = false,
}
```

The player object is intentionally small in the current SDK.

