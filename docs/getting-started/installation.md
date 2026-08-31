# Installation

## Local development

Place `MainModule` and all of its children inside `ServerScriptService`:

```text
ServerScriptService
├── Meridian
└── MainModule
    ├── APIClient
    ├── AntiCheat
    ├── BanStore
    ├── Logger
    ├── PlayerManager
    ├── Security
    ├── Signal
    ├── Types
    └── UnbanQueue
```

Create a server loader next to `MainModule`:

```lua
local Meridian = require(script.Parent.MainModule)

Meridian.Init({
    InstallationKey = "meridian_live_xxxxxxxxx",
})
```

## Public model

After publishing the complete model, require it by asset ID:

```lua
local Meridian = require(111401706699344)

Meridian.Init({
    InstallationKey = "meridian_live_xxxxxxxxx",
})
```

The asset must be a Roblox `Model` containing a `ModuleScript` named `MainModule`. Republish `MainModule` with all child modules before switching a production loader to the public asset.

Next: [Initialization and configuration](configuration.md).

