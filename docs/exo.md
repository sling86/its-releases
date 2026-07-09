# Exchange Online (`exo`)

Exchange Online — distribution groups, shared mailboxes, permissions, mail flow rules, accepted domains.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [groups](#groups)
- [mailboxes](#mailboxes)
- [rules](#rules)
- [domains](#domains)
- [dkim](#dkim)
- [trace](#trace)
- [autoreply](#autoreply)
- [recipients](#recipients)
- [forwarding](#forwarding)

## Setup

```bash
its exo setup           # Interactive wizard
its exo setup --check   # Check configuration status
its exo setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `TENANT_ID` | Microsoft Entra tenant ID (same as Entra provider) |
| `EXO_APP_ID` | App registration with `Exchange.ManageAsApp` permission |
| `EXO_CERT_THUMBPRINT` | SHA1 thumbprint of the certificate in the cert store |
| `EXO_ORGANIZATION` | Exchange Online organisation (e.g. `contoso.onmicrosoft.com`) |

Requires **PowerShell 7** (`pwsh`) and the **ExchangeOnlineManagement** module. Auth uses a self-signed certificate uploaded to a dedicated Entra app registration with `Exchange.ManageAsApp` permission. Run `its exo setup` to check prerequisites and install the module.

On Windows, the certificate lives in the current user's cert store (`Cert:\CurrentUser\My`). On POSIX, PFX file auth is a future enhancement.

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/exo/client.ts` | API client methods |
| `src/providers/exo/types.ts` | TypeScript interfaces |
| `src/providers/exo/commands/` | Command definitions (split by resource) |
| `src/providers/exo/definition.ts` | definition |
| `src/providers/exo/resolve.ts` | resolve |

## Resources

### groups

> Source: `src/providers/exo/commands/groups.ts`

| Command | Description |
|---------|-------------|
| `its exo groups` | List all distribution groups. Surfaces the most common fields; pass --json for raw shape. |
| `its exo groups get <group>` | Get distribution group details. Pass the id (or any natural identifier) as the positional arg. |
| `its exo groups members <group>` | List members of a distribution group. Returns direct members; nested groups aren't expanded. |
| `its exo groups create <name>` | Create a new distribution group. Idempotent on duplicate names — use update/edit to mutate an existing record. |
| `its exo groups delete <group>` | Delete a distribution group. Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record. |
| `its exo groups add-member <group> <member>` | Add a member to a distribution group. Idempotent — already-a-member is a no-op. |
| `its exo groups remove-member <group> <member>` | Remove a member from a distribution group. Reverse of add-member. --confirm required. |

#### `its exo groups`

List all distribution groups. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its exo groups

# Pipe-friendly output — use with jq / scripts.
its exo groups --json

# Re-runs every 10s — handy for dashboards or incident response.
its exo groups --watch
```

#### `its exo groups get <group>`

Get distribution group details. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its exo groups get "All Staff"

# Pipe-friendly output — use with jq / scripts.
its exo groups get "All Staff" --json
```

#### `its exo groups members <group>`

List members of a distribution group. Returns direct members; nested groups aren't expanded.

**Examples:**

```bash
its exo groups members "All Staff"

# Pipe-friendly output — use with jq / scripts.
its exo groups members "All Staff" --json
```

#### `its exo groups create <name>`

Create a new distribution group. Idempotent on duplicate names — use update/edit to mutate an existing record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--alias` | `` | Email alias (before @domain) | — |
| `--managed-by` | `` | Owner email address | — |
| `--type` | `` | Group type | distribution |

**Examples:**

```bash
its exo groups create "Marketing Team" --type distribution --alias marketing

# Pipe-friendly output — use with jq / scripts.
its exo groups create "Marketing Team" --type distribution --alias marketing --json
```

#### `its exo groups delete <group>`

Delete a distribution group. Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Required to perform this destructive deletion | — |

**Examples:**

```bash
its exo groups delete sales@thf.co.uk --confirm

its exo groups delete "Old DL" --confirm
```

#### `its exo groups add-member <group> <member>`

Add a member to a distribution group. Idempotent — already-a-member is a no-op.

**Examples:**

```bash
its exo groups add-member "All Staff" jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its exo groups add-member "All Staff" jane.smith@example.com --json
```

#### `its exo groups remove-member <group> <member>`

Remove a member from a distribution group. Reverse of add-member. --confirm required.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Required to remove the member | — |

**Examples:**

```bash
its exo groups remove-member sales@thf.co.uk jo@thf.co.uk --confirm

its exo groups remove-member "All Staff" jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its exo groups remove-member "All Staff" jane.smith@example.com --json
```

---

### mailboxes

> Source: `src/providers/exo/commands/mailboxes.ts`

| Command | Description |
|---------|-------------|
| `its exo mailboxes` | List mailboxes (optionally filtered by type). Surfaces the most common fields; pass --json for raw shape. |
| `its exo mailboxes get <mailbox>` | Get mailbox details (accepts partial name, alias, or email) |
| `its exo mailboxes stats <mailbox>` | Get mailbox size and item statistics. Aggregated statistics — counts, totals, percentiles. |
| `its exo mailboxes create <name> <alias>` | Create a shared mailbox. Idempotent on duplicate names — use update/edit to mutate an existing record. |
| `its exo mailboxes permissions <mailbox>` | List non-inherited permissions on a mailbox. Returns access grants on the resource. |
| `its exo mailboxes add-permission <mailbox> <user>` | Grant mailbox access to a user. Grant a new permission. Idempotent. |
| `its exo mailboxes remove-permission <mailbox> <user>` | Remove mailbox access from a user. Revoke a permission. --confirm required. |
| `its exo mailboxes forwarding <mailbox>` | Show forwarding configuration for a mailbox. Audit pass — find mailboxes with forwarding configured. |
| `its exo mailboxes user-access <user>` | List all shared mailboxes a user has FullAccess to (scans all 400+ shared mailboxes, may take up to 5 minutes) |
| `its exo mailboxes set-forwarding <mailbox> <target>` | Configure mailbox forwarding. Set a mailbox forward. --confirm required. |
| `its exo mailboxes set-type <mailbox> <type>` | Convert a mailbox between user and shared (Set-Mailbox -Type). Common at offboarding — flip a leaver's mailbox to Shared. |
| `its exo mailboxes set-visibility <mailbox>` | Hide or show a mailbox in the global address list (GAL). Pass exactly one of --hide / --show. |

#### `its exo mailboxes`

List mailboxes (optionally filtered by type). Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `-t` | Mailbox type filter | — |

**Examples:**

```bash
its exo mailboxes

its exo mailboxes --type SharedMailbox

its exo mailboxes --type RoomMailbox
```

#### `its exo mailboxes get <mailbox>`

Get mailbox details (accepts partial name, alias, or email).

**Examples:**

```bash
its exo mailboxes get jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its exo mailboxes get jane.smith@example.com --json
```

#### `its exo mailboxes stats <mailbox>`

Get mailbox size and item statistics. Aggregated statistics — counts, totals, percentiles.

**Examples:**

```bash
its exo mailboxes stats jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its exo mailboxes stats jane.smith@example.com --json
```

#### `its exo mailboxes create <name> <alias>`

Create a shared mailbox. Idempotent on duplicate names — use update/edit to mutate an existing record.

**Examples:**

```bash
its exo mailboxes create "Support" support

# Pipe-friendly output — use with jq / scripts.
its exo mailboxes create "Support" support --json
```

#### `its exo mailboxes permissions <mailbox>`

List non-inherited permissions on a mailbox. Returns access grants on the resource.

**Examples:**

```bash
its exo mailboxes permissions shared@example.com

# Pipe-friendly output — use with jq / scripts.
its exo mailboxes permissions shared@example.com --json
```

#### `its exo mailboxes add-permission <mailbox> <user>`

Grant mailbox access to a user. Grant a new permission. Idempotent.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--rights` | `-r` | Access rights to grant | FullAccess |

**Examples:**

```bash
its exo mailboxes add-permission shared@example.com jane.smith@example.com --rights FullAccess

# Pipe-friendly output — use with jq / scripts.
its exo mailboxes add-permission shared@example.com jane.smith@example.com --rights FullAccess --json
```

#### `its exo mailboxes remove-permission <mailbox> <user>`

Remove mailbox access from a user. Revoke a permission. --confirm required.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--rights` | `-r` | Access rights to revoke | FullAccess |
| `--confirm` | `` | Required to revoke the permission | — |

**Examples:**

```bash
its exo mailboxes remove-permission shared@thf.co.uk jo@thf.co.uk --confirm

its exo mailboxes remove-permission shared@example.com jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its exo mailboxes remove-permission shared@example.com jane.smith@example.com --json
```

#### `its exo mailboxes forwarding <mailbox>`

Show forwarding configuration for a mailbox. Audit pass — find mailboxes with forwarding configured.

**Examples:**

```bash
its exo mailboxes forwarding jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its exo mailboxes forwarding jane.smith@example.com --json
```

#### `its exo mailboxes user-access <user>`

List all shared mailboxes a user has FullAccess to (scans all 400+ shared mailboxes, may take up to 5 minutes).

**Examples:**

```bash
# Every shared mailbox jane has rights on
its exo mailboxes user-access jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its exo mailboxes user-access jane.smith@example.com --json
```

#### `its exo mailboxes set-forwarding <mailbox> <target>`

Configure mailbox forwarding. Set a mailbox forward. --confirm required.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--keep-copy` | `` | Also deliver to the original mailbox | true |
| `--confirm` | `` | Required to set forwarding (mail-exfiltration risk) | — |

**Examples:**

```bash
its exo mailboxes set-forwarding jo@thf.co.uk manager@thf.co.uk --confirm

its exo mailboxes set-forwarding jane.smith@example.com external@vendor.com --confirm --keep-copy

# Pipe-friendly output — use with jq / scripts.
its exo mailboxes set-forwarding jane.smith@example.com external@vendor.com --confirm --keep-copy --json
```

#### `its exo mailboxes set-type <mailbox> <type>`

Convert a mailbox between user and shared (Set-Mailbox -Type). Common at offboarding — flip a leaver's mailbox to Shared.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Required to change the mailbox type | — |

**Examples:**

```bash
its exo mailboxes set-type jo@thf.co.uk SharedMailbox --confirm
```

#### `its exo mailboxes set-visibility <mailbox>`

Hide or show a mailbox in the global address list (GAL). Pass exactly one of --hide / --show.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--hide` | `` | Hide from the GAL | — |
| `--show` | `` | Show in the GAL | — |

```bash
its exo mailboxes set-visibility <mailbox>
```

---

### rules

> Source: `src/providers/exo/commands/rules.ts`

| Command | Description |
|---------|-------------|
| `its exo rules` | List mail flow (transport) rules. Surfaces the most common fields; pass --json for raw shape. |
| `its exo rules get <name>` | Get one transport rule's full condition/action shape (Get-TransportRule). Flags mail-exfiltration verbs (RedirectMessageTo, BlindCopyTo) if present. |
| `its exo rules audit` | Audit every transport rule for mail-exfiltration verbs (RedirectMessageTo, BlindCopyTo, AddToRecipients, CopyTo). A silent redirect/blind-copy to an external address is a classic data-exfiltration tripwire — review every flagged rule. |
| `its exo rules disable <name>` | Disable a transport rule (Disable-TransportRule) — remediate a flagged mail-exfiltration rule. Reversible with `rules enable`. Changes org mail flow — use --confirm. |
| `its exo rules enable <name>` | Enable a transport rule (Enable-TransportRule). Reversible with `rules disable`. Changes org mail flow — use --confirm. |

#### `its exo rules`

List mail flow (transport) rules. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its exo rules

# Transport rules (Exchange admin centre)
its exo rules

# Pipe-friendly output — use with jq / scripts.
its exo rules --json

# Re-runs every 10s — handy for dashboards or incident response.
its exo rules --watch
```

#### `its exo rules get <name>`

Get one transport rule's full condition/action shape (Get-TransportRule). Flags mail-exfiltration verbs (RedirectMessageTo, BlindCopyTo) if present.

**Examples:**

```bash
its exo rules get "External forward block"
```

#### `its exo rules audit`

Audit every transport rule for mail-exfiltration verbs (RedirectMessageTo, BlindCopyTo, AddToRecipients, CopyTo). A silent redirect/blind-copy to an external address is a classic data-exfiltration tripwire — review every flagged rule.

**Examples:**

```bash
its exo rules audit

its exo rules audit --json
```

#### `its exo rules disable <name>`

Disable a transport rule (Disable-TransportRule) — remediate a flagged mail-exfiltration rule. Reversible with `rules enable`. Changes org mail flow — use --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm disabling the rule | — |

**Examples:**

```bash
its exo rules disable "Evil redirect" --confirm
```

#### `its exo rules enable <name>`

Enable a transport rule (Enable-TransportRule). Reversible with `rules disable`. Changes org mail flow — use --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm enabling the rule | — |

**Examples:**

```bash
its exo rules enable "Block external forward" --confirm
```

---

### domains

> Source: `src/providers/exo/commands/domains.ts`

| Command | Description |
|---------|-------------|
| `its exo domains` | List accepted email domains. Surfaces the most common fields; pass --json for raw shape. |

#### `its exo domains`

List accepted email domains. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its exo domains

# Pipe-friendly output — use with jq / scripts.
its exo domains --json

# Re-runs every 10s — handy for dashboards or incident response.
its exo domains --watch
```

---

### dkim

> Source: `src/providers/exo/commands/dkim.ts`

| Command | Description |
|---------|-------------|
| `its exo dkim` | List DKIM signing config for every domain. Pass --json for raw shape. |
| `its exo dkim get <domain>` | Get DKIM signing config for a domain (Enabled, Status, selector key sizes, RotateOnDate). |
| `its exo dkim rotate <domain>` | Rotate the DKIM signing key for a domain (Rotate-DkimSigningConfig). Mutation — requires --confirm. |
| `its exo dkim enable <domain>` | Enable DKIM signing for a domain (Set-DkimSigningConfig). Mutation — requires --confirm. |
| `its exo dkim disable <domain>` | Disable DKIM signing for a domain (Set-DkimSigningConfig). Mutation — requires --confirm. |

#### `its exo dkim`

List DKIM signing config for every domain. Pass --json for raw shape.

```bash
its exo dkim
```

#### `its exo dkim get <domain>`

Get DKIM signing config for a domain (Enabled, Status, selector key sizes, RotateOnDate).

```bash
its exo dkim get <domain>
```

#### `its exo dkim rotate <domain>`

Rotate the DKIM signing key for a domain (Rotate-DkimSigningConfig). Mutation — requires --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Required to perform the key rotation | — |
| `--key-size` | `` | Key size in bits | 2048 |

**Examples:**

```bash
its exo dkim rotate contractcandles.com --confirm
```

#### `its exo dkim enable <domain>`

Enable DKIM signing for a domain (Set-DkimSigningConfig). Mutation — requires --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Required to change the signing state | — |

**Examples:**

```bash
its exo dkim enable contractcandles.com --confirm
```

#### `its exo dkim disable <domain>`

Disable DKIM signing for a domain (Set-DkimSigningConfig). Mutation — requires --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Required to change the signing state | — |

**Examples:**

```bash
its exo dkim disable contractcandles.com --confirm
```

---

### trace

> Source: `src/providers/exo/commands/trace.ts`

| Command | Description |
|---------|-------------|
| `its exo trace` | Search message trace (last N days). Surfaces the most common fields; pass --json for raw shape. |
| `its exo trace detail <trace-id> <recipient>` | Get hop-by-hop detail for a traced message (Get-MessageTraceDetailV2). Surfaces the FAIL reason from each hop's Data blob; --json keeps the raw Data XML. |
| `its exo trace historical` | Submit a historical message-trace search (Start-HistoricalSearch) for the 11–90 day window that `trace list` (10-day cap) can't reach. Async — returns a JobId; poll with `trace historical-status <jobId>`. |
| `its exo trace historical-status <job-id>` | Poll a historical search job (Get-HistoricalSearch). FileUrl is populated once Status is Done. |

#### `its exo trace`

Search message trace (last N days). Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--sender` | `-s` | Sender email address | — |
| `--recipient` | `-r` | Recipient email address | — |
| `--subject` | `` | Subject contains (case-insensitive) | — |
| `--days` | `-d` | Number of days to search back | 2 |

**Examples:**

```bash
its exo trace --sender jane.smith@example.com --days 1

its exo trace --recipient external@vendor.com --days 7

# Re-runs every 10s — handy for dashboards or incident response.
its exo trace --sender jane.smith@example.com --days 1 --watch
```

#### `its exo trace detail <trace-id> <recipient>`

Get hop-by-hop detail for a traced message (Get-MessageTraceDetailV2). Surfaces the FAIL reason from each hop's Data blob; --json keeps the raw Data XML.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--days` | `-d` | Days back to search (max 10 per query) | 10 |

**Examples:**

```bash
its exo trace detail <msg-id>

# Pipe-friendly output — use with jq / scripts.
its exo trace detail <msg-id> --json
```

#### `its exo trace historical`

Submit a historical message-trace search (Start-HistoricalSearch) for the 11–90 day window that `trace list` (10-day cap) can't reach. Async — returns a JobId; poll with `trace historical-status <jobId>`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--sender` | `-s` | Sender email address | — |
| `--recipient` | `-r` | Recipient email address | — |
| `--days` | `-d` | Days back to search (max 90) | 30 |
| `--notify` | `` | Email address to notify when the CSV is ready (required by EXO) | — |

```bash
its exo trace historical
```

#### `its exo trace historical-status <job-id>`

Poll a historical search job (Get-HistoricalSearch). FileUrl is populated once Status is Done.

```bash
its exo trace historical-status <job-id>
```

---

### autoreply

> Source: `src/providers/exo/commands/autoreply.ts`

| Command | Description |
|---------|-------------|
| `its exo autoreply get <mailbox>` | Check out-of-office / auto-reply status for a mailbox. Pass the id (or any natural identifier) as the positional arg. |
| `its exo autoreply enable <mailbox>` | Enable out-of-office auto-reply. Switch to active state. Idempotent. |
| `its exo autoreply disable <mailbox>` | Disable out-of-office auto-reply. Switch to inactive state. Idempotent. |

#### `its exo autoreply get <mailbox>`

Check out-of-office / auto-reply status for a mailbox. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its exo autoreply get jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its exo autoreply get jane.smith@example.com --json
```

#### `its exo autoreply enable <mailbox>`

Enable out-of-office auto-reply. Switch to active state. Idempotent.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--internal` | `` | Internal message (HTML or plain text) | — |
| `--external` | `` | External message (HTML or plain text) | — |
| `--start` | `` | Start time for scheduled OOF (e.g. 2026-03-20T09:00) | — |
| `--end` | `` | End time for scheduled OOF (e.g. 2026-03-25T17:00) | — |

**Examples:**

```bash
its exo autoreply enable jane.smith@example.com --internal "Out of office until Monday" --external "On leave — contact support@example.com"

# Pipe-friendly output — use with jq / scripts.
its exo autoreply enable jane.smith@example.com --internal "Out of office until Monday" --external "On leave — contact support@example.com" --json
```

#### `its exo autoreply disable <mailbox>`

Disable out-of-office auto-reply. Switch to inactive state. Idempotent.

**Examples:**

```bash
its exo autoreply disable jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its exo autoreply disable jane.smith@example.com --json
```

---

### recipients

> Source: `src/providers/exo/commands/recipients.ts`

| Command | Description |
|---------|-------------|
| `its exo recipients search <query>` | Search all recipient types (users, groups, contacts, shared mailboxes) |
| `its exo recipients send-as <identity>` | List Send-As permissions on a mailbox or group. Returns SendAs delegates for a mailbox. |
| `its exo recipients add-send-as <identity> <trustee>` | Grant Send-As permission. Idempotent — already-granted delegates are skipped. |
| `its exo recipients remove-send-as <identity> <trustee>` | Revoke Send-As permission. Reverse of add-send-as. |

#### `its exo recipients search <query>`

Search all recipient types (users, groups, contacts, shared mailboxes).

**Examples:**

```bash
its exo recipients search "jane"

# Pipe-friendly output — use with jq / scripts.
its exo recipients search "jane" --json
```

#### `its exo recipients send-as <identity>`

List Send-As permissions on a mailbox or group. Returns SendAs delegates for a mailbox.

**Examples:**

```bash
its exo recipients send-as shared@example.com

# Pipe-friendly output — use with jq / scripts.
its exo recipients send-as shared@example.com --json
```

#### `its exo recipients add-send-as <identity> <trustee>`

Grant Send-As permission. Idempotent — already-granted delegates are skipped.

**Examples:**

```bash
its exo recipients add-send-as shared@example.com jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its exo recipients add-send-as shared@example.com jane.smith@example.com --json
```

#### `its exo recipients remove-send-as <identity> <trustee>`

Revoke Send-As permission. Reverse of add-send-as.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Required to revoke the Send-As permission | — |

**Examples:**

```bash
its exo recipients remove-send-as shared@thf.co.uk jo@thf.co.uk --confirm

its exo recipients remove-send-as shared@example.com jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its exo recipients remove-send-as shared@example.com jane.smith@example.com --json
```

---

### forwarding

> Source: `src/providers/exo/commands/forwarding.ts`

| Command | Description |
|---------|-------------|
| `its exo forwarding check [upn]` | Audit auto-forwarding posture — transport rules + outbound spam policies (+ optional mailbox) |

#### `its exo forwarding check [upn]`

Audit auto-forwarding posture — transport rules + outbound spam policies (+ optional mailbox).

**Examples:**

```bash
# Every mailbox with external forwarding configured
its exo forwarding check

# Pipe-friendly output — use with jq / scripts.
its exo forwarding check --json
```

---
