# Help UI (`its docs`)

> Auto-generated docs index entry. The full reference for every `its docs` subcommand is at [docs.md](./docs.md).

Interactive help UI for the `its` CLI. Four ways to consume the same registry:

| Command | What it does |
|---------|--------------|
| `its docs serve` | Launch a local browser UI on 127.0.0.1 with command tree, signature, flags, examples, and Cmd-K palette |
| `its docs open <topic>` | Same as above, but deep-links the browser to a specific command page |
| `its docs search <q>` | Fuzzy search every command in the terminal — prints signature + description |
| `its docs show <topic>` | Render one command's help in the terminal (ANSI markdown) |

All four read from the same JSON registry built from live `CommandDef`s — there's no separate doc source to maintain.

## Browser UI

```bash
its docs serve              # random port, opens browser
its docs serve --port 4242  # known port
its docs serve --no-open    # print URL, don't launch browser
its docs serve --theme dark # force dark mode on first paint (default: auto)
```

### Layout

```
┌────────────────────────────────────────────────────────────────────┐
│ ◆ its docs    [Search commands ⌘K]              ☾ theme            │
├──────────┬─────────────────────────────────────────────────────────┤
│ rmm    ▾ │ its rmm agents                                          │
│  agents  │ List all RMM agents with status, hostname, OS, site     │
│   list ●│  [4 flags]  [5 examples]                                 │
│   stale │ ─────────── FLAGS ────────────                            │
│   ping  │ --status, -s   string   Filter by status [online|offline]│
│   ...   │ --client, -c   string   Filter by client name            │
│ entra  ▸ │ ─────────── EXAMPLES ─────────                           │
│ ...      │ [all] [offline] [by site] [json] [pipe to ai]            │
│          │ $ its rmm agents --status offline                       │
│          │ [▶ Run (disabled)] [Copy]                                │
└──────────┴─────────────────────────────────────────────────────────┘
```

### Keyboard shortcuts

| Key | Action |
|-----|--------|
| `⌘K` / `Ctrl+K` / `/` | Open command palette |
| `⌘H` / `Ctrl+H` | Toggle recent-runs panel |
| `T` | Cycle mode (auto → dark → light) on the current palette |
| `↑↓` | Navigate palette results |
| `↵` (in palette) | Open selected command |
| `↵` (in edit input) | Run the edited command |
| `Esc` | Close any open overlay |

### Output mode picker

A small select beside Run lets you pick how the command output appears:

| Mode | Effect |
|------|--------|
| **table** (default) | Sortable DOM table from `tableHeaders` + `tableRows` |
| **JSON** | Collapsible JSON tree of `result.data` — works for any shape |
| **AI-compact** | Same as JSON visually; `--ai` flag added so the server returns the AI-mode envelope |
| **CSV** | `--csv` flag added; JSON tree of the response |

The pick is persisted in `localStorage` and only applies to non-interactive, non-destructive commands.

### Recent runs

Every time Run launches (table, terminal, or post-confirm), the line is pushed onto a 25-entry ring buffer persisted in `localStorage`. Open the panel via the ⟳ icon in the header or `⌘H` / `Ctrl+H`. Click an entry to jump to that command page with the line pre-filled in the edit input — one keystroke away from re-running with tweaks.

### Tree filter

The sidebar has a small filter input at the top. Type any substring — it matches against alias, name, resource, action, or description across all 500+ commands. Matching providers auto-expand; provider counts show `matched/total` while filtering. Esc or × clears.

### Sticky example actions

The Run/mode/Copy/Reset row uses `position: sticky` inside the example body so it pins to the top of the scroll area once you scroll past it. Keeps the Run button one click away on long command pages (heavy flag lists like `dokploy apps deploy` or `bw vault create`).

### Watch mode

Beside the Run button (result-mode only) there's a `↻ Watch` toggle + interval picker (5s / 10s / 30s / 60s, default 10s). Toggle it on and the same command re-runs on that cadence — handy for monitoring `dokploy apps health`, `rmm agents --status offline`, or any table-shaped output that changes over time.

The result header gets a mauve pulsing `watch · tick N · 10s` pill and, once a tick produces a different result, a delta badge like `+2 -1 ~3` showing rows added / removed / changed.

In the table itself:

- **New rows** flash green (`flash-new` 2s ease-out)
- **Changed cells** flash peach (`flash` 2s ease-out)

Row identity is derived from an id-like column if present (`id`, `agent_id`, `alias`, `name`, `upn`, `key`) — otherwise the row's concatenated value. Stops automatically after 60 ticks, on navigation, on the × close, or click ■ Stop.

### Terminal dock

Interactive commands open in a persistent dock anchored to the bottom of the viewport, not inline under the example. The dock:

- Survives navigation — open `cf token request` in a terminal, then browse the rest of the registry; the WS bridge keeps streaming
- Holds up to 4 simultaneous terminals as tabs; oldest gets evicted when you exceed the cap
- ▼ minimises to just the tab strip (~36 px tall) so you can read the doc page underneath; ▲ restores
- × on a tab kills that child + WS, leaving any other terminals running
- Each terminal is a full xterm.js (WebGL renderer) instance, independent of selectedId

### Theme

Five palettes, each with light + dark variants:

- **Catppuccin** — Mocha / Latte (default)
- **GitHub** — Dark Default / Light Default
- **Tokyo Night** — Night / Day
- **Nord** — Polar Night / Snow Storm
- **Rosé Pine** — Main / Dawn

Click the sun/moon icon in the header to open the theme picker — pick palette + mode independently. Both choices persist in `localStorage` (`its-palette`, `its-mode`). Mode supports `auto` (honours OS `prefers-color-scheme`). The `T` key still cycles mode (auto → dark → light) on the current palette.

Tokens live in `src/help-ui/web/styles/theme.css` — components read semantic vars (`--bg-base`, `--fg-default`, `--accent`) which the palette × mode selectors rewire. Adding a new palette is one block of raw vars per mode + two media queries for `data-mode="auto"`.

### Run button

Click `▶ Run` on any example to execute it in-process — same registry the CLI uses, no shell, no `exec`. The result panel shows live status (running / done / error), elapsed time, summary, and a sortable DOM table (or JSON tree if the command doesn't expose `tableHeaders`).

Disabled when:

- `cmd.interactive: true` — Bitwarden setup, prompts, etc. Needs PTY (next phase).
- `cmd.action` is in the destructive denylist (`delete`, `remove`, `wipe`, `reboot`, `kill`, `stop`, `purge`). Confirm-dialog support lands later.
- Example marks itself `noRun: true` (illustrative only).

Endpoint: `POST /api/result` with `{ argv: string[] }`. Enforced caps: 3 in-flight per session, 30s timeout, server-side parse so the argv is never passed to a shell.

## Terminal subcommands

### `its docs show <topic>`

Render a command's help in the terminal — works as a faster `--help` and includes examples by default. Topic resolution is hierarchical:

```bash
its docs show rmm                 # provider overview
its docs show rmm agents          # resource overview (all actions)
its docs show rmm agents list     # single command (flags, examples)
```

If the topic doesn't match exactly, falls through to fuzzy search.

### `its docs search <query>`

```bash
its docs search "offline agents"      # cross-provider fuzzy search
its docs search dokploy apps deploy   # multi-word ranking
its docs search "shared mailbox"      # description match
its docs search --limit 5 ...         # cap results
```

Results table: `signature` + `description`. JSON mode (`--json`/`--ai`) returns score + ids — useful for AI tooling.

### `its docs open <topic>`

Launches the browser UI and jumps straight to a command page via URL hash (`/#/rmm/agents/list`). Same topic resolution as `show`.

```bash
its docs open rmm agents list
its docs open entra users
```

## Security

| Control | Default |
|---------|---------|
| Bind | `127.0.0.1` only — no `--lan` in PR 1 |
| Auth | 32-char hex token in URL `?t=...`, then `its_token` cookie |
| Headers | `X-Its-Token` accepted as alternative |
| Origin | Cookie scoped to host:port, `SameSite=Strict` |
| Idle | Server self-terminates after 1 hour of no requests |
| `/api/health` | Open (liveness probe) — never returns secrets |

The token is printed to stderr on startup. Keep stderr private — anyone with the URL has full read access to your `CommandDef` registry (which is metadata, not credentials).

## Architecture

```
src/help-ui/
├── server.ts              # Bun.serve(), token + cookie auth, asset routing
├── registry-snapshot.ts   # Walks ALL_PROVIDERS → JSON
├── examples.ts            # Central bank for un-migrated examples
├── search-engine.ts       # Weighted-substring scorer (server-side)
├── topic-renderer.ts      # ANSI markdown for `docs show`
└── web/                   # Svelte 5 source (build target)
    ├── App.svelte
    ├── lib/
    │   ├── state.svelte.ts     # $state store (named `app` to avoid $state shadowing)
    │   ├── Tree.svelte         # virtualised command tree
    │   ├── CommandCard.svelte  # signature + flags + examples
    │   ├── Palette.svelte      # Cmd-K + uFuzzy
    │   └── signature.ts        # colourise "its rmm agents list <id>"
    ├── styles/{theme.css, app.css}
    └── fonts/{InterVariable, JetBrainsMono}.woff2

src/providers/docs/        # CLI surface (serve/open/search/show)
└── definition.ts → commands.ts → src/help-ui/server.ts dynamic import
```

## Editable example commands

Every example renders in an inline text input. Click it, edit the line (swap `<id>` placeholders for real values, add `--json`, etc.), then hit Enter or click Run. The unedited Copy button reflects the current edit; once you've diverged from the original, a Reset button appears.

The edit buffer resets when you change tabs or navigate to a different command.

## Interactive auto-detection

You don't have to annotate `interactive: true` by hand. The build walks each `src/providers/**/commands/*.ts` once at server start, finds every `{ resource:"x", action:"y", ... }` block whose body references `promptConfirm` / `promptText` / `promptSecret` / `promptPin` / `promptList`, and merges those keys into the snapshot.

Override is supported either way:

- Manual `interactive: true` in a CommandDef always wins.
- Auto-detect picks up anything that uses the prompt helpers — that's how `bw vault create`, `bw session unlock`, `bw pin reset`, `cf token request`, and `cf tunnels delete` all surface the terminal route without explicit annotation.

## Adding examples to a command

Add an `examples` field on the `CommandDef`:

```ts
{
  resource: "agents",
  action: "list",
  description: "...",
  examples: [
    { title: "all", cmd: "its rmm agents" },
    { title: "offline", cmd: "its rmm agents --status offline",
      description: "Only offline or overdue agents" },
  ],
  handler: ...,
}
```

The doc generator picks them up too — they'll show in both `docs/<provider>.md` and the help UI.

For commands not yet migrated, the central `src/help-ui/examples.ts` bank keys by `"<alias> <resource> <action>"`. Inline takes precedence over bank entries.

## Build

```bash
bun run build:help-ui        # Build Svelte bundle into src/help-ui/dist/
bun run docs                 # postbuild chain now includes the help-ui build
```

Bundle size (gzipped): ~28 KB JS + ~3 KB CSS. Plus ~450 KB woff2 fonts (cached forever).

### Compiled binary

`bun build --compile` bundles the entire UI into the resulting `its.exe`. The pipeline is:

1. `bun run build:help-ui` — Vite emits stable-name assets into `src/help-ui/dist/`
2. The same script writes `src/help-ui/embedded-assets.ts` with one `import x from "./dist/<file>" with { type: "file" }` per asset
3. `bun build --compile` follows the static imports, embeds each file into the binary's virtual filesystem
4. At runtime, `server.ts` reads each asset via `Bun.file(EMBEDDED_ASSETS[key])` — works identically in dev (resolves to disk) and compiled mode (resolves to the embedded path)

Disk-on-dist is still preferred when present so you can iterate on Svelte source via `bun run dev` without rebuilding the binary.

Binary size is ~120 MB (Bun runtime ~119 MB + UI assets ~1 MB).

## Run modes

The Run button picks one of three modes per command:

| Mode | When | What happens |
|------|------|--------------|
| **result** | normal commands | POST `/api/result`, render sortable DOM table |
| **terminal** | `cmd.interactive === true` | Open `TerminalPane`, WebSocket to `/api/run`, live xterm.js pane with WebGL renderer, kill button |
| **confirm-then-result** | destructive action (delete/remove/wipe/reboot/kill/stop/purge) | Modal "⚠ Destructive command" → on confirm, runs via `/api/result` |

The terminal pane uses `Bun.spawn` with pipes (not PTY — `Bun.Terminal` is unavailable on Windows). Practical effect: line-buffered prompts (`promptConfirm` Y/N, `promptText`) appear correctly; raw-mode prompts that mask input (`promptSecret` PIN entry) do not echo. Linux/macOS: full Bun.Terminal PTY available, would require a guard if we wanted to use it. Annotate `interactive: true` on a CommandDef to surface the terminal route.

## Roadmap

- [x] PR 1: browse-only UI, tree, command card, palette, terminal subcommands
- [x] PR 1: `/api/result` + Run button + sortable DOM table for non-destructive commands
- [x] PR 2 phase B: live terminal pane (xterm.js webgl + WS `/api/run` + Bun.spawn child + Catppuccin theme + Kill/Close)
- [x] PR 2 phase B: destructive-action confirm modal
- [x] PR 3: auto-detect interactive commands via static prompt scan
- [x] PR 3: editable command line (swap placeholders inline before Run)
- [x] PR 3: inline-migrated examples across rmm, entra, dokploy, bw, unifi, intune, exo, cf
- [x] PR 4: full binary embedding of dist assets via generated `embedded-assets.ts` manifest
- [ ] Future: `Bun.Terminal` PTY on Mac/Linux (raw-mode prompts, resize)
- [ ] PR 3: `Bun.Terminal` PTY on Mac/Linux (raw-mode prompts, resize)
- [ ] Future: `--lan` mode with QR code for phone testing
- [ ] Future: server-side rendering of the initial tree for instant first paint
