# Failure behavior and production setup

## Fail-open behavior

With `FailOpen = true`:

- API unavailable: the game remains playable.
- DataStore read failure: no uncertain kick occurs.
- An existing confirmed DataStore ban remains enforceable.
- Reconciliation failure does not erase bans.
- Heartbeat failure reconnects.
- Session expiration refreshes authentication.
- Registration failure retries.
- Detection upload failure retains the in-memory batch.
- SDK errors should not terminate unrelated game systems.

Fail-open does not mean “ignore confirmed bans.” It means uncertain failures do not create punishments.

## Recommended production loader

```lua
local Meridian = require(111401706699344)

local initialized, errorCode = Meridian.Init({
    InstallationKey = "meridian_live_xxxxxxxxx",
    BaseUrl = "https://api.mrid.app",
    Environment = "production",
    Debug = false,
    FailOpen = true,
    HeartbeatInterval = 10,
    ReconcileInterval = 90,
    BanAuditInterval = 900,

    Moderation = {
        Enabled = true,
        AutoEnforceBans = true,
        EnforcementMode = "Kick",
        JoinReconciliationTimeout = 10,
    },

    AntiCheat = {
        Enabled = true,
        Profile = "balanced",
        AutoKick = true,
        AutoBan = false,
    },
})

if not initialized then
    warn("[Meridian Loader] Initialization failed:", errorCode)
end
```

Before using the public loader, publish `MainModule` with every child module. Keep the local loader configured for the local module during development.

