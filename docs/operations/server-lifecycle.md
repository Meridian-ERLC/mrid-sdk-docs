# Server lifecycle

## Heartbeat

The default heartbeat interval is 10 seconds.

```http
POST /v1/sdk/servers/heartbeat
Authorization: Bearer <session token>
Content-Type: application/json
```

```json
{ "player_count": 12 }
```

A response can include server time and events:

```json
{
  "ok": true,
  "server_time": "2026-08-31T14:00:00Z",
  "events": [{
    "type": "player.banned",
    "user_id": 123456789,
    "punishment_id": "punishment-id",
    "punishment_type": "ban",
    "reason": "Exploiting",
    "expires_at": null,
    "created_at": "2026-08-31T13:59:48Z"
  }]
}
```

Supported event types are `player.banned`, `player.unbanned`, and `config.updated`. A `player.banned` event does not directly authorize a kick; it triggers reconciliation and DataStore verification first.

## Shutdown

Meridian uses `game:BindToClose()` and reports remaining players as leaving before marking the server offline.

```http
POST /v1/sdk/servers/shutdown
Authorization: Bearer <session token>
Content-Type: application/json
```

```json
{ "player_count": 0 }
```

```json
{
  "ok": true,
  "status": "offline",
  "server_time": "2026-08-31T14:30:00Z"
}
```

