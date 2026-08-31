# Meridian Roblox SDK

Server-side moderation, Anti-Cheat, remote security, and telemetry for Roblox experiences.

## Start here

1. [Install the SDK](docs/getting-started/installation.md)
2. [Initialize and configure Meridian](docs/getting-started/configuration.md)
3. Browse the [API reference](docs/api/README.md)
4. Review [production behavior](docs/operations/failure-behavior.md)

## Documentation

### Getting started

- [Installation](docs/getting-started/installation.md)
- [Initialization and configuration](docs/getting-started/configuration.md)

### API reference

- [Public state and API client](docs/api/core.md)
- [Players](docs/api/players.md)
- [Moderation](docs/api/moderation.md)
- [Anti-Cheat](docs/api/anti-cheat.md)
- [Security and validation types](docs/api/security.md)
- [Events and logging](docs/api/events.md)

### Operations

- [Heartbeat and shutdown](docs/operations/server-lifecycle.md)
- [Failure behavior and production loader](docs/operations/failure-behavior.md)

## Requirements

- The SDK runs on the Roblox server.
- HTTP requests and DataStore access must be enabled where applicable.
- Never expose the installation key or SDK session token to clients.
- Never treat client-supplied data as server authority.

