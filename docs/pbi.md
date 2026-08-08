# Power BI (`pbi`)

Power BI Admin API — workspaces, reports, apps, licences, activity log.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [workspaces](#workspaces)
- [reports](#reports)
- [apps](#apps)
- [licences](#licences)
- [activity](#activity)
- [my](#my)

## Setup

```bash
its pbi setup           # Interactive wizard
its pbi setup --check   # Check configuration status
its pbi setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `TENANT_ID` | Microsoft Entra tenant ID (shared with entra) |
| `CLIENT_ID` | App registration client ID (shared with entra) |
| `CLIENT_SECRET` | App registration client secret (shared with entra) |

Uses the same service principal as `entra`. Authenticates against the Power BI audience (`https://analysis.windows.net/powerbi/api/.default`).

**Tenant setup required:** the SP must be granted read-only Power BI admin API access. In the Power BI Admin Portal → Tenant settings, enable **Service principals can use Fabric admin APIs** (and the legacy **Service principals can use read-only admin APIs** if still present) and add the SP to the permitted security group. Without this, all `pbi` calls return 401.

`pbi licences` queries Microsoft Graph `/subscribedSkus` (not the Power BI API) so it works as long as the SP has `Organization.Read.All` on Graph (which `entra` already needs).

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/pbi/client.ts` | API client methods |
| `src/providers/pbi/types.ts` | TypeScript interfaces |
| `src/providers/pbi/commands/` | Command definitions (split by resource) |
| `src/providers/pbi/definition.ts` | definition |
| `src/providers/pbi/my-auth.ts` | my auth |
| `src/providers/pbi/my-client.ts` | my client |

## Resources

### workspaces

> Source: `src/providers/pbi/commands/workspaces.ts`

| Command | Description |
|---------|-------------|
| `its pbi workspaces` | List Power BI workspaces (admin API). Surfaces the most common fields; pass --json for raw shape. |
| `its pbi workspaces members <workspace_id>` | List users/groups with access to a workspace. Returns direct members; nested groups aren't expanded. |
| `its pbi workspaces add-user <workspace_id>` | Add a user/group/app to a workspace (admin). Add a primary user; idempotent. |
| `its pbi workspaces update-user <workspace_id>` | Update a user's access right on a workspace. Change the primary user. |
| `its pbi workspaces remove-user <workspace_id>` | Remove a user/group/app from a workspace (admin, requires --confirm). Reverse of add-user. |

#### `its pbi workspaces`

List Power BI workspaces (admin API). Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Max results (default 5000) | 5000 |

**Examples:**

```bash
its pbi workspaces

# Re-runs every 10s — handy for dashboards or incident response.
its pbi workspaces --watch
```

#### `its pbi workspaces members <workspace_id>`

List users/groups with access to a workspace. Returns direct members; nested groups aren't expanded.

**Examples:**

```bash
its pbi workspaces members <workspace-id>
```

#### `its pbi workspaces add-user <workspace_id>`

Add a user/group/app to a workspace (admin). Add a primary user; idempotent.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | User UPN, group ID, or app ID to grant access | — |
| `--principal-type` | `` | Principal type (default: User) | User |
| `--access` | `` | Access right (default: Member) | Member |

**Examples:**

```bash
its pbi workspaces add-user <workspace-id> --user jane.smith@example.com --access Member
```

#### `its pbi workspaces update-user <workspace_id>`

Update a user's access right on a workspace. Change the primary user.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | User UPN, group ID, or app ID whose access to update | — |
| `--principal-type` | `` | Principal type (default: User) | User |
| `--access` | `` | New access right | — |

**Examples:**

```bash
its pbi workspaces update-user <workspace-id> --user jane.smith@example.com --access Admin
```

#### `its pbi workspaces remove-user <workspace_id>`

Remove a user/group/app from a workspace (admin, requires --confirm). Reverse of add-user.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | User UPN, group ID, or app ID to remove | — |
| `--confirm` | `` | Confirm the removal | — |

**Examples:**

```bash
its pbi workspaces remove-user <workspace-id> --user jane.smith@example.com --confirm
```

---

### reports

> Source: `src/providers/pbi/commands/reports.ts`

| Command | Description |
|---------|-------------|
| `its pbi reports` | List Power BI reports (all workspaces or --workspace <id>). Surfaces the most common fields; pass --json for raw shape. |
| `its pbi reports search <query>` | Fuzzy search reports by name across all workspaces. Substring match across the most relevant fields; case-insensitive. |
| `its pbi reports url <report_id>` | Print the web URL for a report. Returns a reachable connection URL. |

#### `its pbi reports`

List Power BI reports (all workspaces or --workspace <id>). Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--workspace` | `` | Limit to a single workspace ID | — |

**Examples:**

```bash
its pbi reports

its pbi reports --workspace <workspace-id>

# Re-runs every 10s — handy for dashboards or incident response.
its pbi reports --watch
```

#### `its pbi reports search <query>`

Fuzzy search reports by name across all workspaces. Substring match across the most relevant fields; case-insensitive.

**Examples:**

```bash
its pbi reports search "sales"
```

#### `its pbi reports url <report_id>`

Print the web URL for a report. Returns a reachable connection URL.

**Examples:**

```bash
# Embed/share link
its pbi reports url <report-id>
```

---

### apps

> Source: `src/providers/pbi/commands/apps.ts`

| Command | Description |
|---------|-------------|
| `its pbi apps` | List Power BI apps. Use --user <id|upn> to list one user's app access. |

#### `its pbi apps`

List Power BI apps. Use --user <id|upn> to list one user's app access.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | User ID or UPN — reads artifactAccess for that user | — |
| `--top` | `` | Max results when listing tenant-wide (default 5000) | 5000 |

**Examples:**

```bash
its pbi apps

# Re-runs every 10s — handy for dashboards or incident response.
its pbi apps --watch
```

---

### licences

> Source: `src/providers/pbi/commands/licences.ts`

| Command | Description |
|---------|-------------|
| `its pbi licences` | List Power BI licence SKUs with consumption. Surfaces the most common fields; pass --json for raw shape. |

#### `its pbi licences`

List Power BI licence SKUs with consumption. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its pbi licences

# Re-runs every 10s — handy for dashboards or incident response.
its pbi licences --watch
```

---

### activity

> Source: `src/providers/pbi/commands/activity.ts`

| Command | Description |
|---------|-------------|
| `its pbi activity` | List Power BI activity events. Power BI retains up to 30 days of activity. |

#### `its pbi activity`

List Power BI activity events. Power BI retains up to 30 days of activity.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--since` | `` | Time window (e.g. 1h, 1d, 7d, 2w). Default 1d. | 1d |
| `--user` | `` | Filter to events by user (UPN). Client-side filter. | — |
| `--activity` | `` | Filter to a specific activity name (e.g. ViewReport). Client-side filter. | — |
| `--top` | `` | Limit rows returned (default 200) | 200 |

**Examples:**

```bash
# Audit events for the tenant
its pbi activity --since 24h

# Re-runs every 10s — handy for dashboards or incident response.
its pbi activity --since 24h --watch
```

---

### my

> Source: `src/providers/pbi/commands/my.ts`

| Command | Description |
|---------|-------------|
| `its pbi my login` | Sign in as a Power BI user via device-code flow. Tokens are cached locally (~/.its/secrets/pbi-my-token.json). |
| `its pbi my logout` | Clear the cached Power BI user token. Does not revoke it server-side. |
| `its pbi my whoami` | Show the cached Power BI user account and token expiry. |
| `its pbi my workspaces` | List workspaces accessible to the signed-in user. Workspaces visible to the current principal. |
| `its pbi my reports` | List Power BI reports accessible to the signed-in user |
| `its pbi my datasets` | List Power BI datasets accessible to the signed-in user |
| `its pbi my add-workspace-user <workspace_id>` | Add a user/group/app to a workspace using the signed-in user's permissions (sidesteps SP admin-API restrictions when you're a workspace admin) |
| `its pbi my update-workspace-user <workspace_id>` | Update a user's access right on a workspace using the signed-in user's permissions |
| `its pbi my remove-workspace-user <workspace_id>` | Remove a user/group/app from a workspace using the signed-in user's permissions (requires --confirm) |
| `its pbi my refresh <dataset_id>` | Trigger a refresh of a dataset owned by the signed-in user. Force the provider to re-pull state from the upstream API. |

#### `its pbi my login`

Sign in as a Power BI user via device-code flow. Tokens are cached locally (~/.its/secrets/pbi-my-token.json).

**Examples:**

```bash
# Device-code flow for user-context API
its pbi my login
```

#### `its pbi my logout`

Clear the cached Power BI user token. Does not revoke it server-side.

**Examples:**

```bash
its pbi my logout
```

#### `its pbi my whoami`

Show the cached Power BI user account and token expiry.

**Examples:**

```bash
its pbi my whoami
```

#### `its pbi my workspaces`

List workspaces accessible to the signed-in user. Workspaces visible to the current principal.

**Examples:**

```bash
its pbi my workspaces
```

#### `its pbi my reports`

List Power BI reports accessible to the signed-in user.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--workspace` | `` | Limit to a single workspace ID | — |

**Examples:**

```bash
its pbi my reports
```

#### `its pbi my datasets`

List Power BI datasets accessible to the signed-in user.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--workspace` | `` | Limit to a single workspace ID | — |

**Examples:**

```bash
its pbi my datasets
```

#### `its pbi my add-workspace-user <workspace_id>`

Add a user/group/app to a workspace using the signed-in user's permissions (sidesteps SP admin-API restrictions when you're a workspace admin).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | User UPN, group ID, or app ID to grant access | — |
| `--principal-type` | `` | Principal type (default: User) | User |
| `--access` | `` | Access right (default: Viewer) | Viewer |

**Examples:**

```bash
its pbi my add-workspace-user <workspace-id> --user jane.smith@example.com --access Member
```

#### `its pbi my update-workspace-user <workspace_id>`

Update a user's access right on a workspace using the signed-in user's permissions.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | User UPN, group ID, or app ID whose access to update | — |
| `--principal-type` | `` | Principal type (default: User) | User |
| `--access` | `` | New access right | — |

**Examples:**

```bash
its pbi my update-workspace-user <workspace-id> --user jane.smith@example.com --access Admin
```

#### `its pbi my remove-workspace-user <workspace_id>`

Remove a user/group/app from a workspace using the signed-in user's permissions (requires --confirm).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | User UPN, group ID, or app ID to remove | — |
| `--confirm` | `` | Confirm the removal | — |

**Examples:**

```bash
its pbi my remove-workspace-user <workspace-id> --user jane.smith@example.com --confirm
```

#### `its pbi my refresh <dataset_id>`

Trigger a refresh of a dataset owned by the signed-in user. Force the provider to re-pull state from the upstream API.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--workspace` | `` | Workspace ID containing the dataset (recommended) | — |
| `--notify` | `` | Email notification policy | NoNotification |

**Examples:**

```bash
its pbi my refresh <dataset-id>

# Disambiguates when the dataset isn't in your personal workspace.
its pbi my refresh <dataset-id> --workspace <workspace-id>
```

---
