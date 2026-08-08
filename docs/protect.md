# UniFi Protect (`protect`)

UniFi Protect CCTV — cameras, NVR status, storage, motion events, snapshots.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [cameras](#cameras)
- [nvr](#nvr)
- [events](#events)
- [footage](#footage)
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
| `src/providers/protect/query.ts` | query |
| `src/providers/protect/sites.ts` | sites |

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
| `--site` | `` | Named Protect profile (UNIFI_PROTECT_*_<SITE>), or `all` to fan out across every configured site | — |

**Examples:**

```bash
its protect cameras

# Re-runs every 10s — handy for dashboards or incident response.
its protect cameras --watch
```

#### `its protect cameras get <id>`

Get camera details. Pass the id (or any natural identifier) as the positional arg.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Named Protect profile (UNIFI_PROTECT_*_<SITE>), or `all` to fan out across every configured site | — |

**Examples:**

```bash
its protect cameras get <camera-id>
```

#### `its protect cameras offline`

List disconnected/offline cameras — those whose state is not CONNECTED.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Named Protect profile (UNIFI_PROTECT_*_<SITE>), or `all` to fan out across every configured site | — |

**Examples:**

```bash
its protect cameras offline
```

#### `its protect cameras snapshot <camera_id>`

Get the full-resolution snapshot URL for a camera (authenticated endpoint). Pass the camera id from `cameras list`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--width` | `` | Snapshot width in px | 1920 |
| `--height` | `` | Snapshot height in px | 1080 |
| `--site` | `` | Named Protect profile (UNIFI_PROTECT_*_<SITE>), or `all` to fan out across every configured site | — |

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
| `--site` | `` | Named Protect profile (UNIFI_PROTECT_*_<SITE>), or `all` to fan out across every configured site | — |

**Examples:**

```bash
# Storage usage, retention, health
its protect nvr

# Re-runs every 10s — handy for dashboards or incident response.
its protect nvr --watch
```

---

### events

> Source: `src/providers/protect/commands/events.ts`

| Command | Description |
|---------|-------------|
| `its protect events` | List Protect events (motion, smart detections). Surfaces the most common fields; pass --json for raw shape. |
| `its protect events thumbnail <event_id>` | Download the historical still for an event. Identifiable imagery — --output is required so the write is always deliberate. |

#### `its protect events`

List Protect events (motion, smart detections). Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--hours` | `` | Hours to look back (ignored when --start is given) | 24 |
| `--start` | `` | Window start — ISO, local 'YYYY-MM-DD HH:mm', or epoch ms | — |
| `--end` | `` | Window end (default: now) | — |
| `--camera` | `` | Camera name or ID — comma-separate for several | — |
| `--types` | `` | Event types, server-side filter | — |
| `--smart` | `` | Smart-detection labels, filtered client-side (the NVR ignores the query param) | — |
| `--all` | `` | Page through the whole window — the fix for silently truncated busy sites | — |
| `--limit` | `` | Maximum events to return (default 30, ignored with --all) | 30 |
| `--site` | `` | Named Protect profile (UNIFI_PROTECT_*_<SITE>), or `all` to fan out across every configured site | — |

**Examples:**

```bash
its protect events --hours 1

its protect events --filter camera=<camera-name> --hours 24

# Re-runs every 10s — handy for dashboards or incident response.
its protect events --hours 1 --watch
```

#### `its protect events thumbnail <event_id>`

Download the historical still for an event. Identifiable imagery — --output is required so the write is always deliberate.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--output` | `` | File to write the JPEG to | — |
| `--width` | `` | Thumbnail width in px | 640 |
| `--site` | `` | Named Protect profile (UNIFI_PROTECT_*_<SITE>), or `all` to fan out across every configured site | — |

```bash
its protect events thumbnail <event_id>
```

---

### footage

> Source: `src/providers/protect/commands/footage.ts`

| Command | Description |
|---------|-------------|
| `its protect footage export <camera>` | Export MP4 footage for one camera over an exact window. Identifiable footage — --output is required so the write is always deliberate. |

#### `its protect footage export <camera>`

Export MP4 footage for one camera over an exact window. Identifiable footage — --output is required so the write is always deliberate.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--start` | `` | Window start — ISO, local 'YYYY-MM-DD HH:mm', or epoch ms | — |
| `--end` | `` | Window end (default: now) | — |
| `--minutes` | `` | Window length from --start when --end is omitted | — |
| `--output` | `` | File to write the MP4 to | — |
| `--site` | `` | Named Protect profile (UNIFI_PROTECT_*_<SITE>), or `all` to fan out across every configured site | — |

```bash
its protect footage export <camera>
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
| `--site` | `` | Named Protect profile (UNIFI_PROTECT_*_<SITE>), or `all` to fan out across every configured site | — |

**Examples:**

```bash
its protect dashboard

# Re-runs every 10s — handy for dashboards or incident response.
its protect dashboard --watch
```

---
