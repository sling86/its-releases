# PeopleHR (`hr`)

PeopleHR — bulk employee directory, upcoming and recent starters/leavers. THF tenant key is bulk-read scoped (single-record endpoints return Access Denied), so lookups go through the bulk list + client-side filter.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [drift](#drift)
- [absences](#absences)
- [employees](#employees)
- [starters](#starters)
- [leavers](#leavers)

## Setup

```bash
its hr setup           # Interactive wizard
its hr setup --check   # Check configuration status
its hr setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `PEOPLEHR_API_KEY` | PeopleHR API key — bulk-read scoped |

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/hr/client.ts` | API client methods |
| `src/providers/hr/types.ts` | TypeScript interfaces |
| `src/providers/hr/commands.ts` | Command definitions |
| `src/providers/hr/absence.ts` | absence |
| `src/providers/hr/definition.ts` | definition |
| `src/providers/hr/drift.ts` | drift |

## Resources

### drift

> Source: `src/providers/hr/commands.ts`

| Command | Description |
|---------|-------------|
| `its hr drift detect` | Detect drift between PeopleHR and Entra ID. Reports field mismatches plus PHR-only / Entra-only orphans. Read-only. |

#### `its hr drift detect`

Detect drift between PeopleHR and Entra ID. Reports field mismatches plus PHR-only / Entra-only orphans. Read-only.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--domain` | `` | Entra UPN domain to audit (e.g. example.com). Defaults to every domain seen in active Entra users. | — |
| `--company` | `` | Restrict PHR side to this company (substring match against Company DisplayValue). Default: search globally. | — |
| `--include-disabled` | `` | Include disabled Entra accounts (default: only enabled). | — |

```bash
its hr drift detect
```

---

### absences

> Source: `src/providers/hr/commands.ts`

| Command | Description |
|---------|-------------|
| `its hr absences get <employee>` | Get one employee's sickness-absence records. Contains special-category health data. Dates use YYYY-MM-DD; free-text notes are omitted unless --include-notes is passed. |
| `its hr absences summary <employee>` | Summarise one employee's sickness absence for a calendar year: episodes, days, longest spell, Bradford factor, emergency leave, outstanding return-to-work interviews, and reason breakdown. Contains special-category health data. |
| `its hr absences team` | Rank a manager's direct reports or a department by Bradford factor for one year. Contains special-category health data. Refuses teams larger than 25. |

#### `its hr absences get <employee>`

Get one employee's sickness-absence records. Contains special-category health data. Dates use YYYY-MM-DD; free-text notes are omitted unless --include-notes is passed.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--from` | `` | Start date (YYYY-MM-DD); defaults to 1 January this year | — |
| `--to` | `` | End date (YYYY-MM-DD); defaults to today | — |
| `--include-notes` | `` | Include free-text absence comments | — |

```bash
its hr absences get <employee>
```

#### `its hr absences summary <employee>`

Summarise one employee's sickness absence for a calendar year: episodes, days, longest spell, Bradford factor, emergency leave, outstanding return-to-work interviews, and reason breakdown. Contains special-category health data.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--year` | `` | Calendar year; defaults to current year | — |

```bash
its hr absences summary <employee>
```

#### `its hr absences team`

Rank a manager's direct reports or a department by Bradford factor for one year. Contains special-category health data. Refuses teams larger than 25.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--manager` | `` | Manager email, EmployeeId, or exact full name | — |
| `--department` | `` | Exact department name | — |
| `--year` | `` | Calendar year; defaults to current year | — |

```bash
its hr absences team
```

---

### employees

> Source: `src/providers/hr/commands.ts`

| Command | Description |
|---------|-------------|
| `its hr employees` | List all employees. Surfaces the most common fields; pass --json for raw shape. |
| `its hr employees search <query>` | Search employees by name/email/role/department/location. Substring match across the most relevant fields; case-insensitive. |
| `its hr employees get <email>` | Get employee details by email (client-side filter). Match is exact on email address — not a fuzzy/name lookup. |

#### `its hr employees`

List all employees. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--leavers` | `` | Include employees who have left | — |

**Examples:**

```bash
its hr employees

# Re-runs every 10s — handy for dashboards or incident response.
its hr employees --watch
```

#### `its hr employees search <query>`

Search employees by name/email/role/department/location. Substring match across the most relevant fields; case-insensitive.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--leavers` | `` | Include employees who have left | — |

**Examples:**

```bash
its hr employees search "jane"
```

#### `its hr employees get <email>`

Get employee details by email (client-side filter). Match is exact on email address — not a fuzzy/name lookup.

**Examples:**

```bash
its hr employees get <employee-id>
```

---

### starters

> Source: `src/providers/hr/commands.ts`

| Command | Description |
|---------|-------------|
| `its hr starters` | Upcoming starters — employees with StartDate in the future. Surfaces the most common fields; pass --json for raw shape. |
| `its hr starters recent` | Recent starters — employees with StartDate in the past window |

#### `its hr starters`

Upcoming starters — employees with StartDate in the future. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--days` | `` | Window in days (default 30) | 30 |

**Examples:**

```bash
its hr starters

# Re-runs every 10s — handy for dashboards or incident response.
its hr starters --watch
```

#### `its hr starters recent`

Recent starters — employees with StartDate in the past window.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--days` | `` | Window in days (default 30) | 30 |

**Examples:**

```bash
its hr starters recent --days 30
```

---

### leavers

> Source: `src/providers/hr/commands.ts`

| Command | Description |
|---------|-------------|
| `its hr leavers` | Upcoming leavers — employees with LeavingDate in the future |
| `its hr leavers recent` | Recent leavers — employees with LeavingDate in the past window |

#### `its hr leavers`

Upcoming leavers — employees with LeavingDate in the future.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--days` | `` | Window in days (default 30) | 30 |

**Examples:**

```bash
its hr leavers

# Re-runs every 10s — handy for dashboards or incident response.
its hr leavers --watch
```

#### `its hr leavers recent`

Recent leavers — employees with LeavingDate in the past window.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--days` | `` | Window in days (default 30) | 30 |

**Examples:**

```bash
its hr leavers recent --days 30
```

---
