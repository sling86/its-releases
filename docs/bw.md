# Bitwarden (`bw`)

Bitwarden vault — search items, get passwords, browse folders.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [attendance](./attendance.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

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
- [generate](#generate)
- [sends](#sends)
- [compat](#compat)
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

**Handing a secret to another command** — use `--to-file <path>` on the same three commands. It writes a mode-0600 file and prints only the byte count, needs no terminal and no clipboard tool, and is the one sink that works on a headless box. Pair it with `its dokploy env set <app> KEY --from-file <path>` so the value never appears on argv (`/proc/<pid>/cmdline` is world-readable) or in shell history:

```bash
its bw items get <id> --to-file /dev/shm/sec
its dokploy env set <app> CLIENT_SECRET --from-file /dev/shm/sec --deploy
rm -f /dev/shm/sec
```

`--include-secrets` / `--unsafe` reveals plaintext only in interactive human output and is audit-logged. Machine output and redirected stdout reject plaintext-secret mode rather than leaking a value into a pipe or transcript. Do not use `--json --unsafe`, and never paste revealed output into chat, tickets, or AI tools.

### Checking readiness

`its bw setup --check` distinguishes *configured* from *usable*. The checklist covers the stored values; the line beneath it reports whether a command would actually work right now — `Configured but LOCKED` when no session exists, a warning when the session is nearly expired, silence when all is well. A locked vault is not a setup problem and the wizard cannot fix one, so the check names `its bw session unlock` rather than sending you back through setup. There is no connection test: the vault is unreachable until the master password is decrypted, and that needs a PIN a non-interactive check must not prompt for.

### Using from Claude Code or other AI shells

AI shell commands run non-interactively, so first run `its bw session unlock` yourself in a real terminal. The 8-hour session at `~/.its/sessions/bw.json` is inherited by subsequent calls. For unattended CI, cron, or hooks only, set `BW_PASSWORD_FILE` to a mode-`0600` file. With an unlocked session, use interactive `-c` when a human needs the value; raw secret stdout is deliberately unavailable to the AI process.

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/bw/client.ts` | API client methods |
| `src/providers/bw/types.ts` | TypeScript interfaces |
| `src/providers/bw/commands.ts` | Command definitions |
| `src/providers/bw/audit.ts` | audit |
| `src/providers/bw/commands-tools.ts` | commands tools |
| `src/providers/bw/compat-ops.ts` | compat ops |
| `src/providers/bw/compat.ts` | compat |
| `src/providers/bw/crypto.ts` | crypto |
| `src/providers/bw/definition.ts` | definition |
| `src/providers/bw/doctor.ts` | doctor |
| `src/providers/bw/export.ts` | export |
| `src/providers/bw/generate.ts` | generate |
| `src/providers/bw/serve.ts` | serve |
| `src/providers/bw/ssh.ts` | ssh |
| `src/providers/bw/wordlist.ts` | wordlist |

## Resources

### items

> Source: `src/providers/bw/commands.ts`

| Command | Description |
|---------|-------------|
| `its bw items` | List all vault items. Surfaces the most common fields; pass --json for raw shape. |
| `its bw items search <query>` | Search vault items by name, username, URL, or notes. Substring match across the most relevant fields; case-insensitive. |
| `its bw items find-login [query]` | Rank login candidates by name, URL and username without returning secrets. Timestamps only break equal-score ties; pass an exact ID to `items get` after choosing. |
| `its bw items get <id>` | Get a vault item by ID (includes password and fields). Pass the id (or any natural identifier) as the positional arg. The secret is redacted unless a sink is given: --copy for desk work, --to-file to hand it to another command. --field <name> targets a custom field instead of the login password. |
| `its bw items attachments <id>` | List the files attached to a vault item. Names are decrypted locally — the server never sees them. |
| `its bw items attach <id> <file>` | Attach a local file to a vault item. The file is encrypted locally under its own key before upload — the server never sees the contents or the filename. Additive: existing attachments are untouched. |
| `its bw items download <id> <attachment>` | Download and decrypt one attachment to a mode-0600 file. Never prints the contents — an attachment is usually key material, so it goes straight to disk like every other secret sink. |
| `its bw items detach <id> <attachment>` | Permanently delete one attachment from a vault item. There is no trash for attachments — this cannot be undone. Requires --confirm. |
| `its bw items totp <query>` | Generate current TOTP code for an item. Returns the current TOTP code — refresh every 30s. |
| `its bw items trash` | List trashed vault items. Returns soft-deleted items in the trash bin. |
| `its bw items recent` | List recently modified vault items. Returns the N most recently modified items. |
| `its bw items favourites` | List favourite vault items. Items the user has starred. |
| `its bw items create <name>` | Create a new vault item (login, note, card, or identity). Idempotent on duplicate names — use update/edit to mutate an existing record. |
| `its bw items update <id>` | Update a vault item. Preserve-by-default: only the flags you pass change — everything omitted (password, notes, URIs, TOTP, custom fields) is left intact. --field and --uri add to what is there; use --field-remove / --uri-remove to drop one. |
| `its bw items share <id>` | Irreversibly transfer a personal item to an organisation collection. There is no automatic rollback. |
| `its bw items move <id> [folder]` | Move vault items to a folder. Move an item between folders. --confirm required. |
| `its bw items delete <id>` | Move a vault item to trash (soft-delete, recoverable). Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record. |
| `its bw items restore <id>` | Restore a vault item from the trash. Restore a soft-deleted item from trash. |
| `its bw items purge <id>` | PERMANENTLY delete a vault item. This CANNOT be undone. |
| `its bw items export` | Export the vault in the official Bitwarden formats (json, csv, encrypted_json) so the file imports back into any Bitwarden client. Writes to --output (0600) or prints with --stdout. Trashed items are skipped. |
| `its bw items import <file>` | Import a Bitwarden export (json, csv, or encrypted/password-protected json) — folders come along. Items are added, never merged; import into a fresh folder first if unsure. Requires --confirm. |

#### `its bw items`

List all vault items. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `` | Filter by type (login/note/card/identity/ssh-key) | — |
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

#### `its bw items find-login [query]`

Rank login candidates by name, URL and username without returning secrets. Timestamps only break equal-score ties; pass an exact ID to `items get` after choosing.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--url` | `` | Site URL or hostname to match strongly | — |
| `--username` | `` | Expected username to match strongly | — |
| `--organisation` | `` | Organisation name or ID | — |
| `--collection` | `` | Collection name or ID | — |
| `--personal-only` | `` | Show only personal items | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw items find-login office.com --url login.microsoftonline.com --username admin@company.test --json
```

#### `its bw items get <id>`

Get a vault item by ID (includes password and fields). Pass the id (or any natural identifier) as the positional arg. The secret is redacted unless a sink is given: --copy for desk work, --to-file to hand it to another command. --field <name> targets a custom field instead of the login password.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--copy` | `-c` | Copy the secret to the OS clipboard instead of printing it. Auto-clears after --clear-after seconds. | — |
| `--clear-after` | `` | Seconds before the clipboard is wiped (0 disables). Only meaningful with --copy. | 30 |
| `--to-file` | `` | Write the secret to this path (created 0600 / owner-only) instead of printing it. Unlike --copy this needs no terminal or clipboard tool, so it works headless — the sanctioned way to hand a secret to another command. | — |
| `--field` | `` | Send this custom field to the sink instead of the login password (case-insensitive name match). | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
# Works headless — stdout shows only the byte count. Feed it to `its dokploy env set <app> KEY --from-file`
its bw items get "server-login" --to-file /dev/shm/sec

# For items whose secret lives in a custom field and whose password is empty
its bw items get "Outlook MCP" --field "API KEY" --to-file /dev/shm/sec

its bw items get "Server admin"
```

#### `its bw items attachments <id>`

List the files attached to a vault item. Names are decrypted locally — the server never sees them.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw items attachments "Exchange Online cert"
```

#### `its bw items attach <id> <file>`

Attach a local file to a vault item. The file is encrypted locally under its own key before upload — the server never sees the contents or the filename. Additive: existing attachments are untouched.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | Store under this filename instead of the file's own | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw items attach "Exchange Online cert" ./exo-auth.pfx

its bw items attach 3f2b1c94-... ./key.pem --name exo-key.pem
```

#### `its bw items download <id> <attachment>`

Download and decrypt one attachment to a mode-0600 file. Never prints the contents — an attachment is usually key material, so it goes straight to disk like every other secret sink.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--output` | `-o` | Where to write it (a directory keeps the vault's filename; defaults to the vault's filename in the current directory) | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw items download "Exchange Online cert" exo-auth.pfx --output /dev/shm/exo-auth.pfx

its bw items download 3f2b1c94-... exo-key.pem --output ./certs
```

#### `its bw items detach <id> <attachment>`

Permanently delete one attachment from a vault item. There is no trash for attachments — this cannot be undone. Requires --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Required — attachment deletion is irreversible | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw items detach "Exchange Online cert" exo-key.pem --confirm
```

#### `its bw items totp <query>`

Generate current TOTP code for an item. Returns the current TOTP code — refresh every 30s.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--copy` | `-c` | Copy the secret to the OS clipboard instead of printing it. Auto-clears after --clear-after seconds. | — |
| `--clear-after` | `` | Seconds before the clipboard is wiped (0 disables). Only meaningful with --copy. | 30 |
| `--to-file` | `` | Write the secret to this path (created 0600 / owner-only) instead of printing it. Unlike --copy this needs no terminal or clipboard tool, so it works headless — the sanctioned way to hand a secret to another command. | — |
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
| `--type` | `` | Item type: login (default), note, card, identity, ssh-key | — |
| `--username` | `` | Login (or identity) username | — |
| `--password` | `` | Login password | — |
| `--password-file` | `` | Read the password from a UTF-8 file (keeps the secret out of shell history and the command line) | — |
| `--uri` | `` | Login URL | — |
| `--totp` | `` | TOTP secret or otpauth URI | — |
| `--notes` | `` | Notes | — |
| `--notes-file` | `` | Read notes from a UTF-8 file (use for notes > ~15KB — Windows command-line cap) | — |
| `--folder` | `` | Folder name (created if it does not exist) | — |
| `--field` | `` | Custom text field(s) — comma-separated name=value (e.g. --field lan_ip=10.0.0.1,rack=A3). On update, upserts by name. | — |
| `--field-hidden` | `` | Custom hidden field(s) — comma-separated name=value. Stored as a secret (masked in the UI like a password). | — |
| `--cardholder` | `` | Card: name on the card | — |
| `--card-number` | `` | Card: number | — |
| `--card-brand` | `` | Card: brand (visa, mastercard, amex …) | — |
| `--card-exp` | `` | Card: expiry as MM/YYYY | — |
| `--card-code` | `` | Card: security code | — |
| `--first-name` | `` | Identity: first name | — |
| `--middle-name` | `` | Identity: middle name | — |
| `--last-name` | `` | Identity: last name | — |
| `--title` | `` | Identity: title (Mr, Ms …) | — |
| `--company` | `` | Identity: company | — |
| `--email` | `` | Identity: e-mail | — |
| `--phone` | `` | Identity: phone | — |
| `--address` | `` | Identity: address line 1 | — |
| `--address2` | `` | Identity: address line 2 | — |
| `--address3` | `` | Identity: address line 3 | — |
| `--city` | `` | Identity: city | — |
| `--county` | `` | Identity: county / state | — |
| `--postcode` | `` | Identity: postcode | — |
| `--country` | `` | Identity: country | — |
| `--ssn` | `` | Identity: national insurance / SSN | — |
| `--passport` | `` | Identity: passport number | — |
| `--licence` | `` | Identity: driving licence number | — |
| `--private-key-file` | `` | SSH key: path to the private key (OpenSSH/PEM). Public key + fingerprint are derived | — |
| `--public-key` | `` | SSH key: public key line, if it cannot be derived | — |
| `--fingerprint` | `` | SSH key: fingerprint, if it cannot be derived | — |
| `--organisation` | `` | Organisation name or ID | — |
| `--collection` | `` | Collection name or ID | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw items create "Router" --username admin --password "s3cret"

# Text + hidden custom fields. Multiple via comma: --field a=1,b=2
its bw items create "Router" --field lan_ip=10.0.0.1 --field-hidden api_token=abc123

its bw items create "Company Amex" --type card --cardholder "A Payer" --card-number 4111111111111111 --card-exp 09/2028 --card-code 1234

# Public key and fingerprint are derived from the private key
its bw items create "deploy@prod" --type ssh-key --private-key-file ~/.ssh/id_ed25519

its bw items create "Server admin" --username admin --password "P@ssw0rd" --uri https://server.example.com

its bw items create "API keys" --type note --notes "stuff"
```

#### `its bw items update <id>`

Update a vault item. Preserve-by-default: only the flags you pass change — everything omitted (password, notes, URIs, TOTP, custom fields) is left intact. --field and --uri add to what is there; use --field-remove / --uri-remove to drop one.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | New name | — |
| `--username` | `` | Login username | — |
| `--password` | `` | Login password | — |
| `--password-file` | `` | Read the password from a UTF-8 file (keeps the secret out of shell history and the command line) | — |
| `--uri` | `` | Login URL to add. Appended to the item's existing URIs (no duplicate) — use --uri-remove to drop one. | — |
| `--uri-remove` | `` | Login URL to remove from the item, matched exactly (case-insensitive). | — |
| `--totp` | `` | TOTP secret | — |
| `--notes` | `` | Notes | — |
| `--notes-file` | `` | Read notes from a UTF-8 file (use for notes > ~15KB — Windows command-line cap) | — |
| `--folder` | `` | Folder name (created if needed) | — |
| `--field` | `` | Custom text field(s) — comma-separated name=value (e.g. --field lan_ip=10.0.0.1,rack=A3). On update, upserts by name. | — |
| `--field-hidden` | `` | Custom hidden field(s) — comma-separated name=value. Stored as a secret (masked in the UI like a password). | — |
| `--cardholder` | `` | Card: name on the card | — |
| `--card-number` | `` | Card: number | — |
| `--card-brand` | `` | Card: brand (visa, mastercard, amex …) | — |
| `--card-exp` | `` | Card: expiry as MM/YYYY | — |
| `--card-code` | `` | Card: security code | — |
| `--first-name` | `` | Identity: first name | — |
| `--middle-name` | `` | Identity: middle name | — |
| `--last-name` | `` | Identity: last name | — |
| `--title` | `` | Identity: title (Mr, Ms …) | — |
| `--company` | `` | Identity: company | — |
| `--email` | `` | Identity: e-mail | — |
| `--phone` | `` | Identity: phone | — |
| `--address` | `` | Identity: address line 1 | — |
| `--address2` | `` | Identity: address line 2 | — |
| `--address3` | `` | Identity: address line 3 | — |
| `--city` | `` | Identity: city | — |
| `--county` | `` | Identity: county / state | — |
| `--postcode` | `` | Identity: postcode | — |
| `--country` | `` | Identity: country | — |
| `--ssn` | `` | Identity: national insurance / SSN | — |
| `--passport` | `` | Identity: passport number | — |
| `--licence` | `` | Identity: driving licence number | — |
| `--private-key-file` | `` | SSH key: path to the private key (OpenSSH/PEM). Public key + fingerprint are derived | — |
| `--public-key` | `` | SSH key: public key line, if it cannot be derived | — |
| `--fingerprint` | `` | SSH key: fingerprint, if it cannot be derived | — |
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

#### `its bw items move <id> [folder]`

Move vault items to a folder. Move an item between folders. --confirm required.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--folder` | `` | Destination folder name (created if needed; positional name also accepted) | — |
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

#### `its bw items export`

Export the vault in the official Bitwarden formats (json, csv, encrypted_json) so the file imports back into any Bitwarden client. Writes to --output (0600) or prints with --stdout. Trashed items are skipped.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--format` | `` | json (default), csv, or encrypted_json | — |
| `--password` | `` | encrypted_json only: password-protect instead of account-key encryption (importable to another account) | — |
| `--password-file` | `` | Read the export password from a file | — |
| `--output` | `` | File or directory to write (default: bitwarden_export_<stamp>.<ext> in cwd) | — |
| `--stdout` | `` | Print the export instead of writing a file | — |
| `--organisation` | `` | Export one organisation's vault (name or id) instead of the personal vault | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw items export --output ~/backups/

its bw items export --format encrypted_json --password-file /dev/shm/pw --output vault.json
```

#### `its bw items import <file>`

Import a Bitwarden export (json, csv, or encrypted/password-protected json) — folders come along. Items are added, never merged; import into a fresh folder first if unsure. Requires --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--format` | `` | auto (default), bitwardenjson, bitwardencsv, bitwardenpasswordprotected | — |
| `--password` | `` | Password of a password-protected export | — |
| `--password-file` | `` | Read the export password from a file | — |
| `--organisation` | `` | Import into this organisation (name or id); collections in the file are created | — |
| `--confirm` | `` | Confirm the import | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
# Without --confirm nothing is written — it reports what the file holds
its bw items import ./vault.json

its bw items import ./vault.json --confirm
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
| `its bw organisations members <organisation>` | List an organisation's members with invite status and role. Needs an admin/owner account. |
| `its bw organisations confirm <organisation> <member>` | Confirm a member who has accepted their invite: the organisation key is encrypted to their public key and handed over. After this they can see the collections they were granted. Requires --confirm. |

#### `its bw organisations`

List organisations available to the selected vault account.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--vault` | `` | Named vault profile (omit for default) | — |

```bash
its bw organisations
```

#### `its bw organisations members <organisation>`

List an organisation's members with invite status and role. Needs an admin/owner account.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--status` | `` | Filter: invited, accepted, confirmed, revoked | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw organisations members "THF"

its bw organisations members "THF" --status accepted
```

#### `its bw organisations confirm <organisation> <member>`

Confirm a member who has accepted their invite: the organisation key is encrypted to their public key and handed over. After this they can see the collections they were granted. Requires --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm handing the organisation key to the member | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw organisations confirm "THF" jane@example.com --confirm
```

---

### collections

> Source: `src/providers/bw/commands.ts`

| Command | Description |
|---------|-------------|
| `its bw collections` | List collections, optionally scoped by organisation name or ID. |
| `its bw collections create <organisation> <name>` | Create a collection in an organisation. Access can be granted to groups by id with --group (repeat via comma). |
| `its bw collections update <organisation> <collection>` | Rename a collection or replace its group grants. Groups not passed are removed — pass the full set. |
| `its bw collections delete <organisation> <collection>` | Delete a collection. Items in it stay in the organisation (unassigned). Requires --confirm. |

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

#### `its bw collections create <organisation> <name>`

Create a collection in an organisation. Access can be granted to groups by id with --group (repeat via comma).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--group` | `` | Group id(s) to grant, comma-separated. Suffix :ro for read-only, :manage for manage | — |
| `--external-id` | `` | External id (directory sync) | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw collections create "THF" "IT/Servers"
```

#### `its bw collections update <organisation> <collection>`

Rename a collection or replace its group grants. Groups not passed are removed — pass the full set.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | New name | — |
| `--group` | `` | Group id(s) to grant, comma-separated (replaces existing). :ro / :manage suffixes | — |
| `--confirm` | `` | Confirm the update | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw collections update "THF" "IT/Servers" --name "IT/Infrastructure" --confirm
```

#### `its bw collections delete <organisation> <collection>`

Delete a collection. Items in it stay in the organisation (unassigned). Requires --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the deletion | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw collections delete "THF" "IT/Old" --confirm
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
| `--to-file` | `` | Write the secret to this path (created 0600 / owner-only) instead of printing it. Unlike --copy this needs no terminal or clipboard tool, so it works headless — the sanctioned way to hand a secret to another command. | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
# Print the password (redacted by default; reveal is audit-logged)
its bw password "server-login" --include-secrets

# Copy to clipboard, auto-clear after 30s
its bw password "server-login" --copy

# 0600 file, no TTY needed — the composable sink for scripts
its bw password "server-login" --to-file /dev/shm/sec

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
| `its bw session serve` | Run the `bw serve` REST API on localhost, backed by the its session — same routes and JSON as the official CLI, so tools built against bw serve work unchanged. Ctrl+C to stop. |

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

#### `its bw session serve`

Run the `bw serve` REST API on localhost, backed by the its session — same routes and JSON as the official CLI, so tools built against bw serve work unchanged. Ctrl+C to stop.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--port` | `` | Port (default 8087) | — |
| `--hostname` | `` | Bind address (default localhost; 'all' for every interface) | — |
| `--disable-origin-protection` | `` | Accept requests that carry an Origin header (browser pages). Off by default for a reason. | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw session serve --port 8087
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
| `--type` | `` | Filter by type: login, note, card, identity, ssh-key | — |
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

### generate

> Source: `src/providers/bw/commands.ts`

| Command | Description |
|---------|-------------|
| `its bw generate password` | Generate a random password (default 14 chars, upper+lower+digits — the official bw defaults). Pass any of --upper/--lower/--number/--special to choose classes explicitly. |
| `its bw generate passphrase` | Generate a passphrase from the EFF long wordlist (default 6 words joined by '-', like the official bw). |

#### `its bw generate password`

Generate a random password (default 14 chars, upper+lower+digits — the official bw defaults). Pass any of --upper/--lower/--number/--special to choose classes explicitly.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--length` | `` | Length (5–128, default 14) | — |
| `--upper` | `` | Include uppercase letters | — |
| `--lower` | `` | Include lowercase letters | — |
| `--number` | `` | Include digits | — |
| `--special` | `` | Include !@#$%^&* | — |
| `--min-number` | `` | Minimum digits | — |
| `--min-special` | `` | Minimum special characters | — |
| `--avoid-ambiguous` | `` | Leave out I, O, l, 0, 1 | — |
| `--copy` | `-c` | Copy the secret to the OS clipboard instead of printing it. Auto-clears after --clear-after seconds. | — |
| `--clear-after` | `` | Seconds before the clipboard is wiped (0 disables). Only meaningful with --copy. | 30 |
| `--to-file` | `` | Write the secret to this path (created 0600 / owner-only) instead of printing it. Unlike --copy this needs no terminal or clipboard tool, so it works headless — the sanctioned way to hand a secret to another command. | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw generate password

its bw generate password --length 32 --upper --lower --number --special

its bw generate password --copy
```

#### `its bw generate passphrase`

Generate a passphrase from the EFF long wordlist (default 6 words joined by '-', like the official bw).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--words` | `` | Number of words (3–20, default 6) | — |
| `--separator` | `` | Word separator (default -) | — |
| `--capitalise` | `` | Title-case each word | — |
| `--include-number` | `` | Append a digit to one word | — |
| `--copy` | `-c` | Copy the secret to the OS clipboard instead of printing it. Auto-clears after --clear-after seconds. | — |
| `--clear-after` | `` | Seconds before the clipboard is wiped (0 disables). Only meaningful with --copy. | 30 |
| `--to-file` | `` | Write the secret to this path (created 0600 / owner-only) instead of printing it. Unlike --copy this needs no terminal or clipboard tool, so it works headless — the sanctioned way to hand a secret to another command. | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw generate passphrase

its bw generate passphrase --words 4 --separator . --capitalise --include-number
```

---

### sends

> Source: `src/providers/bw/commands.ts`

| Command | Description |
|---------|-------------|
| `its bw sends` | List your Bitwarden Sends with their share URLs. Surfaces the most common fields; pass --json for raw shape. |
| `its bw sends get <id>` | Show one Send by id or name, including its share URL. --text prints just the text body. |
| `its bw sends create [name]` | Create a Send — a self-expiring share link. --text for a snippet, --file for a file. Deletes after --delete-in (default 7d). The link is printed; anyone with it can open the Send until it expires. |
| `its bw sends update <id>` | Edit a Send's name, notes, limits or password. The content itself cannot change — create a new Send for that. |
| `its bw sends delete <id>` | Delete a Send now. The link stops working immediately; there is no trash. |
| `its bw sends remove-password <id>` | Remove the password from a Send so the link alone opens it. |
| `its bw sends receive <url>` | Open someone else's Send link. Text is printed (or sent to --to-file/--copy); a file is saved 0600 to --output. Works without a vault — the key is in the URL. |

#### `its bw sends`

List your Bitwarden Sends with their share URLs. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--search` | `` | Filter by name or notes | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw sends list
```

#### `its bw sends get <id>`

Show one Send by id or name, including its share URL. --text prints just the text body.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--text` | `` | Print only the text content | — |
| `--output` | `` | For a file Send: download the file here (dir or path) | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw sends get "VPN config"
```

#### `its bw sends create [name]`

Create a Send — a self-expiring share link. --text for a snippet, --file for a file. Deletes after --delete-in (default 7d). The link is printed; anyone with it can open the Send until it expires.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--text` | `` | Text to share | — |
| `--text-file` | `` | Read the text from a UTF-8 file | — |
| `--file` | `` | Path of a file to share (file Send) | — |
| `--notes` | `` | Private notes (only you see them) | — |
| `--hidden` | `` | Hide the text by default in the web view | — |
| `--password` | `` | Require this password to open the Send | — |
| `--password-file` | `` | Read the Send password from a file | — |
| `--emails` | `` | Restrict to these recipient e-mails (comma-separated; they get a one-time code) | — |
| `--max-access` | `` | Maximum number of opens | — |
| `--delete-in` | `` | Delete after e.g. 1h, 7d (default 7d) or an ISO date | — |
| `--expire-in` | `` | Stop access after e.g. 24h (optional, must be before delete-in) | — |
| `--hide-email` | `` | Do not show your e-mail to the recipient | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw sends create "WiFi" --text "guest / s3cret" --delete-in 1d

its bw sends create --file ./cert.pfx --password hunter2 --max-access 1
```

#### `its bw sends update <id>`

Edit a Send's name, notes, limits or password. The content itself cannot change — create a new Send for that.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | New name | — |
| `--notes` | `` | New private notes | — |
| `--hidden` | `` | Hide text by default | — |
| `--password` | `` | Set / replace the password | — |
| `--max-access` | `` | Maximum number of opens | — |
| `--delete-in` | `` | New deletion time (30m, 7d, ISO) | — |
| `--expire-in` | `` | New expiry time | — |
| `--disabled` | `` | Disable the link without deleting | — |
| `--enabled` | `` | Re-enable a disabled Send | — |
| `--confirm` | `` | Confirm the edit | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw sends update "WiFi" --delete-in 14d --confirm
```

#### `its bw sends delete <id>`

Delete a Send now. The link stops working immediately; there is no trash.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the deletion | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw sends delete "WiFi" --confirm
```

#### `its bw sends remove-password <id>`

Remove the password from a Send so the link alone opens it.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm removing the password | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw sends remove-password "WiFi" --confirm
```

#### `its bw sends receive <url>`

Open someone else's Send link. Text is printed (or sent to --to-file/--copy); a file is saved 0600 to --output. Works without a vault — the key is in the URL.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--password` | `` | Send password, if it has one | — |
| `--password-file` | `` | Read the Send password from a file | — |
| `--email` | `` | Your e-mail, for recipient-restricted Sends | — |
| `--otp` | `` | One-time code e-mailed to you (after a first attempt with --email) | — |
| `--output` | `` | Where to save a file Send (dir or path; default current dir) | — |
| `--copy` | `-c` | Copy the secret to the OS clipboard instead of printing it. Auto-clears after --clear-after seconds. | — |
| `--clear-after` | `` | Seconds before the clipboard is wiped (0 disables). Only meaningful with --copy. | 30 |
| `--to-file` | `` | Write the secret to this path (created 0600 / owner-only) instead of printing it. Unlike --copy this needs no terminal or clipboard tool, so it works headless — the sanctioned way to hand a secret to another command. | — |
| `--vault` | `` | Named vault profile (omit for default) | — |

**Examples:**

```bash
its bw sends receive https://send.bitwarden.com/#abc/def

its bw sends receive https://send.bitwarden.com/#abc/def --output ./downloads/
```

---

### compat

> Source: `src/providers/bw/commands.ts`

| Command | Description |
|---------|-------------|
| `its bw compat status` | Show whether the drop-in `bw` shim is installed and which `bw` wins on PATH. |
| `its bw compat install` | Install a `bw` shim so the official Bitwarden CLI syntax runs `its bw`: ~/.local/bin/bw on Linux/macOS (bw.cmd beside its.exe on Windows). Verbs its doesn't cover fall through to the real bw. |
| `its bw compat uninstall` | Remove the `bw` shim. Only deletes a file this command wrote. |

#### `its bw compat status`

Show whether the drop-in `bw` shim is installed and which `bw` wins on PATH.

**Examples:**

```bash
its bw compat status
```

#### `its bw compat install`

Install a `bw` shim so the official Bitwarden CLI syntax runs `its bw`: ~/.local/bin/bw on Linux/macOS (bw.cmd beside its.exe on Windows). Verbs its doesn't cover fall through to the real bw.

**Examples:**

```bash
its bw compat install
```

#### `its bw compat uninstall`

Remove the `bw` shim. Only deletes a file this command wrote.

**Examples:**

```bash
its bw compat uninstall
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
