# Power BI (`pbi`)

Power BI Admin API — workspaces, reports, apps, licences, activity log.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [attendance](./attendance.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [workspaces](#workspaces)
- [reports](#reports)
- [apps](#apps)
- [access](#access)
- [licences](#licences)
- [activity](#activity)
- [scan](#scan)
- [api](#api)
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
| `src/providers/pbi/refresh.ts` | refresh |
| `src/providers/pbi/scan.ts` | scan |

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
its pbi workspaces add-user 8f1c2d3e-... --user jane.smith@example.com --principal-type User --access Member

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
its pbi workspaces update-user 8f1c2d3e-... --user jane.smith@example.com --principal-type User --access Admin

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
its pbi workspaces remove-user 8f1c2d3e-... --user jane.smith@example.com --confirm

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
| `its pbi apps` | List Power BI apps. Use --user <id|upn> to list one user's app access (`its pbi access --user` shows every type). |
| `its pbi apps users <app_id>` | Who can open a Power BI app (its audience): users, groups and service principals with their access right. Admin API, 200 calls/hour. |

#### `its pbi apps`

List Power BI apps. Use --user <id|upn> to list one user's app access (`its pbi access --user` shows every type).

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

#### `its pbi apps users <app_id>`

Who can open a Power BI app (its audience): users, groups and service principals with their access right. Admin API, 200 calls/hour.

**Examples:**

```bash
its pbi apps users f089354e-8366-4e18-aea3-4cb4a3a50b48
```

---

### access

> Source: `src/providers/pbi/commands/access.ts`

| Command | Description |
|---------|-------------|
| `its pbi access` | Every Power BI item one user can reach (reports, apps, datasets, dashboards…) with their access right. Admin API, 200 calls/hour. |

#### `its pbi access`

Every Power BI item one user can reach (reports, apps, datasets, dashboards…) with their access right. Admin API, 200 calls/hour.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | User ID or UPN (required) | — |
| `--type` | `` | Only this artifact type (filtered server-side) | — |

**Examples:**

```bash
its pbi access --user jane.smith@example.com

its pbi access --user jane.smith@example.com --type Report
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

### scan

> Source: `src/providers/pbi/commands/scan.ts`

| Command | Description |
|---------|-------------|
| `its pbi scan workspaces` | Every workspace id in the tenant, personal "My workspace" ones included (the plain `workspaces` list omits those). Admin API, service principal. |
| `its pbi scan run` | Scan the tenant with the Scanner API: workspaces, items, owners, users with access, data sources, lineage, model tables, measures and Power Query text. Saves the full result under --out and prints only counts plus which detail levels the tenant returned. Read-only; limit about 500 scans an hour, 100 workspaces each. |

#### `its pbi scan workspaces`

Every workspace id in the tenant, personal "My workspace" ones included (the plain `workspaces` list omits those). Admin API, service principal.

```bash
its pbi scan workspaces
```

#### `its pbi scan run`

Scan the tenant with the Scanner API: workspaces, items, owners, users with access, data sources, lineage, model tables, measures and Power Query text. Saves the full result under --out and prints only counts plus which detail levels the tenant returned. Read-only; limit about 500 scans an hour, 100 workspaces each.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--out` | `` | Directory for the raw scan JSON (created, mode 0700; files 0600). Required — the content is sensitive, so it never goes to the terminal | — |
| `--workspace` | `` | Comma-separated workspace ids (default: every workspace, personal ones included) | — |
| `--detail` | `` | Which detail to ask for: lineage, datasources, schema, expressions, users (default: all five) | — |
| `--timeout` | `` | Seconds to wait for each batch (default 600) | 600 |
| `--poll-interval` | `` | Seconds between status checks (default 5) | 5 |

**Examples:**

```bash
its pbi scan run --out ./data/scanner

its pbi scan run --workspace <id> --detail schema,expressions --out ./data/scanner
```

---

### api

> Source: `src/providers/pbi/commands/scan.ts`

| Command | Description |
|---------|-------------|
| `its pbi api get <path>` | Raw read-only GET against the Power BI REST API (api.powerbi.com): pass a /v1.0/myorg/... or /v2.0/myorg/... path. Uses the CLI's own admin token; GET only. --to-file saves the JSON (mode 0600) and prints only a count. |

#### `its pbi api get <path>`

Raw read-only GET against the Power BI REST API (api.powerbi.com): pass a /v1.0/myorg/... or /v2.0/myorg/... path. Uses the CLI's own admin token; GET only. --to-file saves the JSON (mode 0600) and prints only a count.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--to-file` | `` | Write the JSON here (mode 0600) instead of printing it | — |

**Examples:**

```bash
its pbi api get /v1.0/myorg/admin/widelySharedArtifacts/publishedToWeb

its pbi api get /v1.0/myorg/admin/datasets/<id>/datasources --to-file ./data/ds.json
```

---

### my

> Source: `src/providers/pbi/commands/my.ts`

| Command | Description |
|---------|-------------|
| `its pbi my login` | Sign in as a Power BI user. Opens your browser (authorisation code + PKCE, like `its auth login`) and caches the token locally (~/.its/secrets/pbi-my-token.json). --device-code uses the device-code flow instead, which THF's Conditional Access blocks (AADSTS53003). |
| `its pbi my logout` | Clear the cached Power BI user token. Does not revoke it server-side. |
| `its pbi my whoami` | Show the cached Power BI user account and token expiry. |
| `its pbi my workspaces` | List workspaces accessible to the signed-in user. Workspaces visible to the current principal. |
| `its pbi my reports` | List Power BI reports accessible to the signed-in user |
| `its pbi my datasets` | List Power BI datasets accessible to the signed-in user |
| `its pbi my add-workspace-user <workspace_id>` | Add a user/group/app to a workspace using the signed-in user's permissions (sidesteps SP admin-API restrictions when you're a workspace admin) |
| `its pbi my update-workspace-user <workspace_id>` | Update a user's access right on a workspace using the signed-in user's permissions |
| `its pbi my remove-workspace-user <workspace_id>` | Remove a user/group/app from a workspace using the signed-in user's permissions (requires --confirm) |
| `its pbi my refresh <dataset_id>` | Trigger a refresh of a dataset owned by the signed-in user. Force the provider to re-pull state from the upstream API. |
| `its pbi my refresh-schedule <dataset_id>` | A dataset's scheduled refresh: enabled, days, times, time zone. Delegated (`its pbi my login`). |
| `its pbi my refreshes <dataset_id>` | A dataset's refresh history, newest first (Power BI keeps the last 60). Shows status, start/end and the error CODE only, never the message (it can hold connection detail). Delegated. |
| `its pbi my datasources <dataset_id>` | The data sources a dataset reads from: type, server/database or URL, and the connection id (what Power BI calls gatewayId; with no gateways it is the owner's personal cloud connection). No credentials — the API never returns them. Delegated. |
| `its pbi my refresh-status` | Which refreshable datasets in a workspace are failing right now: last status and time, failures in the history, the most recent error code, and whether the schedule is on. Two reads per dataset, paced. Delegated. |

#### `its pbi my login`

Sign in as a Power BI user. Opens your browser (authorisation code + PKCE, like `its auth login`) and caches the token locally (~/.its/secrets/pbi-my-token.json). --device-code uses the device-code flow instead, which THF's Conditional Access blocks (AADSTS53003).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--device-code` | `` | Use the device-code flow instead of the browser (blocked at THF) | — |

**Examples:**

```bash
# Browser sign-in; token cached under ~/.its/secrets
its pbi my login

its pbi my login --device-code
```

#### `its pbi my logout`

Clear the cached Power BI user token. Does not revoke it server-side.

**Examples:**

```bash
# Local only — does not revoke server-side
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
# Uses your own permissions — sidesteps the service-principal admin restriction
its pbi my add-workspace-user 8f1c2d3e-... --user jane.smith@example.com --principal-type User --access Member

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
its pbi my update-workspace-user 8f1c2d3e-... --user jane.smith@example.com --principal-type User --access Contributor

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
its pbi my remove-workspace-user 8f1c2d3e-... --user jane.smith@example.com --confirm

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
its pbi my refresh 9a8b7c6d-... --workspace 8f1c2d3e-...

its pbi my refresh <dataset-id>

# Disambiguates when the dataset isn't in your personal workspace.
its pbi my refresh <dataset-id> --workspace <workspace-id>
```

#### `its pbi my refresh-schedule <dataset_id>`

A dataset's scheduled refresh: enabled, days, times, time zone. Delegated (`its pbi my login`).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--workspace` | `` | Workspace ID that holds the dataset (needed for datasets you do not own) | — |

```bash
its pbi my refresh-schedule <dataset_id>
```

#### `its pbi my refreshes <dataset_id>`

A dataset's refresh history, newest first (Power BI keeps the last 60). Shows status, start/end and the error CODE only, never the message (it can hold connection detail). Delegated.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--workspace` | `` | Workspace ID that holds the dataset (needed for datasets you do not own) | — |
| `--top` | `` | How many runs (default 20, max 60) | 20 |

```bash
its pbi my refreshes <dataset_id>
```

#### `its pbi my datasources <dataset_id>`

The data sources a dataset reads from: type, server/database or URL, and the connection id (what Power BI calls gatewayId; with no gateways it is the owner's personal cloud connection). No credentials — the API never returns them. Delegated.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--workspace` | `` | Workspace ID that holds the dataset (needed for datasets you do not own) | — |

```bash
its pbi my datasources <dataset_id>
```

#### `its pbi my refresh-status`

Which refreshable datasets in a workspace are failing right now: last status and time, failures in the history, the most recent error code, and whether the schedule is on. Two reads per dataset, paced. Delegated.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--workspace` | `` | Workspace ID (required) | — |
| `--failing` | `` | Only datasets whose latest run failed | — |

**Examples:**

```bash
its pbi my refresh-status --workspace <id> --failing
```

---
