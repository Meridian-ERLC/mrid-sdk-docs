# Initialization and configuration

Call `Meridian.Init()` once from a server script:

```lua
local Meridian = require(script.Parent.MainModule)

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
    warn("Meridian failed:", errorCode)
end
```

Additional calls return the existing SDK instance.

## Default configuration

```lua
{
    BaseUrl = "https://api.mrid.app",
    Environment = "production",
    HeartbeatInterval = 10,
    ReconcileInterval = 90,
    BanAuditInterval = 900,
    HttpTimeout = 10,
    MaxRetries = 4,
    MaxRetryDelay = 15,
    DataStoreName = "Meridian_Bans_v1",
    UnbanQueueDataStoreName = "Meridian_UnbanQueue_v1",
    FailOpen = true,
    Debug = false,

    Moderation = {
        Enabled = true,
        AutoEnforceBans = true,
        EnforcementMode = "Kick",
        JoinReconciliationTimeout = 10,
    },

    Telemetry = { Enabled = true },

    AntiCheat = {
        Enabled = false,
        Profile = "balanced",
        AutoKick = true,
        AutoBan = false,
        DetectionEndpoint = "/v1/sdk/anticheat/detections/batch",
        SampleInterval = 0.25,
        FlushInterval = 5,
        BatchSize = 50,
        QueueLimit = 500,
        Movement = {
            Enabled = true,
            Speed = { Enabled = true },
            Teleport = { Enabled = true },
            Vertical = { Enabled = true, MaximumSpeed = 140 },
        },
        RemoteSecurity = { Enabled = true },
        Risk = {
            Enabled = true,
            Maximum = 100,
            Decay = true,
            DecayInterval = 60,
            DecayAmount = 2,
            KickScore = 90,
            BanScore = 100,
        },
    },

    RobloxBans = {
        Enabled = true,
        ApplyToUniverse = true,
        ExcludeAltAccounts = false,
        ApplyDeviceBlock = false,
    },

    ServerMetadata = {},
}
```

Runtime configuration returned by Meridian's API can update moderation, telemetry, and Anti-Cheat settings without restarting the server. Treat `Meridian.Config` as read-only.

