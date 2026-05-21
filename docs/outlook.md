# Outlook (`outlook`)

Microsoft Outlook (Graph) mailbox + calendar — list/search/send mail, manage drafts, organise folders, schedule events, check free/busy, configure auto-reply and inbox rules.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md)

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
| `its outlook mail search <query>` | Keyword search across the mailbox using Graph $search (KQL syntax). |
| `its outlook mail thread <conversation_id>` | List every message in the same conversation (entire thread). |
| `its outlook mail move <message_id> <folder_id>` | Move a message to another folder. |
| `its outlook mail copy <message_id> <folder_id>` | Copy a message to another folder. |
| `its outlook mail read <message_id>` | Mark a message as read. |
| `its outlook mail unread <message_id>` | Mark a message as unread. |
| `its outlook mail flag <message_id>` | Set follow-up flag status on a message. |
| `its outlook mail categorise <message_id> <categories>` | Set the category list on a message (replaces existing categories). |
| `its outlook mail delete <message_id>` | Delete a message (moves to Deleted Items). |
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

```bash
its outlook mail get <message_id>
```

#### `its outlook mail search <query>`

Keyword search across the mailbox using Graph $search (KQL syntax).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Max results (max 50) | 25 |

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

```bash
its outlook mail thread <conversation_id>
```

#### `its outlook mail move <message_id> <folder_id>`

Move a message to another folder.

```bash
its outlook mail move <message_id> <folder_id>
```

#### `its outlook mail copy <message_id> <folder_id>`

Copy a message to another folder.

```bash
its outlook mail copy <message_id> <folder_id>
```

#### `its outlook mail read <message_id>`

Mark a message as read.

```bash
its outlook mail read <message_id>
```

#### `its outlook mail unread <message_id>`

Mark a message as unread.

```bash
its outlook mail unread <message_id>
```

#### `its outlook mail flag <message_id>`

Set follow-up flag status on a message.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--status` | `` | Flag status | flagged |

```bash
its outlook mail flag <message_id>
```

#### `its outlook mail categorise <message_id> <categories>`

Set the category list on a message (replaces existing categories).

```bash
its outlook mail categorise <message_id> <categories>
```

#### `its outlook mail delete <message_id>`

Delete a message (moves to Deleted Items).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Skip safety prompt | — |

```bash
its outlook mail delete <message_id>
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
| `--html` | `` | Treat --body as HTML (default text) | — |
| `--importance` | `` | low|normal|high | normal |

**Examples:**

```bash
its outlook mail send --to user@example.com --subject "Hi" --body "Quick note"
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
| `--html` | `` | Treat --body as HTML (default text) | — |
| `--importance` | `` | low|normal|high | normal |

```bash
its outlook drafts create
```

#### `its outlook drafts reply <message_id>`

Create a reply draft. By default replies to the sender only; use --all to Reply-All. Does not send — use `drafts send <id>` after edits.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--comment` | `` | Inline comment prepended to the reply body | — |
| `--all` | `` | Reply to all recipients | — |

```bash
its outlook drafts reply <message_id>
```

#### `its outlook drafts forward <message_id>`

Create a forward draft.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--to` | `` | Comma-separated recipients | — |
| `--comment` | `` | Inline comment prepended to the forwarded body | — |

```bash
its outlook drafts forward <message_id>
```

#### `its outlook drafts update <draft_id>`

Patch an existing draft. Any of subject/body/to/cc/bcc/importance/categories.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--subject` | `` | Replace subject | — |
| `--body` | `` | Replace body | — |
| `--html` | `` | Treat --body as HTML | — |
| `--to` | `` | Replace To recipients (comma-separated) | — |
| `--cc` | `` | Replace CC recipients | — |
| `--bcc` | `` | Replace BCC recipients | — |
| `--importance` | `` | low|normal|high | — |
| `--categories` | `` | Replace categories (comma-separated) | — |

```bash
its outlook drafts update <draft_id>
```

#### `its outlook drafts send <draft_id>`

Send an existing draft.

```bash
its outlook drafts send <draft_id>
```

---

### folders

> Source: `src/providers/outlook/commands/folders.ts`

| Command | Description |
|---------|-------------|
| `its outlook folders` | List mail folders with counts (totalItemCount, unreadItemCount). |
| `its outlook folders get <folder_id>` | Get a single mail folder by ID or well-known name (inbox, sentitems, drafts, deleteditems, archive). |
| `its outlook folders create <name>` | Create a new mail folder (optionally nested under a parent). |

#### `its outlook folders`

List mail folders with counts (totalItemCount, unreadItemCount).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Max folders (default 50, max 100) | 50 |

```bash
its outlook folders
```

#### `its outlook folders get <folder_id>`

Get a single mail folder by ID or well-known name (inbox, sentitems, drafts, deleteditems, archive).

```bash
its outlook folders get <folder_id>
```

#### `its outlook folders create <name>`

Create a new mail folder (optionally nested under a parent).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--parent` | `` | Parent folder ID (omit for top-level) | — |

```bash
its outlook folders create <name>
```

---

### attachments

> Source: `src/providers/outlook/commands/attachments.ts`

| Command | Description |
|---------|-------------|
| `its outlook attachments <message_id>` | List attachments on a message. |
| `its outlook attachments get <message_id> <attachment_id>` | Get a single attachment (includes contentBytes for fileAttachment). |
| `its outlook attachments add <message_id>` | Attach a file to a draft message. Pass --file <path> to read from disk, or --content-bytes (base64) directly. |

#### `its outlook attachments <message_id>`

List attachments on a message.

```bash
its outlook attachments <message_id>
```

#### `its outlook attachments get <message_id> <attachment_id>`

Get a single attachment (includes contentBytes for fileAttachment).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--save` | `` | Write contentBytes to this local path (decoded) | — |

```bash
its outlook attachments get <message_id> <attachment_id>
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

**Examples:**

```bash
# Pair with body HTML containing `<img src="cid:logo-1">`
its outlook attachments add <draft_id> --file logo.png --content-type image/png --is-inline --content-id logo-1
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

**Examples:**

```bash
its outlook events

its outlook events --start 2026-05-25 --end 2026-06-01
```

#### `its outlook events get <event_id>`

Get a single calendar event by ID.

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
| `--html` | `` | Treat --body as HTML | — |
| `--attendees` | `` | Comma-separated email addresses (required) | — |
| `--optional-attendees` | `` | Comma-separated optional attendees | — |
| `--all-day` | `` | Mark as all-day event | — |
| `--online` | `` | Create Teams meeting link | — |

```bash
its outlook events create
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
| `--html` | `` | Treat --body as HTML | — |
| `--all-day` | `` | Set isAllDay | — |
| `--online` | `` | Toggle isOnlineMeeting | — |

```bash
its outlook events update <event_id>
```

#### `its outlook events delete <event_id>`

Delete a calendar event.

```bash
its outlook events delete <event_id>
```

#### `its outlook events respond <event_id> <response>`

Respond to a meeting invite — accept, decline, or tentatively accept.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--comment` | `` | Reply comment to the organiser | — |
| `--no-response` | `` | Don't send a reply to the organiser | — |

```bash
its outlook events respond <event_id> <response>
```

#### `its outlook events availability <schedules>`

Check free/busy across one or more mailboxes between two times.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--start` | `` | Start ISO datetime | — |
| `--end` | `` | End ISO datetime | — |
| `--interval` | `` | View interval in minutes (default 30) | 30 |

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

```bash
its outlook rules create
```

#### `its outlook rules delete <rule_id>`

Delete an inbox rule.

```bash
its outlook rules delete <rule_id>
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

```bash
its outlook contacts search <query>
```

---
