# its — CLI Reference

Usage, options, and output modes for the `its` CLI. For provider-specific commands, see the provider docs.

[Index](./index.md) · [README](../README.md)
Providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md)

## Contents

- [Installation](#installation)
- [Usage](#usage)
- [Providers](#providers)
- [Global Options](#global-options)
- [Output Modes](#output-modes)
- [Setup](#setup)
- [Shell Completions](#shell-completions)

## Installation

```bash
bun install
bun link
```

## Usage

```
its <provider> <resource> [action] [args] [--flags]
```

### Getting Help

| Command | Description |
|---------|-------------|
| `its --help` | Global help — list all providers |
| `its <provider>` | Provider help — list all resources and actions |
| `its <provider> help` | Same as above |
| `its <provider> <resource> help` | Resource help — list all actions, args, and flags |

## Providers

| Provider | Alias | Commands | Docs |
|----------|-------|----------|------|
| Tactical RMM | `rmm` | 65 commands, 16 resources | [rmm.md](./rmm.md) |
| Entra ID | `entra` | 100 commands, 21 resources | [entra.md](./entra.md) |
| Dokploy | `dokploy` | 116 commands, 25 resources | [dokploy.md](./dokploy.md) |
| Bitwarden | `bw` | 37 commands, 10 resources | [bw.md](./bw.md) |
| SharePoint | `sp` | 46 commands, 11 resources | [sp.md](./sp.md) |
| UniFi Network | `unifi` | 40 commands, 16 resources | [unifi.md](./unifi.md) |
| Wrike | `wrike` | 48 commands, 12 resources | [wrike.md](./wrike.md) |
| Azure CLI | `az` | 24 commands, 11 resources | [az.md](./az.md) |
| Exchange Online | `exo` | 35 commands, 8 resources | [exo.md](./exo.md) |
| Intune | `intune` | 41 commands, 16 resources | [intune.md](./intune.md) |
| UniFi Protect | `protect` | 6 commands, 4 resources | [protect.md](./protect.md) |
| Power BI | `pbi` | 21 commands, 6 resources | [pbi.md](./pbi.md) |
| Power Platform | `pa` | 11 commands, 4 resources | [pa.md](./pa.md) |
| Cloudflare | `cf` | 16 commands, 5 resources | [cf.md](./cf.md) |
| PeopleHR | `hr` | 8 commands, 4 resources | [hr.md](./hr.md) |
| Business Central | `bc` | 5 commands, 4 resources | [bc.md](./bc.md) |
| ctxc memories | `ctxc` | 5 commands, 1 resources | [ctxc.md](./ctxc.md) |
| Docs UI | `docs` | 5 commands, 5 resources | [docs.md](./docs.md) |
| GitHub | `gh` | 4 commands, 2 resources | [gh.md](./gh.md) |
| Outlook | `outlook` | 42 commands, 11 resources | [outlook.md](./outlook.md) |
| Microsoft 365 Health | `m365` | 3 commands, 2 resources | [m365.md](./m365.md) |

## Global Options

| Flag | Description |
|------|-------------|
| `--ai`, `--compact` | Minimal JSON for AI piping. Arrays of 5+ rows are compacted into a lossless columnar envelope (see Output Modes) |
| `--ai-flat` | Like `--ai` but plain row objects — skips the columnar envelope, for consumers that don't decode it |
| `--json` | Full raw JSON output |
| `--csv` | CSV output |
| `--tsv` | TSV output |
| `--sort <column>` | Sort table output by column name |
| `--order asc\|desc` | Sort direction (default: asc) |
| `--filter col=val` | Filter rows — case-insensitive substring match, comma for OR |
| `--fields a,b,c` | Select columns — works in all output modes (partial names OK) |
| `--limit N` | Limit output to N rows (server-side where supported) |
| `--count` | Show only the row count |
| `--auth auto\|delegated\|app` | OAuth mode for Graph providers. `auto` (default) tries delegated then falls back to app-only; `delegated` errors if no delegated token; `app` forces SP context |
| `--no-cache` | Bypass response cache |
| `--max-chars N` | Character budget (AI mode only) |
| `--no-colour` | Disable ANSI colours |
| `--include-secrets` | Disable global secret redaction. Audit-logged (one JSON line per use) to `~/.its/audit.log`; secret-bearing flag values are masked before write; perms `0o600` (file) / `0o700` (dir) on POSIX. Never paste output into chat/AI tools |
| `-v`, `--verbose` | Debug output to stderr |
| `-h`, `--help` | Show help |

### Flag Examples

```bash
# Output formats
its rmm agents --json                        # Full JSON
its rmm agents --ai                          # Compact JSON for piping
its rmm agents --csv > agents.csv             # Export to CSV

# Filter: col=value — case-insensitive substring match
its rmm agents --filter status=online         # Single value
its rmm agents --filter status=online,overdue # Multiple values (OR)
its entra users --filter company=candle       # Partial match

# Fields: select specific columns (partial names OK)
its exo groups --fields displayName,email     # Table shows 2 columns
its exo domains --fields domain,type --json   # JSON with 2 keys only

# Sort, limit, count
its rmm agents --sort hostname                # Sort ascending
its rmm agents --sort hostname --order desc   # Sort descending
its exo mailboxes --limit 10                  # First 10 results
its entra users --count                       # Just the total

# Combine flags
its rmm agents --filter status=online --sort hostname --limit 5
its exo groups --fields name,email --csv > groups.csv
```

## Output Modes

| Mode | When | Description |
|------|------|-------------|
| Human | Default in TTY | Summary line + formatted table |
| AI | `--ai` or non-TTY pipe | Minified JSON, respects `--max-chars`. Uniform arrays of 5+ rows become a columnar envelope (below) |
| AI (flat) | `--ai-flat` | Minified JSON of plain row objects — no columnar envelope |
| JSON | `--json` | Pretty-printed full JSON |

```bash
its rmm agents                 # Human-readable table
its rmm agents --json          # Full JSON
its rmm agents --ai            # Compact (columnar) JSON for AI piping
its rmm agents --ai-flat       # Compact JSON, plain rows (no envelope)
its rmm agents --ai | ai "which agents are offline?"
```

### Columnar `--ai` format

To save tokens, `--ai` rewrites a uniform array of 5+ objects into a lossless columnar envelope: field names are stated once, each row becomes a positional array, and any column with one value across every row is factored into `consts`.

```json
{ "_fmt": "cols", "_v": 1, "consts": { "os": "Windows 11" }, "fields": ["hostname", "status"], "rows": [["PC-A", "online"], ["PC-B", "offline"]] }
```

- `_fmt` is always `"cols"`; `_v` is the format version (bumped on any breaking shape change).
- Rehydrate by merging `consts` with `fields`-zipped `rows`. The `its` CLI does this automatically for `… --ai | its … --stdin`.
- Over `--max-chars`, trailing rows are dropped and a `_truncated` count is added — the output stays valid JSON.
- If your consumer can't decode the envelope, use `--ai-flat` to get plain row objects instead.

## Setup

Each provider has an interactive setup wizard that checks prerequisites, prompts for missing config, writes to `.env.local`, and tests the connection.

```bash
its <provider> setup           # Interactive — prompts for missing config
its <provider> setup --check   # Non-interactive status check
its <provider> setup --reset   # Re-run setup (overwrite existing config)
```

See [`.env.example`](../.env.example) for all available environment variables.

## Shell Completions

Tab completion for providers, resources, actions, and flags. Completions query live command definitions at tab-time, so they stay up to date automatically.

### Quick Install

Auto-detects your shells, checks for existing installs, and adds the completion line to the right profile:

```bash
its --completions install           # Auto-detect and install
its --completions install bash       # Install for a specific shell
```

### Manual Install

Add one line to your shell profile:

```bash
# Bash / Git Bash — add to ~/.bashrc
eval "$(its --completions bash)"

# Zsh — add to ~/.zshrc
eval "$(its --completions zsh)"

# Fish — add to ~/.config/fish/config.fish
its --completions fish | source
```

```powershell
# PowerShell — add to $PROFILE
Invoke-Expression (its --completions powershell | Out-String)
```
