# Security API

## Protect remotes

`ProtectRemote` owns the server handler for a `RemoteEvent` or `RemoteFunction` and requires a handler:

```lua
local PurchaseRemote = game.ReplicatedStorage.Remotes.Purchase

Meridian.Security:ProtectRemote(PurchaseRemote, {
    RateLimit = { Requests = 5, Window = 10, OnExceeded = "flag" },
    Arguments = {
        Meridian.Types.Integer({ Min = 1, Max = 1000000 }),
    },
    Validate = function(player, itemId)
        return Shop:CanPurchase(player, itemId)
    end,
    Handler = function(player, itemId)
        Shop:Purchase(player, itemId)
    end,
})
```

For `RemoteFunction`, Meridian replaces `OnServerInvoke` and warns if one exists. For `RemoteEvent`, avoid other unprotected `OnServerEvent` handlers because they can still receive invalid calls.

## Rate limits and validation

```lua
local allowed = Meridian.Security:RateLimit(player, "fire-weapon", {
    Requests = 10,
    Window = 1,
    OnExceeded = "flag",
})

local valid, reason = Meridian.Security:Validate(player, "purchase", {
    Item = itemId,
    Price = price,
}, function(data)
    return Shop:IsValidItem(data.Item)
end)
```

Rate-limit actions are `ignore`, `log`, `flag`, `kick`, and `ban`; `flag` is the recommended default. State is server-local and cleaned when the player leaves. Validation failures create `security.validation_failed` detections.

## Rules and permissions

```lua
Meridian.Security:RegisterRule("purchase.allowed", function(player, context)
    return Shop:CanPurchase(player, context.ItemId)
end)

local allowed, reason = Meridian.Security:CheckRule(player, "purchase.allowed", {
    ItemId = 42,
})

local permitted = Meridian.Security:CheckPermission(player, "admin.command", function(target, permission)
    return Permissions:Has(target, permission)
end)
```

Permission failures create `remote.unauthorized` detections.

## Types API

`Meridian.Types` constructors validate remote arguments:

```lua
Meridian.Types.String({ MinLength = 1, MaxLength = 50, Pattern = "^[%w_]+$" })
Meridian.Types.Number({ Min = 0, Max = 100 })
Meridian.Types.Integer({ Min = 1, Max = 1000000 })
Meridian.Types.Boolean()
Meridian.Types.Vector3({ MaxMagnitude = 10000 })
Meridian.Types.CFrame()
Meridian.Types.Enum({ EnumType = Enum.Material })
```

Numbers reject NaN and infinity. Instances support class, ancestry, and custom checks:

```lua
Meridian.Types.Instance({
    ClassName = "Tool",
    DescendantOf = workspace.Items,
    Validate = function(instance)
        return instance:GetAttribute("Purchasable") == true
    end,
})
```

Collections can be composed:

```lua
local ids = Meridian.Types.Array(
    Meridian.Types.Integer({ Min = 1 }),
    { MinLength = 1, MaxLength = 20 }
)

local order = Meridian.Types.Object({
    ItemId = Meridian.Types.Integer({ Min = 1 }),
    Quantity = Meridian.Types.Integer({ Min = 1, Max = 10 }),
    Coupon = Meridian.Types.Optional(Meridian.Types.String({ MaxLength = 20 })),
}, { AllowUnknown = false })
```

Sparse and dictionary-shaped arrays are rejected.

## Economy security

```lua
local result = Meridian.Security.Economy:ValidatePurchase(player, {
    ItemId = itemId,
    Price = price,
    Balance = balance,
})
```

Reasons include `invalid_data`, `invalid_purchase`, and `insufficient_funds`. Load authoritative prices and balances server-side; never trust client-supplied values.

```lua
local result = Meridian.Security.Economy:Transaction({
    Player = player,
    Type = "purchase",
    Validate = function()
        return playerData.Coins >= item.Price
    end,
    Commit = function()
        playerData.Coins -= item.Price
        Inventory:Add(player, item)
    end,
})
```

`Transaction` structures safe callback execution and monitoring; it does not replace correct DataStore `UpdateAsync` transaction design.

## Combat security

```lua
local result = Meridian.Security.Combat:ValidateShot(player, {
    Weapon = weapon,
    Origin = origin,
    Direction = direction,
    TargetPosition = targetPosition,
    MaximumRange = 500,
})
```

Current validation requires a `Vector3` origin, normalized `Vector3` direction, and target distance within range. The game must still validate ammunition, fire rate, weapon ownership, line of sight, and team rules.

