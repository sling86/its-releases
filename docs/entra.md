# Entra ID (`entra`)

Microsoft Entra ID (Azure AD) identity management — users, groups, licences, roles, sign-ins, security, directory, onboarding.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

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
- [devices](#devices)
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
| `src/providers/entra/apps-audit.ts` | apps audit |
| `src/providers/entra/apps-manifest.ts` | apps manifest |
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
| `its entra users set-password <id>` | Set a user's password (requires --confirm). The calling principal needs Password Administrator / User Administrator — app-only User.ReadWrite.All returns 403. Add --force-change to require a reset at next sign-in. |
| `its entra users delete <id>` | Soft-delete a user (requires --confirm). Recoverable for 30 days via directory/deletedItems. |
| `its entra users transfer <id>` | Internal transfer (Company A→B): rename UPN + swap primary SMTP (optionally keep old as alias) + set company / department / manager in one pass. Requires --confirm. Cloud-only accounts (proxyAddresses must be Entra-writable). |
| `its entra users reinstate <id>` | Reverse a recent offboarding (requires --confirm): re-enable the account, optionally reset the password, and replay the directory audit log from the last N minutes to re-add removed group memberships. Licence removals are reported (re-assign manually). |

#### `its entra users`

List users from the Entra ID directory. Defaults to first 50 — use --all to page through every user, --filter for OData expressions, or --domain for a quick UPN-suffix match.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Number of results (max 100) | 50 |
| `--all` | `` | Fetch all results (overrides --top) | — |
| `--filter` | `` | OData filter expression | — |
| `--search` | `` | Fuzzy substring match across displayName, mail, and UPN (Graph $search) | — |
| `--enabled` | `` | Only show enabled accounts | — |
| `--domain` | `` | Filter by UPN domain (e.g. example.com) — server-side endsWith match | — |

**Examples:**

```bash
# Default — first 50 enabled accounts
its entra users

# Page through every user
its entra users --all

its entra users --enabled

its entra users --domain example.com

# Fuzzy match on name, mail, UPN
its entra users --search smith

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
its entra users update jane.smith@example.com --jobTitle "Finance Manager" --department Finance

its entra users update jane.smith@example.com --manager john.doe@example.com

its entra users update jane.smith@example.com --set extensionAttribute13=Leaver

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
```

#### `its entra users groups <id>`

All groups the user is a direct member of — security, M365, distribution, mail-enabled. Doesn't expand transitive (parent-of-parent) membership; use `users transitive-groups` for that.

**Examples:**

```bash
its entra users groups jane.smith@example.com
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
```

#### `its entra users licences <id>`

Per-SKU licence assignments with service-plan level detail — which SKUs the user has, plus which features are enabled or disabled inside each.

**Examples:**

```bash
its entra users licences jane.smith@example.com
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
its entra users invite --email contractor@partner.com --name "Alex Cole"

# Creates the guest and returns the redemption URL for you to send yourself
its entra users invite --email contractor@partner.com --name "Alex Cole" --no-email

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
its entra users create --displayName "Jane Smith" --upn jane.smith@example.com --department Finance

its entra users create --upn jane.smith@example.com --displayName "Jane Smith" --password "TempP@ss!"
```

#### `its entra users enable <id>`

Enable a user account. Switch to active state. Idempotent.

**Examples:**

```bash
its entra users enable jane.smith@example.com
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
its entra users bootstrap-admin jane.smith@example.com --dry-run

its entra users bootstrap-admin jane.smith@example.com --ttl 60 --one-time

# Creates a TAP and excludes user from blocking CA policies
its entra users bootstrap-admin newadmin@example.com
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

# Kills active refresh tokens too — without this a signed-in session survives the disable
its entra users disable jane.smith@example.com --revoke-sessions --confirm
```

#### `its entra users revoke-sessions <id>`

Revoke all sign-in sessions for a user (POST /users/{id}/revokeSignInSessions). Forces re-auth on all clients without disabling the account.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the revoke | — |

**Examples:**

```bash
# First move on a suspected compromise — signs every client out without disabling the account
its entra users revoke-sessions jane.smith@example.com --confirm

# Force re-auth on every device
its entra users revoke-sessions jane.smith@example.com
```

#### `its entra users set-password <id>`

Set a user's password (requires --confirm). The calling principal needs Password Administrator / User Administrator — app-only User.ReadWrite.All returns 403. Add --force-change to require a reset at next sign-in.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--password` | `` | New password | — |
| `--force-change` | `` | Force a password change at next sign-in | — |
| `--confirm` | `` | Confirm the password set | — |

**Examples:**

```bash
# Caller needs Password Administrator or higher
its entra users set-password jane.smith@example.com --password '<generated>' --force-change --confirm
```

#### `its entra users delete <id>`

Soft-delete a user (requires --confirm). Recoverable for 30 days via directory/deletedItems.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the deletion | — |

**Examples:**

```bash
# Without --confirm, reports the target and stops
its entra users delete jane.smith@example.com

# Soft delete — recoverable for 30 days from directory/deletedItems
its entra users delete jane.smith@example.com --confirm
```

#### `its entra users transfer <id>`

Internal transfer (Company A→B): rename UPN + swap primary SMTP (optionally keep old as alias) + set company / department / manager in one pass. Requires --confirm. Cloud-only accounts (proxyAddresses must be Entra-writable).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--new-upn` | `` | New userPrincipalName (also becomes primary SMTP) | — |
| `--new-company` | `` | New companyName | — |
| `--new-department` | `` | New department | — |
| `--new-manager` | `` | New manager (UPN or GUID) | — |
| `--keep-old-smtp-as-alias` | `` | Keep the old primary SMTP as a secondary alias | — |
| `--confirm` | `` | Confirm the transfer | — |

**Examples:**

```bash
its entra users transfer jane.smith@example.com --new-upn jane.smith@newco.example.com --new-company NewCo --keep-old-smtp-as-alias --confirm
```

#### `its entra users reinstate <id>`

Reverse a recent offboarding (requires --confirm): re-enable the account, optionally reset the password, and replay the directory audit log from the last N minutes to re-add removed group memberships. Licence removals are reported (re-assign manually).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--from-audit-minutes` | `` | Audit window to replay group re-adds from (default 60) | 60 |
| `--password` | `` | Also set a new password | — |
| `--force-change` | `` | Force password change at next sign-in (with --password) | — |
| `--confirm` | `` | Confirm the reinstate | — |

**Examples:**

```bash
# Replays the recent offboarding in reverse — re-enables and restores group membership
its entra users reinstate jane.smith@example.com --from-audit-minutes 120 --confirm
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
| `its entra groups edit-rule <group_id>` | Edit a dynamic group's membershipRule. --add-upn appends an OR exception (grants a user who doesn't match the rule); --remove-upn strips one; --set-rule replaces the whole rule. add/remove never drop existing members. --confirm required; without it, prints the current→new diff (ctxc 1052). |
| `its entra groups audit-rules [group_id]` | Scan dynamic groups' membershipRules for dead user exceptions — hardcoded userPrincipalName/objectId/mail clauses whose account no longer exists (missing) or is disabled (a leaver still pinned). Pass a group ID to scan one; default scans all dynamic groups. --all also lists the OK refs. |

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
```

#### `its entra groups get <group_id>`

Get group details. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its entra groups get <group-id>
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
its entra groups create --displayName "IT Admins" --description "IT team"

its entra groups create --displayName "All Staff" --mail

its entra groups create --displayName "Marketing"
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
# Refuses dynamic-membership groups — Graph accepts the call but the member is silently dropped
its entra groups add-member 8f1c2d3e-4a5b-6c7d-8e9f-0a1b2c3d4e5f --user jane.smith@example.com

# Guards against dynamic groups, disabled users, and namesake collisions
its entra groups add-member <group-id> --user jane.smith@example.com
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
its entra groups remove-member 8f1c2d3e-4a5b-6c7d-8e9f-0a1b2c3d4e5f --user jane.smith@example.com --confirm

its entra groups remove-member <group-id> --user jane.smith@example.com --confirm
```

#### `its entra groups edit-rule <group_id>`

Edit a dynamic group's membershipRule. --add-upn appends an OR exception (grants a user who doesn't match the rule); --remove-upn strips one; --set-rule replaces the whole rule. add/remove never drop existing members. --confirm required; without it, prints the current→new diff (ctxc 1052).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--add-upn` | `` | Append `or (user.userPrincipalName -eq "<upn>")` to the rule | — |
| `--remove-upn` | `` | Strip the OR exception for this UPN from the rule | — |
| `--set-rule` | `` | Replace the ENTIRE membershipRule (re-evaluates the whole group) | — |
| `--confirm` | `` | Apply the change | — |
| `--force` | `` | Skip the --add-upn existence check (allow a UPN that doesn't resolve) | — |

**Examples:**

```bash
# Appends an OR clause so one user joins a dynamic group without matching the rule
its entra groups edit-rule 8f1c2d3e-... --add-upn jane.smith@example.com --confirm

its entra groups edit-rule 8f1c2d3e-... --remove-upn jane.smith@example.com --confirm
```

#### `its entra groups audit-rules [group_id]`

Scan dynamic groups' membershipRules for dead user exceptions — hardcoded userPrincipalName/objectId/mail clauses whose account no longer exists (missing) or is disabled (a leaver still pinned). Pass a group ID to scan one; default scans all dynamic groups. --all also lists the OK refs.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--all` | `` | Include refs that resolve OK, not just dead/disabled | — |

```bash
its entra groups audit-rules [group_id]
```

---

### licences

> Source: `src/providers/entra/commands/licences.ts`

| Command | Description |
|---------|-------------|
| `its entra licences` | List all subscribed SKUs/licences. Available column gets a ⚠ marker when fewer than 10% of seats remain (or none) — useful for catching exhaustion before the next assign request fails. |
| `its entra licences assign <user_id>` | Assign one or more licences to a user. Pass --sku as a comma-separated list to assign several atomically in a single Graph call (parallel single-assigns trip 409 Directory_ConcurrencyViolation). Idempotent. |
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

# Re-runs every 10s — handy for dashboards or incident response.
its entra licences --watch
```

#### `its entra licences assign <user_id>`

Assign one or more licences to a user. Pass --sku as a comma-separated list to assign several atomically in a single Graph call (parallel single-assigns trip 409 Directory_ConcurrencyViolation). Idempotent.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--sku` | `` | SKU ID (GUID), or comma-separated list of GUIDs | — |

**Examples:**

```bash
its entra licences assign jane@x.com --sku <guid>

its entra licences assign jane@x.com --sku <guid1>,<guid2>,<guid3>

its entra licences assign jane.smith@example.com --sku SPB

# assign has no --location flag — set usageLocation first via its entra users update <id> --usage GB. Comma-separated --sku assigns several SKUs in one atomic Graph call.
its entra licences assign jane.smith@example.com --sku SPB,ENTERPRISEPACK
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
its entra licences remove jane.smith@example.com --sku ENTERPRISEPACK --confirm

# Shows which SKUs the user currently holds
its entra users licences jane.smith@example.com

its entra licences remove jane.smith@example.com --sku SPB --confirm

# licences remove has no --all flag (only a single --sku at a time) — onboarding convert-mailbox strips every assigned SKU (and disables the account) in one call, used in offboarding.
its entra onboarding convert-mailbox jane.smith@example.com --confirm
```

#### `its entra licences users <sku_id>`

List users assigned a specific licence SKU. List by resource membership; use --json for the raw shape.

**Examples:**

```bash
its entra licences users SPB
```

#### `its entra licences unlicensed`

Find enabled users with no licence assignments. Read-only — useful for billing audits.

**Examples:**

```bash
# Active users without any licence — billing leak hunt
its entra licences unlicensed
```

#### `its entra licences audit`

Licence usage report with availability and over-allocation warnings.

**Examples:**

```bash
# Per-SKU usage with under/over-allocation flags
its entra licences audit
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

# Re-runs every 10s — handy for dashboards or incident response.
its entra roles --watch
```

#### `its entra roles members <role_id>`

List members of a directory role. Returns direct members; nested groups aren't expanded.

**Examples:**

```bash
its entra roles members <role-id>
```

#### `its entra roles assign <user_id>`

Assign a directory role to a user. Idempotent — assigning twice is a no-op.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--role` | `` | Role definition ID | — |

**Examples:**

```bash
its entra roles assign jane.smith@example.com --role "User Administrator"

its entra roles assign jane.smith@example.com --role <role-def-id>
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
its entra roles remove jane.smith@example.com --role "User Administrator" --confirm

its entra roles remove jane.smith@example.com --role <role-def-id> --confirm
```

#### `its entra roles assignments`

List all role assignments with role names. Returns the assignment record, not the principal — pair with `users get` for the readable name.

**Examples:**

```bash
# Complete map of role → principal
its entra roles assignments
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

# Re-runs every 10s — handy for dashboards or incident response.
its entra signin summary jane.smith@example.com --watch
```

#### `its entra signin explain <id>`

Decode a single sign-in event: error code, failureReason, and the CA policies that fired (appliedConditionalAccessPolicies). Pass the sign-in ID from `signin list`.

**Examples:**

```bash
# Show error code + which CA policies fired
its entra signin explain <signin-id>
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
# Does not work for B2B guests
its entra tap create jane.smith@example.com --ttl 60 --one-time

its entra tap create jane.smith@example.com

its entra tap create jane.smith@example.com --ttl 8h
```

#### `its entra tap <user_id>`

List active Temporary Access Passes for a user (codes redacted).

**Examples:**

```bash
# Temporary access passes outstanding
its entra tap

# Re-runs every 10s — handy for dashboards or incident response.
its entra tap --watch
```

#### `its entra tap revoke <user_id> <method_id>`

Revoke a Temporary Access Pass. Reverse an assignment. --confirm where required.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm revocation | — |

**Examples:**

```bash
its entra tap revoke jane.smith@example.com 3f2b1c94-... --confirm

its entra tap list jane.smith@example.com

its entra tap revoke jane.smith@example.com <method-id>
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

# Re-runs every 10s — handy for dashboards or incident response.
its entra ca --watch
```

#### `its entra ca enabled`

List only enabled CA policies. Filters by `state eq 'enabled'`.

**Examples:**

```bash
its entra ca enabled
```

#### `its entra ca get <id_or_name>`

Show full CA policy JSON by id or displayName. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its entra ca get <policy-id>
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
its entra ca patch "Block legacy auth" --state enabledForReportingButNotEnforced --dry-run

its entra ca patch "Block legacy auth" --state enabled

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
its entra ca exclude-guests "Require compliant device" --dry-run

its entra ca exclude-guests <policy-id>
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
its entra ca create --body @./policy.json --dry-run

its entra ca create @./policy.json
```

#### `its entra ca delete <id_or_name>`

Delete a Conditional Access policy by id or displayName. Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--dry-run` | `` | Print the DELETE request without sending | — |
| `--confirm` | `` | Confirm deletion | — |

**Examples:**

```bash
# Shows the policy that would go, changes nothing
its entra ca delete "Block legacy auth" --dry-run

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
# Atomic — no need to rebuild the whole users object
its entra ca exclude-user "Require MFA" jane.smith@example.com

# Use sparingly — re-include the user once the trip-cause is cleared.
its entra ca exclude-user <policy-id> jane.smith@example.com

# Policy can be addressed by exact displayName instead of id.
its entra ca exclude-user "Require MFA for admins" jane.smith@example.com
```

#### `its entra ca unexclude-user <id_or_name> <user>`

Remove a user from a CA policy's excludeUsers list. Reverse of `exclude-user`. Idempotent.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--dry-run` | `` | Print the PATCH request without sending | — |

**Examples:**

```bash
its entra ca unexclude-user "Require MFA" jane.smith@example.com

its entra ca unexclude-user <policy-id> jane.smith@example.com
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
```

#### `its entra ca named-locations`

List trusted/named locations referenced by CA policies.

**Examples:**

```bash
its entra ca named-locations
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
```

#### `its entra authmethods get <method>`

Get a single authentication method configuration by id (TemporaryAccessPass, Fido2, Sms, MicrosoftAuthenticator, Email, Voice, X509Certificate, etc.).

**Examples:**

```bash
its entra authmethods get TemporaryAccessPass
```

#### `its entra authmethods enable <method>`

Enable a method tenant-wide (sets state=enabled on the method configuration). Pass --target <groupId> or --target all-users to scope.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--target` | `` | Optional include target — group ID or 'all_users'. Adds a default include target if specified. | — |
| `--dry-run` | `` | Print the PATCH request without sending | — |
| `--confirm` | `` | Confirm this tenant-wide policy change | — |

**Examples:**

```bash
its entra authmethods enable fido2 --confirm

its entra authmethods enable fido2 --target 8f1c2d3e-... --confirm

its entra authmethods enable Fido2
```

#### `its entra authmethods disable <method>`

Disable a method tenant-wide (state=disabled). Switch to inactive state. Idempotent.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--dry-run` | `` | Print the PATCH request without sending | — |
| `--confirm` | `` | Confirm this tenant-wide policy change | — |

**Examples:**

```bash
its entra authmethods disable sms --dry-run

its entra authmethods disable Sms
```

#### `its entra authmethods patch <method>`

Patch a method configuration with a custom body (inline or @path/to/file.json).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--body` | `` | Inline JSON or @path/to/body.json — the PATCH body for the method config | — |
| `--dry-run` | `` | Print the PATCH request without sending | — |
| `--confirm` | `` | Confirm this tenant-wide policy change | — |

**Examples:**

```bash
its entra authmethods patch fido2 --body @./fido2.json --confirm

its entra authmethods patch TemporaryAccessPass @./tap-policy.json
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
# Union with the existing grant — never replaces what is already consented
its entra consent add-scope 8f1c2d3e-4a5b-6c7d-8e9f-0a1b2c3d4e5f User.Read,Mail.Read --dry-run

# Unions with existing scopes, never replaces
its entra consent add-scope <app-id> Mail.Read
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
its entra consent remove-scope 8f1c2d3e-... Mail.Read --dry-run

its entra consent remove-scope <app-id> Mail.Read
```

#### `its entra consent app-roles <client_app_id>`

List application-permission grants (appRoleAssignments) for a client app.

**Examples:**

```bash
its entra consent app-roles <app-id>
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
its entra consent app-role-grant 8f1c2d3e-... User.Read.All --dry-run

its entra consent app-role-grant <app-id> User.Read.All
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
its entra consent app-role-revoke 8f1c2d3e-... 9a8b7c6d-... --confirm

its entra consent app-role-revoke <app-id> <assignment-id>
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
```

#### `its entra xtenant trust-mfa <on_off>`

Toggle inboundTrust.isMfaAccepted on the default policy.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--dry-run` | `` | Print the PATCH without sending | — |

**Examples:**

```bash
its entra xtenant trust-mfa on --dry-run

its entra xtenant trust-mfa on
```

#### `its entra xtenant trust-device <on_off>`

Toggle inboundTrust.isCompliantDeviceAccepted on the default policy.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--dry-run` | `` | Print the PATCH without sending | — |

**Examples:**

```bash
its entra xtenant trust-device off --dry-run

its entra xtenant trust-device on
```

#### `its entra xtenant partners`

List per-partner cross-tenant access overrides. Returns per-partner overrides — use `partner-set` to mutate.

**Examples:**

```bash
its entra xtenant partners
```

#### `its entra xtenant partner-add <tenant_id>`

Start a per-partner cross-tenant access config. Bootstraps a fresh per-partner override record.

**Examples:**

```bash
# Creates a cross-tenant partner configuration for the given tenant ID
its entra xtenant partner-add 8f1c2d3e-4a5b-6c7d-8e9f-0a1b2c3d4e5f

its entra xtenant list

its entra xtenant partner-add <partner-tenant-id>
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
its entra xtenant partner-set 8f1c2d3e-4a5b-6c7d-8e9f-0a1b2c3d4e5f --trust-mfa true --dry-run

its entra xtenant partner-set <partner-tenant-id> --trust-mfa on
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
its entra audit --filter "activityDateTime ge 2024-01-01T00:00:00Z"

its entra audit --target <user-id>

# Re-runs every 10s — handy for dashboards or incident response.
its entra audit --filter "activityDateTime ge 2024-01-01T00:00:00Z" --watch
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
```

#### `its entra security mfa <user_id>`

Check MFA readiness for a user. Returns user's strong-auth methods plus a readiness flag.

**Examples:**

```bash
# Users without strong MFA
its entra security mfa
```

#### `its entra security admin-mfa`

Audit all admin users for MFA readiness. Audit pass — flags any admin without strong MFA.

**Examples:**

```bash
# Privileged accounts without MFA
its entra security admin-mfa
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
```

#### `its entra directory domains`

List verified domains. Returns verified + provisioning domains.

**Examples:**

```bash
its entra directory domains
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
```

#### `its entra directory summary`

Aggregate users by company, department, location. Quick one-screen view — designed for dashboards / `--watch`.

**Examples:**

```bash
its entra directory summary

# Re-runs every 10s — handy for dashboards or incident response.
its entra directory summary --watch
```

#### `its entra directory app-usage`

Licence and app usage summary. Aggregated app + licence usage across the tenant.

**Examples:**

```bash
its entra directory app-usage
```

---

### devices

> Source: `src/providers/entra/commands/devices.ts`

| Command | Description |
|---------|-------------|
| `its entra devices` | List Entra-registered and Entra-joined devices. Wider than `its intune devices`, which only sees Intune-managed ones — filter with --filter isManaged=false to find registered-but-unenrolled devices. Needs Device.Read.All. |
| `its entra devices get <device>` | Get one device by directory object id, deviceId, or exact display name. Needs Device.Read.All. |

#### `its entra devices`

List Entra-registered and Entra-joined devices. Wider than `its intune devices`, which only sees Intune-managed ones — filter with --filter isManaged=false to find registered-but-unenrolled devices. Needs Device.Read.All.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--search` | `` | Match on display name | — |
| `--top` | `` | Maximum devices to return | 100 |

**Examples:**

```bash
its entra devices list

its entra devices list --search THF-UD

its entra devices list --filter isManaged=false
```

#### `its entra devices get <device>`

Get one device by directory object id, deviceId, or exact display name. Needs Device.Read.All.

**Examples:**

```bash
its entra devices get THF-UD-MP27XZ31

its entra devices get 12345678-90ab-cdef-1234-567890abcdef
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
# Already-shared groups are skipped, so it is safe to re-run
its entra onboarding copy-groups --source john.doe@example.com --target jane.smith@example.com

its entra onboarding copy-groups --source peer@example.com --target jane.smith@example.com
```

#### `its entra onboarding convert-mailbox <user_id>`

Convert user mailbox to shared (disable + remove licences). Requires --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the conversion | — |

**Examples:**

```bash
# Disables the account and strips licences as part of the conversion
its entra onboarding convert-mailbox jane.smith@example.com --confirm
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
# Disables the account, revokes licences and removes group membership in one pass
its entra offboarding run jane.smith@example.com --confirm
```

---

### break-glass

> Source: `src/providers/entra/commands/break-glass.ts`

| Command | Description |
|---------|-------------|
| `its entra break-glass audit` | Audit your two break-glass accounts (BG01 password + BG02 FIDO2) — GA role, CA exclusions, sign-in hygiene, password age, FIDO2. Non-zero exit on any gap. Set the UPNs via ENTRA_BG01_UPN / ENTRA_BG02_UPN (or --bg01/--bg02). |

#### `its entra break-glass audit`

Audit your two break-glass accounts (BG01 password + BG02 FIDO2) — GA role, CA exclusions, sign-in hygiene, password age, FIDO2. Non-zero exit on any gap. Set the UPNs via ENTRA_BG01_UPN / ENTRA_BG02_UPN (or --bg01/--bg02).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--bg01` | `` | Override BG01 UPN (defaults to ENTRA_BG01_UPN env) | — |
| `--bg02` | `` | Override BG02 UPN (defaults to ENTRA_BG02_UPN env) | — |

**Examples:**

```bash
# Verify emergency-access accounts are healthy + excluded from CA
its entra break-glass audit
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
| `its entra apps secrets` | Credential expiry dashboard — every client secret and certificate across all app registrations, with days-left and a status of EXPIRED, expiring (within --warn-days), long-lived (>2y), or ok. |
| `its entra apps rotate <app>` | Mint a new client secret on an app registration and store it straight into the OS keychain (--env-key) and/or Bitwarden (--bw). Additive — existing credentials are untouched unless --prune-expired. Output carries a fingerprint, never the secret (unless --include-secrets). Fails closed: if every storage target fails (locked vault, unavailable keychain) the new credential is rolled back and nothing is printed — pass --include-secrets to accept the secret on stderr instead. |
| `its entra apps plan` | Diff the apps manifest against live tenant state — read-only. Shows what apply would create or grant (+), change (~), plus warnings (!) and manual steps (✋). Declaration (requiredResourceAccess) and grant (appRoleAssignments / delegated consent) are diffed separately. Manifest resolution: --manifest > ITS_APPS_MANIFEST > manifest/entra-apps.jsonc (cwd-relative). |
| `its entra apps apply` | Execute the manifest plan against the tenant — ADDITIVE ONLY, nothing is ever removed. Per app: create app → create SP → refetch → add owners → one coalesced requiredResourceAccess PATCH → redirect/public-client PATCHes → app-role grants → delegated scope-union grants. Needs Graph permissions Application.ReadWrite.All + AppRoleAssignment.ReadWrite.All + DelegatedPermissionGrant.ReadWrite.All — easiest run as an admin via --auth az, or a delegated admin sign-in (its auth login). Use --dry-run for a write-free preview. |
| `its entra apps permissions <app>` | What an app ACTUALLY holds — declared (requiredResourceAccess) and granted (appRoleAssignments / delegated consent) side by side, with permission GUIDs resolved to names. The two drift independently and in both directions, so neither alone answers "can this app do X?". |
| `its entra apps export <appIds>` | Reverse-map live app registrations into manifest fragments — declared permission GUIDs resolve back to value names via each resource SP (unresolved ids stay as raw GUIDs with a note), owners become UPNs, redirect URIs keep their web/publicClient buckets. Paste the output into the manifest's "apps" array. Never dumps the tenant — pass explicit appIds. |
| `its entra apps audit` | Security-posture audit across all app registrations — god-apps (app-role grant counts near the ~40-role token roles-claim overflow that broke Intune), expired and long-lived secrets, credential-less and secret-only apps, ownerless apps, stale sign-in activity (beta report, degrades gracefully), and apps missing from the manifest (when one is readable). Unused-grants is out of scope — no per-permission usage signal exists without premium logs. |

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

**Examples:**

```bash
its entra apps register "its CLI" --redirect-uri http://localhost --months 12
```

#### `its entra apps add-password <app>`

Rotate / add a client secret on an existing app registration. The plaintext secretText is only returned once.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | Display name for the secret | its-cli generated |
| `--months` | `` | Lifetime in months | 12 |

**Examples:**

```bash
# The plaintext secret is returned once and never again — capture it now
its entra apps add-password "its CLI" --name 2026-rotation --months 12
```

#### `its entra apps secrets`

Credential expiry dashboard — every client secret and certificate across all app registrations, with days-left and a status of EXPIRED, expiring (within --warn-days), long-lived (>2y), or ok.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--warn-days` | `` | Flag credentials expiring within this many days | 30 |
| `--expiring` | `` | Only show EXPIRED and expiring credentials | — |

```bash
its entra apps secrets
```

#### `its entra apps rotate <app>`

Mint a new client secret on an app registration and store it straight into the OS keychain (--env-key) and/or Bitwarden (--bw). Additive — existing credentials are untouched unless --prune-expired. Output carries a fingerprint, never the secret (unless --include-secrets). Fails closed: if every storage target fails (locked vault, unavailable keychain) the new credential is rolled back and nothing is printed — pass --include-secrets to accept the secret on stderr instead.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | Display name for the new secret (default its-rotated-YYYY-MM) | — |
| `--months` | `` | Lifetime in months | 12 |
| `--env-file` | `` | Fall back to ~/.its/.env (plaintext, 0600) when the OS keychain and Bitwarden are both unavailable. Opt-in: without it, rotate fails closed rather than silently downgrading where the secret is stored. | — |
| `--env-key` | `` | Store the secret in the OS keychain under this env var name (must be a known secret key, e.g. CLIENT_SECRET) | — |
| `--bw` | `` | Upsert a "<app> - <secret name>" login item in the Bitwarden Infrastructure folder | — |
| `--prune-expired` | `` | Also remove credentials that had already expired before this rotation (prompts per credential unless --confirm) | — |
| `--confirm` | `` | Skip the interactive confirmation | — |

**Examples:**

```bash
its entra apps rotate "its CLI" --env-key CLIENT_SECRET --months 12 --confirm

its entra apps rotate "its CLI" --bw --prune-expired --confirm
```

#### `its entra apps plan`

Diff the apps manifest against live tenant state — read-only. Shows what apply would create or grant (+), change (~), plus warnings (!) and manual steps (✋). Declaration (requiredResourceAccess) and grant (appRoleAssignments / delegated consent) are diffed separately. Manifest resolution: --manifest > ITS_APPS_MANIFEST > manifest/entra-apps.jsonc (cwd-relative).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--manifest` | `` | Path to the JSONC apps manifest | — |
| `--app` | `` | Only plan one manifest entry (by name or appId) | — |
| `--exit-code` | `` | Exit 1 when there is actionable drift (CI guard) | — |
| `--changes-only` | `` | Hide manual-step and report-only rows — show only actionable changes and warnings | — |

**Examples:**

```bash
its entra apps plan

# Non-zero exit when the tenant has drifted from the manifest
its entra apps plan --exit-code --changes-only
```

#### `its entra apps apply`

Execute the manifest plan against the tenant — ADDITIVE ONLY, nothing is ever removed. Per app: create app → create SP → refetch → add owners → one coalesced requiredResourceAccess PATCH → redirect/public-client PATCHes → app-role grants → delegated scope-union grants. Needs Graph permissions Application.ReadWrite.All + AppRoleAssignment.ReadWrite.All + DelegatedPermissionGrant.ReadWrite.All — easiest run as an admin via --auth az, or a delegated admin sign-in (its auth login). Use --dry-run for a write-free preview.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--manifest` | `` | Path to the JSONC apps manifest | — |
| `--app` | `` | Only apply one manifest entry (by name or appId) | — |
| `--confirm` | `` | Skip the interactive confirmation | — |

**Examples:**

```bash
# Additive only — extras warn, nothing is ever removed
its entra apps apply --confirm
```

#### `its entra apps permissions <app>`

What an app ACTUALLY holds — declared (requiredResourceAccess) and granted (appRoleAssignments / delegated consent) side by side, with permission GUIDs resolved to names. The two drift independently and in both directions, so neither alone answers "can this app do X?".

**Examples:**

```bash
its entra apps permissions c15df88c-3070-4606-a95b-27a871b6a7da

its entra apps permissions <appId> --filter status~^(granted|declared)-only$
```

#### `its entra apps export <appIds>`

Reverse-map live app registrations into manifest fragments — declared permission GUIDs resolve back to value names via each resource SP (unresolved ids stay as raw GUIDs with a note), owners become UPNs, redirect URIs keep their web/publicClient buckets. Paste the output into the manifest's "apps" array. Never dumps the tenant — pass explicit appIds.

```bash
its entra apps export <appIds>
```

#### `its entra apps audit`

Security-posture audit across all app registrations — god-apps (app-role grant counts near the ~40-role token roles-claim overflow that broke Intune), expired and long-lived secrets, credential-less and secret-only apps, ownerless apps, stale sign-in activity (beta report, degrades gracefully), and apps missing from the manifest (when one is readable). Unused-grants is out of scope — no per-permission usage signal exists without premium logs.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--god-threshold` | `` | App-role grant count above which an app is critical | 25 |
| `--stale-days` | `` | Days without a sign-in before an app counts as stale | 90 |
| `--max-secret-months` | `` | Flag secrets whose lifetime exceeds this many months | 24 |
| `--manifest` | `` | Path to the JSONC apps manifest for the not-in-manifest check | — |
| `--severity` | `` | Only show findings at or above this severity | — |

**Examples:**

```bash
its entra apps audit --severity high
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

**Examples:**

```bash
# Tries a TAP first, falls back where TAP is not permitted
its entra admin-bootstrap run jane.smith@example.com --ttl 60 --one-time --dry-run
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
its entra graph get /users

its entra graph get /administrativeUnits --beta

its entra graph get /users --header ConsistencyLevel=eventual

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
its entra graph post /users --body '{"displayName":"Jane Smith"}'

its entra graph post /administrativeUnits --beta

its entra graph post /users --header ConsistencyLevel=eventual

its entra graph post "/users/$count" --body @./payload.json
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
its entra graph patch /users --body '{"displayName":"Jane Smith"}'

its entra graph patch /administrativeUnits --beta

its entra graph patch /users --header ConsistencyLevel=eventual

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
its entra graph put /users --body '{"displayName":"Jane Smith"}'

its entra graph put /administrativeUnits --beta

its entra graph put /users --header ConsistencyLevel=eventual

its entra graph put "/users/<id>/photo/$value" --body @./photo.jpg
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
# No confirmation prompt — the path is sent exactly as given
its entra graph delete /groups/8f1c2d3e-4a5b-6c7d-8e9f-0a1b2c3d4e5f

its entra graph delete /administrativeUnits --beta

its entra graph delete /users --header ConsistencyLevel=eventual

# Raw Graph passthrough — this command has no --confirm guard (unlike typed delete commands such as users delete / ca delete). Double-check the path before running; there is no dry-run.
its entra graph delete "/users/<id>"
```

---
