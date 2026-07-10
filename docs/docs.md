# Docs UI (`docs`)

Interactive help UI for the `its` CLI. `docs serve` launches a browser-based command explorer (tree, search, examples) bound to 127.0.0.1 with a random per-session token. `docs open <command>` deep-links to a single page. `docs search <q>` and `docs show <topic>` render help in the terminal.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [build](#build)
- [serve](#serve)
- [open](#open)
- [search](#search)
- [show](#show)

## Setup

```bash
its docs setup           # Interactive wizard
its docs setup --check   # Check configuration status
its docs setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|

No setup required. The UI is built and bundled into the `its` binary by `bun run build:help-ui` (chained from `bun run docs`).

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/docs/client.ts` | API client methods |
| `src/providers/docs/commands.ts` | Command definitions |
| `src/providers/docs/build-helpers.ts` | build helpers |
| `src/providers/docs/definition.ts` | definition |

## Resources

### build

> Source: `src/providers/docs/commands.ts`

| Command | Description |
|---------|-------------|
| `its docs build` | Compile the interactive help UI browser assets (Vite build + embedded-assets manifest) |

#### `its docs build`

Compile the interactive help UI browser assets (Vite build + embedded-assets manifest).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--force` | `` | Rebuild even if up-to-date assets already exist | false |

**Examples:**

```bash
its docs build

# Rebuild even if assets are present
its docs build --force
```

---

### serve

> Source: `src/providers/docs/commands.ts`

| Command | Description |
|---------|-------------|
| `its docs serve` | Launch the interactive help UI on 127.0.0.1 and open a browser |

#### `its docs serve`

Launch the interactive help UI on 127.0.0.1 and open a browser.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--port` | `` | Bind port (0 = random) | 0 |
| `--no-open` | `` | Don't auto-open the browser | false |
| `--theme` | `` | Theme | auto |

**Examples:**

```bash
its docs serve

# Bind a known port (useful for bookmarks)
its docs serve --port 4242

# Print URL only — don't launch a browser
its docs serve --no-open
```

---

### open

> Source: `src/providers/docs/commands.ts`

| Command | Description |
|---------|-------------|
| `its docs open <topic>` | Launch the help UI and open the browser to a specific command page |

#### `its docs open <topic>`

Launch the help UI and open the browser to a specific command page.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--port` | `` | Bind port (0 = random) | 0 |
| `--theme` | `` | Theme | auto |

**Examples:**

```bash
its docs open rmm agents list

its docs open entra users
```

---

### search

> Source: `src/providers/docs/commands.ts`

| Command | Description |
|---------|-------------|
| `its docs search <query>` | Fuzzy search every command — terminal output. Surfaces the most common fields; pass --json for raw shape. |

#### `its docs search <query>`

Fuzzy search every command — terminal output. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--limit` | `` | Max results | 25 |

**Examples:**

```bash
its docs search "offline agents"

its docs search dokploy apps
```

---

### show

> Source: `src/providers/docs/commands.ts`

| Command | Description |
|---------|-------------|
| `its docs show <topic>` | Show one command's help in the terminal (ANSI markdown). Surfaces the most common fields; pass --json for raw shape. |

#### `its docs show <topic>`

Show one command's help in the terminal (ANSI markdown). Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its docs show rmm

its docs show rmm agents

its docs show rmm agents list
```

---
