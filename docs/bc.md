# Business Central (`bc`)

Business Central (Dynamics 365) — tenant-level companies list, multi-company entity queries via OData, record get. Reuses Entra app credentials (TENANT_ID/CLIENT_ID/CLIENT_SECRET) but requires Business Central API permission granted and an ApplicationUser with a Permission Set inside each BC company..

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md)

## Contents

- [Setup](#setup)
- [companies](#companies)
- [query](#query)
- [record](#record)
- [health](#health)

## Setup

```bash
its bc setup           # Interactive wizard
its bc setup --check   # Check configuration status
its bc setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `TENANT_ID` | Entra tenant ID (shared) |
| `CLIENT_ID` | Entra app client ID (shared) |
| `CLIENT_SECRET` | Entra app client secret (shared) |
| `BC_ENVIRONMENT` | BC environment name (default: Production) |

In addition to the Entra app permissions, Business Central requires the app registration to be granted `https://api.businesscentral.dynamics.com/.default` API permission AND registered as an ApplicationUser inside each BC company with a Permission Set (SUPER for full access). See Microsoft docs: 'Use service-to-service authentication with Business Central'.

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/bc/client.ts` | API client methods |
| `src/providers/bc/types.ts` | TypeScript interfaces |
| `src/providers/bc/commands.ts` | Command definitions |
| `src/providers/bc/definition.ts` | definition |

## Resources

### companies

> Source: `src/providers/bc/commands.ts`

| Command | Description |
|---------|-------------|
| `its bc companies` | List all BC companies visible to the service principal. Surfaces the most common fields; pass --json for raw shape. |
| `its bc companies get <nameOrId>` | Resolve a company by name/id/partial match. Pass the id (or any natural identifier) as the positional arg. |

#### `its bc companies`

List all BC companies visible to the service principal. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its bc companies

# Pipe-friendly output — use with jq / scripts.
its bc companies --json

# Re-runs every 10s — handy for dashboards or incident response.
its bc companies --watch
```

#### `its bc companies get <nameOrId>`

Resolve a company by name/id/partial match. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its bc companies get "Head Office"

# Pipe-friendly output — use with jq / scripts.
its bc companies get "Head Office" --json
```

---

### query

> Source: `src/providers/bc/commands.ts`

| Command | Description |
|---------|-------------|
| `its bc query get <entity>` | Query any BC entity — OData passthrough with filter/top/select |

#### `its bc query get <entity>`

Query any BC entity — OData passthrough with filter/top/select.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--company` | `` | Company name/id (default: first company) | — |
| `--filter` | `` | OData $filter expression | — |
| `--top` | `` | Max records (default 50) | 50 |
| `--select` | `` | $select fields (comma-separated) | — |
| `--orderby` | `` | $orderby expression | — |

**Examples:**

```bash
its bc query get --company <company-id> --entity items

# Pipe-friendly output — use with jq / scripts.
its bc query get --company <company-id> --entity items --json
```

---

### record

> Source: `src/providers/bc/commands.ts`

| Command | Description |
|---------|-------------|
| `its bc record get <entity> <id>` | Get a single BC record by entity + ID. Pass the id (or any natural identifier) as the positional arg. |

#### `its bc record get <entity> <id>`

Get a single BC record by entity + ID. Pass the id (or any natural identifier) as the positional arg.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--company` | `` | Company name/id (default: first company) | — |

**Examples:**

```bash
its bc record get items <item-id>

# Pipe-friendly output — use with jq / scripts.
its bc record get items <item-id> --json
```

---

### health

> Source: `src/providers/bc/commands.ts`

| Command | Description |
|---------|-------------|
| `its bc health get` | Probe BC connectivity (lists companies). Pass the id (or any natural identifier) as the positional arg. |

#### `its bc health get`

Probe BC connectivity (lists companies). Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its bc health

# Pipe-friendly output — use with jq / scripts.
its bc health --json
```

---
