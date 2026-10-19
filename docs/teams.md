# Teams (`teams`)

Microsoft Teams (Graph) for the logged-in user — list chats, read messages (with edits and reactions), send messages (plain text, Markdown or HTML; URLs become real links; @mentions, quote-replies, files, inline images, Adaptive Cards, importance), edit and delete your own messages, react, check your presence. Delegated-only (runs as you via `its auth login`; sending needs ChatMessage.Send, editing/reacting/deleting need Chat.ReadWrite).

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [attendance](./attendance.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [yodeck](./yodeck.md)

## Contents

- [Setup](#setup)
- [chats](#chats)
- [presence](#presence)
- [teams](#teams)
- [files](#files)

## Setup

```bash
its teams setup           # Interactive wizard
its teams setup --check   # Check configuration status
its teams setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `TENANT_ID` | Microsoft Entra tenant ID (shared) |
| `CLIENT_ID` | App registration client ID (shared) |

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/teams/client.ts` | API client methods |
| `src/providers/teams/types.ts` | TypeScript interfaces |
| `src/providers/teams/commands/` | Command definitions (split by resource) |
| `src/providers/teams/definition.ts` | definition |
| `src/providers/teams/format.ts` | format |

## Resources

### chats

> Source: `src/providers/teams/commands/chats.ts`

| Command | Description |
|---------|-------------|
| `its teams chats` | Your Teams chats (1:1, group, meeting), newest message first. Delegated — run `its auth login` first. |
| `its teams chats messages [chat_id]` | Read messages in one of your chats, newest first. Pass a chat ID from `its teams chats`, or --all-chats with --since to search every chat active in that window. |
| `its teams chats images <chat_id>` | Download pasted and attached images from a chat using delegated Graph auth. Pages to the time boundary, checks image signatures and never overwrites an existing file. |
| `its teams chats send <chat_id>` | Send a message to one of your chats, as you. Pass a chat ID from `its teams chats`. Delegated-only, and needs the ChatMessage.Send scope — after granting it, run `its auth login` again or the cached token still won't carry it. |
| `its teams chats delete-message <chat_id> <message_id>` | Delete one of YOUR messages in a chat (Teams shows 'This message has been deleted'). Without --confirm, shows the message it would delete. A file the message shared stays in your OneDrive — delete that separately to remove access. Needs delegated Chat.ReadWrite. |
| `its teams chats edit-message <chat_id> <message_id>` | Edit one of YOUR messages (Teams then shows it as edited). Shows before and after without --confirm, then reads it back to check. Takes the same --html / --markdown / --mention / --importance as send; a file, card or image already on the message is left alone. Needs delegated Chat.ReadWrite. |
| `its teams chats react <chat_id> <message_id>` | React to a message with an emoji, as you (default 👍). Also takes like, heart, laugh, surprised, sad, angry. Teams keeps ONE reaction per person per message, so a new one replaces yours. Needs Chat.ReadWrite. |
| `its teams chats unreact <chat_id> <message_id>` | Take your reaction off a message. Give the same emoji you reacted with (default 👍). |

#### `its teams chats`

Your Teams chats (1:1, group, meeting), newest message first. Delegated — run `its auth login` first.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--limit` | `` | Max chats to show (0 for every chat) | 25 |
| `--with` | `` | Only chats that include this person (name or email, part match, case-insensitive) | — |

**Examples:**

```bash
its teams chats

its teams chats --limit 0

its teams chats --with mark

its teams chats messages self
```

#### `its teams chats messages [chat_id]`

Read messages in one of your chats, newest first. Pass a chat ID from `its teams chats`, or --all-chats with --since to search every chat active in that window.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--all-chats` | `` | Search every chat whose last message is inside --since (needs --since; up to 50 chats, newest first) | — |
| `--limit` | `` | Max matching messages (0 for all; default 20) | 20 |
| `--since` | `` | Oldest message time (ISO, local date or -7d/-24h) | — |
| `--until` | `` | Newest message time (ISO or local date/time) | — |
| `--from` | `` | Sender display-name substring | — |

**Examples:**

```bash
its teams chats messages 19:abc...@thread.v2

its teams chats messages --all-chats --since -2d --from mark
```

#### `its teams chats images <chat_id>`

Download pasted and attached images from a chat using delegated Graph auth. Pages to the time boundary, checks image signatures and never overwrites an existing file.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--since` | `` | Oldest message time (default -1d) | -1d |
| `--until` | `` | Newest message time | — |
| `--from` | `` | Sender display-name substring | — |
| `--output` | `` | Directory to write images (default current directory) | . |

**Examples:**

```bash
its teams chats images 19:abc...@thread.v2 --since -1d --output ./teams-images
```

#### `its teams chats send <chat_id>`

Send a message to one of your chats, as you. Pass a chat ID from `its teams chats`. Delegated-only, and needs the ChatMessage.Send scope — after granting it, run `its auth login` again or the cached token still won't carry it.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--message` | `` | Message text | — |
| `--message-file` | `` | Read the message from a UTF-8 file (use for long bodies — Windows command-line cap) | — |
| `--html` | `` | Treat the message as HTML, sent as-is. Without it, a multi-line message is converted for you: blank lines make paragraphs, '- ' lines make a bulleted list | — |
| `--mention` | `` | @mention chat members, comma-separated: email, full name or first name. Replaces '@Name' in the message, or goes at the start | — |
| `--reply-to` | `` | Reply with a quote of this message (its id from `chats messages`), like Teams' 'Reply with quote'. Your Notes chat (self) shows the quote as an empty box — Teams limitation | — |
| `--file` | `` | Attach a local file (up to 4 MB): uploaded to your OneDrive 'Microsoft Teams Chat Files' and shared read-only with the chat's members. Needs Files.ReadWrite | — |
| `--markdown` | `` | Treat the message as Markdown: # headings (h1-h3), **bold**, *italic*, ~~strike~~, `code`, ``` code blocks, [links](https://…), - and 1. lists (flat), > quotes, --- rules, \| tables \| | — |
| `--importance` | `` | Mark the message important — Teams shows a banner | — |
| `--subject` | `` | Subject line (Graph accepts it; Teams may not show it in chats) | — |
| `--card` | `` | Send an Adaptive Card from this JSON file. Buttons that open a URL work; Action.Submit does not (no bot to receive it). The message text, if any, goes above the card | — |
| `--image` | `` | Inline image(s) in the message body: a .png/.jpg/.gif path, comma-separated for several (4 MB each) | — |

**Examples:**

```bash
its teams chats send self --markdown --importance high --message-file update.md

its teams chats send 19:abc...@thread.v2 --card card.json --message 'Approval needed'

its teams chats send self --file evidence.pdf --message "for the file"

its teams chats send 19:abc...@thread.v2 --message "on my way"

its teams chats send 19:abc...@thread.v2 --mention enrique --message-file update.txt

its teams chats send 19:abc...@thread.v2 --reply-to 1790767448630 --message "agreed"

its teams chats send 19:abc...@thread.v2 --html --message-file note.html
```

#### `its teams chats delete-message <chat_id> <message_id>`

Delete one of YOUR messages in a chat (Teams shows 'This message has been deleted'). Without --confirm, shows the message it would delete. A file the message shared stays in your OneDrive — delete that separately to remove access. Needs delegated Chat.ReadWrite.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Delete it | — |

**Examples:**

```bash
its teams chats delete-message self 1775589175306 --confirm
```

#### `its teams chats edit-message <chat_id> <message_id>`

Edit one of YOUR messages (Teams then shows it as edited). Shows before and after without --confirm, then reads it back to check. Takes the same --html / --markdown / --mention / --importance as send; a file, card or image already on the message is left alone. Needs delegated Chat.ReadWrite.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--message` | `` | New message text | — |
| `--message-file` | `` | Read the new text from a UTF-8 file | — |
| `--html` | `` | Treat the text as HTML, sent as-is | — |
| `--markdown` | `` | Treat the text as Markdown (same subset as send) | — |
| `--mention` | `` | @mention chat members, comma-separated (as on send) | — |
| `--importance` | `` | Change importance | — |
| `--confirm` | `` | Apply the edit | — |

**Examples:**

```bash
its teams chats edit-message self 1775589175306 --message "corrected text" --confirm

its teams chats edit-message 19:abc...@thread.v2 1775589175306 --markdown --message-file new.md --confirm
```

#### `its teams chats react <chat_id> <message_id>`

React to a message with an emoji, as you (default 👍). Also takes like, heart, laugh, surprised, sad, angry. Teams keeps ONE reaction per person per message, so a new one replaces yours. Needs Chat.ReadWrite.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--emoji` | `` | Emoji or name (default 👍) | — |

**Examples:**

```bash
its teams chats react self 1775589175306 --emoji 👍
```

#### `its teams chats unreact <chat_id> <message_id>`

Take your reaction off a message. Give the same emoji you reacted with (default 👍).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--emoji` | `` | Emoji or name (default 👍) | — |

**Examples:**

```bash
its teams chats unreact self 1775589175306 --emoji 👍
```

---

### presence

> Source: `src/providers/teams/commands/chats.ts`

| Command | Description |
|---------|-------------|
| `its teams presence get` | Your current Teams presence (availability + activity). |

#### `its teams presence get`

Your current Teams presence (availability + activity).

**Examples:**

```bash
its teams presence
```

---

### teams

> Source: `src/providers/teams/commands/teams.ts`

| Command | Description |
|---------|-------------|
| `its teams teams` | Teams you are a member of. Needs delegated Team.ReadBasic.All. |
| `its teams teams create <name>` | Create a team (the standard template). Graph does this in the background, so this waits for it to finish (up to --timeout) and reads the team back. Without --confirm it only shows what it would create. You become the owner unless --owner says otherwise. Needs delegated Team.Create. |

#### `its teams teams`

Teams you are a member of. Needs delegated Team.ReadBasic.All.

**Examples:**

```bash
its teams teams list
```

#### `its teams teams create <name>`

Create a team (the standard template). Graph does this in the background, so this waits for it to finish (up to --timeout) and reads the team back. Without --confirm it only shows what it would create. You become the owner unless --owner says otherwise. Needs delegated Team.Create.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--description` | `` | Team description | — |
| `--owner` | `` | Owner's UPN (default: you) | — |
| `--visibility` | `` | private (default) or public | — |
| `--timeout` | `` | Seconds to wait for creation (default 120) | — |
| `--confirm` | `` | Create it | — |

**Examples:**

```bash
its teams teams create "Store Ops" --description "Store ops" --visibility private --dry-run

its teams teams create "Store Ops" --owner tony@example.com --confirm
```

---

### files

> Source: `src/providers/teams/commands/files.ts`

| Command | Description |
|---------|-------------|
| `its teams files <chat_id> <message_id>` | Files shared in one chat message, with size and name (resolved from the SharePoint/OneDrive link). Use the attachment id with `files download`. |
| `its teams files download <chat_id> <message_id> <attachment_id>` | Download one file shared in a chat message (SharePoint/OneDrive-hosted) to a local path. Never overwrites; file is written mode 600. Delegated, as you. |

#### `its teams files <chat_id> <message_id>`

Files shared in one chat message, with size and name (resolved from the SharePoint/OneDrive link). Use the attachment id with `files download`.

**Examples:**

```bash
its teams files <chatId> <messageId> --json
```

#### `its teams files download <chat_id> <message_id> <attachment_id>`

Download one file shared in a chat message (SharePoint/OneDrive-hosted) to a local path. Never overwrites; file is written mode 600. Delegated, as you.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--to` | `` | Destination file path, or an existing directory | — |
| `--max-mb` | `` | Refuse files larger than this (default 50) | 50 |

**Examples:**

```bash
its teams files download <chatId> <messageId> <attachmentId> --to ./report.xlsx
```

---
