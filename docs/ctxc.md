# ctxc memories (`ctxc`)

Readonly access to the local ctxc memory database (`~/.claude/ctxc.db`). Provides shell-side recall without going through the MCP server — useful in scripts, cron jobs, and agent bodies. Writes (save/update/delete) still belong to the ctxc MCP plugin..

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md)

## Contents

- [Setup](#setup)
- [memories](#memories)

## Setup

```bash
its ctxc setup           # Interactive wizard
its ctxc setup --check   # Check configuration status
its ctxc setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `CTXC_DB_PATH` | Optional override for the ctxc database path (defaults to ~/.claude/ctxc.db) |

Install the ctxc plugin (`claude plugin install ctxc`) — it owns the database at `~/.claude/ctxc.db`. This provider only reads from it.

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/ctxc/client.ts` | API client methods |
| `src/providers/ctxc/types.ts` | TypeScript interfaces |
| `src/providers/ctxc/commands.ts` | Command definitions |
| `src/providers/ctxc/definition.ts` | definition |

## Resources

### memories

> Source: `src/providers/ctxc/commands.ts`

| Command | Description |
|---------|-------------|
| `its ctxc memories search <query>` | Full-text search across ctxc memories (FTS5). Supports query operators: AND OR NOT, quoted phrases. |
| `its ctxc memories recall` | Project-scoped recall — recent memories for a project plus globals. Mirrors ctxc_recall MCP semantics. |
| `its ctxc memories get <id>` | Fetch a single memory by id (includes full content). Pass the id (or any natural identifier) as the positional arg. |
| `its ctxc memories` | List memories filtered by project/type/tags, newest first. Use for tag-driven scans (e.g. --tags backlog,resume-prompt). |
| `its ctxc memories stats` | Memory counts broken down by type and project (handy for grooming) |

#### `its ctxc memories search <query>`

Full-text search across ctxc memories (FTS5). Supports query operators: AND OR NOT, quoted phrases.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--project` | `` | Filter by project | — |
| `--type` | `` | Filter by memory type | — |
| `--tags` | `` | Comma-separated tags (OR match) | — |
| `--limit` | `` | Maximum results (default 20) | 20 |

**Examples:**

```bash
its ctxc memories search "auth"

# Pipe-friendly output — use with jq / scripts.
its ctxc memories search "auth" --json
```

#### `its ctxc memories recall`

Project-scoped recall — recent memories for a project plus globals. Mirrors ctxc_recall MCP semantics.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--project` | `` | Scope to project | — |
| `--topic` | `` | Optional topic search (falls back to recency if absent) | — |
| `--limit` | `` | Maximum results (default 10) | 10 |

**Examples:**

```bash
its ctxc memories recall "graph api throttling"

# Pipe-friendly output — use with jq / scripts.
its ctxc memories recall "graph api throttling" --json
```

#### `its ctxc memories get <id>`

Fetch a single memory by id (includes full content). Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its ctxc memories get <memory-id>

# Pipe-friendly output — use with jq / scripts.
its ctxc memories get <memory-id> --json
```

#### `its ctxc memories`

List memories filtered by project/type/tags, newest first. Use for tag-driven scans (e.g. --tags backlog,resume-prompt).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--project` | `` | Filter by project | — |
| `--type` | `` | Filter by memory type | — |
| `--tags` | `` | Comma-separated tags (OR match) | — |
| `--limit` | `` | Maximum results (default 50) | 50 |

**Examples:**

```bash
its ctxc memories

its ctxc memories --project it-cli

# Re-runs every 10s — handy for dashboards or incident response.
its ctxc memories --watch
```

#### `its ctxc memories stats`

Memory counts broken down by type and project (handy for grooming).

**Examples:**

```bash
# Counts by type, project, recency
its ctxc memories stats

# Pipe-friendly output — use with jq / scripts.
its ctxc memories stats --json
```

---
