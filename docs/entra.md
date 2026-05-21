# Entra ID (`entra`)

Microsoft Entra ID (Azure AD) identity management — users, groups, licences, roles, sign-ins, security, directory, onboarding.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md)

## Contents

- [Setup](#setup)
- [users](#users)
- [groups](#groups)
- [licences](#licences)
- [roles](#roles)
- [signin](#signin)
- [tap](#tap)
- [ca](#ca)
- [authmethods](#authmethods)
- [consent](#consent)
- [xtenant](#xtenant)
- [audit](#audit)
- [security](#security)
- [directory](#directory)
- [onboarding](#onboarding)
- [offboarding](#offboarding)
- [break-glass](#break-glass)
- [whoami](#whoami)
- [doctor](#doctor)
- [apps](#apps)
- [admin-bootstrap](#admin-bootstrap)
- [graph](#graph)

## Setup

```bash
its entra setup           # Interactive wizard
its entra setup --check   # Check configuration status
its entra setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `TENANT_ID` | Microsoft Entra tenant ID (Entra admin centre > Overview) |
| `CLIENT_ID` | App registration client ID |
| `CLIENT_SECRET` | App registration client secret |

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/entra/client.ts` | API client methods |
| `src/providers/entra/types.ts` | TypeScript interfaces |
| `src/providers/entra/commands/` | Command definitions (split by resource) |
| `src/providers/entra/auth-methods.ts` | auth methods |
| `src/providers/entra/definition.ts` | definition |
| `src/providers/entra/sku-names.ts` | sku names |

## Resources

### users

> Source: `src/providers/entra/commands/users.ts`

| Command | Description |
|---------|-------------|
| `its entra users` | List users from the Entra ID directory. Defaults to first 50 — use --all to page through every user, --filter for OData expressions, or --domain for a quick UPN-suffix match. |
| `its entra users update <id>` | Update user properties. Use named flags for common fields, --set k=v,k=v for any other Graph user property. |
| `its entra users search <query>` | Fuzzy substring search across displayName, mail, UPN, jobTitle, department. Server-side startsWith for fast filtering — best for partial-name lookups. |
| `its entra users get <id>` | Full user profile — display fields, manager, sign-in state, employee metadata, on-prem sync flag, business phones, additional mail aliases. |
| `its entra users groups <id>` | All groups the user is a direct member of — security, M365, distribution, mail-enabled. Doesn't expand transitive (parent-of-parent) membership; use `users transitive-groups` for that. |
| `its entra users chain <id>` | Follow manager pointers up the org tree and render the chain. Useful for access-request escalation — stops at no-manager or a cycle. |
| `its entra users licences <id>` | Per-SKU licence assignments with service-plan level detail — which SKUs the user has, plus which features are enabled or disabled inside each. |
| `its entra users invite` | Send a B2B guest invitation to an external user. Send an invitation; the recipient takes the action. |
| `its entra users create` | Create a new user. Idempotent on duplicate names — use update/edit to mutate an existing record. |
| `its entra users enable <id>` | Enable a user account. Switch to active state. Idempotent. |
| `its entra users bootstrap-admin <id>` | Macro: create a TAP for the user, then optionally exclude them from CA policies that would block initial sign-in (e.g. phishing-resistant MFA admin policy). Emits a checklist with re-inclusion reminders. |
| `its entra users stale` | Find users with no recent successful sign-in — candidates for offboarding/clean-up. Defaults to enabled accounts with `lastSuccessfulSignInDateTime` older than 90 days; pass --days, --include-disabled, --include-guests to widen. |
| `its entra users disable <id>` | Disable a user account (requires --confirm). Pass --revoke-sessions to also kill active refresh tokens + cookies — recommended during offboarding so the user is forced out of M365 immediately. |
| `its entra users revoke-sessions <id>` | Revoke all sign-in sessions for a user (POST /users/{id}/revokeSignInSessions). Forces re-auth on all clients without disabling the account. |

#### `its entra users`

List users from the Entra ID directory. Defaults to first 50 — use --all to page through every user, --filter for OData expressions, or --domain for a quick UPN-suffix match.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Number of results (max 100) | 50 |
| `--all` | `` | Fetch all results (overrides --top) | — |
| `--filter` | `` | OData filter expression | — |
| `--enabled` | `` | Only show enabled accounts | — |
| `--domain` | `` | Filter by UPN domain (e.g. contractcandles.com) — server-side endsWith match | — |

**Examples:**

```bash
# Default — first 50 enabled accounts
its entra users

# Page through every user
its entra users --all

its entra users --enabled

its entra users --domain contractcandles.com

its entra users --fields displayName,upn,jobTitle
```

#### `its entra users update <id>`

Update user properties. Use named flags for common fields, --set k=v,k=v for any other Graph user property.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--displayName` | `` | Display name | — |
| `--jobTitle` | `` | Job title | — |
| `--department` | `` | Department | — |
| `--company` | `` | Company name | — |
| `--office` | `` | Office location | — |
| `--phone` | `` | Mobile phone | — |
| `--employeeId` | `` | Employee ID (PeopleHR EmployeeId) | — |
| `--streetAddress` | `` | Street address | — |
| `--city` | `` | City | — |
| `--state` | `` | State/county | — |
| `--postalCode` | `` | Postal code | — |
| `--country` | `` | Country | — |
| `--usage` | `` | Usage location (e.g. GB) | — |
| `--manager` | `` | Set manager (UPN or GUID) | — |
| `--clear-manager` | `` | Remove the manager assignment (top of org) | — |
| `--set` | `` | Freeform Graph property passthrough, comma-separated key=value pairs (e.g. --set extensionAttribute1=foo,givenName=Bar) | — |

**Examples:**

```bash
its entra users update jane.smith@example.com --department "Marketing"

its entra users update jane.smith@example.com --manager boss@example.com

its entra users update jane.smith@example.com --set "officeLocation=London,jobTitle=Lead"
```

#### `its entra users search <query>`

Fuzzy substring search across displayName, mail, UPN, jobTitle, department. Server-side startsWith for fast filtering — best for partial-name lookups.

**Examples:**

```bash
its entra users search "jane"

# Matches displayName, mail, jobTitle, department.
its entra users search "marketing"
```

#### `its entra users get <id>`

Full user profile — display fields, manager, sign-in state, employee metadata, on-prem sync flag, business phones, additional mail aliases.

**Examples:**

```bash
its entra users get jane.smith@example.com

# Object IDs work in place of UPN — useful from sign-in logs.
its entra users get 8d3b9c8f-...-aaa

its entra users get jane.smith@example.com --json
```

#### `its entra users groups <id>`

All groups the user is a direct member of — security, M365, distribution, mail-enabled. Doesn't expand transitive (parent-of-parent) membership; use `users transitive-groups` for that.

**Examples:**

```bash
its entra users groups jane.smith@example.com

# Pipe to jq to diff against a target user.
its entra users groups jane.smith@example.com --json
```

#### `its entra users chain <id>`

Follow manager pointers up the org tree and render the chain. Useful for access-request escalation — stops at no-manager or a cycle.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--depth` | `` | Maximum manager depth to follow (default 10) | 10 |

**Examples:**

```bash
# Walk up the manager tree
its entra users chain jane.smith@example.com

its entra users chain jane.smith@example.com --json
```

#### `its entra users licences <id>`

Per-SKU licence assignments with service-plan level detail — which SKUs the user has, plus which features are enabled or disabled inside each.

**Examples:**

```bash
its entra users licences jane.smith@example.com

# Surfaces serviceplan-level enabled/disabled detail.
its entra users licences jane.smith@example.com --json
```

#### `its entra users invite`

Send a B2B guest invitation to an external user. Send an invitation; the recipient takes the action.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--email` | `` | External email address (required) | — |
| `--name` | `` | Display name (defaults to email prefix) | — |
| `--redirect` | `` | Post-redemption redirect URL | https://myapps.microsoft.com |
| `--message` | `` | Custom invitation message body | — |
| `--no-email` | `` | Create guest without sending the invitation email | — |
| `--usage` | `` | ISO country code — also sets usageLocation on the created user | — |

**Examples:**

```bash
its entra users invite contractor@vendor.com --redirect "https://myapps.microsoft.com"

# Returns the redeem URL so you can hand it over manually.
its entra users invite contractor@vendor.com --no-email --redirect "https://myapps.microsoft.com"
```

#### `its entra users create`

Create a new user. Idempotent on duplicate names — use update/edit to mutate an existing record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--displayName` | `` | Display name | — |
| `--upn` | `` | User principal name | — |
| `--password` | `` | Initial password | — |
| `--jobTitle` | `` | Job title | — |
| `--department` | `` | Department | — |
| `--company` | `` | Company name | — |
| `--office` | `` | Office location | — |
| `--usage` | `` | Usage location (e.g. GB) | — |

**Examples:**

```bash
its entra users create --upn jane.smith@example.com --displayName "Jane Smith" --password "TempP@ss!" --force-change

# Pipe-friendly output — use with jq / scripts.
its entra users create --upn jane.smith@example.com --displayName "Jane Smith" --password "TempP@ss!" --force-change --json
```

#### `its entra users enable <id>`

Enable a user account. Switch to active state. Idempotent.

**Examples:**

```bash
its entra users enable jane.smith@example.com

its entra users enable jane.smith@example.com --json
```

#### `its entra users bootstrap-admin <id>`

Macro: create a TAP for the user, then optionally exclude them from CA policies that would block initial sign-in (e.g. phishing-resistant MFA admin policy). Emits a checklist with re-inclusion reminders.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--ttl` | `` | TAP lifetime (e.g. 1h, 4h, 1d). Default 1h | — |
| `--one-time` | `` | TAP usable only once | — |
| `--no-tap` | `` | Skip TAP creation (only run policy exclusions) | — |
| `--exclude-policies` | `` | Comma-separated CA policy IDs or displayNames to exclude the user from. Default empty — pass the Microsoft-managed phishing-resistant admin policy ID if you need it. | — |
| `--dry-run` | `` | Print actions without applying | — |

**Examples:**

```bash
# Creates a TAP and excludes user from blocking CA policies
its entra users bootstrap-admin newadmin@example.com

# Pipe-friendly output — use with jq / scripts.
its entra users bootstrap-admin newadmin@example.com --json
```

#### `its entra users stale`

Find users with no recent successful sign-in — candidates for offboarding/clean-up. Defaults to enabled accounts with `lastSuccessfulSignInDateTime` older than 90 days; pass --days, --include-disabled, --include-guests to widen.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--days` | `` | Threshold — flag accounts with no successful sign-in within this many days (default 90) | 90 |
| `--include-disabled` | `` | Include accounts where accountEnabled is false (default: only enabled) | — |
| `--include-guests` | `` | Include guest (#EXT#) accounts (default: only members) | — |
| `--include-never` | `` | Also flag accounts that have never had a successful sign-in (lastSuccessfulSignInDateTime is null) | — |
| `--top` | `` | Page size for the underlying Graph query (default 999) | 999 |
| `--all` | `` | Fetch every user (paginates) instead of capping at --top | — |

**Examples:**

```bash
# Users with no sign-in in 90 days
its entra users stale

its entra users stale --days 180

# By default we hide already-disabled accounts.
its entra users stale --include-disabled
```

#### `its entra users disable <id>`

Disable a user account (requires --confirm). Pass --revoke-sessions to also kill active refresh tokens + cookies — recommended during offboarding so the user is forced out of M365 immediately.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the disable | — |
| `--revoke-sessions` | `` | Also call /revokeSignInSessions — invalidates refresh tokens and forces re-auth on all clients | — |

**Examples:**

```bash
its entra users disable jane.smith@example.com --confirm

# Disable account + force re-auth on every existing session.
its entra users disable jane.smith@example.com --revoke --confirm
```

#### `its entra users revoke-sessions <id>`

Revoke all sign-in sessions for a user (POST /users/{id}/revokeSignInSessions). Forces re-auth on all clients without disabling the account.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the revoke | — |

**Examples:**

```bash
# Force re-auth on every device
its entra users revoke-sessions jane.smith@example.com

# Standard offboarding step — paired with disable.
its entra users revoke-sessions jane.smith@example.com --confirm
```

---

### groups

> Source: `src/providers/entra/commands/groups.ts`

| Command | Description |
|---------|-------------|
| `its entra groups` | List Entra ID groups. Surfaces the most common fields; pass --json for raw shape. Use --dynamic to narrow to dynamic-membership groups via server-side $filter (auto-fetches all pages). |
| `its entra groups search <query>` | Search groups by name. Substring match across the most relevant fields; case-insensitive. |
| `its entra groups get <group_id>` | Get group details. Pass the id (or any natural identifier) as the positional arg. |
| `its entra groups members <group_id>` | List members of a group. Pass --include-sps (or --all) to also surface service principals, nested groups, and devices — useful for groups like 'Fabric Admin API Service Principals' that the user-only view shows as empty. |
| `its entra groups create` | Create a new security group. Idempotent on duplicate names — use update/edit to mutate an existing record. |
| `its entra groups add-member <group_id>` | Add a user to a group. Refuses dynamic-membership groups (ctxc 41) — Graph accepts the call but the dynamic engine immediately overrides. Also refuses if the candidate is disabled / a leaver / has an active namesake (lesson 326 — Adam picked the wrong Steve/Colette/Nick). Pass --force to override either guard. |
| `its entra groups remove-member <group_id>` | Remove a user from a group (requires --confirm). Refuses dynamic-membership groups (ctxc 41); pass --force to override. |

#### `its entra groups`

List Entra ID groups. Surfaces the most common fields; pass --json for raw shape. Use --dynamic to narrow to dynamic-membership groups via server-side $filter (auto-fetches all pages).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Number of results | 50 |
| `--all` | `` | Fetch all results (overrides --top) | — |
| `--filter` | `` | OData filter expression | — |
| `--dynamic` | `` | Only dynamic-membership groups. Server-side $filter (groupTypes/any(t:t eq 'DynamicMembership')); auto-fetches all pages so the result is the true tenant count. | — |

**Examples:**

```bash
its entra groups --top 200

its entra groups --filter "startswith(displayName,'IT')"

# Re-runs every 10s — handy for dashboards or incident response.
its entra groups --top 200 --watch
```

#### `its entra groups search <query>`

Search groups by name. Substring match across the most relevant fields; case-insensitive.

**Examples:**

```bash
its entra groups search "all staff"

# Pipe-friendly output — use with jq / scripts.
its entra groups search "all staff" --json
```

#### `its entra groups get <group_id>`

Get group details. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its entra groups get <group-id>

# Pipe-friendly output — use with jq / scripts.
its entra groups get <group-id> --json
```

#### `its entra groups members <group_id>`

List members of a group. Pass --include-sps (or --all) to also surface service principals, nested groups, and devices — useful for groups like 'Fabric Admin API Service Principals' that the user-only view shows as empty.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--include-sps` | `` | Include service principals + non-user members (SPs, nested groups, devices) | — |
| `--all` | `` | Alias for --include-sps | — |

**Examples:**

```bash
its entra groups members <group-id>

# Surface service principals and nested groups too
its entra groups members <group-id> --all
```

#### `its entra groups create`

Create a new security group. Idempotent on duplicate names — use update/edit to mutate an existing record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--displayName` | `` | Group display name | — |
| `--description` | `` | Group description | — |
| `--mail` | `` | Mail-enabled | false |

**Examples:**

```bash
its entra groups create --displayName "Marketing"

# Pipe-friendly output — use with jq / scripts.
its entra groups create --displayName "Marketing" --json
```

#### `its entra groups add-member <group_id>`

Add a user to a group. Refuses dynamic-membership groups (ctxc 41) — Graph accepts the call but the dynamic engine immediately overrides. Also refuses if the candidate is disabled / a leaver / has an active namesake (lesson 326 — Adam picked the wrong Steve/Colette/Nick). Pass --force to override either guard.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | User ID to add | — |
| `--force` | `` | Override the dynamic-group guard AND the leaver / disabled / namesake guard | — |
| `--skip-leaver-check` | `` | Skip just the leaver/disabled/namesake guard (still enforces the dynamic-group guard) | — |

**Examples:**

```bash
# Guards against dynamic groups, disabled users, and namesake collisions
its entra groups add-member <group-id> --user jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its entra groups add-member <group-id> --user jane.smith@example.com --json
```

#### `its entra groups remove-member <group_id>`

Remove a user from a group (requires --confirm). Refuses dynamic-membership groups (ctxc 41); pass --force to override.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | User ID to remove | — |
| `--confirm` | `` | Confirm the removal | — |
| `--force` | `` | Override the dynamic-group guard | — |

**Examples:**

```bash
its entra groups remove-member <group-id> --user jane.smith@example.com --confirm

# Pipe-friendly output — use with jq / scripts.
its entra groups remove-member <group-id> --user jane.smith@example.com --confirm --json
```

---

### licences

> Source: `src/providers/entra/commands/licences.ts`

| Command | Description |
|---------|-------------|
| `its entra licences` | List all subscribed SKUs/licences. Available column gets a ⚠ marker when fewer than 10% of seats remain (or none) — useful for catching exhaustion before the next assign request fails. |
| `its entra licences assign <user_id>` | Assign a licence to a user. Idempotent — assigning twice is a no-op. |
| `its entra licences remove <user_id>` | Remove a licence from a user (requires --confirm). Permanent — use --confirm. |
| `its entra licences users <sku_id>` | List users assigned a specific licence SKU. List by resource membership; use --json for the raw shape. |
| `its entra licences unlicensed` | Find enabled users with no licence assignments. Read-only — useful for billing audits. |
| `its entra licences audit` | Licence usage report with availability and over-allocation warnings |
| `its entra licences waste` | Find disabled accounts with reclaimable licences. Read-only — useful for finding reclaimable seats. |

#### `its entra licences`

List all subscribed SKUs/licences. Available column gets a ⚠ marker when fewer than 10% of seats remain (or none) — useful for catching exhaustion before the next assign request fails.

**Examples:**

```bash
# All SKUs with consumed/available counts and capacity warnings
its entra licences

# Show only SKUs at zero seats
its entra licences --filter available=0

its entra licences --json

# All SKUs with consumed/available counts
its entra licences

# Compact view for piping to billing / capacity scripts.
its entra licences --json

# Re-runs every 10s — handy for dashboards or incident response.
its entra licences --watch
```

#### `its entra licences assign <user_id>`

Assign a licence to a user. Idempotent — assigning twice is a no-op.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--sku` | `` | SKU ID (GUID) | — |

**Examples:**

```bash
its entra licences assign jane.smith@example.com --sku SPB

# Some SKUs require usageLocation — set it inline.
its entra licences assign jane.smith@example.com --sku SPB --location GB
```

#### `its entra licences remove <user_id>`

Remove a licence from a user (requires --confirm). Permanent — use --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--sku` | `` | SKU ID (GUID) | — |
| `--confirm` | `` | Confirm the removal | — |

**Examples:**

```bash
its entra licences remove jane.smith@example.com --sku SPB --confirm

# Strips every assigned SKU — used in offboarding.
its entra licences remove jane.smith@example.com --all --confirm
```

#### `its entra licences users <sku_id>`

List users assigned a specific licence SKU. List by resource membership; use --json for the raw shape.

**Examples:**

```bash
its entra licences users --sku SPB

its entra licences users --sku SPB --json
```

#### `its entra licences unlicensed`

Find enabled users with no licence assignments. Read-only — useful for billing audits.

**Examples:**

```bash
# Active users without any licence — billing leak hunt
its entra licences unlicensed

its entra licences unlicensed --json
```

#### `its entra licences audit`

Licence usage report with availability and over-allocation warnings.

**Examples:**

```bash
# Per-SKU usage with under/over-allocation flags
its entra licences audit

# Feed the audit into a scheduled spend report.
its entra licences audit --json
```

#### `its entra licences waste`

Find disabled accounts with reclaimable licences. Read-only — useful for finding reclaimable seats.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--sku` | `` | Filter to specific SKU ID | — |

**Examples:**

```bash
# Disabled users still holding licences
its entra licences waste

# Use the json envelope's totals to file an exec report.
its entra licences waste --json
```

---

### roles

> Source: `src/providers/entra/commands/roles.ts`

| Command | Description |
|---------|-------------|
| `its entra roles` | List activated directory roles. Surfaces the most common fields; pass --json for raw shape. |
| `its entra roles members <role_id>` | List members of a directory role. Returns direct members; nested groups aren't expanded. |
| `its entra roles assign <user_id>` | Assign a directory role to a user. Idempotent — assigning twice is a no-op. |
| `its entra roles remove <user_id>` | Remove a directory role from a user (requires --confirm). Permanent — use --confirm. |
| `its entra roles assignments` | List all role assignments with role names. Returns the assignment record, not the principal — pair with `users get` for the readable name. |

#### `its entra roles`

List activated directory roles. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
# Currently active directory roles
its entra roles

# Pipe-friendly output — use with jq / scripts.
its entra roles --json

# Re-runs every 10s — handy for dashboards or incident response.
its entra roles --watch
```

#### `its entra roles members <role_id>`

List members of a directory role. Returns direct members; nested groups aren't expanded.

**Examples:**

```bash
its entra roles members <role-id>

# Pipe-friendly output — use with jq / scripts.
its entra roles members <role-id> --json
```

#### `its entra roles assign <user_id>`

Assign a directory role to a user. Idempotent — assigning twice is a no-op.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--role` | `` | Role definition ID | — |

**Examples:**

```bash
its entra roles assign jane.smith@example.com --role <role-def-id>

# Pipe-friendly output — use with jq / scripts.
its entra roles assign jane.smith@example.com --role <role-def-id> --json
```

#### `its entra roles remove <user_id>`

Remove a directory role from a user (requires --confirm). Permanent — use --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--role` | `` | Role definition ID | — |
| `--confirm` | `` | Confirm the removal | — |

**Examples:**

```bash
its entra roles remove jane.smith@example.com --role <role-def-id> --confirm
```

#### `its entra roles assignments`

List all role assignments with role names. Returns the assignment record, not the principal — pair with `users get` for the readable name.

**Examples:**

```bash
# Complete map of role → principal
its entra roles assignments

# Pipe-friendly output — use with jq / scripts.
its entra roles assignments --json
```

---

### signin

> Source: `src/providers/entra/commands/signin.ts`

| Command | Description |
|---------|-------------|
| `its entra signin` | List recent sign-in logs. Surfaces the most common fields; pass --json for raw shape. |
| `its entra signin summary <user_id>` | Sign-in summary for a user (last sign-in timestamps). Quick one-screen view — designed for dashboards / `--watch`. |
| `its entra signin explain <id>` | Decode a single sign-in event: error code, failureReason, and the CA policies that fired (appliedConditionalAccessPolicies). Pass the sign-in ID from `signin list`. |
| `its entra signin suspicious` | List sign-ins with risk indicators. Bounded by --since; defaults to last 24h. |

#### `its entra signin`

List recent sign-in logs. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--filter` | `` | OData filter expression | — |
| `--top` | `` | Number of results | 50 |
| `--all` | `` | Fetch all results (overrides --top) | — |
| `--user` | `` | Filter by user ID or UPN (auto-detected) | — |
| `--app` | `` | Filter by app display name | — |
| `--failed` | `` | Only failed sign-ins (errorCode ne 0) | — |
| `--status` | `` | Filter by sign-in status (alias for --failed=true when 'failure', --failed=false when 'success') | — |
| `--since` | `` | Look-back window (e.g. 1h, 6h, 1d, 7d) | — |

**Examples:**

```bash
its entra signin --since 1h

its entra signin --failed --since 24h

its entra signin --user jane.smith@example.com --since 7d

its entra signin --app "Microsoft Teams" --since 1d
```

#### `its entra signin summary <user_id>`

Sign-in summary for a user (last sign-in timestamps). Quick one-screen view — designed for dashboards / `--watch`.

**Examples:**

```bash
its entra signin summary jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its entra signin summary jane.smith@example.com --json

# Re-runs every 10s — handy for dashboards or incident response.
its entra signin summary jane.smith@example.com --watch
```

#### `its entra signin explain <id>`

Decode a single sign-in event: error code, failureReason, and the CA policies that fired (appliedConditionalAccessPolicies). Pass the sign-in ID from `signin list`.

**Examples:**

```bash
# Show error code + which CA policies fired
its entra signin explain <signin-id>

# Pipe-friendly output — use with jq / scripts.
its entra signin explain <signin-id> --json
```

#### `its entra signin suspicious`

List sign-ins with risk indicators. Bounded by --since; defaults to last 24h.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--days` | `` | Look-back period in days | 7 |

**Examples:**

```bash
its entra signin suspicious --days 7

# Pipe-friendly output — use with jq / scripts.
its entra signin suspicious --days 7 --json
```

---

### tap

> Source: `src/providers/entra/commands/tap.ts`

| Command | Description |
|---------|-------------|
| `its entra tap create <user_id>` | Create a Temporary Access Pass for a user (does not work for B2B guests) |
| `its entra tap <user_id>` | List active Temporary Access Passes for a user (codes redacted) |
| `its entra tap revoke <user_id> <method_id>` | Revoke a Temporary Access Pass. Reverse an assignment. --confirm where required. |

#### `its entra tap create <user_id>`

Create a Temporary Access Pass for a user (does not work for B2B guests).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--ttl` | `` | Lifetime (e.g. 30m, 1h, 8h, 1d). Default 1h | — |
| `--one-time` | `` | Pass is usable only once | — |
| `--start-time` | `` | ISO 8601 timestamp for scheduled start (e.g. 2026-04-21T09:00:00Z) | — |

**Examples:**

```bash
its entra tap create jane.smith@example.com

its entra tap create jane.smith@example.com --lifetime 480 --usable-once false
```

#### `its entra tap <user_id>`

List active Temporary Access Passes for a user (codes redacted).

**Examples:**

```bash
# Temporary access passes outstanding
its entra tap

# Pipe-friendly output — use with jq / scripts.
its entra tap --json

# Re-runs every 10s — handy for dashboards or incident response.
its entra tap --watch
```

#### `its entra tap revoke <user_id> <method_id>`

Revoke a Temporary Access Pass. Reverse an assignment. --confirm where required.

**Examples:**

```bash
its entra tap revoke jane.smith@example.com <method-id>

# Pipe-friendly output — use with jq / scripts.
its entra tap revoke jane.smith@example.com <method-id> --json
```

---

### ca

> Source: `src/providers/entra/commands/ca.ts`

| Command | Description |
|---------|-------------|
| `its entra ca` | List all Conditional Access policies. Surfaces the most common fields; pass --json for raw shape. |
| `its entra ca enabled` | List only enabled CA policies. Filters by `state eq 'enabled'`. |
| `its entra ca get <id_or_name>` | Show full CA policy JSON by id or displayName. Pass the id (or any natural identifier) as the positional arg. |
| `its entra ca patch <id_or_name>` | Patch a CA policy (state or full JSON body). PATCH semantics — only the supplied fields change. |
| `its entra ca exclude-guests <id_or_name>` | Add 'All guest and external users' to excludeGuestsOrExternalUsers on a policy |
| `its entra ca create` | Create a Conditional Access policy from a JSON body or file (@path) |
| `its entra ca delete <id_or_name>` | Delete a Conditional Access policy by id or displayName. Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record. |
| `its entra ca exclude-user <id_or_name> <user>` | Add a user to a CA policy's excludeUsers — atomic mutation, no need to rebuild the users object |
| `its entra ca unexclude-user <id_or_name> <user>` | Remove a user from a CA policy's excludeUsers list. Reverse of `exclude-user`. Idempotent. |
| `its entra ca why-blocked <user>` | Show which enabled CA policies target a user, plus the grant controls each requires. Resolves group + role membership. |
| `its entra ca named-locations` | List trusted/named locations referenced by CA policies |

#### `its entra ca`

List all Conditional Access policies. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--state` | `` | Filter by state | — |

**Examples:**

```bash
# Conditional access policies (state + targets)
its entra ca

# Pipe-friendly output — use with jq / scripts.
its entra ca --json

# Re-runs every 10s — handy for dashboards or incident response.
its entra ca --watch
```

#### `its entra ca enabled`

List only enabled CA policies. Filters by `state eq 'enabled'`.

**Examples:**

```bash
its entra ca enabled

# Pipe-friendly output — use with jq / scripts.
its entra ca enabled --json
```

#### `its entra ca get <id_or_name>`

Show full CA policy JSON by id or displayName. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its entra ca get <policy-id>

# Pipe-friendly output — use with jq / scripts.
its entra ca get <policy-id> --json
```

#### `its entra ca patch <id_or_name>`

Patch a CA policy (state or full JSON body). PATCH semantics — only the supplied fields change.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--state` | `` | New state | — |
| `--body` | `` | Raw JSON body to PATCH (overrides --state) — inline or @path/to/file.json | — |
| `--dry-run` | `` | Print the PATCH request without sending | — |

**Examples:**

```bash
its entra ca patch <policy-id> --state enabled

its entra ca patch <policy-id> --state enabledForReportingButNotEnforced
```

#### `its entra ca exclude-guests <id_or_name>`

Add 'All guest and external users' to excludeGuestsOrExternalUsers on a policy.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--dry-run` | `` | Print the PATCH request without sending | — |

**Examples:**

```bash
its entra ca exclude-guests <policy-id>

# Pipe-friendly output — use with jq / scripts.
its entra ca exclude-guests <policy-id> --json
```

#### `its entra ca create`

Create a Conditional Access policy from a JSON body or file (@path).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--body` | `` | Inline JSON or @path/to/policy.json — the POST body for the new policy | — |
| `--dry-run` | `` | Print the POST request without sending | — |

**Examples:**

```bash
its entra ca create @./policy.json

# Pipe-friendly output — use with jq / scripts.
its entra ca create @./policy.json --json
```

#### `its entra ca delete <id_or_name>`

Delete a Conditional Access policy by id or displayName. Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--dry-run` | `` | Print the DELETE request without sending | — |

**Examples:**

```bash
its entra ca delete "Block legacy auth" --confirm
```

#### `its entra ca exclude-user <id_or_name> <user>`

Add a user to a CA policy's excludeUsers — atomic mutation, no need to rebuild the users object.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--dry-run` | `` | Print the PATCH request without sending | — |

**Examples:**

```bash
# Use sparingly — re-include the user once the trip-cause is cleared.
its entra ca exclude-user <policy-id> --user jane.smith@example.com

# Policy can be addressed by exact displayName instead of id.
its entra ca exclude-user "Require MFA for admins" --user jane.smith@example.com
```

#### `its entra ca unexclude-user <id_or_name> <user>`

Remove a user from a CA policy's excludeUsers list. Reverse of `exclude-user`. Idempotent.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--dry-run` | `` | Print the PATCH request without sending | — |

**Examples:**

```bash
its entra ca unexclude-user <policy-id> jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its entra ca unexclude-user <policy-id> jane.smith@example.com --json
```

#### `its entra ca why-blocked <user>`

Show which enabled CA policies target a user, plus the grant controls each requires. Resolves group + role membership.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--include-disabled` | `` | Also evaluate disabled and reportOnly policies | — |

**Examples:**

```bash
# Which CA policy is blocking sign-in?
its entra ca why-blocked jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its entra ca why-blocked jane.smith@example.com --json
```

#### `its entra ca named-locations`

List trusted/named locations referenced by CA policies.

**Examples:**

```bash
its entra ca named-locations

# Pipe-friendly output — use with jq / scripts.
its entra ca named-locations --json
```

---

### authmethods

> Source: `src/providers/entra/commands/auth-methods.ts`

| Command | Description |
|---------|-------------|
| `its entra authmethods policy` | Fetch the full Authentication Methods Policy — shows which methods (TAP, FIDO2, SMS, etc.) are enabled tenant-wide and any per-method targets |
| `its entra authmethods get <method>` | Get a single authentication method configuration by id (TemporaryAccessPass, Fido2, Sms, MicrosoftAuthenticator, Email, Voice, X509Certificate, etc.) |
| `its entra authmethods enable <method>` | Enable a method tenant-wide (sets state=enabled on the method configuration). Pass --target <groupId> or --target all-users to scope. |
| `its entra authmethods disable <method>` | Disable a method tenant-wide (state=disabled). Switch to inactive state. Idempotent. |
| `its entra authmethods patch <method>` | Patch a method configuration with a custom body (inline or @path/to/file.json) |

#### `its entra authmethods policy`

Fetch the full Authentication Methods Policy — shows which methods (TAP, FIDO2, SMS, etc.) are enabled tenant-wide and any per-method targets.

**Examples:**

```bash
# Tenant-wide TAP / FIDO2 / Authenticator state
its entra authmethods policy

# Pipe-friendly output — use with jq / scripts.
its entra authmethods policy --json
```

#### `its entra authmethods get <method>`

Get a single authentication method configuration by id (TemporaryAccessPass, Fido2, Sms, MicrosoftAuthenticator, Email, Voice, X509Certificate, etc.).

**Examples:**

```bash
its entra authmethods get TemporaryAccessPass

# Pipe-friendly output — use with jq / scripts.
its entra authmethods get TemporaryAccessPass --json
```

#### `its entra authmethods enable <method>`

Enable a method tenant-wide (sets state=enabled on the method configuration). Pass --target <groupId> or --target all-users to scope.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--target` | `` | Optional include target — group ID or 'all_users'. Adds a default include target if specified. | — |
| `--dry-run` | `` | Print the PATCH request without sending | — |

**Examples:**

```bash
its entra authmethods enable Fido2

# Pipe-friendly output — use with jq / scripts.
its entra authmethods enable Fido2 --json
```

#### `its entra authmethods disable <method>`

Disable a method tenant-wide (state=disabled). Switch to inactive state. Idempotent.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--dry-run` | `` | Print the PATCH request without sending | — |

**Examples:**

```bash
its entra authmethods disable Sms

# Pipe-friendly output — use with jq / scripts.
its entra authmethods disable Sms --json
```

#### `its entra authmethods patch <method>`

Patch a method configuration with a custom body (inline or @path/to/file.json).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--body` | `` | Inline JSON or @path/to/body.json — the PATCH body for the method config | — |
| `--dry-run` | `` | Print the PATCH request without sending | — |

**Examples:**

```bash
its entra authmethods patch TemporaryAccessPass @./tap-policy.json

# Pipe-friendly output — use with jq / scripts.
its entra authmethods patch TemporaryAccessPass @./tap-policy.json --json
```

---

### consent

> Source: `src/providers/entra/commands/consent.ts`

| Command | Description |
|---------|-------------|
| `its entra consent <client_app_id>` | List delegated OAuth2 permission grants for a client app (existing consent) |
| `its entra consent add-scope <client_app_id> <scopes>` | Safely add delegated scope(s) to an app — UNION with existing scope, never replaces |
| `its entra consent remove-scope <client_app_id> <scopes>` | Remove delegated scope(s) from an existing grant — keeps others intact |
| `its entra consent app-roles <client_app_id>` | List application-permission grants (appRoleAssignments) for a client app |
| `its entra consent app-role-grant <client_app_id> <role>` | Grant an application-permission (app-role) to a client app — equivalent to consenting an Application permission in the portal |
| `its entra consent app-role-revoke <client_app_id> <assignment_id>` | Revoke an app-role assignment by id. Revokes a previously-granted app-role grant. Permanent. |

#### `its entra consent <client_app_id>`

List delegated OAuth2 permission grants for a client app (existing consent).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--resource-app-id` | `` | Filter by resource appId (default: Microsoft Graph). Use '*' for all. | — |

**Examples:**

```bash
its entra consent <app-id>

# Pipe-friendly output — use with jq / scripts.
its entra consent <app-id> --json

# Re-runs every 10s — handy for dashboards or incident response.
its entra consent <app-id> --watch
```

#### `its entra consent add-scope <client_app_id> <scopes>`

Safely add delegated scope(s) to an app — UNION with existing scope, never replaces.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--resource-app-id` | `` | Resource appId (default: Microsoft Graph) | — |
| `--principal-id` | `` | Grant on behalf of a specific user (consentType=Principal). Default: AllPrincipals (tenant-wide) | — |
| `--dry-run` | `` | Print the request without sending | — |

**Examples:**

```bash
# Unions with existing scopes, never replaces
its entra consent add-scope <app-id> Mail.Read

# Pipe-friendly output — use with jq / scripts.
its entra consent add-scope <app-id> Mail.Read --json
```

#### `its entra consent remove-scope <client_app_id> <scopes>`

Remove delegated scope(s) from an existing grant — keeps others intact.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--resource-app-id` | `` | Resource appId (default: Microsoft Graph) | — |
| `--principal-id` | `` | Per-user grant — default AllPrincipals (tenant-wide) | — |
| `--dry-run` | `` | Print the request without sending | — |

**Examples:**

```bash
its entra consent remove-scope <app-id> Mail.Read

# Pipe-friendly output — use with jq / scripts.
its entra consent remove-scope <app-id> Mail.Read --json
```

#### `its entra consent app-roles <client_app_id>`

List application-permission grants (appRoleAssignments) for a client app.

**Examples:**

```bash
its entra consent app-roles <app-id>

# Pipe-friendly output — use with jq / scripts.
its entra consent app-roles <app-id> --json
```

#### `its entra consent app-role-grant <client_app_id> <role>`

Grant an application-permission (app-role) to a client app — equivalent to consenting an Application permission in the portal.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--resource-app-id` | `` | Resource appId (default: Microsoft Graph) | — |
| `--dry-run` | `` | Print the request without sending | — |

**Examples:**

```bash
its entra consent app-role-grant <app-id> User.Read.All

# Pipe-friendly output — use with jq / scripts.
its entra consent app-role-grant <app-id> User.Read.All --json
```

#### `its entra consent app-role-revoke <client_app_id> <assignment_id>`

Revoke an app-role assignment by id. Revokes a previously-granted app-role grant. Permanent.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--resource-app-id` | `` | Resource appId (default: Microsoft Graph) | — |
| `--confirm` | `` | Confirm the revoke | — |

**Examples:**

```bash
its entra consent app-role-revoke <app-id> <assignment-id>

# Pipe-friendly output — use with jq / scripts.
its entra consent app-role-revoke <app-id> <assignment-id> --json
```

---

### xtenant

> Source: `src/providers/entra/commands/xtenant.ts`

| Command | Description |
|---------|-------------|
| `its entra xtenant default` | Show the default cross-tenant access policy. Returns the tenant-default policy / record. |
| `its entra xtenant trust-mfa <on_off>` | Toggle inboundTrust.isMfaAccepted on the default policy |
| `its entra xtenant trust-device <on_off>` | Toggle inboundTrust.isCompliantDeviceAccepted on the default policy |
| `its entra xtenant partners` | List per-partner cross-tenant access overrides. Returns per-partner overrides — use `partner-set` to mutate. |
| `its entra xtenant partner-add <tenant_id>` | Start a per-partner cross-tenant access config. Bootstraps a fresh per-partner override record. |
| `its entra xtenant partner-set <tenant_id>` | Patch a partner cross-tenant access record. Patches an existing partner record. |

#### `its entra xtenant default`

Show the default cross-tenant access policy. Returns the tenant-default policy / record.

**Examples:**

```bash
# Default cross-tenant access settings
its entra xtenant default

# Pipe-friendly output — use with jq / scripts.
its entra xtenant default --json
```

#### `its entra xtenant trust-mfa <on_off>`

Toggle inboundTrust.isMfaAccepted on the default policy.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--dry-run` | `` | Print the PATCH without sending | — |

**Examples:**

```bash
its entra xtenant trust-mfa on

# Pipe-friendly output — use with jq / scripts.
its entra xtenant trust-mfa on --json
```

#### `its entra xtenant trust-device <on_off>`

Toggle inboundTrust.isCompliantDeviceAccepted on the default policy.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--dry-run` | `` | Print the PATCH without sending | — |

**Examples:**

```bash
its entra xtenant trust-device on

# Pipe-friendly output — use with jq / scripts.
its entra xtenant trust-device on --json
```

#### `its entra xtenant partners`

List per-partner cross-tenant access overrides. Returns per-partner overrides — use `partner-set` to mutate.

**Examples:**

```bash
its entra xtenant partners

# Pipe-friendly output — use with jq / scripts.
its entra xtenant partners --json
```

#### `its entra xtenant partner-add <tenant_id>`

Start a per-partner cross-tenant access config. Bootstraps a fresh per-partner override record.

**Examples:**

```bash
its entra xtenant partner-add <partner-tenant-id>

# Pipe-friendly output — use with jq / scripts.
its entra xtenant partner-add <partner-tenant-id> --json
```

#### `its entra xtenant partner-set <tenant_id>`

Patch a partner cross-tenant access record. Patches an existing partner record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--trust-mfa` | `` | on or off | — |
| `--trust-device` | `` | on or off | — |
| `--trust-haadj` | `` | on or off (Hybrid Azure AD Joined) | — |
| `--dry-run` | `` | Print the PATCH without sending | — |

**Examples:**

```bash
its entra xtenant partner-set <partner-tenant-id> --trust-mfa on

# Pipe-friendly output — use with jq / scripts.
its entra xtenant partner-set <partner-tenant-id> --trust-mfa on --json
```

---

### audit

> Source: `src/providers/entra/commands/audit.ts`

| Command | Description |
|---------|-------------|
| `its entra audit` | List directory audit logs. Surfaces the most common fields; pass --json for raw shape. |

#### `its entra audit`

List directory audit logs. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--filter` | `` | OData filter expression | — |
| `--top` | `` | Number of results | 50 |
| `--all` | `` | Fetch all results (overrides --top) | — |
| `--category` | `` | Filter by category | — |
| `--initiated-by` | `` | Filter by initiating user ID (GUID) or app ID | — |
| `--target` | `` | Filter by target resource ID (GUID) | — |
| `--activity` | `` | Filter by activityDisplayName (exact match, e.g. 'Add member to group') | — |
| `--result` | `` | Filter by result | — |

**Examples:**

```bash
its entra audit --since 24h

its entra audit --user jane.smith@example.com

# Re-runs every 10s — handy for dashboards or incident response.
its entra audit --since 24h --watch
```

---

### security

> Source: `src/providers/entra/commands/security.ts`

| Command | Description |
|---------|-------------|
| `its entra security risky` | List risky users from Identity Protection. Returns users with non-none risk score. |
| `its entra security mfa <user_id>` | Check MFA readiness for a user. Returns user's strong-auth methods plus a readiness flag. |
| `its entra security admin-mfa` | Audit all admin users for MFA readiness. Audit pass — flags any admin without strong MFA. |

#### `its entra security risky`

List risky users from Identity Protection. Returns users with non-none risk score.

**Examples:**

```bash
# Identity Protection risk score > none
its entra security risky

# Pipe-friendly output — use with jq / scripts.
its entra security risky --json
```

#### `its entra security mfa <user_id>`

Check MFA readiness for a user. Returns user's strong-auth methods plus a readiness flag.

**Examples:**

```bash
# Users without strong MFA
its entra security mfa

# Pipe-friendly output — use with jq / scripts.
its entra security mfa --json
```

#### `its entra security admin-mfa`

Audit all admin users for MFA readiness. Audit pass — flags any admin without strong MFA.

**Examples:**

```bash
# Privileged accounts without MFA
its entra security admin-mfa

# Pipe-friendly output — use with jq / scripts.
its entra security admin-mfa --json
```

---

### directory

> Source: `src/providers/entra/commands/directory.ts`

| Command | Description |
|---------|-------------|
| `its entra directory org` | Get tenant organisation details. Returns the tenant organisation profile. |
| `its entra directory domains` | List verified domains. Returns verified + provisioning domains. |
| `its entra directory deleted` | List recently deleted users. Returns soft-deleted users still within the 30-day restore window. |
| `its entra directory tree <user_id>` | Build org tree from a root user (recursive). Recursive — pass --depth to bound the walk. |
| `its entra directory summary` | Aggregate users by company, department, location. Quick one-screen view — designed for dashboards / `--watch`. |
| `its entra directory app-usage` | Licence and app usage summary. Aggregated app + licence usage across the tenant. |

#### `its entra directory org`

Get tenant organisation details. Returns the tenant organisation profile.

**Examples:**

```bash
its entra directory org

# Pipe-friendly output — use with jq / scripts.
its entra directory org --json
```

#### `its entra directory domains`

List verified domains. Returns verified + provisioning domains.

**Examples:**

```bash
its entra directory domains

# Pipe-friendly output — use with jq / scripts.
its entra directory domains --json
```

#### `its entra directory deleted`

List recently deleted users. Returns soft-deleted users still within the 30-day restore window.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Number of results | 50 |
| `--all` | `` | Fetch all results (overrides --top) | — |

**Examples:**

```bash
# Soft-deleted users (30-day window)
its entra directory deleted

# Pipe-friendly output — use with jq / scripts.
its entra directory deleted --json
```

#### `its entra directory tree <user_id>`

Build org tree from a root user (recursive). Recursive — pass --depth to bound the walk.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--depth` | `` | Max depth (1-5, default 3) | 3 |

**Examples:**

```bash
# Recursive manager hierarchy
its entra directory tree

# Pipe-friendly output — use with jq / scripts.
its entra directory tree --json
```

#### `its entra directory summary`

Aggregate users by company, department, location. Quick one-screen view — designed for dashboards / `--watch`.

**Examples:**

```bash
its entra directory summary

# Pipe-friendly output — use with jq / scripts.
its entra directory summary --json

# Re-runs every 10s — handy for dashboards or incident response.
its entra directory summary --watch
```

#### `its entra directory app-usage`

Licence and app usage summary. Aggregated app + licence usage across the tenant.

**Examples:**

```bash
its entra directory app-usage

# Pipe-friendly output — use with jq / scripts.
its entra directory app-usage --json
```

---

### onboarding

> Source: `src/providers/entra/commands/onboarding.ts`

| Command | Description |
|---------|-------------|
| `its entra onboarding summary <user_id>` | Onboarding snapshot: user profile, licences, groups, manager |
| `its entra onboarding copy-groups` | Copy group memberships from one user to another. Idempotent — already-shared groups are skipped. |
| `its entra onboarding convert-mailbox <user_id>` | Convert user mailbox to shared (disable + remove licences). Requires --confirm. |

#### `its entra onboarding summary <user_id>`

Onboarding snapshot: user profile, licences, groups, manager.

**Examples:**

```bash
its entra onboarding summary jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its entra onboarding summary jane.smith@example.com --json

# Re-runs every 10s — handy for dashboards or incident response.
its entra onboarding summary jane.smith@example.com --watch
```

#### `its entra onboarding copy-groups`

Copy group memberships from one user to another. Idempotent — already-shared groups are skipped.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--source` | `` | Source user ID or UPN | — |
| `--target` | `` | Target user ID or UPN | — |

**Examples:**

```bash
its entra onboarding copy-groups --from peer@example.com --to jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its entra onboarding copy-groups --from peer@example.com --to jane.smith@example.com --json
```

#### `its entra onboarding convert-mailbox <user_id>`

Convert user mailbox to shared (disable + remove licences). Requires --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the conversion | — |

**Examples:**

```bash
# Disables user + removes licences after conversion
its entra onboarding convert-mailbox jane.smith@example.com --confirm

# Pipe-friendly output — use with jq / scripts.
its entra onboarding convert-mailbox jane.smith@example.com --confirm --json
```

---

### offboarding

> Source: `src/providers/entra/commands/onboarding.ts`

| Command | Description |
|---------|-------------|
| `its entra offboarding summary <user_id>` | Offboarding checklist: disable account, list licences and groups to remove |
| `its entra offboarding run <user_id>` | Execute offboarding: disable account, revoke licences, remove from groups. Requires --confirm. |

#### `its entra offboarding summary <user_id>`

Offboarding checklist: disable account, list licences and groups to remove.

**Examples:**

```bash
its entra offboarding summary jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its entra offboarding summary jane.smith@example.com --json

# Re-runs every 10s — handy for dashboards or incident response.
its entra offboarding summary jane.smith@example.com --watch
```

#### `its entra offboarding run <user_id>`

Execute offboarding: disable account, revoke licences, remove from groups. Requires --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the offboarding | — |

**Examples:**

```bash
# Disable, revoke, strip licences, remove groups
its entra offboarding run jane.smith@example.com --confirm

# Pipe-friendly output — use with jq / scripts.
its entra offboarding run jane.smith@example.com --confirm --json
```

---

### break-glass

> Source: `src/providers/entra/commands/break-glass.ts`

| Command | Description |
|---------|-------------|
| `its entra break-glass audit` | Audit THF break-glass accounts (BG01 + BG02) — GA role, CA exclusions, sign-in hygiene, password age, FIDO2. Non-zero exit on any gap. Override UPNs with ENTRA_BG01_UPN / ENTRA_BG02_UPN. |

#### `its entra break-glass audit`

Audit THF break-glass accounts (BG01 + BG02) — GA role, CA exclusions, sign-in hygiene, password age, FIDO2. Non-zero exit on any gap. Override UPNs with ENTRA_BG01_UPN / ENTRA_BG02_UPN.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--bg01` | `` | Override BG01 UPN (defaults to ENTRA_BG01_UPN env) | — |
| `--bg02` | `` | Override BG02 UPN (defaults to ENTRA_BG02_UPN env) | — |

**Examples:**

```bash
# Verify emergency-access accounts are healthy + excluded from CA
its entra break-glass audit

# Pipe-friendly output — use with jq / scripts.
its entra break-glass audit --json
```

---

### whoami

> Source: `src/providers/entra/commands/whoami.ts`

| Command | Description |
|---------|-------------|
| `its entra whoami show` | Dump the current Entra CLIENT_ID, app display name, Graph app-role assignments (application permissions), delegated scopes, and the live access token's claims. Use before any mutation to check 'will I have permission'. See ctxc 244. |

#### `its entra whoami show`

Dump the current Entra CLIENT_ID, app display name, Graph app-role assignments (application permissions), delegated scopes, and the live access token's claims. Use before any mutation to check 'will I have permission'. See ctxc 244.

**Examples:**

```bash
its entra whoami

# Pipe-friendly output — use with jq / scripts.
its entra whoami --json
```

---

### doctor

> Source: `src/providers/entra/commands/doctor.ts`

| Command | Description |
|---------|-------------|
| `its entra doctor` | Parallel health check on tenant — expiring secrets, admin MFA gaps, risky users, licensed-disabled accounts, stale guests |

#### `its entra doctor`

Parallel health check on tenant — expiring secrets, admin MFA gaps, risky users, licensed-disabled accounts, stale guests.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--secret-warn-days` | `` | Flag app credentials expiring within this many days (default 60) | 60 |
| `--stale-days` | `` | Threshold for stale enabled accounts (default 90) | 90 |

**Examples:**

```bash
its entra doctor

# Pipe-friendly output — use with jq / scripts.
its entra doctor --json

# Re-runs every 10s — handy for dashboards or incident response.
its entra doctor --watch
```

---

### apps

> Source: `src/providers/entra/commands/apps.ts`

| Command | Description |
|---------|-------------|
| `its entra apps register <name>` | Bootstrap a new Entra app registration end-to-end — creates the application, its service principal, and a 12-month client secret. Returns the appId, objectId, SP id, and the plaintext secret (only shown once). |
| `its entra apps add-password <app>` | Rotate / add a client secret on an existing app registration. The plaintext secretText is only returned once. |

#### `its entra apps register <name>`

Bootstrap a new Entra app registration end-to-end — creates the application, its service principal, and a 12-month client secret. Returns the appId, objectId, SP id, and the plaintext secret (only shown once).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--redirect-uri` | `` | Web redirect URI. Repeatable — pass multiple --redirect-uri flags or comma-separate. | — |
| `--audience` | `` | Sign-in audience | single |
| `--secret-name` | `` | Display name for the client secret | its-cli generated |
| `--months` | `` | Client secret lifetime in months | 12 |
| `--no-secret` | `` | Skip generating a client secret | — |

```bash
its entra apps register <name>
```

#### `its entra apps add-password <app>`

Rotate / add a client secret on an existing app registration. The plaintext secretText is only returned once.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | Display name for the secret | its-cli generated |
| `--months` | `` | Lifetime in months | 12 |

```bash
its entra apps add-password <app>
```

---

### admin-bootstrap

> Source: `src/providers/entra/commands/admin-bootstrap.ts`

| Command | Description |
|---------|-------------|
| `its entra admin-bootstrap run <user_id>` | Unblock an admin who can't sign in for phishing-resistant MFA. Tries TAP first; falls back to a per-policy excludeUsers on the Microsoft-managed phish-resistant admin policy. Always emits a re-inclusion checklist. |

#### `its entra admin-bootstrap run <user_id>`

Unblock an admin who can't sign in for phishing-resistant MFA. Tries TAP first; falls back to a per-policy excludeUsers on the Microsoft-managed phish-resistant admin policy. Always emits a re-inclusion checklist.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--ttl` | `` | TAP lifetime in minutes (default 60) | 60 |
| `--one-time` | `` | TAP is single-use | — |
| `--no-fallback` | `` | Don't exclude from the phish-resistant policy if TAP fails — just report. | — |
| `--dry-run` | `` | Show the plan without mutating | — |

```bash
its entra admin-bootstrap run <user_id>
```

---

### graph

| Command | Description |
|---------|-------------|
| `its entra graph get <path>` | Raw Graph GET — pass any /v1.0 or /beta path (use --beta for beta) |
| `its entra graph post <path>` | Raw Graph POST — pass any /v1.0 or /beta path (use --beta for beta) |
| `its entra graph patch <path>` | Raw Graph PATCH — pass any /v1.0 or /beta path (use --beta for beta) |
| `its entra graph put <path>` | Raw Graph PUT — pass any /v1.0 or /beta path (use --beta for beta) |
| `its entra graph delete <path>` | Raw Graph DELETE — pass any /v1.0 or /beta path (use --beta for beta) |

#### `its entra graph get <path>`

Raw Graph GET — pass any /v1.0 or /beta path (use --beta for beta).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--beta` | `` | Use /beta instead of /v1.0 | — |
| `--header` | `` | Extra headers as comma-separated K=V pairs (e.g. Prefer=return=minimal) | — |
| `--raw` | `` | Return the response body as raw bytes (no JSON decode). Required for binary endpoints like /content. Currently honoured by the `sp` provider. | — |
| `--out` | `` | Write the response to this file path instead of stdout. Implies --raw. | — |

**Examples:**

```bash
its entra graph get "/users?$top=5"

its entra graph get "/identityGovernance" --beta
```

#### `its entra graph post <path>`

Raw Graph POST — pass any /v1.0 or /beta path (use --beta for beta).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--body` | `` | Request body — inline JSON string or @file.json to read from disk | — |
| `--beta` | `` | Use /beta instead of /v1.0 | — |
| `--header` | `` | Extra headers as comma-separated K=V pairs (e.g. Prefer=return=minimal) | — |

**Examples:**

```bash
its entra graph post "/users/$count" --body @./payload.json

# Pipe-friendly output — use with jq / scripts.
its entra graph post "/users/$count" --body @./payload.json --json
```

#### `its entra graph patch <path>`

Raw Graph PATCH — pass any /v1.0 or /beta path (use --beta for beta).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--body` | `` | Request body — inline JSON string or @file.json to read from disk | — |
| `--beta` | `` | Use /beta instead of /v1.0 | — |
| `--header` | `` | Extra headers as comma-separated K=V pairs (e.g. Prefer=return=minimal) | — |

**Examples:**

```bash
its entra graph patch "/users/<id>" --body '{"jobTitle":"Lead"}'

# Pipe-friendly output — use with jq / scripts.
its entra graph patch "/users/<id>" --body  --json
```

#### `its entra graph put <path>`

Raw Graph PUT — pass any /v1.0 or /beta path (use --beta for beta).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--body` | `` | Request body — inline JSON string or @file.json to read from disk | — |
| `--beta` | `` | Use /beta instead of /v1.0 | — |
| `--header` | `` | Extra headers as comma-separated K=V pairs (e.g. Prefer=return=minimal) | — |

**Examples:**

```bash
its entra graph put "/users/<id>/photo/$value" --body @./photo.jpg

# Pipe-friendly output — use with jq / scripts.
its entra graph put "/users/<id>/photo/$value" --body @./photo.jpg --json
```

#### `its entra graph delete <path>`

Raw Graph DELETE — pass any /v1.0 or /beta path (use --beta for beta).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--beta` | `` | Use /beta instead of /v1.0 | — |
| `--header` | `` | Extra headers as comma-separated K=V pairs (e.g. Prefer=return=minimal) | — |

**Examples:**

```bash
its entra graph delete "/users/<id>" --confirm
```

---
