# Wrike (`wrike`)

Wrike project management — IT tickets, tasks, contacts, spaces, folders, workflows, onboarding, leavers.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [tickets](#tickets)
- [tasks](#tasks)
- [projects](#projects)
- [contacts](#contacts)
- [groups](#groups)
- [audit](#audit)
- [access](#access)
- [forms](#forms)
- [spaces](#spaces)
- [folders](#folders)
- [workflows](#workflows)
- [custom-fields](#custom-fields)
- [item-types](#item-types)
- [onboarding](#onboarding)
- [leavers](#leavers)
- [dashboard](#dashboard)
- [auth](#auth)

## Setup

```bash
its wrike setup           # Interactive wizard
its wrike setup --check   # Check configuration status
its wrike setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `WRIKE_CLIENT_ID` | Wrike OAuth app client id — enables `its wrike auth login` (redirect URI http://localhost:8730) |
| `WRIKE_CLIENT_SECRET` | Wrike OAuth app client secret |
| `WRIKE_USER_ID` | Your immutable Wrike contact ID — resolves "you" for `tickets mine` and `tickets narrative --mine`. Falls back to the signed-in contact; required for service/robot tokens |
| `WRIKE_TOKEN` | Wrike permanent access token (Apps & Integrations > API) — fallback if not using OAuth |

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/wrike/client.ts` | API client methods |
| `src/providers/wrike/types.ts` | TypeScript interfaces |
| `src/providers/wrike/commands/` | Command definitions (split by resource) |
| `src/providers/wrike/access-audit.ts` | access audit |
| `src/providers/wrike/auth.ts` | auth |
| `src/providers/wrike/definition.ts` | definition |
| `src/providers/wrike/identity.ts` | identity |

## Resources

### tickets

> Source: `src/providers/wrike/commands/tickets.ts`

| Command | Description |
|---------|-------------|
| `its wrike tickets` | List IT support tickets. Surfaces the most common fields; pass --json for raw shape. |
| `its wrike tickets stats` | Ticket analytics — median resolve time, backlog, bus-factor risk by assignee |
| `its wrike tickets active` | List active IT tickets — those not in a completed/cancelled state. |
| `its wrike tickets mine` | List tickets assigned to you (--user-id, else WRIKE_USER_ID, else the signed-in Wrike contact) |
| `its wrike tickets get <idOrPermalink>` | Get full ticket details with comments. Pass the id (or any natural identifier) as the positional arg. |
| `its wrike tickets audit <idOrPermalink>` | Who created, assigned and unassigned a ticket — from the Wrike Enterprise audit log. Accepts a task ID, a permalink, or a bare numeric permalink ID. |
| `its wrike tickets search <query>` | Search IT/BC support tickets by title. Substring match across the most relevant fields; case-insensitive. |
| `its wrike tickets create` | Create a new IT support ticket. Idempotent on duplicate names — use update/edit to mutate an existing record. |
| `its wrike tickets set-due <taskId> <dueDate>` | Set due date on a ticket (YYYY-MM-DD). Set the task's due date. |
| `its wrike tickets assign <taskId> <userId>` | Add an assignee to a ticket. Idempotent — assigning twice is a no-op. |
| `its wrike tickets update-title <taskId> <title>` | Change a ticket's title. Rename the task — keeps the same ID. |
| `its wrike tickets update-importance <taskId> <importance>` | Change ticket importance (High, Normal, Low). Set Low / Normal / High importance. |
| `its wrike tickets update-status <taskId> <status>` | Change ticket status. Move the task to a different workflow stage. |
| `its wrike tickets update-description <taskId> [text]` | Replace a ticket's description. Same formatting rules as add-comment: plain text by default (\n → <br>, @Name auto-mentions), --markdown for bold/italic/links/bullets, --html for raw. Wrike renders <br>, <a>, <b>, <i>, and mention anchors only. |
| `its wrike tickets add-comment <taskId> [text]` | Add a comment to a ticket. Prefer --markdown (bold/italic/links/bullets + plain @Name auto-resolves to mentions). Plain text also works (\n → <br>, @Name auto-resolved). Use --html only when you need raw anchor-based mentions; the mention markup guard (ctxc 258) rejects mismatched modes unless --skip-mention-check is set. Wrike only renders <br>, <a>, <b>, <i>, and mention anchors. |
| `its wrike tickets update-comment <commentId> [text]` | Replace the body of an existing comment. Same formatting rules as add-comment (--markdown / --html / plain text). Use the commentId returned by add-comment, or read it from `tickets get`. |
| `its wrike tickets delete-comment <commentId>` | Delete a comment by ID. Requires --confirm. Use the commentId from add-comment output or `tickets get`. |
| `its wrike tickets narrative [taskId]` | Deep per-ticket narrative with recent comments + ball-is-with heuristic. Prioritises active chatter > waiting-on-them > waiting-on-us > ancient. Pass a single taskId, or --active / --mine for a batch view. |
| `its wrike tickets attachments <taskId>` | List attachments on a ticket. List attachments; pair with `attach`. |
| `its wrike tickets download <taskId> [attachmentId]` | Download an attachment from a ticket. Stream the resource to a local file. |
| `its wrike tickets attach <taskId> <filePath>` | Attach a file to a ticket. Upload a local file as an attachment. |

#### `its wrike tickets`

List IT support tickets. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--status` | `-s` | Filter by status | — |
| `--priority` | `-p` | Filter by priority/importance | — |
| `--limit` | `-l` | Maximum number of tickets | 50 |

**Examples:**

```bash
its wrike tickets

its wrike tickets --priority High

# Re-runs every 10s — handy for dashboards or incident response.
its wrike tickets --watch
```

#### `its wrike tickets stats`

Ticket analytics — median resolve time, backlog, bus-factor risk by assignee.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--since` | `` | Only include tickets created in the last N days (default 90) | 90 |

**Examples:**

```bash
its wrike tickets stats
```

#### `its wrike tickets active`

List active IT tickets — those not in a completed/cancelled state.

**Examples:**

```bash
its wrike tickets active
```

#### `its wrike tickets mine`

List tickets assigned to you (--user-id, else WRIKE_USER_ID, else the signed-in Wrike contact).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--status` | `` | Filter by status (default: Active) | Active |
| `--user-id` | `` | Wrike contact ID to treat as 'you' (overrides WRIKE_USER_ID) | — |

**Examples:**

```bash
# Assignments are matched on your immutable Wrike contact ID, not your email
its wrike tickets mine

# Another person's queue — find their contact ID with `its wrike contacts list`
its wrike tickets mine --user-id KUAXXXXX
```

#### `its wrike tickets get <idOrPermalink>`

Get full ticket details with comments. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
# Full ticket with comments + attachments
its wrike tickets get <task-id>
```

#### `its wrike tickets audit <idOrPermalink>`

Who created, assigned and unassigned a ticket — from the Wrike Enterprise audit log. Accepts a task ID, a permalink, or a bare numeric permalink ID.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--days` | `` | Days of audit history to scan (default: back to ticket creation) | — |

**Examples:**

```bash
# Creation + assignment history — needs Wrike Enterprise and the 'Create user activity reports' right
its wrike tickets audit 4541454107

# Scan only the last 7 days instead of everything since the ticket was created
its wrike tickets audit MAAAAAEOsRcb --days 7
```

#### `its wrike tickets search <query>`

Search IT/BC support tickets by title. Substring match across the most relevant fields; case-insensitive.

**Examples:**

```bash
its wrike tickets search "printer"
```

#### `its wrike tickets create`

Create a new IT support ticket. Idempotent on duplicate names — use update/edit to mutate an existing record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--title` | `-t` | Ticket title (required) | — |
| `--description` | `-d` | Ticket description (inline plain text) | — |
| `--description-file` | `` | Read description body from a file path | — |
| `--description-stdin` | `` | Read description body from stdin | — |
| `--markdown` | `` | Treat description as Markdown (converts to Wrike-safe HTML). Mutually exclusive with --html. | — |
| `--html` | `` | Description is raw Wrike-safe HTML. Mutually exclusive with --markdown. | — |
| `--assignee` | `-a` | Comma-separated user IDs or names (e.g. 'KUAABCDE,Jane Smith') | — |
| `--due` | `` | Due date (YYYY-MM-DD) | — |
| `--importance` | `` | Importance (High, Normal, Low) | Normal |

**Examples:**

```bash
its wrike tickets create --title "Printer offline in Finance" --description "Label printer not responding"

its wrike tickets create --title "Server migration plan" --description-file ./plan.md --markdown --importance High

its wrike tickets create --title "Outlook crashes" --description "Repro: open shared mailbox"
```

#### `its wrike tickets set-due <taskId> <dueDate>`

Set due date on a ticket (YYYY-MM-DD). Set the task's due date.

**Examples:**

```bash
its wrike tickets set-due IEACW7BXKQ2R4KMT 2026-09-01

its wrike tickets set-due <task-id> 2026-06-01
```

#### `its wrike tickets assign <taskId> <userId>`

Add an assignee to a ticket. Idempotent — assigning twice is a no-op.

**Examples:**

```bash
# Find the user ID with `its wrike users list`
its wrike tickets assign IEACW7BXKQ2R4KMT KUAEXAMP

its wrike tickets assign <task-id> <user-id>
```

#### `its wrike tickets update-title <taskId> <title>`

Change a ticket's title. Rename the task — keeps the same ID.

**Examples:**

```bash
its wrike tickets update-title IEACW7BXKQ2R4KMT "Printer offline — Finance, resolved"

its wrike tickets update-title <task-id> "New title"
```

#### `its wrike tickets update-importance <taskId> <importance>`

Change ticket importance (High, Normal, Low). Set Low / Normal / High importance.

**Examples:**

```bash
its wrike tickets update-importance IEACW7BXKQ2R4KMT High

its wrike tickets update-importance <task-id> High
```

#### `its wrike tickets update-status <taskId> <status>`

Change ticket status. Move the task to a different workflow stage.

**Examples:**

```bash
its wrike tickets update-status IEACW7BXKQ2R4KMT Completed

its wrike tickets update-status <task-id> "Completed"
```

#### `its wrike tickets update-description <taskId> [text]`

Replace a ticket's description. Same formatting rules as add-comment: plain text by default (\n → <br>, @Name auto-mentions), --markdown for bold/italic/links/bullets, --html for raw. Wrike renders <br>, <a>, <b>, <i>, and mention anchors only.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--file` | `` | Read comment body from a file path | — |
| `--stdin` | `` | Read comment body from stdin | — |
| `--html` | `` | Send body as raw HTML. Wrike only renders <br>, <a>, <b>, <i>, and mention anchors — other tags are ignored. Use --markdown for richer formatting. | — |
| `--markdown` | `` | Convert Markdown to Wrike-safe HTML. Supported: **bold**, _italic_, `code` (rendered italic), # headings (rendered bold), [text](url), - bullets (flattened to •). In this mode, mention users with plain @Name (auto-resolves). Mutually exclusive with --html. Recommended over --html. | — |
| `--skip-mention-check` | `` | Bypass the @mention markup guard (ctxc 258). Use only when posting a literal string that happens to contain mention-like markup. | — |

**Examples:**

```bash
its wrike tickets update-description IEACW7BXKQ2R4KMT "**Root cause:** stale spooler" --markdown

cat notes.md | its wrike tickets update-description IEACW7BXKQ2R4KMT --stdin --markdown

its wrike tickets update-description <task-id> --markdown "**Repro:** open Outlook → file → ..."
```

#### `its wrike tickets add-comment <taskId> [text]`

Add a comment to a ticket. Prefer --markdown (bold/italic/links/bullets + plain @Name auto-resolves to mentions). Plain text also works (\n → <br>, @Name auto-resolved). Use --html only when you need raw anchor-based mentions; the mention markup guard (ctxc 258) rejects mismatched modes unless --skip-mention-check is set. Wrike only renders <br>, <a>, <b>, <i>, and mention anchors.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--file` | `` | Read comment body from a file path | — |
| `--stdin` | `` | Read comment body from stdin | — |
| `--html` | `` | Send body as raw HTML. Wrike only renders <br>, <a>, <b>, <i>, and mention anchors — other tags are ignored. Use --markdown for richer formatting. | — |
| `--markdown` | `` | Convert Markdown to Wrike-safe HTML. Supported: **bold**, _italic_, `code` (rendered italic), # headings (rendered bold), [text](url), - bullets (flattened to •). In this mode, mention users with plain @Name (auto-resolves). Mutually exclusive with --html. Recommended over --html. | — |
| `--skip-mention-check` | `` | Bypass the @mention markup guard (ctxc 258). Use only when posting a literal string that happens to contain mention-like markup. | — |

**Examples:**

```bash
its wrike tickets add-comment IEACW7BXKQ2R4KMT "Fixed — restarted spooler on WKS-9" --markdown

its wrike tickets add-comment IEACW7BXKQ2R4KMT --file ./update.md --markdown

its wrike tickets add-comment <task-id> "Replicated, escalating"
```

#### `its wrike tickets update-comment <commentId> [text]`

Replace the body of an existing comment. Same formatting rules as add-comment (--markdown / --html / plain text). Use the commentId returned by add-comment, or read it from `tickets get`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--file` | `` | Read comment body from a file path | — |
| `--stdin` | `` | Read comment body from stdin | — |
| `--html` | `` | Send body as raw HTML. Wrike only renders <br>, <a>, <b>, <i>, and mention anchors — other tags are ignored. Use --markdown for richer formatting. | — |
| `--markdown` | `` | Convert Markdown to Wrike-safe HTML. Supported: **bold**, _italic_, `code` (rendered italic), # headings (rendered bold), [text](url), - bullets (flattened to •). In this mode, mention users with plain @Name (auto-resolves). Mutually exclusive with --html. Recommended over --html. | — |
| `--skip-mention-check` | `` | Bypass the @mention markup guard (ctxc 258). Use only when posting a literal string that happens to contain mention-like markup. | — |

**Examples:**

```bash
its wrike tickets update-comment IEACW7BXIMBB3B7F "Corrected: it was the driver, not the spooler" --markdown

its wrike tickets update-comment <comment-id> "fixed typo"
```

#### `its wrike tickets delete-comment <commentId>`

Delete a comment by ID. Requires --confirm. Use the commentId from add-comment output or `tickets get`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm deletion (irreversible) | — |

**Examples:**

```bash
its wrike tickets delete-comment IEACW7BXIMBB3B7F --confirm

its wrike tickets delete-comment <comment-id> --confirm
```

#### `its wrike tickets narrative [taskId]`

Deep per-ticket narrative with recent comments + ball-is-with heuristic. Prioritises active chatter > waiting-on-them > waiting-on-us > ancient. Pass a single taskId, or --active / --mine for a batch view.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--active` | `` | All active IT tickets (ignores --mine) | — |
| `--mine` | `` | Only tickets assigned to you (via --user-id, WRIKE_USER_ID, or the signed-in contact) | — |
| `--user-id` | `` | Wrike contact ID to treat as 'you' (overrides WRIKE_USER_ID) | — |
| `--comments` | `` | Comments to show per ticket (default 3) | 3 |
| `--limit` | `` | Maximum tickets to process for batch modes (default 25) | 25 |

**Examples:**

```bash
# AI-friendly chronological narrative
its wrike tickets narrative <task-id>
```

#### `its wrike tickets attachments <taskId>`

List attachments on a ticket. List attachments; pair with `attach`.

**Examples:**

```bash
its wrike tickets attachments <task-id>
```

#### `its wrike tickets download <taskId> [attachmentId]`

Download an attachment from a ticket. Stream the resource to a local file.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--output` | `-o` | Output file path (defaults to original filename in cwd) | — |

**Examples:**

```bash
its wrike tickets download IEACW7BXKQ2R4KMT --output ./ticket-files

its wrike tickets download IEACW7BXKQ2R4KMT IEAExampleAtt --output ./screenshot.png

its wrike tickets download <task-id> <attachment-id> --output ./screenshot.png
```

#### `its wrike tickets attach <taskId> <filePath>`

Attach a file to a ticket. Upload a local file as an attachment.

**Examples:**

```bash
its wrike tickets attach IEACW7BXKQ2R4KMT ./screenshot.png

its wrike tickets attach <task-id> ./screenshot.png
```

---

### tasks

> Source: `src/providers/wrike/commands/tasks.ts`

| Command | Description |
|---------|-------------|
| `its wrike tasks <folder>` | List tasks in a folder or project (accepts name or ID). Surfaces the most common fields; pass --json for raw shape. |
| `its wrike tasks search <query>` | Search tasks across all of Wrike (use --folder or --space to scope, accepts names) |
| `its wrike tasks get <idOrPermalink>` | Get full task details with comments. Pass the id (or any natural identifier) as the positional arg. |
| `its wrike tasks create <folder>` | Create a new task in a folder or project (accepts name or ID) |
| `its wrike tasks set-due <taskId> <dueDate>` | Set due date on a task (YYYY-MM-DD). Set the task's due date. |
| `its wrike tasks update-title <taskId> <title>` | Change a task's title. Rename the task — keeps the same ID. |
| `its wrike tasks update-importance <taskId> <importance>` | Change task importance (High, Normal, Low). Set Low / Normal / High importance. |
| `its wrike tasks update-status <taskId> <status>` | Change task status. Move the task to a different workflow stage. |
| `its wrike tasks update-description <taskId> [text]` | Replace a task's description. Same formatting rules as add-comment: plain text by default (\n → <br>, @Name auto-mentions), --markdown for bold/italic/links/bullets, --html for raw. Wrike renders <br>, <a>, <b>, <i>, and mention anchors only. |
| `its wrike tasks add-comment <taskId> [text]` | Add a comment to a task. Prefer --markdown (bold/italic/links/bullets + plain @Name auto-resolves to mentions). Plain text also works (\n → <br>, @Name auto-resolved). Use --html only when you need raw anchor-based mentions; the mention markup guard (ctxc 258) rejects mismatched modes unless --skip-mention-check is set. Wrike only renders <br>, <a>, <b>, <i>, and mention anchors. |
| `its wrike tasks attach <taskId> <filePath>` | Attach a file to a task. Upload a local file as an attachment. |

#### `its wrike tasks <folder>`

List tasks in a folder or project (accepts name or ID). Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--status` | `-s` | Filter by status | — |
| `--limit` | `-l` | Maximum number of tasks | 100 |

**Examples:**

```bash
its wrike tasks

# Re-runs every 10s — handy for dashboards or incident response.
its wrike tasks --watch
```

#### `its wrike tasks search <query>`

Search tasks across all of Wrike (use --folder or --space to scope, accepts names).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--folder` | `` | Scope to a folder/project (name or ID) | — |
| `--space` | `` | Scope to a space (name or ID) | — |

**Examples:**

```bash
its wrike tasks search "deploy"
```

#### `its wrike tasks get <idOrPermalink>`

Get full task details with comments. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its wrike tasks get https://www.wrike.com/open.htm?id=...
```

#### `its wrike tasks create <folder>`

Create a new task in a folder or project (accepts name or ID).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--title` | `-t` | Task title | — |
| `--description` | `-d` | Task description | — |
| `--description-file` | `` | Read description from a UTF-8 file (use for descriptions > ~15KB — Windows command-line cap) | — |
| `--status` | `-s` | Task status | — |
| `--importance` | `` | Task importance (High, Normal, Low) | — |

**Examples:**

```bash
its wrike tasks create "IT Projects" --title "Roll out FIDO2 keys" --importance High

its wrike tasks create "My folder" --title "New task"
```

#### `its wrike tasks set-due <taskId> <dueDate>`

Set due date on a task (YYYY-MM-DD). Set the task's due date.

**Examples:**

```bash
its wrike tasks set-due IEACW7BXKQ2R4KMT 2026-09-01

its wrike tasks set-due <task-id> 2026-06-01
```

#### `its wrike tasks update-title <taskId> <title>`

Change a task's title. Rename the task — keeps the same ID.

**Examples:**

```bash
its wrike tasks update-title IEACW7BXKQ2R4KMT "Roll out FIDO2 keys — phase 2"

its wrike tasks update-title <task-id> "New title"
```

#### `its wrike tasks update-importance <taskId> <importance>`

Change task importance (High, Normal, Low). Set Low / Normal / High importance.

**Examples:**

```bash
its wrike tasks update-importance IEACW7BXKQ2R4KMT Low

its wrike tasks update-importance <task-id> Low
```

#### `its wrike tasks update-status <taskId> <status>`

Change task status. Move the task to a different workflow stage.

**Examples:**

```bash
its wrike tasks update-status IEACW7BXKQ2R4KMT Completed

its wrike tasks update-status <task-id> "In Progress"
```

#### `its wrike tasks update-description <taskId> [text]`

Replace a task's description. Same formatting rules as add-comment: plain text by default (\n → <br>, @Name auto-mentions), --markdown for bold/italic/links/bullets, --html for raw. Wrike renders <br>, <a>, <b>, <i>, and mention anchors only.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--file` | `` | Read comment body from a file path | — |
| `--stdin` | `` | Read comment body from stdin | — |
| `--html` | `` | Send body as raw HTML. Wrike only renders <br>, <a>, <b>, <i>, and mention anchors — other tags are ignored. Use --markdown for richer formatting. | — |
| `--markdown` | `` | Convert Markdown to Wrike-safe HTML. Supported: **bold**, _italic_, `code` (rendered italic), # headings (rendered bold), [text](url), - bullets (flattened to •). In this mode, mention users with plain @Name (auto-resolves). Mutually exclusive with --html. Recommended over --html. | — |
| `--skip-mention-check` | `` | Bypass the @mention markup guard (ctxc 258). Use only when posting a literal string that happens to contain mention-like markup. | — |

**Examples:**

```bash
its wrike tasks update-description IEACW7BXKQ2R4KMT "## Plan\n- Order keys" --markdown

its wrike tasks update-description <task-id> --markdown "## Plan\n- step 1"
```

#### `its wrike tasks add-comment <taskId> [text]`

Add a comment to a task. Prefer --markdown (bold/italic/links/bullets + plain @Name auto-resolves to mentions). Plain text also works (\n → <br>, @Name auto-resolved). Use --html only when you need raw anchor-based mentions; the mention markup guard (ctxc 258) rejects mismatched modes unless --skip-mention-check is set. Wrike only renders <br>, <a>, <b>, <i>, and mention anchors.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--file` | `` | Read comment body from a file path | — |
| `--stdin` | `` | Read comment body from stdin | — |
| `--html` | `` | Send body as raw HTML. Wrike only renders <br>, <a>, <b>, <i>, and mention anchors — other tags are ignored. Use --markdown for richer formatting. | — |
| `--markdown` | `` | Convert Markdown to Wrike-safe HTML. Supported: **bold**, _italic_, `code` (rendered italic), # headings (rendered bold), [text](url), - bullets (flattened to •). In this mode, mention users with plain @Name (auto-resolves). Mutually exclusive with --html. Recommended over --html. | — |
| `--skip-mention-check` | `` | Bypass the @mention markup guard (ctxc 258). Use only when posting a literal string that happens to contain mention-like markup. | — |

**Examples:**

```bash
its wrike tasks add-comment IEACW7BXKQ2R4KMT "Keys ordered, ETA Friday" --markdown

its wrike tasks add-comment <task-id> --markdown "**done** — pushed to main"
```

#### `its wrike tasks attach <taskId> <filePath>`

Attach a file to a task. Upload a local file as an attachment.

**Examples:**

```bash
its wrike tasks attach IEACW7BXKQ2R4KMT ./quote.pdf

its wrike tasks attach <task-id> ./diagram.png
```

---

### projects

> Source: `src/providers/wrike/commands/tasks.ts`

| Command | Description |
|---------|-------------|
| `its wrike projects` | List Wrike projects (optionally within a space — accepts name or ID) |
| `its wrike projects search <query>` | Search Wrike projects by title (accepts space name or ID). Substring match across the most relevant fields; case-insensitive. |
| `its wrike projects tasks <project>` | List tasks in a project (accepts project name or ID) |

#### `its wrike projects`

List Wrike projects (optionally within a space — accepts name or ID).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--space` | `` | Filter to a space (name or ID) | — |

**Examples:**

```bash
its wrike projects

its wrike projects "IT Operations"

# Re-runs every 10s — handy for dashboards or incident response.
its wrike projects --watch
```

#### `its wrike projects search <query>`

Search Wrike projects by title (accepts space name or ID). Substring match across the most relevant fields; case-insensitive.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--space` | `` | Scope to a space (name or ID) | — |

**Examples:**

```bash
its wrike projects search "migration"
```

#### `its wrike projects tasks <project>`

List tasks in a project (accepts project name or ID).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--status` | `-s` | Filter by task status | — |
| `--limit` | `-l` | Maximum number of tasks | 100 |

**Examples:**

```bash
its wrike projects tasks "Sprint 23"
```

---

### contacts

> Source: `src/providers/wrike/commands/contacts.ts`

| Command | Description |
|---------|-------------|
| `its wrike contacts` | List all Wrike contacts. Surfaces the most common fields; pass --json for raw shape. |
| `its wrike contacts search <query>` | Search contacts by name or email. Substring match across the most relevant fields; case-insensitive. |
| `its wrike contacts find [query]` | Find contacts by name and/or email domain (for populating --assignee) |
| `its wrike contacts get <user_id>` | Get a single contact by user ID. Pass the id (or any natural identifier) as the positional arg. |

#### `its wrike contacts`

List all Wrike contacts. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--filter` | `-f` | Filter by name | — |

**Examples:**

```bash
its wrike contacts

# Re-runs every 10s — handy for dashboards or incident response.
its wrike contacts --watch
```

#### `its wrike contacts search <query>`

Search contacts by name or email. Substring match across the most relevant fields; case-insensitive.

**Examples:**

```bash
its wrike contacts search jane.smith@example.com
```

#### `its wrike contacts find [query]`

Find contacts by name and/or email domain (for populating --assignee).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--email` | `-e` | Filter by email substring (e.g. '@example.com' for the whole domain) | — |

**Examples:**

```bash
its wrike contacts find jane.smith@example.com
```

#### `its wrike contacts get <user_id>`

Get a single contact by user ID. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its wrike contacts get <user-id>
```

---

### groups

> Source: `src/providers/wrike/commands/groups.ts`

| Command | Description |
|---------|-------------|
| `its wrike groups` | List Wrike user groups with member counts. Surfaces the most common fields; pass --json for raw shape. |
| `its wrike groups members <group>` | List the members of one Wrike group. Pass the group ID or title as the positional arg. |
| `its wrike groups for <contact>` | Every Wrike group one person belongs to — their group footprint. Accepts a contact ID, any of their emails, or their name. |
| `its wrike groups add-member <group> <contact>` | Add a person to a Wrike group. Needs --confirm. |
| `its wrike groups remove-member <group> <contact>` | Remove a person from a Wrike group — the offboarding step `leavers complete` covers automatically. Needs --confirm. |

#### `its wrike groups`

List Wrike user groups with member counts. Surfaces the most common fields; pass --json for raw shape.

```bash
its wrike groups
```

#### `its wrike groups members <group>`

List the members of one Wrike group. Pass the group ID or title as the positional arg.

```bash
its wrike groups members <group>
```

#### `its wrike groups for <contact>`

Every Wrike group one person belongs to — their group footprint. Accepts a contact ID, any of their emails, or their name.

**Examples:**

```bash
# What a departing user still has access to through Wrike groups
its wrike groups for jane.doe@example.com
```

#### `its wrike groups add-member <group> <contact>`

Add a person to a Wrike group. Needs --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Actually apply the change | — |

**Examples:**

```bash
# Accepts a group ID or title, and a contact ID, email or name
its wrike groups add-member 'My Team' jane.doe@example.com --confirm
```

#### `its wrike groups remove-member <group> <contact>`

Remove a person from a Wrike group — the offboarding step `leavers complete` covers automatically. Needs --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Actually apply the change | — |

**Examples:**

```bash
# Revokes group access. Does not release the Wrike seat — that is an admin-UI action
its wrike groups remove-member 'My Team' jane.doe@example.com --confirm
```

---

### audit

> Source: `src/providers/wrike/commands/audit.ts`

| Command | Description |
|---------|-------------|
| `its wrike audit log` | Account audit trail — who did what, when. Filter by operation, object type, object ID or acting user. Requires Wrike Enterprise and the 'Create user activity reports' admin right. |

#### `its wrike audit log`

Account audit trail — who did what, when. Filter by operation, object type, object ID or acting user. Requires Wrike Enterprise and the 'Create user activity reports' admin right.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--days` | `` | Days of history to scan (default 7) | 7 |
| `--operations` | `` | Comma-separated operations to include (e.g. TaskDeleted,PublicLinkCreated) | — |
| `--object-type` | `` | Restrict to one object type (needs --operations too) | — |
| `--object-id` | `` | Comma-separated object IDs, max 10 (needs --object-type) | — |
| `--user` | `` | Acting user — contact ID, email or name. Comma-separated, max 10 | — |
| `--limit` | `` | Maximum events to return, newest first (default 100) | 100 |

**Examples:**

```bash
# Everything logged in the last 24 hours
its wrike audit log --days 1

# Who removed work in the last month
its wrike audit log --operations TaskDeleted,TaskErased,RecycleBinErased --days 30

# Public links and shares — the data-exposure surface
its wrike audit log --operations PublicLinkCreated,TaskShared --days 90

# Everything one user did — accepts a contact ID, any of their emails, or their name
its wrike audit log --user jane.doe@example.com --days 7
```

---

### access

> Source: `src/providers/wrike/commands/access.ts`

| Command | Description |
|---------|-------------|
| `its wrike access drift` | Live Wrike accounts that no longer line up with Entra — leavers still active in Wrike, plus people with no Entra identity. Deactivated contacts are excluded: a successful SCIM deprovision leaves those behind and they are the correct end state. Read-only. |
| `its wrike access last-seen` | Every live Wrike person and when they were last active, longest-dormant first. Activity means any audit event, not just a sign-in — with SSO and long sessions an active user can go months without a fresh login. Needs Enterprise audit-log rights, and sweeps the whole window, so it is a slow command. |
| `its wrike access admins` | Wrike account admins and owners — the blast radius of one compromised login. |

#### `its wrike access drift`

Live Wrike accounts that no longer line up with Entra — leavers still active in Wrike, plus people with no Entra identity. Deactivated contacts are excluded: a successful SCIM deprovision leaves those behind and they are the correct end state. Read-only.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--issue` | `` | Restrict to one finding | — |
| `--licensed` | `` | Only accounts holding a paid seat (role User) | — |

**Examples:**

```bash
# Only drifted accounts holding a licensed seat — Collaborators are free
its wrike access drift --licensed

# Wrike accounts whose Entra account is disabled — always wrong
its wrike access drift --issue disabled

# Suppliers, clients and stale invitations — a list to review, not to action blindly
its wrike access drift --issue no-account
```

#### `its wrike access last-seen`

Every live Wrike person and when they were last active, longest-dormant first. Activity means any audit event, not just a sign-in — with SSO and long sessions an active user can go months without a fresh login. Needs Enterprise audit-log rights, and sweeps the whole window, so it is a slow command.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--days` | `` | Window to scan (default 90). Longer is slower — roughly one request per 1000 events. | 90 |
| `--stale-days` | `` | Only people whose last activity is older than this | — |
| `--never` | `` | Only people with no activity inside the window | — |
| `--licensed` | `` | Only people holding a billable seat — a regular User or an External User, not a free Collaborator | — |

**Examples:**

```bash
# Licensed users with no activity for 90 days — the reclaimable spend
its wrike access last-seen --licensed --stale-days 90

# Accounts with no activity at all inside the window — provisioned and never used
its wrike access last-seen --never
```

#### `its wrike access admins`

Wrike account admins and owners — the blast radius of one compromised login.

```bash
its wrike access admins
```

---

### forms

> Source: `src/providers/wrike/commands/forms.ts`

| Command | Description |
|---------|-------------|
| `its wrike forms` | List Wrike request forms with their field counts. Output routing and default assignees are not exposed by the API — only the questions. |
| `its wrike forms get <form>` | Every question on one request form, in page order. Pass the form ID or title. |

#### `its wrike forms`

List Wrike request forms with their field counts. Output routing and default assignees are not exposed by the API — only the questions.

```bash
its wrike forms
```

#### `its wrike forms get <form>`

Every question on one request form, in page order. Pass the form ID or title.

**Examples:**

```bash
# Field titles, types and which are mandatory — matched on ID or title
its wrike forms get 'Design Work Request Form'
```

---

### spaces

> Source: `src/providers/wrike/commands/spaces.ts`

| Command | Description |
|---------|-------------|
| `its wrike spaces` | List all Wrike spaces. Surfaces the most common fields; pass --json for raw shape. |

#### `its wrike spaces`

List all Wrike spaces. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its wrike spaces

# Re-runs every 10s — handy for dashboards or incident response.
its wrike spaces --watch
```

---

### folders

> Source: `src/providers/wrike/commands/spaces.ts`

| Command | Description |
|---------|-------------|
| `its wrike folders <space>` | List folders in a space (accepts space name or ID). Surfaces the most common fields; pass --json for raw shape. |

#### `its wrike folders <space>`

List folders in a space (accepts space name or ID). Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its wrike folders "IT Operations"

# Re-runs every 10s — handy for dashboards or incident response.
its wrike folders "IT Operations" --watch
```

---

### workflows

> Source: `src/providers/wrike/commands/workflows.ts`

| Command | Description |
|---------|-------------|
| `its wrike workflows` | List all workflows with statuses. Surfaces the most common fields; pass --json for raw shape. |

#### `its wrike workflows`

List all workflows with statuses. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its wrike workflows

# Re-runs every 10s — handy for dashboards or incident response.
its wrike workflows --watch
```

---

### custom-fields

> Source: `src/providers/wrike/commands/workflows.ts`

| Command | Description |
|---------|-------------|
| `its wrike custom-fields` | List custom field definitions. Surfaces the most common fields; pass --json for raw shape. |

#### `its wrike custom-fields`

List custom field definitions. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--filter` | `-f` | Filter by field name | — |

**Examples:**

```bash
its wrike custom-fields

# Re-runs every 10s — handy for dashboards or incident response.
its wrike custom-fields --watch
```

---

### item-types

> Source: `src/providers/wrike/commands/workflows.ts`

| Command | Description |
|---------|-------------|
| `its wrike item-types` | List custom item types (request types). Surfaces the most common fields; pass --json for raw shape. |

#### `its wrike item-types`

List custom item types (request types). Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its wrike item-types

# Re-runs every 10s — handy for dashboards or incident response.
its wrike item-types --watch
```

---

### onboarding

> Source: `src/providers/wrike/commands/onboarding.ts`

| Command | Description |
|---------|-------------|
| `its wrike onboarding get <permalink>` | Extract new starter onboarding details from a task. Pass the id (or any natural identifier) as the positional arg. |

#### `its wrike onboarding get <permalink>`

Extract new starter onboarding details from a task. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
# Parse onboarding task into structured fields
its wrike onboarding get <task-id>
```

---

### leavers

> Source: `src/providers/wrike/commands/leavers.ts`

| Command | Description |
|---------|-------------|
| `its wrike leavers` | List IT - Leaver tickets. Defaults to Active; pass --status Completed for closed ones. Surfaces Last Day + days-until-leave. |
| `its wrike leavers complete <idOrPermalink>` | Orchestrate the IT offboarding flow for a leaver ticket — disable + revoke sessions, clear manager, set extensionAttribute13=Leaver, convert mailbox to shared + GAL-hide, remove licences and groups, then mark the Wrike ticket Completed (only on full success). The status comment is returned as a draft — never posted directly, per wrike-comment-approval. Destructive — needs --confirm. Use --dry-run first. |
| `its wrike leavers get <idOrPermalink>` | Get full leaver-ticket details with comments. Pass the id (or Wrike permalink) as the positional arg. |

#### `its wrike leavers`

List IT - Leaver tickets. Defaults to Active; pass --status Completed for closed ones. Surfaces Last Day + days-until-leave.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--status` | `-s` | Filter by status (default: Active; 'all' for any) | Active |
| `--limit` | `-l` | Maximum results | 50 |

```bash
its wrike leavers
```

#### `its wrike leavers complete <idOrPermalink>`

Orchestrate the IT offboarding flow for a leaver ticket — disable + revoke sessions, clear manager, set extensionAttribute13=Leaver, convert mailbox to shared + GAL-hide, remove licences and groups, then mark the Wrike ticket Completed (only on full success). The status comment is returned as a draft — never posted directly, per wrike-comment-approval. Destructive — needs --confirm. Use --dry-run first.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Required to execute mutations — otherwise dry-run | — |
| `--dry-run` | `` | Print the plan without running anything | — |
| `--user` | `` | Override the resolved Entra UPN (skip ticket → user lookup) | — |
| `--skip` | `` | Comma-separated steps to skip (disable, sessions, manager, ext13, mailbox, licences, groups, wrike-groups, wrike) | — |

**Examples:**

```bash
# Shows every step it would run without touching the account
its wrike leavers complete IEACW7BXKQ2R4KMT --dry-run

its wrike leavers complete IEACW7BXKQ2R4KMT --confirm

its wrike leavers complete IEACW7BXKQ2R4KMT --skip convert-mailbox --confirm
```

#### `its wrike leavers get <idOrPermalink>`

Get full leaver-ticket details with comments. Pass the id (or Wrike permalink) as the positional arg.

```bash
its wrike leavers get <idOrPermalink>
```

---

### dashboard

> Source: `src/providers/wrike/commands/dashboard.ts`

| Command | Description |
|---------|-------------|
| `its wrike dashboard` | Overview of IT tickets by status. Surfaces the most common fields; pass --json for raw shape. |

#### `its wrike dashboard`

Overview of IT tickets by status. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
# Open/in-progress/blocked counts
its wrike dashboard

# Re-runs every 10s — handy for dashboards or incident response.
its wrike dashboard --watch
```

---

### auth

> Source: `src/providers/wrike/commands/auth.ts`

| Command | Description |
|---------|-------------|
| `its wrike auth login` | Sign in to Wrike via OAuth (browser). Requires WRIKE_CLIENT_ID/WRIKE_CLIENT_SECRET from a Wrike app registration — run `its wrike setup` first. |
| `its wrike auth logout` | Clear the persisted Wrike OAuth token. |
| `its wrike auth status` | Show Wrike OAuth sign-in state. |

#### `its wrike auth login`

Sign in to Wrike via OAuth (browser). Requires WRIKE_CLIENT_ID/WRIKE_CLIENT_SECRET from a Wrike app registration — run `its wrike setup` first.

**Examples:**

```bash
# Opens a browser — needs WRIKE_CLIENT_ID and WRIKE_CLIENT_SECRET configured first
its wrike auth login
```

#### `its wrike auth logout`

Clear the persisted Wrike OAuth token.

**Examples:**

```bash
its wrike auth logout
```

#### `its wrike auth status`

Show Wrike OAuth sign-in state.

```bash
its wrike auth status
```

---
