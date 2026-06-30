# UniFi Protect (`protect`)

UniFi Protect CCTV — cameras, NVR status, storage, motion events, snapshots.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [cameras](#cameras)
- [nvr](#nvr)
- [events](#events)
- [dashboard](#dashboard)

## Setup

```bash
its protect setup           # Interactive wizard
its protect setup --check   # Check configuration status
its protect setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `UNIFI_PROTECT_HOST` | Protect controller IP or hostname (e.g. `192.168.60.50`) |
| `UNIFI_PROTECT_USERNAME` | Protect username (falls back to `UNIFI_USERNAME`) |
| `UNIFI_PROTECT_PASSWORD` | Protect password (falls back to `UNIFI_PASSWORD`) |

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/protect/client.ts` | API client methods |
| `src/providers/protect/types.ts` | TypeScript interfaces |
| `src/providers/protect/commands/` | Command definitions (split by resource) |
| `src/providers/protect/definition.ts` | definition |

## Resources

### cameras

> Source: `src/providers/protect/commands/cameras.ts`

| Command | Description |
|---------|-------------|
| `its protect cameras` | List all Protect cameras with status. Surfaces the most common fields; pass --json for raw shape. |
| `its protect cameras get <id>` | Get camera details. Pass the id (or any natural identifier) as the positional arg. |
| `its protect cameras offline` | List disconnected/offline cameras. Returns recently-disconnected clients. |

#### `its protect cameras`

List all Protect cameras with status. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its protect cameras

# Pipe-friendly output — use with jq / scripts.
its protect cameras --json

# Re-runs every 10s — handy for dashboards or incident response.
its protect cameras --watch
```

#### `its protect cameras get <id>`

Get camera details. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its protect cameras get <camera-id>

# Pipe-friendly output — use with jq / scripts.
its protect cameras get <camera-id> --json
```

#### `its protect cameras offline`

List disconnected/offline cameras. Returns recently-disconnected clients.

**Examples:**

```bash
its protect cameras offline

# Pipe-friendly output — use with jq / scripts.
its protect cameras offline --json
```

---

### nvr

> Source: `src/providers/protect/commands/nvr.ts`

| Command | Description |
|---------|-------------|
| `its protect nvr` | Show NVR status, storage, and capacity. Surfaces the most common fields; pass --json for raw shape. |

#### `its protect nvr`

Show NVR status, storage, and capacity. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
# Storage usage, retention, health
its protect nvr

# Pipe-friendly output — use with jq / scripts.
its protect nvr --json

# Re-runs every 10s — handy for dashboards or incident response.
its protect nvr --watch
```

---

### events

> Source: `src/providers/protect/commands/events.ts`

| Command | Description |
|---------|-------------|
| `its protect events` | List recent Protect events (motion, smart detections). Surfaces the most common fields; pass --json for raw shape. |

#### `its protect events`

List recent Protect events (motion, smart detections). Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--hours` | `` | Hours to look back (default 24) | 24 |
| `--limit` | `` | Maximum events to return (default 30) | 30 |

**Examples:**

```bash
its protect events --since 1h

its protect events --camera <camera-id> --since 24h

# Re-runs every 10s — handy for dashboards or incident response.
its protect events --since 1h --watch
```

---

### dashboard

> Source: `src/providers/protect/commands/dashboard.ts`

| Command | Description |
|---------|-------------|
| `its protect dashboard` | Protect overview — NVR, cameras, storage, recent motion. Surfaces the most common fields; pass --json for raw shape. |

#### `its protect dashboard`

Protect overview — NVR, cameras, storage, recent motion. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its protect dashboard

# Pipe-friendly output — use with jq / scripts.
its protect dashboard --json

# Re-runs every 10s — handy for dashboards or incident response.
its protect dashboard --watch
```

---
