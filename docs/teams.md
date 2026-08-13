# Teams (`teams`)

Microsoft Teams (Graph) for the logged-in user — list recent chats, read chat messages, check your presence. Delegated-only (runs as you via `its auth login`).

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md)

## Contents

- [Setup](#setup)
- [chats](#chats)
- [presence](#presence)

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

## Resources

### chats

> Source: `src/providers/teams/commands/chats.ts`

| Command | Description |
|---------|-------------|
| `its teams chats` | Your recent Teams chats (1:1, group, meeting), most-recently-active first. Delegated — run `its auth login` first. |
| `its teams chats messages <chat_id>` | Read messages in one of your chats, newest first. Pass a chat ID from `its teams chats`. |
| `its teams chats send <chat_id>` | Send a message to one of your chats, as you. Pass a chat ID from `its teams chats`. Delegated-only, and needs the ChatMessage.Send scope — after granting it, run `its auth login` again or the cached token still won't carry it. |

#### `its teams chats`

Your recent Teams chats (1:1, group, meeting), most-recently-active first. Delegated — run `its auth login` first.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--limit` | `` | Max chats to return (1-50) | 25 |

**Examples:**

```bash
its teams chats

its teams chats --limit 50
```

#### `its teams chats messages <chat_id>`

Read messages in one of your chats, newest first. Pass a chat ID from `its teams chats`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--limit` | `` | Max messages to return (1-50) | 20 |

**Examples:**

```bash
its teams chats messages 19:abc...@thread.v2
```

#### `its teams chats send <chat_id>`

Send a message to one of your chats, as you. Pass a chat ID from `its teams chats`. Delegated-only, and needs the ChatMessage.Send scope — after granting it, run `its auth login` again or the cached token still won't carry it.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--message` | `` | Message text | — |
| `--message-file` | `` | Read the message from a UTF-8 file (use for long bodies — Windows command-line cap) | — |
| `--html` | `` | Treat the message as HTML (default plain text) | — |

**Examples:**

```bash
its teams chats send 19:abc...@thread.v2 --message "on my way"

its teams chats send 19:abc...@thread.v2 --html --message-file note.html
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
