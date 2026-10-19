# Yodeck (`yodeck`)

Yodeck digital signage (API v2) — list players (Yodeck calls them screens) with workspace, online/offline, last seen and TV status; get one player in full; `doctor` checks the token. Read-only.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [attendance](./attendance.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [players](#players)
- [doctor](#doctor)

## Setup

```bash
its yodeck setup           # Interactive wizard
its yodeck setup --check   # Check configuration status
its yodeck setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `YODECK_API_TOKEN` | Yodeck API token as <label>:<key> (sent as `Authorization: Token <label>:<key>`) |

Create the token in Yodeck (Account settings → API Tokens), then run `its yodeck setup` and paste it as `label:key`. It goes in the OS keychain. `its yodeck doctor` says "not set up" if it is missing.

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/yodeck/client.ts` | API client methods |
| `src/providers/yodeck/types.ts` | TypeScript interfaces |
| `src/providers/yodeck/commands/` | Command definitions (split by resource) |
| `src/providers/yodeck/definition.ts` | definition |

## Resources

### players

| Command | Description |
|---------|-------------|
| `its yodeck players` | List Yodeck players (screens): id, name, workspace, online/offline, last seen, screen (TV) status |
| `its yodeck players get <id>` | One Yodeck player in full: state, hardware, software, resolution, uptime |

#### `its yodeck players`

List Yodeck players (screens): id, name, workspace, online/offline, last seen, screen (TV) status.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--online` | `` | Only online players | — |
| `--offline` | `` | Only offline players | — |
| `--workspace` | `` | Workspace ID | — |
| `--q` | `` | Search by name | — |

**Examples:**

```bash
its yodeck players --json

its yodeck players --offline
```

#### `its yodeck players get <id>`

One Yodeck player in full: state, hardware, software, resolution, uptime.

```bash
its yodeck players get <id>
```

---

### doctor

| Command | Description |
|---------|-------------|
| `its yodeck doctor` | Is Yodeck set up and reachable? Says 'not set up' plainly if the token is missing |

#### `its yodeck doctor`

Is Yodeck set up and reachable? Says 'not set up' plainly if the token is missing.

```bash
its yodeck doctor
```

---
