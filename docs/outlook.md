# Outlook (`outlook`)

Microsoft Outlook (Graph) mailbox + calendar — list/search/send mail, manage drafts, organise folders, schedule events, check free/busy, configure auto-reply and inbox rules.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [mail](#mail)
- [drafts](#drafts)
- [folders](#folders)
- [attachments](#attachments)
- [events](#events)
- [settings](#settings)
- [autoreply](#autoreply)
- [categories](#categories)
- [rules](#rules)
- [contacts](#contacts)
- [triage](#triage)

## Setup

```bash
its outlook setup           # Interactive wizard
its outlook setup --check   # Check configuration status
its outlook setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `TENANT_ID` | Microsoft Entra tenant ID (shared) |
| `CLIENT_ID` | App registration client ID (shared fallback) |
| `CLIENT_SECRET` | App registration client secret (shared fallback) |
| `OUTLOOK_TENANT_ID` | Tenant ID override for the Outlook app (rarely needed) |
| `OUTLOOK_CLIENT_ID` | Outlook-specific app registration client ID (preferred over CLIENT_ID) |
| `OUTLOOK_CLIENT_SECRET` | Outlook-specific app registration client secret |
| `OUTLOOK_DEFAULT_USER` | UPN of the mailbox to operate on under app-only auth (ignored when delegated token is cached) |
| `OUTLOOK_DEFAULT_TZ` | Default time zone for `events create/update` when --tz is not passed (default UTC) |

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/outlook/client.ts` | API client methods |
| `src/providers/outlook/types.ts` | TypeScript interfaces |
| `src/providers/outlook/commands/` | Command definitions (split by resource) |
| `src/providers/outlook/definition.ts` | definition |
| `src/providers/outlook/helpers.ts` | helpers |

## Resources

### mail

> Source: `src/providers/outlook/commands/mail.ts`

| Command | Description |
|---------|-------------|
| `its outlook mail` | List messages from the mailbox. Defaults to Inbox (top 25 by receivedDateTime desc). Use --folder for a specific folder, --filter for OData expressions, --search to switch to keyword search. |
| `its outlook mail get <message_id>` | Get a single message including body, recipients, and flags. |
| `its outlook mail headers <message_id>` | Show a message's internet headers + parsed antispam verdict (SCL/SFV/CAT/BCL + spf/dkim/dmarc/compauth). The 'why did this land in Junk' tool. Add --all for every raw header. |
| `its outlook mail search <query>` | Keyword search across the mailbox using Graph $search (KQL syntax). |
| `its outlook mail thread <conversation_id>` | List every message in the same conversation (entire thread). |
| `its outlook mail move <message_id> <folder_id>` | Move a message to another folder. |
| `its outlook mail copy <message_id> <folder_id>` | Copy a message to another folder. |
| `its outlook mail read <message_id>` | Mark a message as read. |
| `its outlook mail unread <message_id>` | Mark a message as unread. |
| `its outlook mail flag <message_id>` | Set follow-up flag status on a message. |
| `its outlook mail categorise <message_id> <categories>` | Set the category list on a message (replaces existing categories). |
| `its outlook mail delete [message_id]` | Delete one or more messages (moves to Deleted Items). Pass a single id positionally, or pipe `mail list --json` to stdin with --stdin for bulk delete. |
| `its outlook mail send` | Send a new email directly (no draft step). Saves a copy in Sent Items. |

#### `its outlook mail`

List messages from the mailbox. Defaults to Inbox (top 25 by receivedDateTime desc). Use --folder for a specific folder, --filter for OData expressions, --search to switch to keyword search.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--folder` | `` | Folder ID or well-known name (inbox, sentitems, drafts, deleteditems, archive) | — |
| `--top` | `` | Number of messages (max 50) | 25 |
| `--skip` | `` | Skip first N messages (pagination) | 0 |
| `--filter` | `` | OData $filter expression | — |
| `--search` | `` | KQL-style search query (alternative to --filter) | — |
| `--unread` | `` | Only unread messages | — |
| `--has-attachments` | `` | Only messages with attachments | — |
| `--from` | `` | Filter by sender email address (substring match via OData) | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
its outlook mail

its outlook mail --unread

its outlook mail --from boss@example.com

its outlook mail --search invoice

its outlook mail --folder sentitems
```

#### `its outlook mail get <message_id>`

Get a single message including body, recipients, and flags.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--no-body` | `` | Skip the body content (faster, smaller output) | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

```bash
its outlook mail get <message_id>
```

#### `its outlook mail headers <message_id>`

Show a message's internet headers + parsed antispam verdict (SCL/SFV/CAT/BCL + spf/dkim/dmarc/compauth). The 'why did this land in Junk' tool. Add --all for every raw header.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--all` | `` | Show every raw internet header, not just the antispam summary | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

```bash
its outlook mail headers <message_id>
```

#### `its outlook mail search <query>`

Keyword search across the mailbox using Graph $search (KQL syntax).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Max results (max 50) | 25 |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
its outlook mail search "subject:invoice"

its outlook mail search "from:boss@example.com"

its outlook mail search renewal
```

#### `its outlook mail thread <conversation_id>`

List every message in the same conversation (entire thread).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Max messages (max 50) | 50 |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

```bash
its outlook mail thread <conversation_id>
```

#### `its outlook mail move <message_id> <folder_id>`

Move a message to another folder.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
its outlook mail move AAMkAGI1AAAt0M0AAA= AQMkAGI1Archive --user jane.smith@example.com
```

#### `its outlook mail copy <message_id> <folder_id>`

Copy a message to another folder.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
its outlook mail copy AAMkAGI1AAAt0M0AAA= AQMkAGI1Archive --user jane.smith@example.com
```

#### `its outlook mail read <message_id>`

Mark a message as read.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
its outlook mail read AAMkAGI1AAAt0M0AAA= --user jane.smith@example.com
```

#### `its outlook mail unread <message_id>`

Mark a message as unread.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
its outlook mail unread AAMkAGI1AAAt0M0AAA= --user jane.smith@example.com
```

#### `its outlook mail flag <message_id>`

Set follow-up flag status on a message.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--status` | `` | Flag status | flagged |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
its outlook mail flag AAMkAGI1AAAt0M0AAA= --status flagged --user jane.smith@example.com

its outlook mail flag AAMkAGI1AAAt0M0AAA= --status complete --user jane.smith@example.com
```

#### `its outlook mail categorise <message_id> <categories>`

Set the category list on a message (replaces existing categories).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
# Replaces the existing category list rather than appending
its outlook mail categorise AAMkAGI1AAAt0M0AAA= "IT,Follow up" --user jane.smith@example.com
```

#### `its outlook mail delete [message_id]`

Delete one or more messages (moves to Deleted Items). Pass a single id positionally, or pipe `mail list --json` to stdin with --stdin for bulk delete.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Required for every delete (single and bulk) | — |
| `--stdin` | `` | Read newline/JSON list of message ids from stdin | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
its outlook mail delete <message_id> --confirm

its outlook mail --filter "from/emailAddress/address eq 'spammer@x'" --json | its outlook mail delete --stdin --confirm
```

#### `its outlook mail send`

Send a new email directly (no draft step). Saves a copy in Sent Items.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--to` | `` | Comma-separated recipients (Name <addr> or addr) | — |
| `--cc` | `` | Comma-separated CC recipients | — |
| `--bcc` | `` | Comma-separated BCC recipients | — |
| `--subject` | `` | Subject line | — |
| `--body` | `` | Body content | — |
| `--body-file` | `` | Read body from a UTF-8 file (use for bodies > ~15KB — Windows command-line cap) | — |
| `--html` | `` | Treat --body / --body-file as HTML (default text) | — |
| `--importance` | `` | low|normal|high | normal |
| `--attach` | `` | File to attach. Comma-separated for multiple. `path:cid:<id>` syntax marks an attachment inline with that cid (pair with `<img src="cid:<id>">` in body). | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |
| `--var` | `` | Template substitution — `--var k1=v1,k2=v2`. Substitutes `${key}` in body/comment after --body-file read. | — |

**Examples:**

```bash
its outlook mail send --to user@example.com --subject "Hi" --body "Quick note"

its outlook mail send --to user@example.com --subject "Report" --html --body-file report.html

its outlook mail send --to user@example.com --subject "Welcome" --html --body-file welcome.html --attach "logo.png:cid:logo-1"
```

---

### drafts

> Source: `src/providers/outlook/commands/drafts.ts`

| Command | Description |
|---------|-------------|
| `its outlook drafts create` | Create a draft email (does not send). |
| `its outlook drafts reply <message_id>` | Create a reply draft. By default replies to the sender only; use --all to Reply-All. Does not send — use `drafts send <id>` after edits. |
| `its outlook drafts forward <message_id>` | Create a forward draft. |
| `its outlook drafts update <draft_id>` | Patch an existing draft. Any of subject/body/to/cc/bcc/importance/categories. |
| `its outlook drafts send <draft_id>` | Send an existing draft. |
| `its outlook drafts` | List draft messages (convenience for `mail list --folder drafts`). |
| `its outlook drafts delete <id>` | Delete a draft by message id. Refuses anything that is not a draft, so a mistyped id cannot delete a sent or received message through this command. Permanent — use --confirm. |

#### `its outlook drafts create`

Create a draft email (does not send).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--to` | `` | Comma-separated recipients | — |
| `--cc` | `` | CC recipients | — |
| `--bcc` | `` | BCC recipients | — |
| `--subject` | `` | Subject line | — |
| `--body` | `` | Body content | — |
| `--body-file` | `` | Read body from a UTF-8 file (use this for bodies > ~15KB — Windows command-line cap) | — |
| `--html` | `` | Treat --body / --body-file as HTML (default text) | — |
| `--importance` | `` | low|normal|high | normal |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |
| `--var` | `` | Template substitution — `--var k1=v1,k2=v2`. Substitutes `${key}` in body/comment after --body-file read. | — |

**Examples:**

```bash
its outlook drafts create --to jane.smith@example.com --subject "Laptop ready" --body "Come and collect when convenient." --user jane.smith@example.com

its outlook drafts create --to jane.smith@example.com --subject "Change notice" --body-file ./notice.html --html --user jane.smith@example.com
```

#### `its outlook drafts reply <message_id>`

Create a reply draft. By default replies to the sender only; use --all to Reply-All. Does not send — use `drafts send <id>` after edits.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--comment` | `` | Inline comment prepended to the reply body | — |
| `--comment-file` | `` | Read --comment from a UTF-8 file (bypasses Windows ~32K command-line cap) | — |
| `--all` | `` | Reply to all recipients | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |
| `--var` | `` | Template substitution — `--var k1=v1,k2=v2`. Substitutes `${key}` in body/comment after --body-file read. | — |

**Examples:**

```bash
its outlook drafts reply AAMkAGI1AAAt0M0AAA= --comment "On it now." --user jane.smith@example.com

its outlook drafts reply AAMkAGI1AAAt0M0AAA= --all --comment "Adding IT." --user jane.smith@example.com
```

#### `its outlook drafts forward <message_id>`

Create a forward draft.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--to` | `` | Comma-separated recipients | — |
| `--comment` | `` | Inline comment prepended to the forwarded body | — |
| `--comment-file` | `` | Read --comment from a UTF-8 file (bypasses Windows ~32K command-line cap) | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |
| `--var` | `` | Template substitution — `--var k1=v1,k2=v2`. Substitutes `${key}` in body/comment after --body-file read. | — |

**Examples:**

```bash
its outlook drafts forward AAMkAGI1AAAt0M0AAA= --to jane.smith@example.com --comment "FYI" --user jane.smith@example.com
```

#### `its outlook drafts update <draft_id>`

Patch an existing draft. Any of subject/body/to/cc/bcc/importance/categories.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--subject` | `` | Replace subject | — |
| `--body` | `` | Replace body | — |
| `--body-file` | `` | Read replacement body from a UTF-8 file (use for bodies > ~15KB — Windows command-line cap) | — |
| `--html` | `` | Treat --body / --body-file as HTML | — |
| `--to` | `` | Replace To recipients (comma-separated) | — |
| `--cc` | `` | Replace CC recipients | — |
| `--bcc` | `` | Replace BCC recipients | — |
| `--importance` | `` | low|normal|high | — |
| `--categories` | `` | Replace categories (comma-separated) | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |
| `--var` | `` | Template substitution — `--var k1=v1,k2=v2`. Substitutes `${key}` in body/comment after --body-file read. | — |

**Examples:**

```bash
its outlook drafts update AAMkAGDraft01 --subject "Revised: laptop ready" --user jane.smith@example.com
```

#### `its outlook drafts send <draft_id>`

Send an existing draft.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
# Sends an existing draft — compose first with `drafts create`
its outlook drafts send AAMkAGDraft01 --user jane.smith@example.com
```

#### `its outlook drafts`

List draft messages (convenience for `mail list --folder drafts`).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Number of drafts (max 50) | 25 |
| `--skip` | `` | Skip first N (pagination) | 0 |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

```bash
its outlook drafts
```

#### `its outlook drafts delete <id>`

Delete a draft by message id. Refuses anything that is not a draft, so a mistyped id cannot delete a sent or received message through this command. Permanent — use --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Required to perform this destructive deletion | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
its outlook drafts delete AAMkAGI2... --confirm
```

---

### folders

> Source: `src/providers/outlook/commands/folders.ts`

| Command | Description |
|---------|-------------|
| `its outlook folders` | List mail folders with counts (totalItemCount, unreadItemCount). |
| `its outlook folders get <folder_id>` | Get a single mail folder by ID or well-known name (inbox, sentitems, drafts, deleteditems, archive). |
| `its outlook folders create <name>` | Create a new mail folder (optionally nested under a parent). |
| `its outlook folders delete <folder>` | Delete a mail folder by id or display name. Graph deletes the folder AND every message and child folder inside it — there is no move-contents-out option, and no undo. Use --confirm. |

#### `its outlook folders`

List mail folders with counts (totalItemCount, unreadItemCount).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Max folders (default 50, max 100) | 50 |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

```bash
its outlook folders
```

#### `its outlook folders get <folder_id>`

Get a single mail folder by ID or well-known name (inbox, sentitems, drafts, deleteditems, archive).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

```bash
its outlook folders get <folder_id>
```

#### `its outlook folders create <name>`

Create a new mail folder (optionally nested under a parent).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--parent` | `` | Parent folder ID (omit for top-level) | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
its outlook folders create "IT Archive" --user jane.smith@example.com

its outlook folders create "2026" --parent "IT Archive" --user jane.smith@example.com
```

#### `its outlook folders delete <folder>`

Delete a mail folder by id or display name. Graph deletes the folder AND every message and child folder inside it — there is no move-contents-out option, and no undo. Use --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Required to perform this destructive deletion | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
its outlook folders delete "IT Archive" --user jane.smith@example.com --confirm
```

---

### attachments

> Source: `src/providers/outlook/commands/attachments.ts`

| Command | Description |
|---------|-------------|
| `its outlook attachments <message_id>` | List attachments on a message. |
| `its outlook attachments get <message_id> [attachment_id]` | Get a single attachment (includes contentBytes for fileAttachment). Pass --save-all <dir> to dump every attachment on the message instead of one. |
| `its outlook attachments add <message_id>` | Attach a file to a draft message. Pass --file <path> to read from disk, or --content-bytes (base64) directly. |
| `its outlook attachments delete <message_id> [attachment_id]` | Delete attachment(s) from a message. Single: pass <message_id> <attachment_id>. Bulk: pass <message_id> and pipe `attachments list --json` to stdin with --stdin. |

#### `its outlook attachments <message_id>`

List attachments on a message.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

```bash
its outlook attachments <message_id>
```

#### `its outlook attachments get <message_id> [attachment_id]`

Get a single attachment (includes contentBytes for fileAttachment). Pass --save-all <dir> to dump every attachment on the message instead of one.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--save` | `` | Write contentBytes of the single attachment to this local path (decoded) | — |
| `--save-all` | `` | Save every attachment on the message into this directory (created if missing) | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
its outlook attachments get <message_id> <attachment_id> --save invoice.pdf

its outlook attachments get <message_id> --save-all ./out
```

#### `its outlook attachments add <message_id>`

Attach a file to a draft message. Pass --file <path> to read from disk, or --content-bytes (base64) directly.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--file` | `` | Local file path — read + base64-encoded | — |
| `--name` | `` | Override attachment display name (defaults to basename of --file) | — |
| `--content-type` | `` | MIME type (default application/octet-stream) | — |
| `--content-bytes` | `` | Base64-encoded file content (alternative to --file) | — |
| `--is-inline` | `` | Mark as inline attachment (renders in body via cid: ref, not as paperclip) | — |
| `--content-id` | `` | Content-ID for cid: targeting in HTML body (e.g. `cid:logo-1` ↔ --content-id logo-1) | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
# Pair with body HTML containing `<img src="cid:logo-1">`
its outlook attachments add <draft_id> --file logo.png --content-type image/png --is-inline --content-id logo-1
```

#### `its outlook attachments delete <message_id> [attachment_id]`

Delete attachment(s) from a message. Single: pass <message_id> <attachment_id>. Bulk: pass <message_id> and pipe `attachments list --json` to stdin with --stdin.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Required for bulk (--stdin) deletes | — |
| `--stdin` | `` | Read newline/JSON list of attachment ids from stdin | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
its outlook attachments delete <message_id> <attachment_id> --confirm

its outlook attachments <message_id> --json | its outlook attachments delete <message_id> --stdin --confirm
```

---

### events

> Source: `src/providers/outlook/commands/events.ts`

| Command | Description |
|---------|-------------|
| `its outlook events` | List calendar events between two dates (calendarView — includes expanded recurrences). |
| `its outlook events get <event_id>` | Get a single calendar event by ID. |
| `its outlook events create` | Create a calendar event. Times default to UTC unless --tz is set. |
| `its outlook events update <event_id>` | Update a calendar event (subject, times, location, all-day, online-meeting). |
| `its outlook events delete <event_id>` | Delete a calendar event. |
| `its outlook events respond <event_id> <response>` | Respond to a meeting invite — accept, decline, or tentatively accept. |
| `its outlook events availability <schedules>` | Check free/busy across one or more mailboxes between two times. |

#### `its outlook events`

List calendar events between two dates (calendarView — includes expanded recurrences).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--start` | `` | Start ISO date/datetime (default: today) | — |
| `--end` | `` | End ISO date/datetime (default: 7 days from start) | — |
| `--top` | `` | Max events (default 50, max 100) | 50 |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
its outlook events

its outlook events --start 2026-05-25 --end 2026-06-01
```

#### `its outlook events get <event_id>`

Get a single calendar event by ID.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

```bash
its outlook events get <event_id>
```

#### `its outlook events create`

Create a calendar event. Times default to UTC unless --tz is set.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--subject` | `` | Event subject | — |
| `--start` | `` | Start datetime (ISO, e.g. 2026-05-25T09:00:00) | — |
| `--end` | `` | End datetime (ISO) | — |
| `--tz` | `` | Time zone (default UTC) | — |
| `--location` | `` | Location display name | — |
| `--body` | `` | Body content | — |
| `--body-file` | `` | Read body from a UTF-8 file (use for bodies > ~15KB — Windows command-line cap) | — |
| `--html` | `` | Treat --body / --body-file as HTML | — |
| `--attendees` | `` | Comma-separated email addresses (required) | — |
| `--optional-attendees` | `` | Comma-separated optional attendees | — |
| `--all-day` | `` | Mark as all-day event | — |
| `--online` | `` | Create Teams meeting link | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |
| `--var` | `` | Template substitution — `--var k1=v1,k2=v2`. Substitutes `${key}` in body/comment after --body-file read. | — |

**Examples:**

```bash
its outlook events create --subject "IT catch-up" --start 2026-09-01T10:00 --end 2026-09-01T10:30 --tz Europe/London --attendees jane.smith@example.com --online --user jane.smith@example.com

its outlook events create --subject "Server migration" --start 2026-09-05 --all-day --user jane.smith@example.com
```

#### `its outlook events update <event_id>`

Update a calendar event (subject, times, location, all-day, online-meeting).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--subject` | `` | Replace subject | — |
| `--start` | `` | Replace start datetime (ISO) | — |
| `--end` | `` | Replace end datetime (ISO) | — |
| `--tz` | `` | Time zone for --start/--end (default UTC) | — |
| `--location` | `` | Replace location | — |
| `--body` | `` | Replace body | — |
| `--body-file` | `` | Read replacement body from a UTF-8 file (use for bodies > ~15KB — Windows command-line cap) | — |
| `--html` | `` | Treat --body / --body-file as HTML | — |
| `--all-day` | `` | Set isAllDay | — |
| `--online` | `` | Toggle isOnlineMeeting | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |
| `--var` | `` | Template substitution — `--var k1=v1,k2=v2`. Substitutes `${key}` in body/comment after --body-file read. | — |

**Examples:**

```bash
its outlook events update AAMkAGEvent01 --start 2026-09-01T11:00 --end 2026-09-01T11:30 --tz Europe/London --user jane.smith@example.com
```

#### `its outlook events delete <event_id>`

Delete a calendar event.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |
| `--confirm` | `` | Confirm deletion | — |

**Examples:**

```bash
# Cancels the event and notifies attendees if the user is the organiser
its outlook events delete AAMkAGI1... --user jane.smith@example.com --confirm

its outlook events list --user jane.smith@example.com
```

#### `its outlook events respond <event_id> <response>`

Respond to a meeting invite — accept, decline, or tentatively accept.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--comment` | `` | Reply comment to the organiser | — |
| `--comment-file` | `` | Read --comment from a UTF-8 file (bypasses Windows ~32K command-line cap) | — |
| `--no-response` | `` | Don't send a reply to the organiser | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |
| `--var` | `` | Template substitution — `--var k1=v1,k2=v2`. Substitutes `${key}` in body/comment after --body-file read. | — |

**Examples:**

```bash
its outlook events respond AAMkAGEvent01 accept --user jane.smith@example.com

its outlook events respond AAMkAGEvent01 decline --comment "On leave" --user jane.smith@example.com
```

#### `its outlook events availability <schedules>`

Check free/busy across one or more mailboxes between two times.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--start` | `` | Start ISO datetime | — |
| `--end` | `` | End ISO datetime | — |
| `--interval` | `` | View interval in minutes (default 30) | 30 |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
its outlook events availability "a@b.com,c@d.com" --start 2026-05-21T09:00:00Z --end 2026-05-21T17:00:00Z
```

---

### settings

> Source: `src/providers/outlook/commands/settings.ts`

| Command | Description |
|---------|-------------|
| `its outlook settings get` | Get full mailbox settings (time zone, locale, working hours, auto-reply state). |

#### `its outlook settings get`

Get full mailbox settings (time zone, locale, working hours, auto-reply state).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

```bash
its outlook settings get
```

---

### autoreply

> Source: `src/providers/outlook/commands/settings.ts`

| Command | Description |
|---------|-------------|
| `its outlook autoreply get` | Get current automatic reply (out-of-office) settings. |
| `its outlook autoreply set` | Configure automatic reply. Use --status to toggle disabled/alwaysEnabled/scheduled. For scheduled, pass --start and --end. |

#### `its outlook autoreply get`

Get current automatic reply (out-of-office) settings.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

```bash
its outlook autoreply get
```

#### `its outlook autoreply set`

Configure automatic reply. Use --status to toggle disabled/alwaysEnabled/scheduled. For scheduled, pass --start and --end.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--status` | `` | disabled | alwaysEnabled | scheduled | — |
| `--audience` | `` | External audience: none | contactsOnly | all | — |
| `--internal` | `` | Internal reply message | — |
| `--external` | `` | External reply message | — |
| `--start` | `` | Scheduled start (ISO datetime) | — |
| `--end` | `` | Scheduled end (ISO datetime) | — |
| `--tz` | `` | Time zone for --start/--end (default UTC) | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
its outlook autoreply set --status disabled

its outlook autoreply set --status scheduled --start 2026-05-25T09:00:00 --end 2026-05-30T17:00:00 --tz "GMT Standard Time" --internal "Out of office, back Mon." --external "Out of office."
```

---

### categories

> Source: `src/providers/outlook/commands/settings.ts`

| Command | Description |
|---------|-------------|
| `its outlook categories` | List master categories (named colour labels available for `mail categorise`). |

#### `its outlook categories`

List master categories (named colour labels available for `mail categorise`).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

```bash
its outlook categories
```

---

### rules

> Source: `src/providers/outlook/commands/rules.ts`

| Command | Description |
|---------|-------------|
| `its outlook rules` | List inbox message rules. |
| `its outlook rules create` | Create an inbox rule. Conditions + actions take raw JSON (Graph schema). See https://learn.microsoft.com/graph/api/resources/messagerule for shape. |
| `its outlook rules delete <rule_id>` | Delete an inbox rule. |

#### `its outlook rules`

List inbox message rules.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

```bash
its outlook rules
```

#### `its outlook rules create`

Create an inbox rule. Conditions + actions take raw JSON (Graph schema). See https://learn.microsoft.com/graph/api/resources/messagerule for shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | Display name | — |
| `--sequence` | `` | Run-order (lower runs first) | 100 |
| `--disabled` | `` | Create in disabled state | — |
| `--conditions` | `` | Conditions JSON (e.g. '{"fromAddresses":[{"emailAddress":{"address":"x@y"}}]}') | — |
| `--actions` | `` | Actions JSON (e.g. '{"moveToFolder":"<folderId>"}') | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
# Conditions and actions take raw Graph JSON
its outlook rules create --name "IT to folder" --conditions '{"senderContains":["helpdesk"]}' --actions '{"moveToFolder":"AQMkAGI1IT"}' --user jane.smith@example.com
```

#### `its outlook rules delete <rule_id>`

Delete an inbox rule.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |
| `--confirm` | `` | Confirm deletion | — |

**Examples:**

```bash
its outlook rules delete AQAAAL7z... --user jane.smith@example.com --confirm

its outlook rules list --user jane.smith@example.com
```

---

### contacts

> Source: `src/providers/outlook/commands/contacts.ts`

| Command | Description |
|---------|-------------|
| `its outlook contacts search <query>` | Search the /people graph for matching contacts and frequent collaborators. |

#### `its outlook contacts search <query>`

Search the /people graph for matching contacts and frequent collaborators.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Max results (default 10, max 25) | 10 |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

```bash
its outlook contacts search <query>
```

---

### triage

> Source: `src/providers/outlook/commands/triage.ts`

| Command | Description |
|---------|-------------|
| `its outlook triage` | Triage the unread inbox — buckets messages into ACTION REQUIRED / FYI / NOISE with a one-line recommendation. Read-only; no mutations. |

#### `its outlook triage`

Triage the unread inbox — buckets messages into ACTION REQUIRED / FYI / NOISE with a one-line recommendation. Read-only; no mutations.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Max unread messages to scan (default 25, max 50) | 25 |
| `--include-read` | `` | Also include read messages (default: unread only) | — |
| `--user` | `` | Override mailbox UPN (app-only auth). Default: OUTLOOK_DEFAULT_USER or /me. | — |

**Examples:**

```bash
its outlook triage

its outlook triage --top 50
```

---
