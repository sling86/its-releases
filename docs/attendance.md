# Factory attendance (`attendance`)

Read-only ZKTeco attendance collection over TCP 4370. Uses an explicit terminal allow-list, sequential transfers, device-count verification, Europe/London wall time and a private 30-day cache. Raw punches remain separate from inferred first/last summaries. Optional PeopleHR joins use only reviewed TimeAndAttendanceId values — never name matching or EmployeeId guesses.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [terminals](#terminals)
- [events](#events)
- [summary](#summary)
- [exceptions](#exceptions)

## Setup

```bash
its attendance setup           # Interactive wizard
its attendance setup --check   # Check configuration status
its attendance setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `ATTENDANCE_TERMINALS` | JSON array of explicitly allowed ZKTeco terminal names, hosts, ports and optional sites |
| `ATTENDANCE_TIMEOUT_MS` | Per-response timeout in milliseconds (default 15000) |
| `ATTENDANCE_RETRIES` | Full-transfer retries (default 2) |
| `ATTENDANCE_CACHE_MINUTES` | Complete-transfer cache lifetime in minutes (default 5) |
| `PEOPLEHR_API_KEY` | Optional: enrich --peoplehr output using reviewed TimeAndAttendanceId mappings |

Only ZKTeco terminals accepting communication key 0 are supported. Collection fails closed on any partial terminal transfer. The CLI does not infer clock-in/out, paid hours, overtime, lateness, absence or current presence.

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/attendance/client.ts` | API client methods |
| `src/providers/attendance/types.ts` | TypeScript interfaces |
| `src/providers/attendance/commands.ts` | Command definitions |
| `src/providers/attendance/definition.ts` | definition |
| `src/providers/attendance/zkteco.ts` | zkteco |

## Resources

### terminals

> Source: `src/providers/attendance/commands.ts`

| Command | Description |
|---------|-------------|
| `its attendance terminals` | Check every explicitly configured ZKTeco terminal sequentially. Read-only: reports user/log counts and never requests fingerprint templates or device changes. |

#### `its attendance terminals`

Check every explicitly configured ZKTeco terminal sequentially. Read-only: reports user/log counts and never requests fingerprint templates or device changes.

**Examples:**

```bash
its attendance terminals
```

---

### events

> Source: `src/providers/attendance/commands.ts`

| Command | Description |
|---------|-------------|
| `its attendance events` | List raw terminal punches in Europe/London time. Records have no IN/OUT state. Every fresh transfer is sequential and rejected unless received rows equal the device's advertised log count. |

#### `its attendance events`

List raw terminal punches in Europe/London time. Records have no IN/OUT state. Every fresh transfer is sequential and rejected unless received rows equal the device's advertised log count.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--since` | `` | Start of the local Europe/London window; default -30d, maximum 30 days | -30d |
| `--until` | `` | End of the local Europe/London window; default now | — |
| `--terminal` | `` | Restrict collection to one configured terminal name or host | — |
| `--peoplehr` | `` | Join only reviewed PeopleHR TimeAndAttendanceId mappings; never guesses by name or EmployeeId | — |
| `--refresh` | `` | Ignore the short-lived private cache and perform a fresh, count-checked transfer | — |

**Examples:**

```bash
its attendance events

its attendance events --since -7d --until 2026-09-14

its attendance events --terminal "Factory A" --since -1d

its attendance events --peoplehr --since -7d --json
```

---

### summary

> Source: `src/providers/attendance/commands.ts`

| Command | Description |
|---------|-------------|
| `its attendance summary` | Group raw punches by person and local day. First/last and observed-window fields are inference only — not clock-in/out, paid hours, lateness, overtime, absence or current presence. |

#### `its attendance summary`

Group raw punches by person and local day. First/last and observed-window fields are inference only — not clock-in/out, paid hours, lateness, overtime, absence or current presence.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--since` | `` | Start of the local Europe/London window; default -30d, maximum 30 days | -30d |
| `--until` | `` | End of the local Europe/London window; default now | — |
| `--terminal` | `` | Restrict collection to one configured terminal name or host | — |
| `--peoplehr` | `` | Join only reviewed PeopleHR TimeAndAttendanceId mappings; never guesses by name or EmployeeId | — |
| `--refresh` | `` | Ignore the short-lived private cache and perform a fresh, count-checked transfer | — |

**Examples:**

```bash
its attendance summary

its attendance summary --peoplehr --since -7d
```

---

### exceptions

> Source: `src/providers/attendance/commands.ts`

| Command | Description |
|---------|-------------|
| `its attendance exceptions` | Report single/odd-punch person-days and punches whose terminal user record has been deleted. Only complete local days inside the requested window are checked, so a cut-off cannot invent a missing punch. These are review prompts, not proof of absence. |

#### `its attendance exceptions`

Report single/odd-punch person-days and punches whose terminal user record has been deleted. Only complete local days inside the requested window are checked, so a cut-off cannot invent a missing punch. These are review prompts, not proof of absence.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--since` | `` | Start of the local Europe/London window; default -30d, maximum 30 days | -30d |
| `--until` | `` | End of the local Europe/London window; default now | — |
| `--terminal` | `` | Restrict collection to one configured terminal name or host | — |
| `--peoplehr` | `` | Join only reviewed PeopleHR TimeAndAttendanceId mappings; never guesses by name or EmployeeId | — |
| `--refresh` | `` | Ignore the short-lived private cache and perform a fresh, count-checked transfer | — |
| `--type` | `` | Restrict exception type | all |

**Examples:**

```bash
its attendance exceptions --since -7d

its attendance exceptions --type single-punch-day --since -30d
```

---
