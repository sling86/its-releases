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
| `ITS_PROTECT_SITE` | Default named Protect profile; override per command with `--site <name>` |
| `UNIFI_PROTECT_HOST_<SITE>` | Controller for a named profile, e.g. `_HEAD_OFFICE` or `_WAREHOUSE` |
| `UNIFI_PROTECT_USERNAME_<SITE>` | Username for a named profile |
| `UNIFI_PROTECT_PASSWORD_<SITE>` | Password for a named profile |
| `UNIFI_PROTECT_HOST` | Protect controller IP or hostname (e.g. `192.168.60.50`) |
| `UNIFI_PROTECT_USERNAME` | Protect username (falls back to `UNIFI_USERNAME`) |
| `UNIFI_PROTECT_PASSWORD` | Protect password (falls back to `UNIFI_PASSWORD`) |

Named profiles use upper-case suffixed variables: `UNIFI_PROTECT_HOST_<SITE>`, `UNIFI_PROTECT_USERNAME_<SITE>`, and `UNIFI_PROTECT_PASSWORD_<SITE>`. Select one with `--site <name>` or set `ITS_PROTECT_SITE`. Explicit profiles never fall back to unsuffixed credentials.

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
| `its protect cameras offline` | List disconnected/offline cameras — those whose state is not CONNECTED. |
| `its protect cameras snapshot <camera_id>` | Get the full-resolution snapshot URL for a camera (authenticated endpoint). Pass the camera id from `cameras list`. |

#### `its protect cameras`

List all Protect cameras with status. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Named Protect profile (uses UNIFI_PROTECT_*_<SITE> environment variables) | — |

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

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Named Protect profile (uses UNIFI_PROTECT_*_<SITE> environment variables) | — |

**Examples:**

```bash
its protect cameras get <camera-id>

# Pipe-friendly output — use with jq / scripts.
its protect cameras get <camera-id> --json
```

#### `its protect cameras offline`

List disconnected/offline cameras — those whose state is not CONNECTED.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Named Protect profile (uses UNIFI_PROTECT_*_<SITE> environment variables) | — |

**Examples:**

```bash
its protect cameras offline

# Pipe-friendly output — use with jq / scripts.
its protect cameras offline --json
```

#### `its protect cameras snapshot <camera_id>`

Get the full-resolution snapshot URL for a camera (authenticated endpoint). Pass the camera id from `cameras list`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--width` | `` | Snapshot width in px | 1920 |
| `--height` | `` | Snapshot height in px | 1080 |
| `--site` | `` | Named Protect profile (uses UNIFI_PROTECT_*_<SITE> environment variables) | — |

```bash
its protect cameras snapshot <camera_id>
```

---

### nvr

> Source: `src/providers/protect/commands/nvr.ts`

| Command | Description |
|---------|-------------|
| `its protect nvr` | Show NVR status, storage, and capacity. Surfaces the most common fields; pass --json for raw shape. |

#### `its protect nvr`

Show NVR status, storage, and capacity. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Named Protect profile (uses UNIFI_PROTECT_*_<SITE> environment variables) | — |

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
| `--site` | `` | Named Protect profile (uses UNIFI_PROTECT_*_<SITE> environment variables) | — |

**Examples:**

```bash
its protect events --hours 1

its protect events --filter camera=<camera-name> --hours 24

# Re-runs every 10s — handy for dashboards or incident response.
its protect events --hours 1 --watch
```

---

### dashboard

> Source: `src/providers/protect/commands/dashboard.ts`

| Command | Description |
|---------|-------------|
| `its protect dashboard` | Protect overview — NVR, cameras, storage, recent motion. Surfaces the most common fields; pass --json for raw shape. |

#### `its protect dashboard`

Protect overview — NVR, cameras, storage, recent motion. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Named Protect profile (uses UNIFI_PROTECT_*_<SITE> environment variables) | — |

**Examples:**

```bash
its protect dashboard

# Pipe-friendly output — use with jq / scripts.
its protect dashboard --json

# Re-runs every 10s — handy for dashboards or incident response.
its protect dashboard --watch
```

---
