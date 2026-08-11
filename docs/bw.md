# Bitwarden (`bw`)

Bitwarden vault — search items, get passwords, browse folders.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [items](#items)
- [folders](#folders)
- [organisations](#organisations)
- [collections](#collections)
- [password](#password)
- [profile](#profile)
- [dashboard](#dashboard)
- [pin](#pin)
- [session](#session)
- [vaults](#vaults)
- [audit](#audit)
- [doctor](#doctor)

## Setup

```bash
its bw setup           # Interactive wizard
its bw setup --check   # Check configuration status
its bw setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `BW_URL` | Bitwarden/Vaultwarden server URL |
| `BW_EMAIL` | Email address (recommended — like the desktop app) |
| `BW_PASSWORD_FILE` | Path to a 0600-permission file containing the master password (CI / unattended use only). Trailing newline trimmed. Safer than passing the password as an env var (which leaks via /proc/<pid>/environ and shell history). For interactive use, prefer the PIN-encrypted store at ~/.its/secrets/bw-password.enc. |

**Auth options:** Use `BW_EMAIL` (recommended) for email+password auth, or `BW_CLIENTID` + `BW_CLIENTSECRET` for API key auth. The setup wizard (`its bw setup`) lets you choose.

The master password is stored PIN-encrypted in `~/.its/secrets/`. During setup you choose a PIN (8+ characters) that encrypts your master password locally. The PIN is required each time you use `its bw`.

**Multiple vaults** — the default vault is configured via env vars (`BW_URL`, `BW_EMAIL`/`BW_CLIENTID`+`BW_CLIENTSECRET`) plus the PIN-encrypted master password at `~/.its/secrets/bw-password.enc`. For additional vaults, run `its bw vaults create <name>`; credentials are encrypted into `~/.its/secrets/bw-vault-<name>.enc`. Switch per command with `--vault <name>`, list profiles with `its bw vaults`, and remove local profile configuration with `its bw vaults delete <name> --confirm`.

**Identity print** — every authentication prints `Authenticating <email-or-clientid> @ <host>` to stderr so a missing `--vault <name>` is visible before any command runs.

**2FA behaviour** — after successful TOTP/email authentication, a 30-day remember token is stored per email (OS keychain preferred, encrypted-file fallback). If a server challenges again, only that vault profile's token is cleared.

### Viewing secrets

Secret fields are redacted by default. For interactive desk work, use `--copy` / `-c` on `its bw password <q>`, `its bw items get <id>`, or `its bw items totp <q>`; the value goes directly to the OS clipboard and is cleared after `--clear-after` seconds (default 30). `--copy` requires an interactive terminal and cannot be combined with `--json`, `--ai`, `--jsonl`, CSV/TSV, or redirected stdout.

`--include-secrets` / `--unsafe` reveals plaintext only in interactive human output and is audit-logged. Machine output and redirected stdout reject plaintext-secret mode rather than leaking a value into a pipe or transcript. Do not use `--json --unsafe`, and never paste revealed output into chat, tickets, or AI tools.

### Using from Claude Code or other AI shells

AI shell commands run non-interactively, so first run `its bw session unlock` yourself in a real terminal. The 8-hour session at `~/.its/sessions/bw.json` is inherited by subsequent calls. For unattended CI, cron, or hooks only, set `BW_PASSWORD_FILE` to a mode-`0600` file. With an unlocked session, use interactive `-c` when a human needs the value; raw secret stdout is deliberately unavailable to the AI process.

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/bw/client.ts` | API client methods |
| `src/providers/bw/types.ts` | TypeScript interfaces |
| `src/providers/bw/commands.ts` | Command definitions |
| `src/providers/bw/audit.ts` | audit |
| `src/providers/bw/crypto.ts` | crypto |
| `src/providers/bw/definition.ts` | definition |
| `src/providers/bw/doctor.ts` | doctor |

## Resources

### items

> Source: `src/providers/bw/commands.ts`

| Command | Description |
|---------|-------------|
| `its bw items` | List all vault items. Surfaces the most common fields; pass --json for raw shape. |
| `its bw items search <query>` | Search vault items by name, username, URL, or notes. Substring match across the most relevant fields; case-insensitive. |
| `its bw items get <id>` | Get a vault item by ID (includes password and fields). Pass the id (or any natural identifier) as the positional arg. |
| `its bw items totp <query>` | Generate current TOTP code for an item. Returns the current TOTP code — refresh every 30s. |
| `its bw items trash` | List trashed vault items. Returns soft-deleted items in the trash bin. |
| `its bw items recent` | List recently modified vault items. Returns the N most recently modified items. |
| `its bw items favourites` | List favourite vault items. Items the user has starred. |
| `its bw items create <name>` | Create a new vault item (login, note, card, or identity). Idempotent on duplicate names — use update/edit to mutate an existing record. |
| `its bw items update <id>` | Update a vault item. Preserve-by-default: only the flags you pass change — everything omitted (password, notes, URIs, TOTP, custom fields) is left intact. Use --field-remove to drop a custom field. |
| `its bw items share <id>` | Irreversibly transfer a personal item to an organisation collection. There is no automatic rollback. |
| `its bw items move <id>` | Move vault items to a folder. Move an item between folders. --confirm required. |
| `its bw items delete <id>` | Move a vault item to trash (soft-delete, recoverable). Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record. |
| `its bw items restore <id>` | Restore a vault item from the trash. Restore a soft-deleted item from trash. |
| `its bw items purge <id>` | PERMANENTLY delete a vault item. This CANNOT be undone. |

#### `its bw items`

List all vault items. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `` | Filter by type (login/note/card/identity) | — |
| `--folder` | `` | Filter by folder name | — |
| `--favourite` | `` | Show only favourites | — |
| `--organisation` | `` | Organisation name or ID | — |
| `--collection` | `` | Collection name or ID | — |
| `--personal-only` | `` | Show only personal items | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw items

its bw items --folder "Servers"

# Re-runs every 10s — handy for dashboards or incident response.
its bw items --watch
```

#### `its bw items search <query>`

Search vault items by name, username, URL, or notes. Substring match across the most relevant fields; case-insensitive.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--organisation` | `` | Organisation name or ID | — |
| `--collection` | `` | Collection name or ID | — |
| `--personal-only` | `` | Show only personal items | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw items search "github"
```

#### `its bw items get <id>`

Get a vault item by ID (includes password and fields). Pass the id (or any natural identifier) as the positional arg.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--copy` | `-c` | Copy the secret to the OS clipboard instead of printing it. Auto-clears after --clear-after seconds. | — |
| `--clear-after` | `` | Seconds before the clipboard is wiped (0 disables). Only meaningful with --copy. | 30 |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw items get "Server admin"
```

#### `its bw items totp <query>`

Generate current TOTP code for an item. Returns the current TOTP code — refresh every 30s.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--copy` | `-c` | Copy the secret to the OS clipboard instead of printing it. Auto-clears after --clear-after seconds. | — |
| `--clear-after` | `` | Seconds before the clipboard is wiped (0 disables). Only meaningful with --copy. | 30 |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw items totp "GitHub"

its bw items totp "GitHub" --copy --clear-after 30

# Generate current TOTP — refreshes every 30s
its bw items totp "Server admin"
```

#### `its bw items trash`

List trashed vault items. Returns soft-deleted items in the trash bin.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw items trash
```

#### `its bw items recent`

List recently modified vault items. Returns the N most recently modified items.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--days` | `` | Look-back period in days | 7 |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw items recent
```

#### `its bw items favourites`

List favourite vault items. Items the user has starred.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw items favourites
```

#### `its bw items create <name>`

Create a new vault item (login, note, card, or identity). Idempotent on duplicate names — use update/edit to mutate an existing record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `` | Item type: login (default), note, card, identity | — |
| `--username` | `` | Login username | — |
| `--password` | `` | Login password | — |
| `--password-file` | `` | Read the password from a UTF-8 file (keeps the secret out of shell history and the command line) | — |
| `--uri` | `` | Login URL | — |
| `--totp` | `` | TOTP secret or otpauth URI | — |
| `--notes` | `` | Notes | — |
| `--notes-file` | `` | Read notes from a UTF-8 file (use for notes > ~15KB — Windows command-line cap) | — |
| `--folder` | `` | Folder name (created if it does not exist) | — |
| `--field` | `` | Custom text field(s) — comma-separated name=value (e.g. --field lan_ip=10.0.0.1,rack=A3). On update, upserts by name. | — |
| `--field-hidden` | `` | Custom hidden field(s) — comma-separated name=value. Stored as a secret (masked in the UI like a password). | — |
| `--organisation` | `` | Organisation name or ID | — |
| `--collection` | `` | Collection name or ID | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw items create "Router" --username admin --password "s3cret"

# Text + hidden custom fields. Multiple via comma: --field a=1,b=2
its bw items create "Router" --field lan_ip=10.0.0.1 --field-hidden api_token=abc123

its bw items create "Server admin" --username admin --password "P@ssw0rd" --uri https://server.example.com

its bw items create "API keys" --type note --notes "stuff"
```

#### `its bw items update <id>`

Update a vault item. Preserve-by-default: only the flags you pass change — everything omitted (password, notes, URIs, TOTP, custom fields) is left intact. Use --field-remove to drop a custom field.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | New name | — |
| `--username` | `` | Login username | — |
| `--password` | `` | Login password | — |
| `--password-file` | `` | Read the password from a UTF-8 file (keeps the secret out of shell history and the command line) | — |
| `--uri` | `` | Login URL | — |
| `--totp` | `` | TOTP secret | — |
| `--notes` | `` | Notes | — |
| `--notes-file` | `` | Read notes from a UTF-8 file (use for notes > ~15KB — Windows command-line cap) | — |
| `--folder` | `` | Folder name (created if needed) | — |
| `--field` | `` | Custom text field(s) — comma-separated name=value (e.g. --field lan_ip=10.0.0.1,rack=A3). On update, upserts by name. | — |
| `--field-hidden` | `` | Custom hidden field(s) — comma-separated name=value. Stored as a secret (masked in the UI like a password). | — |
| `--field-remove` | `` | Custom field name(s) to remove — comma-separated (e.g. --field-remove old_ip,legacy_token). | — |
| `--confirm` | `` | Confirm the update | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
# Upserts by name; password/notes/other fields untouched.
its bw items update <id> --field lan_ip=10.0.0.2 --confirm

its bw items update <id> --field-remove lan_ip --confirm

its bw items update <item-id> --password "NewP@ss" --confirm
```

#### `its bw items share <id>`

Irreversibly transfer a personal item to an organisation collection. There is no automatic rollback.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--organisation` | `` | Organisation name or ID | — |
| `--collection` | `` | Collection name or ID | — |
| `--confirm` | `` | Confirm the irreversible ownership transfer | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
# Moves the item into the organisation — it leaves your personal vault and cannot be moved back by this command
its bw items share 3f2b1c94-... --organisation Acme --collection IT --confirm
```

#### `its bw items move <id>`

Move vault items to a folder. Move an item between folders. --confirm required.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--folder` | `` | Destination folder name (created if needed) | — |
| `--confirm` | `` | Confirm the move | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw items move 3f2b1c94-... --folder "Infrastructure" --confirm

its bw items move <item-id> --folder "Servers" --confirm
```

#### `its bw items delete <id>`

Move a vault item to trash (soft-delete, recoverable). Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the deletion | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
# Sends the item to the trash — recoverable until purged
its bw items delete 3f2b1c94-... --confirm

# Moves to trash, recoverable for 30 days
its bw items delete <item-id> --confirm
```

#### `its bw items restore <id>`

Restore a vault item from the trash. Restore a soft-deleted item from trash.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the restore | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw items restore 3f2b1c94-... --confirm

its bw items restore <item-id> --confirm
```

#### `its bw items purge <id>`

PERMANENTLY delete a vault item. This CANNOT be undone.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm permanent deletion (REQUIRED) | — |
| `--yes-permanently-delete` | `` | Double-confirm that you understand this is irreversible | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
# Unrecoverable. Both flags are required — there is no trash to restore from afterwards
its bw items purge 3f2b1c94-... --confirm --yes-permanently-delete

# CANNOT be undone
its bw items purge <item-id> --confirm --yes-permanently-delete
```

---

### folders

> Source: `src/providers/bw/commands.ts`

| Command | Description |
|---------|-------------|
| `its bw folders` | List all vault folders. Surfaces the most common fields; pass --json for raw shape. |
| `its bw folders get <name>` | List items in a folder by name. Pass the id (or any natural identifier) as the positional arg. |
| `its bw folders summary` | List folders with item counts. Quick one-screen view — designed for dashboards / `--watch`. |
| `its bw folders create <name>` | Create a new folder. Idempotent on duplicate names — use update/edit to mutate an existing record. |
| `its bw folders delete <name>` | Delete a folder (items in it are moved to No Folder, not deleted) |

#### `its bw folders`

List all vault folders. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw folders

# Re-runs every 10s — handy for dashboards or incident response.
its bw folders --watch
```

#### `its bw folders get <name>`

List items in a folder by name. Pass the id (or any natural identifier) as the positional arg.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw folders get "Servers"
```

#### `its bw folders summary`

List folders with item counts. Quick one-screen view — designed for dashboards / `--watch`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw folders summary

# Re-runs every 10s — handy for dashboards or incident response.
its bw folders summary --watch
```

#### `its bw folders create <name>`

Create a new folder. Idempotent on duplicate names — use update/edit to mutate an existing record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw folders create "Infrastructure"

its bw folders create "Servers"
```

#### `its bw folders delete <name>`

Delete a folder (items in it are moved to No Folder, not deleted).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm folder deletion | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
# Items in the folder are kept — they become unfiled
its bw folders delete "Old kit" --confirm

its bw folders delete "Old stuff" --confirm
```

---

### organisations

> Source: `src/providers/bw/commands.ts`

| Command | Description |
|---------|-------------|
| `its bw organisations` | List organisations available to the selected vault account. |

#### `its bw organisations`

List organisations available to the selected vault account.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

```bash
its bw organisations
```

---

### collections

> Source: `src/providers/bw/commands.ts`

| Command | Description |
|---------|-------------|
| `its bw collections` | List collections, optionally scoped by organisation name or ID. |

#### `its bw collections`

List collections, optionally scoped by organisation name or ID.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--organisation` | `` | Organisation name or ID | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

```bash
its bw collections
```

---

### password

> Source: `src/providers/bw/commands.ts`

| Command | Description |
|---------|-------------|
| `its bw password <query>` | Get the password for an item by search query. Surfaces the most common fields; pass --json for raw shape. |

#### `its bw password <query>`

Get the password for an item by search query. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--copy` | `-c` | Copy the secret to the OS clipboard instead of printing it. Auto-clears after --clear-after seconds. | — |
| `--clear-after` | `` | Seconds before the clipboard is wiped (0 disables). Only meaningful with --copy. | 30 |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
# Print the password (redacted by default; reveal is audit-logged)
its bw password "server-login" --include-secrets

# Copy to clipboard, auto-clear after 30s
its bw password "server-login" --copy

# Print password (mask in shared terminals)
its bw password "server-login"

# Re-runs every 10s — handy for dashboards or incident response.
its bw password "server-login" --watch
```

---

### profile

> Source: `src/providers/bw/commands.ts`

| Command | Description |
|---------|-------------|
| `its bw profile` | Show vault profile information. Surfaces the most common fields; pass --json for raw shape. |

#### `its bw profile`

Show vault profile information. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw profile

# Re-runs every 10s — handy for dashboards or incident response.
its bw profile --watch
```

---

### dashboard

> Source: `src/providers/bw/commands.ts`

| Command | Description |
|---------|-------------|
| `its bw dashboard` | Vault summary statistics. Surfaces the most common fields; pass --json for raw shape. |

#### `its bw dashboard`

Vault summary statistics. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw dashboard

# Re-runs every 10s — handy for dashboards or incident response.
its bw dashboard --watch
```

---

### pin

> Source: `src/providers/bw/commands.ts`

| Command | Description |
|---------|-------------|
| `its bw pin reset` | Change the PIN used to encrypt the master password. Drop the resource's state — use --confirm. |

#### `its bw pin reset`

Change the PIN used to encrypt the master password. Drop the resource's state — use --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw pin reset
```

---

### session

> Source: `src/providers/bw/commands.ts`

| Command | Description |
|---------|-------------|
| `its bw session unlock` | Unlock vault — skip PIN prompt for subsequent commands. Begin an interactive session — see `bw session unlock`. |
| `its bw session lock` | Lock vault and destroy the active session. End the current session. |
| `its bw session` | Check if a vault session is active. Surfaces the most common fields; pass --json for raw shape. |

#### `its bw session unlock`

Unlock vault — skip PIN prompt for subsequent commands. Begin an interactive session — see `bw session unlock`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--ttl` | `` | Session duration in minutes (default 480 = 8 hours) | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw session unlock

# Subsequent bw commands skip the PIN prompt until it expires
its bw session unlock --ttl 3600
```

#### `its bw session lock`

Lock vault and destroy the active session. End the current session.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw session lock
```

#### `its bw session`

Check if a vault session is active. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw session list

# Re-runs every 10s — handy for dashboards or incident response.
its bw session list --watch
```

---

### vaults

> Source: `src/providers/bw/commands.ts`

| Command | Description |
|---------|-------------|
| `its bw vaults` | List configured vault profiles. Surfaces the most common fields; pass --json for raw shape. |
| `its bw vaults create <name>` | Save a named vault profile — its own host, account and master password (use for a second vault on a different server) |
| `its bw vaults delete <name>` | Delete a named vault profile (local config only — does NOT touch the actual vault or its data) |

#### `its bw vaults`

List configured vault profiles. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its bw vaults

# Re-runs every 10s — handy for dashboards or incident response.
its bw vaults --watch
```

#### `its bw vaults create <name>`

Save a named vault profile — its own host, account and master password (use for a second vault on a different server).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--url` | `` | Server URL (defaults to current BW_URL) | — |
| `--email` | `` | Email (defaults to current BW_EMAIL) | — |
| `--client-id` | `` | API client ID (for API key auth) | — |
| `--client-secret` | `` | API client secret (for API key auth) | — |

**Examples:**

```bash
its bw vaults create work --url https://vault.example.com --email jane.smith@example.com

its bw vaults create "personal"
```

#### `its bw vaults delete <name>`

Delete a named vault profile (local config only — does NOT touch the actual vault or its data).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Required to actually remove the profile | — |

**Examples:**

```bash
# Removes the local vault profile and its stored credentials
its bw vaults delete work --confirm

its bw vaults list

# Local config only — doesn't touch the actual vault
its bw vaults delete "personal" --confirm
```

---

### audit

> Source: `src/providers/bw/commands.ts`

| Command | Description |
|---------|-------------|
| `its bw audit` | Full vault health audit (weak passwords, reuse, duplicates, cleanup issues) |
| `its bw audit weak` | Find logins with weak passwords. Identifies weak passwords; pair with `bw audit reused`. |
| `its bw audit reused` | Find passwords reused across multiple logins. Identifies passwords shared across multiple items. |
| `its bw audit exposed` | Check passwords against Have I Been Pwned breaches (k-anonymity safe) |
| `its bw audit duplicates` | Detect duplicate logins (domain+username, name+username matching) |
| `its bw audit unfiled` | Vault items with no folder assigned (hygiene issue). Items with no folder assignment. |
| `its bw audit cleanup` | Detect vault hygiene issues (skeleton logins, missing fields, empty items) |
| `its bw audit vault-report` | One-shot vault hygiene snapshot — counts, unfiled breakdown, weak/reused/duplicates, and per-folder coverage. Composes audit weak/reused/duplicates/unfiled so the numbers stay in sync with the individual commands. |

#### `its bw audit`

Full vault health audit (weak passwords, reuse, duplicates, cleanup issues).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
# Weak passwords, reuse, duplicates, cleanup
its bw audit

# Re-runs every 10s — handy for dashboards or incident response.
its bw audit --watch
```

#### `its bw audit weak`

Find logins with weak passwords. Identifies weak passwords; pair with `bw audit reused`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw audit weak
```

#### `its bw audit reused`

Find passwords reused across multiple logins. Identifies passwords shared across multiple items.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw audit reused
```

#### `its bw audit exposed`

Check passwords against Have I Been Pwned breaches (k-anonymity safe).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
# k-anonymity safe — sends only password hash prefix
its bw audit exposed
```

#### `its bw audit duplicates`

Detect duplicate logins (domain+username, name+username matching).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
# Items with identical name/username
its bw audit duplicates
```

#### `its bw audit unfiled`

Vault items with no folder assigned (hygiene issue). Items with no folder assignment.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `` | Filter by type: login, note, card, identity | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
# Items missing folder/collection
its bw audit unfiled
```

#### `its bw audit cleanup`

Detect vault hygiene issues (skeleton logins, missing fields, empty items).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
# Skeleton logins, missing fields, empty items
its bw audit cleanup
```

#### `its bw audit vault-report`

One-shot vault hygiene snapshot — counts, unfiled breakdown, weak/reused/duplicates, and per-folder coverage. Composes audit weak/reused/duplicates/unfiled so the numbers stay in sync with the individual commands.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--min-items` | `` | Only show folders with this many items or more (default 1) | 1 |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw audit vault-report
```

---

### doctor

> Source: `src/providers/bw/commands.ts`

| Command | Description |
|---------|-------------|
| `its bw doctor` | Local health check — vault profiles, active sessions, 2FA-remember token age, master-password store presence. No network calls. |

#### `its bw doctor`

Local health check — vault profiles, active sessions, 2FA-remember token age, master-password store presence. No network calls.

**Examples:**

```bash
# Auth, session, 2FA, PIN status
its bw doctor

# Re-runs every 10s — handy for dashboards or incident response.
its bw doctor --watch
```

---
