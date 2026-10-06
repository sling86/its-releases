# Business Central (`bc`)

Business Central (Dynamics 365) — companies, multi-company entity queries via OData (standard v2.0 or a custom `--api publisher/group/version` route), record get/create/update/delete and bound actions (writes need an explicit `--env` and `--confirm`), environments (list, sandbox copy/delete, operations via the Admin Centre API), installed apps (list, updates, install, uninstall, update) and BC users/permission sets. Runs app-only as a dedicated app registration (BC_CLIENT_ID/BC_CLIENT_SECRET; falls back to the shared Entra credentials). That app needs: the Business Central API roles, an entry in each environment's BC page *Microsoft Entra Applications* (registration is per environment — production and every sandbox) with a user holding D365 AUTOMATION + D365 READ (add a write permission set only to test writes), and authorisation under the Admin Centre's *Authorized Microsoft Entra apps* for environments and apps.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [attendance](./attendance.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [companies](#companies)
- [environments](#environments)
- [entities](#entities)
- [extensions](#extensions)
- [query](#query)
- [record](#record)
- [users](#users)
- [apps](#apps)
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
| `src/providers/bc/write-guards.ts` | write guards |

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
| `its bc environments copy <source>` | Copy an environment into a new sandbox (Admin Centre API; sandbox only). The copy carries the source's per-tenant extensions and Entra app registrations, so copy Production to get a sandbox the CLI app can already use. Previews without --confirm; the copy runs in the background — follow it with `environments operations`. |
| `its bc environments delete <name>` | Delete a SANDBOX environment (Admin Centre API). Refuses anything that isn't a sandbox. Soft-deleted for 14 days. Previews without --confirm. |
| `its bc environments operations <name>` | Operations on an environment (copy, create, delete, update) with status — how to follow a copy. |

#### `its bc environments`

List all BC environments (Production/Sandbox) on the tenant.

```bash
its bc environments
```

#### `its bc environments copy <source>`

Copy an environment into a new sandbox (Admin Centre API; sandbox only). The copy carries the source's per-tenant extensions and Entra app registrations, so copy Production to get a sandbox the CLI app can already use. Previews without --confirm; the copy runs in the background — follow it with `environments operations`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | Name for the new environment | — |
| `--confirm` | `` | Apply the change | — |

**Examples:**

```bash
its bc environments copy Production --name Sandbox-Test --confirm
```

#### `its bc environments delete <name>`

Delete a SANDBOX environment (Admin Centre API). Refuses anything that isn't a sandbox. Soft-deleted for 14 days. Previews without --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Apply the change | — |

**Examples:**

```bash
its bc environments delete Sandbox-Test --confirm
```

#### `its bc environments operations <name>`

Operations on an environment (copy, create, delete, update) with status — how to follow a copy.

```bash
its bc environments operations <name>
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
| `its bc extensions` | List published AL extensions with their versions — the direct answer to "which environment is on which app version?". Prefers the Automation API (per-company, includes per-tenant extensions); where that identity is refused, falls back to the Admin Centre route, which needs a delegated Dynamics 365 admin (--auth az) and covers AppSource apps only. The summary says which answered. |
| `its bc extensions get <name>` | Get one extension's published version by name (substring, case-insensitive) — for gating on "is this environment on >= x.y.z?". |

#### `its bc extensions`

List published AL extensions with their versions — the direct answer to "which environment is on which app version?". Prefers the Automation API (per-company, includes per-tenant extensions); where that identity is refused, falls back to the Admin Centre route, which needs a delegated Dynamics 365 admin (--auth az) and covers AppSource apps only. The summary says which answered.

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
| `its bc record create <entity>` | Create a record. Needs --env <name> spelled out (never the default) and --body '{…}' or @file.json. Previews without --confirm. |
| `its bc record update <entity> <id>` | Change fields on a record (PATCH). Reads the record first and sends its etag, so a record changed since fails instead of being overwritten. Previews the changed fields without --confirm. |
| `its bc record delete <entity> <id>` | Delete a record. Reads it first and shows what would go; sends its etag. Previews without --confirm. |
| `its bc record call <entity> <id> <action>` | Run a bound action on a record (POST …/Microsoft.NAV.<action>), e.g. post or release. The action name must match the AL page's [ServiceEnabled] procedure. Previews without --confirm. |

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

#### `its bc record create <entity>`

Create a record. Needs --env <name> spelled out (never the default) and --body '{…}' or @file.json. Previews without --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--company` | `` | Company name/id (default: first company) | — |
| `--env` | `` | BC environment name — REQUIRED for writes, never defaulted | — |
| `--api` | `` | Custom API route <publisher>/<group>/<version> (default: standard v2.0 API) | — |
| `--body` | `` | JSON body — inline or @path/to/file.json | — |
| `--confirm` | `` | Apply the change | — |

**Examples:**

```bash
its bc record create customers --env Support --body '{"displayName":"Test Ltd"}' --confirm
```

#### `its bc record update <entity> <id>`

Change fields on a record (PATCH). Reads the record first and sends its etag, so a record changed since fails instead of being overwritten. Previews the changed fields without --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--company` | `` | Company name/id (default: first company) | — |
| `--env` | `` | BC environment name — REQUIRED for writes, never defaulted | — |
| `--api` | `` | Custom API route <publisher>/<group>/<version> (default: standard v2.0 API) | — |
| `--body` | `` | JSON body — inline or @path/to/file.json | — |
| `--confirm` | `` | Apply the change | — |

**Examples:**

```bash
its bc record update customers <id> --env Support --body '{"phoneNumber":"01234"}' --confirm
```

#### `its bc record delete <entity> <id>`

Delete a record. Reads it first and shows what would go; sends its etag. Previews without --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--company` | `` | Company name/id (default: first company) | — |
| `--env` | `` | BC environment name — REQUIRED for writes, never defaulted | — |
| `--api` | `` | Custom API route <publisher>/<group>/<version> (default: standard v2.0 API) | — |
| `--confirm` | `` | Apply the change | — |

**Examples:**

```bash
its bc record delete customers <id> --env Support --confirm
```

#### `its bc record call <entity> <id> <action>`

Run a bound action on a record (POST …/Microsoft.NAV.<action>), e.g. post or release. The action name must match the AL page's [ServiceEnabled] procedure. Previews without --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--company` | `` | Company name/id (default: first company) | — |
| `--env` | `` | BC environment name — REQUIRED for writes, never defaulted | — |
| `--api` | `` | Custom API route <publisher>/<group>/<version> (default: standard v2.0 API) | — |
| `--body` | `` | Optional JSON body for the action — inline or @file.json | — |
| `--confirm` | `` | Apply the change | — |

**Examples:**

```bash
its bc record call salesInvoices <id> post --env Support --confirm
```

---

### users

> Source: `src/providers/bc/commands.ts`

| Command | Description |
|---------|-------------|
| `its bc users` | Business Central users: user name, state, contact email. Automation API — if it 403s, rerun with --auth az or give the app's BC user the D365 AUTOMATION permission set. |
| `its bc users get <user>` | One BC user and every permission set they hold (with the company each applies to — blank = all companies). |
| `its bc users permission-sets` | Every permission set that can be assigned (system and extension-supplied). |
| `its bc users add-permission <user>` | Give a BC user a permission set — in one company (--in-company) or all (default). Previews without --confirm; reads the user's permissions back. |
| `its bc users remove-permission <user>` | Take a permission set off a BC user (the assignment matching --set and --in-company). Previews without --confirm. |

#### `its bc users`

Business Central users: user name, state, contact email. Automation API — if it 403s, rerun with --auth az or give the app's BC user the D365 AUTOMATION permission set.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--company` | `` | Company (name or id) whose API endpoint to use | — |
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |

```bash
its bc users
```

#### `its bc users get <user>`

One BC user and every permission set they hold (with the company each applies to — blank = all companies).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--company` | `` | Company (name or id) whose API endpoint to use | — |
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |

```bash
its bc users get <user>
```

#### `its bc users permission-sets`

Every permission set that can be assigned (system and extension-supplied).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--company` | `` | Company (name or id) whose API endpoint to use | — |
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |

```bash
its bc users permission-sets
```

#### `its bc users add-permission <user>`

Give a BC user a permission set — in one company (--in-company) or all (default). Previews without --confirm; reads the user's permissions back.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--set` | `` | Permission set id, e.g. D365 BUS FULL ACCESS | — |
| `--in-company` | `` | Company NAME the set applies in (omit = all companies) | — |
| `--confirm` | `` | Apply the change | — |
| `--company` | `` | Company (name or id) whose API endpoint to use | — |
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |

**Examples:**

```bash
its bc users add-permission jo.bloggs@example.com --set "D365 BUS FULL ACCESS" --in-company "CRONUS UK Ltd." --confirm
```

#### `its bc users remove-permission <user>`

Take a permission set off a BC user (the assignment matching --set and --in-company). Previews without --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--set` | `` | Permission set id, e.g. D365 BUS FULL ACCESS | — |
| `--in-company` | `` | Company NAME the set applies in (omit = all companies) | — |
| `--confirm` | `` | Apply the change | — |
| `--company` | `` | Company (name or id) whose API endpoint to use | — |
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |

**Examples:**

```bash
its bc users remove-permission jo.bloggs@example.com --set "D365 BUS FULL ACCESS" --in-company "CRONUS UK Ltd." --confirm
```

---

### apps

> Source: `src/providers/bc/commands.ts`

| Command | Description |
|---------|-------------|
| `its bc apps` | Apps installed on an environment, with appId, state and whether they can be uninstalled (Admin Centre API). |
| `its bc apps updates` | Newer versions available for installed global apps, with anything that must be installed or updated first. |
| `its bc apps upload <file>` | Upload a per-tenant extension (.app up to 50 MB) and install or update it (Admin Centre API pteInstall). Needs --env spelled out, --accept-eula and --confirm; previews the file and settings without --confirm. The install runs in the background — follow it with `apps scheduled` and `apps operations`. Try it in a sandbox first. |
| `its bc apps scheduled` | Per-tenant extension installs and updates that are scheduled but haven't run yet. |
| `its bc apps operations <appId>` | Install, update and uninstall operations for one app, with status and any error — how to follow an upload. |
| `its bc apps install <appId>` | Install a global app on an environment (Admin Centre API). Needs --env spelled out. --accept-eula is you agreeing to the publisher's terms. Previews without --confirm. |
| `its bc apps uninstall <appId>` | Uninstall an app. Data is kept unless --delete-data. Lists dependents that must go first. Needs --env spelled out. Previews without --confirm. |
| `its bc apps update <appId>` | Update a global app to --version (find it with `apps updates`). Needs --env spelled out. Previews without --confirm. |

#### `its bc apps`

Apps installed on an environment, with appId, state and whether they can be uninstalled (Admin Centre API).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |

```bash
its bc apps
```

#### `its bc apps updates`

Newer versions available for installed global apps, with anything that must be installed or updated first.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |

```bash
its bc apps updates
```

#### `its bc apps upload <file>`

Upload a per-tenant extension (.app up to 50 MB) and install or update it (Admin Centre API pteInstall). Needs --env spelled out, --accept-eula and --confirm; previews the file and settings without --confirm. The install runs in the background — follow it with `apps scheduled` and `apps operations`. Try it in a sandbox first.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |
| `--schedule` | `` | When to install: Immediate (default), UpdateWindow, NextMinorUpdate, NextMajorUpdate (a brand-new app can only be Immediate or UpdateWindow) | — |
| `--sync-mode` | `` | Schema sync: Add (default) or ForceSync (can drop data — only when you mean it) | — |
| `--with-dependencies` | `` | Also install or update apps this one needs (default: stop and list them) | — |
| `--accept-eula` | `` | Agree to the publisher's terms (required) | — |
| `--confirm` | `` | Upload and install | — |

**Examples:**

```bash
its bc apps upload ./THF_Reporting_1.0.0.0.app --env Support --accept-eula --confirm
```

#### `its bc apps scheduled`

Per-tenant extension installs and updates that are scheduled but haven't run yet.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |

```bash
its bc apps scheduled
```

#### `its bc apps operations <appId>`

Install, update and uninstall operations for one app, with status and any error — how to follow an upload.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |

```bash
its bc apps operations <appId>
```

#### `its bc apps install <appId>`

Install a global app on an environment (Admin Centre API). Needs --env spelled out. --accept-eula is you agreeing to the publisher's terms. Previews without --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |
| `--accept-eula` | `` | Agree to the publisher's EULA and Microsoft's marketplace terms (required to install) | — |
| `--version` | `` | Target version (default: latest) | — |
| `--with-dependencies` | `` | Also install or update apps this one needs (default: stop and list them) | — |
| `--confirm` | `` | Apply the change | — |

**Examples:**

```bash
its bc apps install <appId> --env Support --confirm
```

#### `its bc apps uninstall <appId>`

Uninstall an app. Data is kept unless --delete-data. Lists dependents that must go first. Needs --env spelled out. Previews without --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |
| `--delete-data` | `` | Also delete the app's data (cannot be undone) | — |
| `--confirm` | `` | Apply the change | — |

**Examples:**

```bash
its bc apps uninstall <appId> --env Support --confirm
```

#### `its bc apps update <appId>`

Update a global app to --version (find it with `apps updates`). Needs --env spelled out. Previews without --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--env` | `` | BC environment name (default: BC_ENVIRONMENT) | — |
| `--version` | `` | Target version (required) | — |
| `--with-dependencies` | `` | Also install or update apps this one needs (default: stop and list them) | — |
| `--confirm` | `` | Apply the change | — |

**Examples:**

```bash
its bc apps update <appId> --env Support --version 28.5.1.0 --confirm
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
