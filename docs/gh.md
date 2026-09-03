# GitHub (`gh`)

GitHub via the local `gh` CLI. Piggybacks on the user's existing `gh auth` — no PAT needed. Today: standard branch-protection (THF block), per-repo webhook setup, webhook list.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [branch-protect](#branch-protect)
- [webhook](#webhook)

## Setup

```bash
its gh setup           # Interactive wizard
its gh setup --check   # Check configuration status
its gh setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|

Requires the GitHub CLI (`gh`) on PATH and authenticated against your account/org (`gh auth login` once). Every command shells out to `gh api` under the hood so you inherit whatever scopes your existing `gh` token has — no separate PAT or env var. Verify with `gh auth status`.

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/gh/client.ts` | API client methods |
| `src/providers/gh/commands/` | Command definitions (split by resource) |
| `src/providers/gh/definition.ts` | definition |

## Resources

### branch-protect

| Command | Description |
|---------|-------------|
| `its gh branch-protect apply <repo>` | Apply the THF standard branch-protection block to <owner/repo> on the named branch (default `main`). 1 approving review, dismiss stale, no force push, no delete, conversation resolution required. |
| `its gh branch-protect show <repo>` | Show current branch-protection settings for <owner/repo>@<branch>. Read-only — no mutation. |
| `its gh branch-protect remove <repo>` | Remove ALL branch protection from <owner/repo>@<branch> (default `main`). Permanent — use --confirm. This does not restore the previous rules; re-apply with `branch-protect apply`. |

#### `its gh branch-protect apply <repo>`

Apply the THF standard branch-protection block to <owner/repo> on the named branch (default `main`). 1 approving review, dismiss stale, no force push, no delete, conversation resolution required.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--branch` | `` | Branch to protect | main |
| `--dry-run` | `` | Print the planned PUT body without sending | — |

**Examples:**

```bash
its gh branch-protect apply acme/storefront --dry-run

its gh branch-protect apply acme/storefront --branch main
```

#### `its gh branch-protect show <repo>`

Show current branch-protection settings for <owner/repo>@<branch>. Read-only — no mutation.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--branch` | `` | Branch | main |

```bash
its gh branch-protect show <repo>
```

#### `its gh branch-protect remove <repo>`

Remove ALL branch protection from <owner/repo>@<branch> (default `main`). Permanent — use --confirm. This does not restore the previous rules; re-apply with `branch-protect apply`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--branch` | `` | Branch to unprotect (default main) | main |
| `--confirm` | `` | Required to perform this destructive removal | — |

**Examples:**

```bash
its gh branch-protect remove acme/storefront --confirm
```

---

### webhook

| Command | Description |
|---------|-------------|
| `its gh webhook setup <repo> <url>` | Create a push-event webhook on <owner/repo> pointing at <url>. Idempotent — bails if a hook with the same URL already exists. |
| `its gh webhook <repo>` | List webhooks on <owner/repo> |
| `its gh webhook delete <repo> <hook>` | Delete a webhook from <owner/repo> by id (from `webhook list`) or by its exact URL. Permanent — use --confirm. Deliveries stop immediately; GitHub keeps no undo. |

#### `its gh webhook setup <repo> <url>`

Create a push-event webhook on <owner/repo> pointing at <url>. Idempotent — bails if a hook with the same URL already exists.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--secret` | `` | Webhook secret (defaults to a random secret, printed once - save it) | — |
| `--events` | `` | Comma-separated events (default: push) | push |

**Examples:**

```bash
# Idempotent — bails if a hook with that URL already exists
its gh webhook setup acme/storefront https://dok.example.com/api/deploy --events push
```

#### `its gh webhook <repo>`

List webhooks on <owner/repo>.

```bash
its gh webhook <repo>
```

#### `its gh webhook delete <repo> <hook>`

Delete a webhook from <owner/repo> by id (from `webhook list`) or by its exact URL. Permanent — use --confirm. Deliveries stop immediately; GitHub keeps no undo.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Required to perform this destructive deletion | — |

**Examples:**

```bash
its gh webhook delete acme/storefront 12345678 --confirm

its gh webhook delete acme/storefront https://dok.example.com/api/deploy --confirm
```

---
