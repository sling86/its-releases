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
| `WRIKE_TOKEN` | Wrike permanent access token (Apps & Integrations > API) — fallback if not using OAuth |

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/wrike/client.ts` | API client methods |
| `src/providers/wrike/types.ts` | TypeScript interfaces |
| `src/providers/wrike/commands/` | Command definitions (split by resource) |
| `src/providers/wrike/auth.ts` | auth |
| `src/providers/wrike/definition.ts` | definition |

## Resources

### tickets

> Source: `src/providers/wrike/commands/tickets.ts`

| Command | Description |
|---------|-------------|
| `its wrike tickets` | List IT support tickets. Surfaces the most common fields; pass --json for raw shape. |
| `its wrike tickets stats` | Ticket analytics — median resolve time, backlog, bus-factor risk by assignee |
| `its wrike tickets active` | List active IT tickets. Returns whichever record the API key currently targets. |
| `its wrike tickets mine` | List tickets assigned to you (reads ITS_USER_EMAIL or --email) |
| `its wrike tickets get <idOrPermalink>` | Get full ticket details with comments. Pass the id (or any natural identifier) as the positional arg. |
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

# Pipe-friendly output — use with jq / scripts.
its wrike tickets stats --json
```

#### `its wrike tickets active`

List active IT tickets. Returns whichever record the API key currently targets.

**Examples:**

```bash
its wrike tickets active

# Pipe-friendly output — use with jq / scripts.
its wrike tickets active --json
```

#### `its wrike tickets mine`

List tickets assigned to you (reads ITS_USER_EMAIL or --email).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--status` | `` | Filter by status (default: Active) | Active |
| `--email` | `` | Email to match on (overrides ITS_USER_EMAIL env var) | — |

**Examples:**

```bash
its wrike tickets mine

# Pipe-friendly output — use with jq / scripts.
its wrike tickets mine --json
```

#### `its wrike tickets get <idOrPermalink>`

Get full ticket details with comments. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
# Full ticket with comments + attachments
its wrike tickets get <task-id>

# Pipe-friendly output — use with jq / scripts.
its wrike tickets get <task-id> --json
```

#### `its wrike tickets search <query>`

Search IT/BC support tickets by title. Substring match across the most relevant fields; case-insensitive.

**Examples:**

```bash
its wrike tickets search "printer"

# Pipe-friendly output — use with jq / scripts.
its wrike tickets search "printer" --json
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
its wrike tickets create --title "Outlook crashes" --description "Repro: open shared mailbox"

# Pipe-friendly output — use with jq / scripts.
its wrike tickets create --title "Outlook crashes" --description "Repro: open shared mailbox" --json
```

#### `its wrike tickets set-due <taskId> <dueDate>`

Set due date on a ticket (YYYY-MM-DD). Set the task's due date.

**Examples:**

```bash
its wrike tickets set-due <task-id> 2026-06-01

# Pipe-friendly output — use with jq / scripts.
its wrike tickets set-due <task-id> 2026-06-01 --json
```

#### `its wrike tickets assign <taskId> <userId>`

Add an assignee to a ticket. Idempotent — assigning twice is a no-op.

**Examples:**

```bash
its wrike tickets assign <task-id> <user-id>

# Pipe-friendly output — use with jq / scripts.
its wrike tickets assign <task-id> <user-id> --json
```

#### `its wrike tickets update-title <taskId> <title>`

Change a ticket's title. Rename the task — keeps the same ID.

**Examples:**

```bash
its wrike tickets update-title <task-id> "New title"

# Pipe-friendly output — use with jq / scripts.
its wrike tickets update-title <task-id> "New title" --json
```

#### `its wrike tickets update-importance <taskId> <importance>`

Change ticket importance (High, Normal, Low). Set Low / Normal / High importance.

**Examples:**

```bash
its wrike tickets update-importance <task-id> High

# Pipe-friendly output — use with jq / scripts.
its wrike tickets update-importance <task-id> High --json
```

#### `its wrike tickets update-status <taskId> <status>`

Change ticket status. Move the task to a different workflow stage.

**Examples:**

```bash
its wrike tickets update-status <task-id> "Completed"

# Pipe-friendly output — use with jq / scripts.
its wrike tickets update-status <task-id> "Completed" --json
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
its wrike tickets update-description <task-id> --markdown "**Repro:** open Outlook → file → ..."

# Pipe-friendly output — use with jq / scripts.
its wrike tickets update-description <task-id> --markdown "**Repro:** open Outlook → file → ..." --json
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
its wrike tickets add-comment <task-id> "Replicated, escalating"

# Pipe-friendly output — use with jq / scripts.
its wrike tickets add-comment <task-id> "Replicated, escalating" --json
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
its wrike tickets update-comment <comment-id> "fixed typo"

# Pipe-friendly output — use with jq / scripts.
its wrike tickets update-comment <comment-id> "fixed typo" --json
```

#### `its wrike tickets delete-comment <commentId>`

Delete a comment by ID. Requires --confirm. Use the commentId from add-comment output or `tickets get`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm deletion (irreversible) | — |

**Examples:**

```bash
its wrike tickets delete-comment <comment-id> --confirm

# Pipe-friendly output — use with jq / scripts.
its wrike tickets delete-comment <comment-id> --confirm --json
```

#### `its wrike tickets narrative [taskId]`

Deep per-ticket narrative with recent comments + ball-is-with heuristic. Prioritises active chatter > waiting-on-them > waiting-on-us > ancient. Pass a single taskId, or --active / --mine for a batch view.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--active` | `` | All active IT tickets (ignores --mine) | — |
| `--mine` | `` | Only tickets assigned to you (via ITS_USER_EMAIL or --email) | — |
| `--email` | `` | Email to match for --mine (overrides ITS_USER_EMAIL) | — |
| `--comments` | `` | Comments to show per ticket (default 3) | 3 |
| `--limit` | `` | Maximum tickets to process for batch modes (default 25) | 25 |

**Examples:**

```bash
# AI-friendly chronological narrative
its wrike tickets narrative <task-id>

# Pipe-friendly output — use with jq / scripts.
its wrike tickets narrative <task-id> --json
```

#### `its wrike tickets attachments <taskId>`

List attachments on a ticket. List attachments; pair with `attach`.

**Examples:**

```bash
its wrike tickets attachments <task-id>

# Pipe-friendly output — use with jq / scripts.
its wrike tickets attachments <task-id> --json
```

#### `its wrike tickets download <taskId> [attachmentId]`

Download an attachment from a ticket. Stream the resource to a local file.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--output` | `-o` | Output file path (defaults to original filename in cwd) | — |

**Examples:**

```bash
its wrike tickets download <task-id> <attachment-id> --output ./screenshot.png

# Pipe-friendly output — use with jq / scripts.
its wrike tickets download <task-id> <attachment-id> --output ./screenshot.png --json
```

#### `its wrike tickets attach <taskId> <filePath>`

Attach a file to a ticket. Upload a local file as an attachment.

**Examples:**

```bash
its wrike tickets attach <task-id> ./screenshot.png

# Pipe-friendly output — use with jq / scripts.
its wrike tickets attach <task-id> ./screenshot.png --json
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

# Pipe-friendly output — use with jq / scripts.
its wrike tasks --json

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

# Pipe-friendly output — use with jq / scripts.
its wrike tasks search "deploy" --json
```

#### `its wrike tasks get <idOrPermalink>`

Get full task details with comments. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its wrike tasks get https://www.wrike.com/open.htm?id=...

# Pipe-friendly output — use with jq / scripts.
its wrike tasks get https://www.wrike.com/open.htm?id=... --json
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
its wrike tasks create "My folder" --title "New task"

# Pipe-friendly output — use with jq / scripts.
its wrike tasks create "My folder" --title "New task" --json
```

#### `its wrike tasks set-due <taskId> <dueDate>`

Set due date on a task (YYYY-MM-DD). Set the task's due date.

**Examples:**

```bash
its wrike tasks set-due <task-id> 2026-06-01

# Pipe-friendly output — use with jq / scripts.
its wrike tasks set-due <task-id> 2026-06-01 --json
```

#### `its wrike tasks update-title <taskId> <title>`

Change a task's title. Rename the task — keeps the same ID.

**Examples:**

```bash
its wrike tasks update-title <task-id> "New title"

# Pipe-friendly output — use with jq / scripts.
its wrike tasks update-title <task-id> "New title" --json
```

#### `its wrike tasks update-importance <taskId> <importance>`

Change task importance (High, Normal, Low). Set Low / Normal / High importance.

**Examples:**

```bash
its wrike tasks update-importance <task-id> Low

# Pipe-friendly output — use with jq / scripts.
its wrike tasks update-importance <task-id> Low --json
```

#### `its wrike tasks update-status <taskId> <status>`

Change task status. Move the task to a different workflow stage.

**Examples:**

```bash
its wrike tasks update-status <task-id> "In Progress"

# Pipe-friendly output — use with jq / scripts.
its wrike tasks update-status <task-id> "In Progress" --json
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
its wrike tasks update-description <task-id> --markdown "## Plan\n- step 1"

# Pipe-friendly output — use with jq / scripts.
its wrike tasks update-description <task-id> --markdown "## Plan\n- step 1" --json
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
its wrike tasks add-comment <task-id> --markdown "**done** — pushed to main"

# Pipe-friendly output — use with jq / scripts.
its wrike tasks add-comment <task-id> --markdown "**done** — pushed to main" --json
```

#### `its wrike tasks attach <taskId> <filePath>`

Attach a file to a task. Upload a local file as an attachment.

**Examples:**

```bash
its wrike tasks attach <task-id> ./diagram.png

# Pipe-friendly output — use with jq / scripts.
its wrike tasks attach <task-id> ./diagram.png --json
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

# Pipe-friendly output — use with jq / scripts.
its wrike projects search "migration" --json
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

# Pipe-friendly output — use with jq / scripts.
its wrike projects tasks "Sprint 23" --json
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

# Pipe-friendly output — use with jq / scripts.
its wrike contacts --json

# Re-runs every 10s — handy for dashboards or incident response.
its wrike contacts --watch
```

#### `its wrike contacts search <query>`

Search contacts by name or email. Substring match across the most relevant fields; case-insensitive.

**Examples:**

```bash
its wrike contacts search jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its wrike contacts search jane.smith@example.com --json
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

# Pipe-friendly output — use with jq / scripts.
its wrike contacts find jane.smith@example.com --json
```

#### `its wrike contacts get <user_id>`

Get a single contact by user ID. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its wrike contacts get <user-id>

# Pipe-friendly output — use with jq / scripts.
its wrike contacts get <user-id> --json
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

# Pipe-friendly output — use with jq / scripts.
its wrike spaces --json

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

# Pipe-friendly output — use with jq / scripts.
its wrike folders "IT Operations" --json

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

# Pipe-friendly output — use with jq / scripts.
its wrike workflows --json

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

# Pipe-friendly output — use with jq / scripts.
its wrike custom-fields --json

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

# Pipe-friendly output — use with jq / scripts.
its wrike item-types --json

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

# Pipe-friendly output — use with jq / scripts.
its wrike onboarding get <task-id> --json
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
| `--skip` | `` | Comma-separated steps to skip (disable, sessions, manager, ext13, mailbox, licences, groups, wrike) | — |

```bash
its wrike leavers complete <idOrPermalink>
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

# Pipe-friendly output — use with jq / scripts.
its wrike dashboard --json

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

```bash
its wrike auth login
```

#### `its wrike auth logout`

Clear the persisted Wrike OAuth token.

```bash
its wrike auth logout
```

#### `its wrike auth status`

Show Wrike OAuth sign-in state.

```bash
its wrike auth status
```

---
