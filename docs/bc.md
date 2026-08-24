# Business Central (`bc`)

Business Central (Dynamics 365) — tenant-level companies list, multi-company entity queries via OData, record get. Uses a dedicated BC app registration (BC_CLIENT_ID/BC_CLIENT_SECRET) or falls back to the shared Entra credentials (TENANT_ID/CLIENT_ID/CLIENT_SECRET); either app needs the Business Central API permission granted and an ApplicationUser with a Permission Set inside each BC company.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [companies](#companies)
- [environments](#environments)
- [entities](#entities)
- [extensions](#extensions)
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
| `BC_CLIENT_ID` | Dedicated BC app client ID (falls back to CLIENT_ID) |
| `BC_CLIENT_SECRET` | Dedicated BC app client secret (falls back to CLIENT_SECRET) |
| `CLIENT_ID` | Shared Entra app client ID (fallback) |
| `CLIENT_SECRET` | Shared Entra app client secret (fallback) |
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

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |

**Examples:**

```bash
its bc companies

# Re-runs every 10s — handy for dashboards or incident response.
its bc companies --watch
```

#### `its bc companies get <nameOrId>`

Resolve a company by name/id/partial match. Pass the id (or any natural identifier) as the positional arg.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |

**Examples:**

```bash
its bc companies get "Head Office"
```

---

### environments

> Source: `src/providers/bc/commands.ts`

| Command | Description |
|---------|-------------|
| `its bc environments` | List all BC environments (Production/Sandbox) on the tenant |

#### `its bc environments`

List all BC environments (Production/Sandbox) on the tenant.

```bash
its bc environments
```

---

### entities

> Source: `src/providers/bc/commands.ts`

| Command | Description |
|---------|-------------|
| `its bc entities` | List entity sets exposed by the BC API (OData service document) |

#### `its bc entities`

List entity sets exposed by the BC API (OData service document).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |
| `--api` | `` | Custom API route <publisher>/<group>/<version> (default: standard v2.0 API) | — |

```bash
its bc entities
```

---

### extensions

> Source: `src/providers/bc/commands.ts`

| Command | Description |
|---------|-------------|
| `its bc extensions` | List published AL extensions with their versions — the direct answer to "which environment is on which app version?". Reads the Automation API, so it needs the SP's BC user to hold D365 EXTENSION MGT (not the Admin Centre grant). |
| `its bc extensions get <name>` | Get one extension's published version by name (substring, case-insensitive) — for gating on "is this environment on >= x.y.z?". |

#### `its bc extensions`

List published AL extensions with their versions — the direct answer to "which environment is on which app version?". Reads the Automation API, so it needs the SP's BC user to hold D365 EXTENSION MGT (not the Admin Centre grant).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |
| `--company` | `` | Company name/id (default: first company) | — |
| `--publisher` | `` | Only extensions from this publisher (substring) | — |

**Examples:**

```bash
its bc extensions list --env Production

its bc extensions list --env Production --publisher THF
```

#### `its bc extensions get <name>`

Get one extension's published version by name (substring, case-insensitive) — for gating on "is this environment on >= x.y.z?".

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |
| `--company` | `` | Company name/id (default: first company) | — |

**Examples:**

```bash
its bc extensions get platformSync --env Production
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
| `--all` | `` | Fetch all pages (up to 10000 records; overrides --top) | — |
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |
| `--api` | `` | Custom API route <publisher>/<group>/<version> (default: standard v2.0 API) | — |

**Examples:**

```bash
its bc query get items --company <company-id>
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
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |
| `--api` | `` | Custom API route <publisher>/<group>/<version> (default: standard v2.0 API) | — |

**Examples:**

```bash
its bc record get items <item-id>
```

---

### health

> Source: `src/providers/bc/commands.ts`

| Command | Description |
|---------|-------------|
| `its bc health get` | Probe BC connectivity by listing companies. Takes no positional argument; use --env to target a non-default environment. |

#### `its bc health get`

Probe BC connectivity by listing companies. Takes no positional argument; use --env to target a non-default environment.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |

**Examples:**

```bash
its bc health
```

---
