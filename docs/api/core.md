# Core API

## Public state

| Member | Type | Description |
| --- | --- | --- |
| `Meridian.Ready` | `boolean` | Becomes `true` after the initial connection attempt, including degraded mode. |
| `Meridian.Status` | `string` | Current SDK lifecycle state. |
| `Meridian.Version` | `string` | SDK version, for example `"0.2.0"`. |
| `Meridian.Config` | `table` | Effective merged local and runtime configuration. Read-only. |

Status values include `NotInitialized`, `Starting`, `Ready`, `Degraded`, `Stopped`, and `ConfigurationError`.

```lua
print(Meridian.Version)
print(Meridian.Config.Moderation.Enabled)
print(Meridian.Config.AntiCheat.Profile)
```

## API client

`Meridian.API` provides:

```lua
Meridian.API:Get(path, options?)
Meridian.API:Post(path, body?, options?)
Meridian.API:Delete(path, body?, options?)
```

It handles authentication, JSON encoding and decoding, timeouts, retries, request IDs, session refresh, HTTP/API errors, rate limiting, debug logging, and backoff.

```lua
local response, err = Meridian.API:Get("/v1/sdk/example")

local created, createErr = Meridian.API:Post("/v1/sdk/example", {
    value = 123,
})

local removed, removeErr = Meridian.API:Delete("/v1/sdk/example", {
    resource_id = "example",
})
```

When a request fails, the response is `nil` and the second return value is an error:

```lua
{
    Code = "http_error",
    Message = "...",
    StatusCode = 500,
}
```

### Request options

```lua
{
    Token = "custom_token",
    Timeout = 5,
    MaxRetries = 1,
    SkipSessionRefresh = false,
}
```

Developers normally should not pass authentication tokens manually. Never treat client-supplied API data as trusted server authority.

