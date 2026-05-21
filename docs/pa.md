# Power Platform (`pa`)

Power Platform admin API — environments, Power Automate cloud flows, Power Apps canvas apps, connections.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md)

## Contents

- [Setup](#setup)
- [environments](#environments)
- [flows](#flows)
- [apps](#apps)
- [connections](#connections)

## Setup

```bash
its pa setup           # Interactive wizard
its pa setup --check   # Check configuration status
its pa setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `TENANT_ID` | Microsoft Entra tenant ID (shared with entra) |
| `CLIENT_ID` | App registration client ID (shared with entra) |
| `CLIENT_SECRET` | App registration client secret (shared with entra) |

Reuses the Entra service principal (TENANT_ID/CLIENT_ID/CLIENT_SECRET). Two audiences are used internally — `service.powerapps.com/.default` for BAP endpoints (environments, apps, connections) and `service.flow.microsoft.com/.default` for Power Automate cloud flow endpoints.

**Tenant setup required** (one-off, tenant admin or Power Platform admin role):

```powershell
Install-Module Microsoft.PowerApps.Administration.PowerShell -Scope CurrentUser
Add-PowerAppsAccount
New-PowerAppManagementApp -ApplicationId <CLIENT_ID>
```

Without this every `/scopes/admin/*` call returns 401/403. Default per-user environments are usually invisible even to registered admin SPs — `flows list` and `apps list` silently skip environments where access is denied so the aggregated view still works.

Known limitation: some legacy/personal Power Automate flows are unfindable via the admin API even with Power Platform Admin role assigned (memory: ctxc lesson #348).

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/pa/client.ts` | API client methods |
| `src/providers/pa/types.ts` | TypeScript interfaces |
| `src/providers/pa/commands/` | Command definitions (split by resource) |
| `src/providers/pa/definition.ts` | definition |

## Resources

### environments

> Source: `src/providers/pa/commands/environments.ts`

| Command | Description |
|---------|-------------|
| `its pa environments` | List Power Platform environments (admin). Surfaces the most common fields; pass --json for raw shape. |
| `its pa environments get <environment_id>` | Show details for one environment. Pass the id (or any natural identifier) as the positional arg. |

#### `its pa environments`

List Power Platform environments (admin). Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its pa environments

# Pipe-friendly output — use with jq / scripts.
its pa environments --json

# Re-runs every 10s — handy for dashboards or incident response.
its pa environments --watch
```

#### `its pa environments get <environment_id>`

Show details for one environment. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its pa environments get <env-id>

# Pipe-friendly output — use with jq / scripts.
its pa environments get <env-id> --json
```

---

### flows

> Source: `src/providers/pa/commands/flows.ts`

| Command | Description |
|---------|-------------|
| `its pa flows` | List Power Automate cloud flows. Defaults to all environments — use --environment <id> to scope |
| `its pa flows get <flow_id>` | Show flow details (definition, triggers, actions). Pass the id (or any natural identifier) as the positional arg. |
| `its pa flows stop <flow_id>` | Turn a flow off (admin). Stop the resource. Use --confirm if the action is destructive. |
| `its pa flows start <flow_id>` | Turn a flow on (admin). Start the resource. Idempotent. |
| `its pa flows delete <flow_id>` | Delete a flow (admin). Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record. |
| `its pa flows set-owner <flow_id>` | Change ownership / permissions on a cloud flow (admin). --owner <upn|guid> upserts a principal (default role Owner); --remove <upn|guid> revokes one. Idempotent. Used to reclaim flows from disabled accounts during licence reclaim (ctxc 858). |
| `its pa flows runs <flow_id>` | List recent runs for a flow. Returns historical run records. |

#### `its pa flows`

List Power Automate cloud flows. Defaults to all environments — use --environment <id> to scope.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--environment` | `-e` | Limit to a single environment (id from `its pa environments`) | — |
| `--state` | `` | Filter by state | — |

**Examples:**

```bash
its pa flows --env <env-id>

its pa flows --env <env-id> --status error

# Re-runs every 10s — handy for dashboards or incident response.
its pa flows --env <env-id> --watch
```

#### `its pa flows get <flow_id>`

Show flow details (definition, triggers, actions). Pass the id (or any natural identifier) as the positional arg.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--environment` | `-e` | Environment id (required) | — |

**Examples:**

```bash
its pa flows get <flow-id> --env <env-id>

# Pipe-friendly output — use with jq / scripts.
its pa flows get <flow-id> --env <env-id> --json
```

#### `its pa flows stop <flow_id>`

Turn a flow off (admin). Stop the resource. Use --confirm if the action is destructive.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--environment` | `-e` | Environment id (required) | — |

**Examples:**

```bash
its pa flows stop <flow-id> --env <env-id>
```

#### `its pa flows start <flow_id>`

Turn a flow on (admin). Start the resource. Idempotent.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--environment` | `-e` | Environment id (required) | — |

**Examples:**

```bash
its pa flows start <flow-id> --env <env-id>

# Pipe-friendly output — use with jq / scripts.
its pa flows start <flow-id> --env <env-id> --json
```

#### `its pa flows delete <flow_id>`

Delete a flow (admin). Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--environment` | `-e` | Environment id (required) | — |

**Examples:**

```bash
its pa flows delete <flow-id> --env <env-id> --confirm
```

#### `its pa flows set-owner <flow_id>`

Change ownership / permissions on a cloud flow (admin). --owner <upn|guid> upserts a principal (default role Owner); --remove <upn|guid> revokes one. Idempotent. Used to reclaim flows from disabled accounts during licence reclaim (ctxc 858).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--environment` | `-e` | Environment id (required) | — |
| `--owner` | `` | Principal to grant the role to (UPN or AAD object id). Mutually exclusive with --remove. | — |
| `--remove` | `` | Principal to revoke (UPN or AAD object id). Mutually exclusive with --owner. | — |
| `--role` | `` | Permission tier when granting | Owner |
| `--confirm` | `` | Required to execute the mutation | — |

```bash
its pa flows set-owner <flow_id>
```

#### `its pa flows runs <flow_id>`

List recent runs for a flow. Returns historical run records.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--environment` | `-e` | Environment id (required) | — |
| `--top` | `` | Max runs to fetch (default 50) | 50 |

**Examples:**

```bash
its pa flows runs <flow-id> --env <env-id>

# Pipe-friendly output — use with jq / scripts.
its pa flows runs <flow-id> --env <env-id> --json
```

---

### apps

> Source: `src/providers/pa/commands/apps.ts`

| Command | Description |
|---------|-------------|
| `its pa apps` | List Power Apps canvas apps. Defaults to all envs — scope with --environment <id> |

#### `its pa apps`

List Power Apps canvas apps. Defaults to all envs — scope with --environment <id>.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--environment` | `-e` | Limit to a single environment | — |

**Examples:**

```bash
its pa apps --env <env-id>

# Pipe-friendly output — use with jq / scripts.
its pa apps --env <env-id> --json

# Re-runs every 10s — handy for dashboards or incident response.
its pa apps --env <env-id> --watch
```

---

### connections

> Source: `src/providers/pa/commands/connections.ts`

| Command | Description |
|---------|-------------|
| `its pa connections` | List connections in an environment. Surfaces the most common fields; pass --json for raw shape. |

#### `its pa connections`

List connections in an environment. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--environment` | `-e` | Environment id (required) | — |

**Examples:**

```bash
its pa connections --env <env-id>

# Pipe-friendly output — use with jq / scripts.
its pa connections --env <env-id> --json

# Re-runs every 10s — handy for dashboards or incident response.
its pa connections --env <env-id> --watch
```

---
