# Dokploy (`dokploy`)

Dokploy self-hosted PaaS management — projects, apps, databases, deployments, domains, registries, notifications.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [projects](#projects)
- [apps](#apps)
- [databases](#databases)
- [deployments](#deployments)
- [domains](#domains)
- [env](#env)
- [environments](#environments)
- [registries](#registries)
- [destinations](#destinations)
- [notifications](#notifications)
- [dashboard](#dashboard)
- [mounts](#mounts)
- [webhook](#webhook)
- [nodes](#nodes)
- [cluster](#cluster)
- [containers](#containers)
- [maintenance](#maintenance)
- [traefik](#traefik)
- [github](#github)
- [users](#users)
- [orgs](#orgs)
- [compose](#compose)
- [backup](#backup)
- [schedule](#schedule)
- [git](#git)

## Setup

```bash
its dokploy setup           # Interactive wizard
its dokploy setup --check   # Check configuration status
its dokploy setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `DOKPLOY_URL` | Dokploy instance URL (e.g. `https://dokploy.example.com`) |
| `DOKPLOY_TOKEN` | API token from Dokploy dashboard > Settings > API Tokens |

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/dokploy/client.ts` | API client methods |
| `src/providers/dokploy/types.ts` | TypeScript interfaces |
| `src/providers/dokploy/commands/` | Command definitions (split by resource) |
| `src/providers/dokploy/definition.ts` | definition |
| `src/providers/dokploy/resolve.ts` | resolve |
| `src/providers/dokploy/runtime.ts` | runtime |
| `src/providers/dokploy/ssh.ts` | ssh |

## Resources

### projects

> Source: `src/providers/dokploy/commands/projects.ts`

| Command | Description |
|---------|-------------|
| `its dokploy projects` | Every project in this Dokploy server, with a per-project tally of applications and databases. The starting point for navigating most other dokploy commands. |
| `its dokploy projects get <projectId>` | Full project detail — owner, env, plus the contained apps/databases/composes. Use the project ID from `dokploy projects`. |
| `its dokploy projects create` | Create an empty Dokploy project. Apps and databases are added afterwards via `dokploy apps create` / `databases create`. |
| `its dokploy projects delete <projectId>` | Permanently delete a project AND every app/database it contains. Destructive — needs --confirm. |

#### `its dokploy projects`

Every project in this Dokploy server, with a per-project tally of applications and databases. The starting point for navigating most other dokploy commands.

**Examples:**

```bash
its dokploy projects

its dokploy projects --json

# Re-runs every 10s — handy for dashboards or incident response.
its dokploy projects --watch
```

#### `its dokploy projects get <projectId>`

Full project detail — owner, env, plus the contained apps/databases/composes. Use the project ID from `dokploy projects`.

**Examples:**

```bash
its dokploy projects get <project-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy projects get <project-id> --json
```

#### `its dokploy projects create`

Create an empty Dokploy project. Apps and databases are added afterwards via `dokploy apps create` / `databases create`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `-n` | Project name | — |
| `--description` | `-d` | Project description | — |

**Examples:**

```bash
its dokploy projects create --name "my-app"

# Pipe-friendly output — use with jq / scripts.
its dokploy projects create --name "my-app" --json
```

#### `its dokploy projects delete <projectId>`

Permanently delete a project AND every app/database it contains. Destructive — needs --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm deletion | — |

**Examples:**

```bash
its dokploy projects delete <project-id> --confirm
```

---

### apps

> Source: `src/providers/dokploy/commands/apps.ts`

| Command | Description |
|---------|-------------|
| `its dokploy apps` | Every application across every project — name, project, status, last-deploy. Filter with --project to scope to one project. |
| `its dokploy apps get <applicationId>` | Get full detail for an application — source config, mounts, env keys, replicas, last deploy, and live container image |
| `its dokploy apps create` | Scaffold an empty application inside a project. Wire it to a source (git, docker, dockerfile) afterwards via `dokploy apps set-source`. |
| `its dokploy apps delete <applicationId>` | Permanently delete an application — container, build artefacts, env. Destructive — needs --confirm. The project remains. |
| `its dokploy apps deploy <app>` | Trigger deployment, optionally wait for healthy and show logs |
| `its dokploy apps stop <applicationId>` | Stop the running container without deleting it. Restart via `dokploy apps start`. |
| `its dokploy apps start <applicationId>` | Start a previously stopped container. Idempotent — no-op if already running. |
| `its dokploy apps restart <applicationId>` | Stop + start in one call. Doesn't rebuild — use `dokploy apps deploy` if the image needs to change. |
| `its dokploy apps set-source <app>` | Wire an application to a source provider. --type github links a GitHub App repo; --type docker pins to a registry image. |
| `its dokploy apps set-build <app>` | Set build type for an application (dockerfile, nixpacks, heroku_buildpacks, paketo_buildpacks, static, railpack). Dockerfile path/context configurable. |
| `its dokploy apps rebuild <app>` | Force a fresh build from source (clones, builds image, deploys). Distinct from `redeploy` (re-uses last image) and `deploy` which is the alias for this. Internally maps to application.deploy — `application.rebuild` returns 404. |
| `its dokploy apps wait-deploy <app>` | Poll an application's deployments until the latest (or --since <id>) transitions from running to done/error. Exit code reflects the final state: 0 on done, 1 on error or timeout. Suitable for GitHub Actions. See ctxc 238. |
| `its dokploy apps redeploy <applicationId>` | Redeploy an application without rebuilding. Redeploys the existing container; doesn't rebuild from source. |
| `its dokploy apps logs <app>` | Show container logs for an application via Dokploy API (no SSH). Pass --follow to stream live (SSH-based) or --build to read the docker build log (the build log is the only place where 'image build failed' errors are visible). |
| `its dokploy apps monitoring <app>` | Resource-usage snapshot for a running app — CPU%, memory MB, disk, network rx/tx, block I/O. Empty arrays mean Dokploy hasn't sampled the container yet (first sample takes ~1 min). |
| `its dokploy apps traefik <app>` | Show the Traefik routing config (router + service + middlewares) Dokploy generated for the application. Useful for debugging 404/SSL/redirect issues at the proxy layer. |
| `its dokploy apps status <app>` | Show container state, uptime, restart count, image, and domain |
| `its dokploy apps clone <source> <newName>` | Clone an application — copies Docker settings, env vars, and creates a domain |
| `its dokploy apps shell <app> [cmd]` | Open an interactive shell in the running container via SSH + docker exec. Pass [cmd] for a one-shot (e.g. 'bun run db:migrate'). Defaults to sh. |
| `its dokploy apps migrate <app>` | Run a migration command inside the running app container. Defaults to `bun run db:migrate`. Lighter wrapper than `apps shell <id> '<cmd>'` — useful for ad-hoc migrations on apps that don't auto-migrate on boot. |
| `its dokploy apps env-runtime <app>` | Read the env vars actually present in the running container (via docker exec) — the ground truth, not the saved record. Use this to catch the Swarm bug where saved vars never reach the container (ctxc 1549). Keys shown; values redacted unless --show-values. Pass --diff to compare against the saved record. |
| `its dokploy apps apply-env <app>` | Make the saved env actually reach the container. On Docker Swarm a plain redeploy does NOT converge env-only changes (ctxc 1549) — this stops the app (removes the swarm service), redeploys so Dokploy recreates it from the current env, then verifies every saved var is live. Causes a brief outage. |
| `its dokploy apps health` | Lightweight health sweep across ALL apps — replicas / last deploy / domain reachable. One row per app. Use `apps doctor <id>` for the deep dive on a specific app. |
| `its dokploy apps env-drift` | Fleet sweep comparing each app's SAVED env record against its LIVE runtime env (via docker exec printenv). Flags apps where saved vars never reached the container — the generalised Swarm env-freeze bug (ctxc 1549). One row per app. Fix a flagged app with `its dokploy apps apply-env <id>`. |
| `its dokploy apps doctor <app>` | Run a battery of parallel checks (env, mounts, webhook, replicas, image, healthcheck, recent log scan, last build). One green/red checklist instead of probing five separate commands. |
| `its dokploy apps bootstrap <name>` | Bootstrap a new Dokploy app end-to-end: resolves or creates the project, creates the application, wires the GitHub source, sets the build type, pushes env vars, creates the domain, and triggers the first deploy. Idempotent — re-running with the same --name skips already-completed steps. Use --dry-run first. |
| `its dokploy apps cert-status <app>` | Read Traefik acme.json for the app's domains — surfaces certificate state, expiry, last LE error. SSH-backed (reads from the dokploy-traefik container). |

#### `its dokploy apps`

Every application across every project — name, project, status, last-deploy. Filter with --project to scope to one project.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--project` | `-p` | Filter by project name | — |

**Examples:**

```bash
# Every app across every project
its dokploy apps

its dokploy apps --project ccd-platform

its dokploy apps --ai | its dokploy apps health --stdin
```

#### `its dokploy apps get <applicationId>`

Get full detail for an application — source config, mounts, env keys, replicas, last deploy, and live container image.

**Examples:**

```bash
its dokploy apps get <app-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy apps get <app-id> --json
```

#### `its dokploy apps create`

Scaffold an empty application inside a project. Wire it to a source (git, docker, dockerfile) afterwards via `dokploy apps set-source`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--project` | `-p` | Project ID | — |
| `--name` | `-n` | Application display name | — |
| `--appName` | `` | Docker app name (auto-slugified from name if omitted) | — |
| `--description` | `-d` | Application description | — |

**Examples:**

```bash
its dokploy apps create --project <project-id> --name "my-api"

# Pipe-friendly output — use with jq / scripts.
its dokploy apps create --project <project-id> --name "my-api" --json
```

#### `its dokploy apps delete <applicationId>`

Permanently delete an application — container, build artefacts, env. Destructive — needs --confirm. The project remains.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm deletion | — |

**Examples:**

```bash
its dokploy apps delete <app-id> --confirm
```

#### `its dokploy apps deploy <app>`

Trigger deployment, optionally wait for healthy and show logs.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--wait` | `` | Wait for container to become healthy | — |
| `--logs` | `` | Show last 50 lines of logs after deploy | — |

**Examples:**

```bash
its dokploy apps deploy <app-id>

its dokploy apps deploy <app-id> --wait
```

#### `its dokploy apps stop <applicationId>`

Stop the running container without deleting it. Restart via `dokploy apps start`.

**Examples:**

```bash
its dokploy apps stop <app-id>
```

#### `its dokploy apps start <applicationId>`

Start a previously stopped container. Idempotent — no-op if already running.

**Examples:**

```bash
its dokploy apps start <app-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy apps start <app-id> --json
```

#### `its dokploy apps restart <applicationId>`

Stop + start in one call. Doesn't rebuild — use `dokploy apps deploy` if the image needs to change.

**Examples:**

```bash
its dokploy apps restart <app-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy apps restart <app-id> --json
```

#### `its dokploy apps set-source <app>`

Wire an application to a source provider. --type github links a GitHub App repo; --type docker pins to a registry image.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `` | Source type | — |
| `--github-id` | `` | Dokploy GitHub provider ID (its dokploy git list) | — |
| `--owner` | `` | GitHub owner / org | — |
| `--repo` | `` | GitHub repository name | — |
| `--branch` | `` | Branch (default main) | main |
| `--build-path` | `` | Path inside the repo to use as build context (default /) | / |
| `--watch-paths` | `` | Comma-separated glob list to limit auto-deploy triggers (default ** = any change) | — |
| `--image` | `` | Docker image reference (for --type docker) | — |
| `--username` | `` | Registry username (optional, for private registries) | — |
| `--password` | `` | Registry password / token (optional) | — |
| `--registry-url` | `` | Registry URL — e.g. ghcr.io or docker.io. Leave blank for Docker Hub. | — |
| `--dry-run` | `` | Print the planned request without sending | — |

**Examples:**

```bash
its dokploy apps set-source <app-id> --type github --repo owner/repo --branch main

# Pipe-friendly output — use with jq / scripts.
its dokploy apps set-source <app-id> --type github --repo owner/repo --branch main --json
```

#### `its dokploy apps set-build <app>`

Set build type for an application (dockerfile, nixpacks, heroku_buildpacks, paketo_buildpacks, static, railpack). Dockerfile path/context configurable.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `` | Build type | — |
| `--dockerfile` | `` | Path to the Dockerfile (default Dockerfile) | Dockerfile |
| `--context` | `` | Docker build context (default .) | . |
| `--stage` | `` | Multi-stage build target stage | — |
| `--publish-dir` | `` | Static SPA publish directory (type=static only) | — |
| `--dry-run` | `` | Print the planned request without sending | — |

**Examples:**

```bash
its dokploy apps set-build <app-id> --type nixpacks

its dokploy apps set-build <app-id> --type dockerfile --dockerfile "./Dockerfile"
```

#### `its dokploy apps rebuild <app>`

Force a fresh build from source (clones, builds image, deploys). Distinct from `redeploy` (re-uses last image) and `deploy` which is the alias for this. Internally maps to application.deploy — `application.rebuild` returns 404.

**Examples:**

```bash
# Distinct from deploy — clones, builds image, deploys
its dokploy apps rebuild <app-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy apps rebuild <app-id> --json
```

#### `its dokploy apps wait-deploy <app>`

Poll an application's deployments until the latest (or --since <id>) transitions from running to done/error. Exit code reflects the final state: 0 on done, 1 on error or timeout. Suitable for GitHub Actions. See ctxc 238.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--timeout` | `` | Timeout in seconds (default 600) | 600 |
| `--interval` | `` | Poll interval in seconds (default 5) | 5 |
| `--since` | `` | Wait for this specific deployment ID. Avoids the race where two deploys fire close together and the latest snapshot changes mid-poll. | — |

**Examples:**

```bash
its dokploy apps wait-deploy <app-id> --timeout 600

# Pipe-friendly output — use with jq / scripts.
its dokploy apps wait-deploy <app-id> --timeout 600 --json
```

#### `its dokploy apps redeploy <applicationId>`

Redeploy an application without rebuilding. Redeploys the existing container; doesn't rebuild from source.

**Examples:**

```bash
its dokploy apps redeploy <app-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy apps redeploy <app-id> --json
```

#### `its dokploy apps logs <app>`

Show container logs for an application via Dokploy API (no SSH). Pass --follow to stream live (SSH-based) or --build to read the docker build log (the build log is the only place where 'image build failed' errors are visible).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--follow` | `-f` | Follow log output (uses SSH stream) | — |
| `--tail` | `` | Number of lines to show | 100 |
| `--since` | `` | Show logs since duration (e.g. 1h, 30m). Only honoured by --follow / --build paths — readLogs API ignores it. | — |
| `--build` | `` | Read the docker build log (/etc/dokploy/logs/<appName>/*.log) instead of the running container log | — |
| `--ssh` | `` | Force SSH path even without --follow (legacy fallback) | — |

**Examples:**

```bash
its dokploy apps logs <app-id>

its dokploy apps logs <app-id> --tail 200
```

#### `its dokploy apps monitoring <app>`

Resource-usage snapshot for a running app — CPU%, memory MB, disk, network rx/tx, block I/O. Empty arrays mean Dokploy hasn't sampled the container yet (first sample takes ~1 min).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--samples` | `` | Latest N samples to show (default 5) | 5 |

**Examples:**

```bash
its dokploy apps monitoring <app-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy apps monitoring <app-id> --json
```

#### `its dokploy apps traefik <app>`

Show the Traefik routing config (router + service + middlewares) Dokploy generated for the application. Useful for debugging 404/SSL/redirect issues at the proxy layer.

**Examples:**

```bash
its dokploy apps traefik <app-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy apps traefik <app-id> --json
```

#### `its dokploy apps status <app>`

Show container state, uptime, restart count, image, and domain.

**Examples:**

```bash
its dokploy apps status <app-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy apps status <app-id> --json

# Re-runs every 10s — handy for dashboards or incident response.
its dokploy apps status <app-id> --watch
```

#### `its dokploy apps clone <source> <newName>`

Clone an application — copies Docker settings, env vars, and creates a domain.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--domain` | `` | Domain hostname for the new app | — |
| `--image` | `` | Override the Docker image | — |
| `--env-override` | `` | Override env var (KEY=VALUE, repeatable via comma-separated) | — |

**Examples:**

```bash
its dokploy apps clone <source-id> "my-api-staging"

# Pipe-friendly output — use with jq / scripts.
its dokploy apps clone <source-id> "my-api-staging" --json
```

#### `its dokploy apps shell <app> [cmd]`

Open an interactive shell in the running container via SSH + docker exec. Pass [cmd] for a one-shot (e.g. 'bun run db:migrate'). Defaults to sh.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--shell` | `` | Shell to use when no [cmd] is given (default sh) | sh |

**Examples:**

```bash
# Opens xterm dock — bash inside the container
its dokploy apps shell <app-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy apps shell <app-id> --json
```

#### `its dokploy apps migrate <app>`

Run a migration command inside the running app container. Defaults to `bun run db:migrate`. Lighter wrapper than `apps shell <id> '<cmd>'` — useful for ad-hoc migrations on apps that don't auto-migrate on boot.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--cmd` | `` | Migration command (default `bun run db:migrate`). Use `npm run db:migrate` etc. for non-bun projects. | bun run db:migrate |

**Examples:**

```bash
# Defaults to `bun run db:migrate` inside the container
its dokploy apps migrate <app-id>

its dokploy apps migrate <app-id> --cmd "bun run db:seed"
```

#### `its dokploy apps env-runtime <app>`

Read the env vars actually present in the running container (via docker exec) — the ground truth, not the saved record. Use this to catch the Swarm bug where saved vars never reach the container (ctxc 1549). Keys shown; values redacted unless --show-values. Pass --diff to compare against the saved record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--show-values` | `` | Show actual values (otherwise redacted as ***) | — |
| `--diff` | `` | Compare the saved record against the runtime — flags vars in the record but missing from the container (un-applied) and runtime-only vars | — |

```bash
its dokploy apps env-runtime <app>
```

#### `its dokploy apps apply-env <app>`

Make the saved env actually reach the container. On Docker Swarm a plain redeploy does NOT converge env-only changes (ctxc 1549) — this stops the app (removes the swarm service), redeploys so Dokploy recreates it from the current env, then verifies every saved var is live. Causes a brief outage.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--timeout` | `` | Deploy timeout in seconds (default 600) | 600 |

```bash
its dokploy apps apply-env <app>
```

#### `its dokploy apps health`

Lightweight health sweep across ALL apps — replicas / last deploy / domain reachable. One row per app. Use `apps doctor <id>` for the deep dive on a specific app.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--project` | `-p` | Filter by project name | — |
| `--fail-only` | `` | Show only apps with non-OK verdict | — |

**Examples:**

```bash
# Container status + healthcheck result
its dokploy apps health <app-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy apps health <app-id> --json

# Re-runs every 10s — handy for dashboards or incident response.
its dokploy apps health <app-id> --watch
```

#### `its dokploy apps env-drift`

Fleet sweep comparing each app's SAVED env record against its LIVE runtime env (via docker exec printenv). Flags apps where saved vars never reached the container — the generalised Swarm env-freeze bug (ctxc 1549). One row per app. Fix a flagged app with `its dokploy apps apply-env <id>`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--project` | `-p` | Filter by project name | — |
| `--drift-only` | `` | Show only apps where the record and container diverge | — |

**Examples:**

```bash
its dokploy apps env-drift

its dokploy apps env-drift --project ccd-platform

its dokploy apps env-drift --drift-only
```

#### `its dokploy apps doctor <app>`

Run a battery of parallel checks (env, mounts, webhook, replicas, image, healthcheck, recent log scan, last build). One green/red checklist instead of probing five separate commands.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--no-ssh` | `` | Skip checks that need SSH (host-path verify, build/service log scan) | — |

**Examples:**

```bash
# Walks deploy logs, traefik, env, source. Flags issues.
its dokploy apps doctor <app-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy apps doctor <app-id> --json
```

#### `its dokploy apps bootstrap <name>`

Bootstrap a new Dokploy app end-to-end: resolves or creates the project, creates the application, wires the GitHub source, sets the build type, pushes env vars, creates the domain, and triggers the first deploy. Idempotent — re-running with the same --name skips already-completed steps. Use --dry-run first.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--repo` | `` | GitHub source as <owner/repo> | — |
| `--branch` | `` | Git branch (default main) | main |
| `--domain` | `` | Domain to assign (e.g. app.contractcandles.com) | — |
| `--port` | `` | Container port the app listens on (default 3000) | 3000 |
| `--env-file` | `` | Path to .env file to push | — |
| `--project` | `` | Project name or id (default: same as --name; created if missing) | — |
| `--github-id` | `` | Dokploy GitHub provider id (default: first provider from `github providers`) | — |
| `--dockerfile` | `` | Dockerfile path inside the repo (default Dockerfile) | Dockerfile |
| `--context` | `` | Docker build context (default .) | . |
| `--build-type` | `` | Build type (default dockerfile) | dockerfile |
| `--no-deploy` | `` | Skip the final rebuild/deploy step | — |
| `--dry-run` | `` | Print the plan without sending any mutations | — |

```bash
its dokploy apps bootstrap <name>
```

#### `its dokploy apps cert-status <app>`

Read Traefik acme.json for the app's domains — surfaces certificate state, expiry, last LE error. SSH-backed (reads from the dokploy-traefik container).

```bash
its dokploy apps cert-status <app>
```

---

### databases

> Source: `src/providers/dokploy/commands/databases.ts`

| Command | Description |
|---------|-------------|
| `its dokploy databases` | List all databases across projects. Surfaces the most common fields; pass --json for raw shape. |
| `its dokploy databases url <id>` | Compose a connection URL for a database — uses the docker-network appName by default (sibling apps in the same project), or external host:port with --external. URL is printed plain to stdout for `eval` / `export DATABASE_URL=$(its dokploy databases url ...)`. |
| `its dokploy databases get` | Get full details for a database (appName, credentials, port) |
| `its dokploy databases create` | Create a database (postgres, mysql, mariadb, mongo, or redis) |
| `its dokploy databases deploy <databaseId>` | Deploy a database instance. Trigger a fresh deploy from the wired source. |
| `its dokploy databases stop <databaseId>` | Stop a database instance. Stop the resource. Use --confirm if the action is destructive. |
| `its dokploy databases delete <databaseId>` | Delete a database instance. Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record. |

#### `its dokploy databases`

List all databases across projects. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its dokploy databases

# Pipe-friendly output — use with jq / scripts.
its dokploy databases --json

# Re-runs every 10s — handy for dashboards or incident response.
its dokploy databases --watch
```

#### `its dokploy databases url <id>`

Compose a connection URL for a database — uses the docker-network appName by default (sibling apps in the same project), or external host:port with --external. URL is printed plain to stdout for `eval` / `export DATABASE_URL=$(its dokploy databases url ...)`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `-t` | Database type: postgres, mysql, mariadb, mongo, redis | — |
| `--external` | `` | Compose using the externally-mapped port instead of the docker-network appName (requires the DB to have an external port configured) | — |
| `--host` | `` | Override hostname (default: appName for internal, dok host for external) | — |

**Examples:**

```bash
# Print connection string (mask in shared terminals)
its dokploy databases url <db-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy databases url <db-id> --json
```

#### `its dokploy databases get`

Get full details for a database (appName, credentials, port).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `-t` | Database type: postgres, mysql, mariadb, mongo, redis | — |

**Examples:**

```bash
its dokploy databases get <db-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy databases get <db-id> --json
```

#### `its dokploy databases create`

Create a database (postgres, mysql, mariadb, mongo, or redis).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--project` | `-p` | Project ID | — |
| `--type` | `-t` | Database type: postgres, mysql, mariadb, mongo, redis | — |
| `--name` | `-n` | Database display name | — |
| `--dbName` | `` | Internal database name | — |
| `--dbUser` | `` | Database user (default varies by type) | — |
| `--dbPassword` | `` | Database password | — |
| `--image` | `` | Docker image (default varies by type) | — |
| `--description` | `-d` | Description | — |

**Examples:**

```bash
its dokploy databases create --type postgres --name "app-db" --project <project-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy databases create --type postgres --name "app-db" --project <project-id> --json
```

#### `its dokploy databases deploy <databaseId>`

Deploy a database instance. Trigger a fresh deploy from the wired source.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `-t` | Database type: postgres, mysql, mariadb, mongo, redis | — |

**Examples:**

```bash
its dokploy databases deploy <db-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy databases deploy <db-id> --json
```

#### `its dokploy databases stop <databaseId>`

Stop a database instance. Stop the resource. Use --confirm if the action is destructive.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `-t` | Database type: postgres, mysql, mariadb, mongo, redis | — |

**Examples:**

```bash
its dokploy databases stop <db-id>
```

#### `its dokploy databases delete <databaseId>`

Delete a database instance. Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `-t` | Database type: postgres, mysql, mariadb, mongo, redis | — |
| `--confirm` | `` | Confirm deletion | — |

**Examples:**

```bash
its dokploy databases delete <db-id> --confirm
```

---

### deployments

> Source: `src/providers/dokploy/commands/deployments.ts`

| Command | Description |
|---------|-------------|
| `its dokploy deployments <applicationId>` | List deployments for an application. Surfaces the most common fields; pass --json for raw shape. |
| `its dokploy deployments queue` | Show pending/active deployments across the whole Dokploy server (not scoped to one app). Useful when a deploy seems stuck — empty queue means the worker is idle. |
| `its dokploy deployments kill <deploymentId>` | Kill an in-flight deployment by ID (sends SIGTERM to the build process) |

#### `its dokploy deployments <applicationId>`

List deployments for an application. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
# Last N deploys with status + duration
its dokploy deployments <app-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy deployments <app-id> --json

# Re-runs every 10s — handy for dashboards or incident response.
its dokploy deployments <app-id> --watch
```

#### `its dokploy deployments queue`

Show pending/active deployments across the whole Dokploy server (not scoped to one app). Useful when a deploy seems stuck — empty queue means the worker is idle.

**Examples:**

```bash
its dokploy deployments queue

# Pipe-friendly output — use with jq / scripts.
its dokploy deployments queue --json
```

#### `its dokploy deployments kill <deploymentId>`

Kill an in-flight deployment by ID (sends SIGTERM to the build process).

**Examples:**

```bash
# Sends SIGTERM to the build process
its dokploy deployments kill <deployment-id>
```

---

### domains

> Source: `src/providers/dokploy/commands/domains.ts`

| Command | Description |
|---------|-------------|
| `its dokploy domains <applicationId>` | List domains for an application. Surfaces the most common fields; pass --json for raw shape. |
| `its dokploy domains create <applicationId>` | Add a domain to an application. Idempotent on duplicate names — use update/edit to mutate an existing record. |
| `its dokploy domains check [host]` | Inspect the live TLS certificate for a domain — issuer, validity window, days-to-expiry, SAN list. Hits the host directly, no Dokploy API. Use without args to check ALL domains across all apps. |
| `its dokploy domains delete <domainId>` | Remove a domain. Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record. |

#### `its dokploy domains <applicationId>`

List domains for an application. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its dokploy domains <app-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy domains <app-id> --json

# Re-runs every 10s — handy for dashboards or incident response.
its dokploy domains <app-id> --watch
```

#### `its dokploy domains create <applicationId>`

Add a domain to an application. Idempotent on duplicate names — use update/edit to mutate an existing record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--host` | `` | Domain hostname (e.g. app.example.com) | — |
| `--port` | `` | Container port (default: 3000) | — |
| `--https` | `` | Enable HTTPS (default: true) | — |
| `--cert` | `` | Certificate type (default: letsencrypt) | — |

**Examples:**

```bash
its dokploy domains create <app-id> --host "app.example.com" --port 3000

# Pipe-friendly output — use with jq / scripts.
its dokploy domains create <app-id> --host "app.example.com" --port 3000 --json
```

#### `its dokploy domains check [host]`

Inspect the live TLS certificate for a domain — issuer, validity window, days-to-expiry, SAN list. Hits the host directly, no Dokploy API. Use without args to check ALL domains across all apps.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--warn-days` | `` | Days remaining below which to flag warn (default 30) | 30 |

**Examples:**

```bash
# Issuer, validity window, days-to-expiry
its dokploy domains check "app.example.com"

# Pipe-friendly output — use with jq / scripts.
its dokploy domains check "app.example.com" --json
```

#### `its dokploy domains delete <domainId>`

Remove a domain. Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm deletion | — |

**Examples:**

```bash
its dokploy domains delete <domain-id> --confirm
```

---

### env

> Source: `src/providers/dokploy/commands/env.ts`

| Command | Description |
|---------|-------------|
| `its dokploy env <applicationId>` | Show environment variables for an application. Keys are visible by default; values are redacted unless --show-values is set. |
| `its dokploy env push <applicationId>` | Push an env file to an application. Upload local state to the upstream. Pass --dry-run to preview which keys would be written (no values, nothing saved). |
| `its dokploy env set <applicationId> <pairs>` | Set one or more env vars (KEY=value) without affecting others. NB: a plain set updates the record only — on Docker Swarm the running container will NOT pick the change up until the service is recreated. Pass --deploy to recreate + verify (brief outage), or run `dokploy apps apply-env <app>` later. Pass --dry-run to preview which keys would change (no values, nothing saved). |
| `its dokploy env unset <applicationId> <keys>` | Remove one or more env vars by key, preserving all others. The inverse of `env set`. Record-only unless --deploy is passed. Pass --dry-run to preview which keys would be removed (nothing saved). |
| `its dokploy env pull <applicationId>` | Pull env vars from an application to a local file. Download upstream state to local. |
| `its dokploy env copy <srcApp> <dstApp>` | Securely copy env vars from one app to another. Reads the REAL values from <srcApp> and writes them to <dstApp> — the values NEVER touch stdout, JSON, or the transcript. The safe way to move secrets like GITHUB_APP_* between staging and prod. Pick keys with --keys K1,K2 or take everything with --all. Honours the redaction-mask guard (won't copy a masked value). Pass --dry-run to preview the set/unchanged/skipped classification without writing. |
| `its dokploy env reveal <applicationId> <key>` | Read ONE secret value to a 0600 file or the OS clipboard — never to stdout. For when a human genuinely needs a single value (e.g. paste into a GUI) without it landing in shell history or the transcript. stdout shows only `wrote <KEY> (<n> bytes) to <sink>`. Exactly one sink required. PEM values arrive single-line `\n`-escaped — unescape with `.replace(/\n/g,"\n")`. |

#### `its dokploy env <applicationId>`

Show environment variables for an application. Keys are visible by default; values are redacted unless --show-values is set.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--show-values` | `` | Show actual values (otherwise redacted as ***) | — |

**Examples:**

```bash
its dokploy env <app-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy env <app-id> --json

# Re-runs every 10s — handy for dashboards or incident response.
its dokploy env <app-id> --watch
```

#### `its dokploy env push <applicationId>`

Push an env file to an application. Upload local state to the upstream. Pass --dry-run to preview which keys would be written (no values, nothing saved).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--file` | `-f` | Path to env file | .env |
| `--force` | `` | Allow pushing an empty file (wipes ALL env on the app). Required because Dokploy keeps no env history — an accidental empty push is unrecoverable. | — |
| `--force-mask` | `` | Override the redaction-mask guard and write values like `***REDACTED***` verbatim. Almost never what you want — the guard exists to stop a masked round-trip clobbering real secrets (ctxc 1705). | — |

**Examples:**

```bash
its dokploy env push <app-id> --file .env.production

# Pipe-friendly output — use with jq / scripts.
its dokploy env push <app-id> --file .env.production --json
```

#### `its dokploy env set <applicationId> <pairs>`

Set one or more env vars (KEY=value) without affecting others. NB: a plain set updates the record only — on Docker Swarm the running container will NOT pick the change up until the service is recreated. Pass --deploy to recreate + verify (brief outage), or run `dokploy apps apply-env <app>` later. Pass --dry-run to preview which keys would change (no values, nothing saved).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--deploy` | `` | After saving, recreate the swarm service (stop + deploy) and verify the new vars reached the container. Causes a brief outage. Without this, the change only lands in the record. | — |
| `--force-mask` | `` | Override the redaction-mask guard and write values like `***REDACTED***` verbatim. Almost never what you want — the guard exists to stop a masked round-trip clobbering real secrets (ctxc 1705). | — |

**Examples:**

```bash
its dokploy env set <app-id> DEBUG=true

# Pipe-friendly output — use with jq / scripts.
its dokploy env set <app-id> DEBUG=true --json
```

#### `its dokploy env unset <applicationId> <keys>`

Remove one or more env vars by key, preserving all others. The inverse of `env set`. Record-only unless --deploy is passed. Pass --dry-run to preview which keys would be removed (nothing saved).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--deploy` | `` | After saving, recreate the swarm service (stop + deploy) so the removal takes effect. Causes a brief outage. | — |

```bash
its dokploy env unset <applicationId> <keys>
```

#### `its dokploy env pull <applicationId>`

Pull env vars from an application to a local file. Download upstream state to local.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--file` | `-f` | Output file path | .env |

**Examples:**

```bash
its dokploy env pull <app-id> --file .env.local

# Pipe-friendly output — use with jq / scripts.
its dokploy env pull <app-id> --file .env.local --json
```

#### `its dokploy env copy <srcApp> <dstApp>`

Securely copy env vars from one app to another. Reads the REAL values from <srcApp> and writes them to <dstApp> — the values NEVER touch stdout, JSON, or the transcript. The safe way to move secrets like GITHUB_APP_* between staging and prod. Pick keys with --keys K1,K2 or take everything with --all. Honours the redaction-mask guard (won't copy a masked value). Pass --dry-run to preview the set/unchanged/skipped classification without writing.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--keys` | `` | Comma-separated list of keys to copy (e.g. K1,K2,K3) | — |
| `--all` | `` | Copy every key from the source app | — |
| `--deploy` | `` | After writing, recreate the destination's swarm service so the new vars go live. Causes a brief outage on the destination. | — |
| `--force-mask` | `` | Override the redaction-mask guard and write values like `***REDACTED***` verbatim. Almost never what you want — the guard exists to stop a masked round-trip clobbering real secrets (ctxc 1705). | — |

```bash
its dokploy env copy <srcApp> <dstApp>
```

#### `its dokploy env reveal <applicationId> <key>`

Read ONE secret value to a 0600 file or the OS clipboard — never to stdout. For when a human genuinely needs a single value (e.g. paste into a GUI) without it landing in shell history or the transcript. stdout shows only `wrote <KEY> (<n> bytes) to <sink>`. Exactly one sink required. PEM values arrive single-line `\n`-escaped — unescape with `.replace(/\n/g,"\n")`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--to-file` | `` | Write the value to this path (created 0600 / owner-only) | — |
| `--to-clipboard` | `` | Copy the value to the OS clipboard instead of a file | — |
| `--clear-after` | `` | Seconds before the clipboard is wiped (0 disables). Only meaningful with --to-clipboard. | 30 |

```bash
its dokploy env reveal <applicationId> <key>
```

---

### environments

> Source: `src/providers/dokploy/commands/environments.ts`

| Command | Description |
|---------|-------------|
| `its dokploy environments <projectId>` | List a project's environments with their environmentId, name and shared-variable count. Pass the projectId. |
| `its dokploy environments env <environmentId>` | Show an environment's SHARED env vars. Keys are visible by default; values are redacted unless --show-values. Pass the environmentId (from `its dokploy environments <projectId>`). |
| `its dokploy environments push <environmentId>` | Push an env file as an environment's SHARED env vars (replaces all). Upload local state to the upstream. |
| `its dokploy environments pull <environmentId>` | Pull an environment's SHARED env vars to a local file. Download upstream state to local. |
| `its dokploy environments set <environmentId> <pairs>` | Set one or more SHARED env vars (KEY=value) on an environment without affecting the others. |

#### `its dokploy environments <projectId>`

List a project's environments with their environmentId, name and shared-variable count. Pass the projectId.

```bash
its dokploy environments <projectId>
```

#### `its dokploy environments env <environmentId>`

Show an environment's SHARED env vars. Keys are visible by default; values are redacted unless --show-values. Pass the environmentId (from `its dokploy environments <projectId>`).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--show-values` | `` | Show actual values (otherwise redacted as ***) | — |

```bash
its dokploy environments env <environmentId>
```

#### `its dokploy environments push <environmentId>`

Push an env file as an environment's SHARED env vars (replaces all). Upload local state to the upstream.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--file` | `-f` | Path to env file | .env |
| `--force` | `` | Allow pushing an empty file (wipes ALL shared env on the environment). Required because Dokploy keeps no env history — an accidental empty push is unrecoverable. | — |

```bash
its dokploy environments push <environmentId>
```

#### `its dokploy environments pull <environmentId>`

Pull an environment's SHARED env vars to a local file. Download upstream state to local.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--file` | `-f` | Output file path | .env |

```bash
its dokploy environments pull <environmentId>
```

#### `its dokploy environments set <environmentId> <pairs>`

Set one or more SHARED env vars (KEY=value) on an environment without affecting the others.

```bash
its dokploy environments set <environmentId> <pairs>
```

---

### registries

> Source: `src/providers/dokploy/commands/infrastructure.ts`

| Command | Description |
|---------|-------------|
| `its dokploy registries` | List configured container registries. Surfaces the most common fields; pass --json for raw shape. |

#### `its dokploy registries`

List configured container registries. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its dokploy registries

# Pipe-friendly output — use with jq / scripts.
its dokploy registries --json

# Re-runs every 10s — handy for dashboards or incident response.
its dokploy registries --watch
```

---

### destinations

> Source: `src/providers/dokploy/commands/infrastructure.ts`

| Command | Description |
|---------|-------------|
| `its dokploy destinations` | List configured backup destinations (S3/Wasabi). Surfaces the most common fields; pass --json for raw shape. |

#### `its dokploy destinations`

List configured backup destinations (S3/Wasabi). Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its dokploy destinations

# Pipe-friendly output — use with jq / scripts.
its dokploy destinations --json

# Re-runs every 10s — handy for dashboards or incident response.
its dokploy destinations --watch
```

---

### notifications

> Source: `src/providers/dokploy/commands/infrastructure.ts`

| Command | Description |
|---------|-------------|
| `its dokploy notifications` | List configured notification channels. Surfaces the most common fields; pass --json for raw shape. |

#### `its dokploy notifications`

List configured notification channels. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its dokploy notifications

# Pipe-friendly output — use with jq / scripts.
its dokploy notifications --json

# Re-runs every 10s — handy for dashboards or incident response.
its dokploy notifications --watch
```

---

### dashboard

> Source: `src/providers/dokploy/commands/dashboard.ts`

| Command | Description |
|---------|-------------|
| `its dokploy dashboard` | Overview of all projects, applications, and databases. Surfaces the most common fields; pass --json for raw shape. |

#### `its dokploy dashboard`

Overview of all projects, applications, and databases. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
# Apps, projects, databases counts + last deploy times
its dokploy dashboard

# Pipe-friendly output — use with jq / scripts.
its dokploy dashboard --json

# Re-runs every 10s — handy for dashboards or incident response.
its dokploy dashboard --watch
```

---

### mounts

> Source: `src/providers/dokploy/commands/mounts.ts`

| Command | Description |
|---------|-------------|
| `its dokploy mounts <app>` | List mounts (bind/volume/file) for an application. Surfaces the most common fields; pass --json for raw shape. |
| `its dokploy mounts add <app>` | Add a mount to an application (--type bind|volume|file). Use --ensure-host-path with bind to create the host directory. |
| `its dokploy mounts update <mountId>` | Update an existing mount. PATCH semantics — only the supplied fields change. |
| `its dokploy mounts remove <mountId>` | Remove a mount. Permanent — use --confirm. |

#### `its dokploy mounts <app>`

List mounts (bind/volume/file) for an application. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its dokploy mounts <app-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy mounts <app-id> --json

# Re-runs every 10s — handy for dashboards or incident response.
its dokploy mounts <app-id> --watch
```

#### `its dokploy mounts add <app>`

Add a mount to an application (--type bind|volume|file). Use --ensure-host-path with bind to create the host directory.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `-t` | Mount type | — |
| `--src` | `` | Source: host path (bind), volume name (volume), or initial content path (file) | — |
| `--dst` | `` | Mount path inside the container | — |
| `--content` | `` | Inline file content (--type file) | — |
| `--content-file` | `` | Read --content from a local UTF-8 file (use for content > ~15KB — Windows command-line cap) | — |
| `--ensure-host-path` | `` | For --type bind: create the host directory on the swarm node and chown to uid 1000 (avoids 'bind source path does not exist' deploy failures). Requires SSH. | — |

**Examples:**

```bash
its dokploy mounts add <app-id> --type bind --src /data --dst /app/data

# Pipe-friendly output — use with jq / scripts.
its dokploy mounts add <app-id> --type bind --src /data --dst /app/data --json
```

#### `its dokploy mounts update <mountId>`

Update an existing mount. PATCH semantics — only the supplied fields change.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--src` | `` | New source (host path / volume name / file path) | — |
| `--dst` | `` | New mount path inside the container | — |
| `--content` | `` | New inline file content (file mounts) | — |
| `--content-file` | `` | Read --content from a local UTF-8 file (use for content > ~15KB — Windows command-line cap) | — |

**Examples:**

```bash
its dokploy mounts update <mount-id> --src "/data" --dst "/app/data"

# Pipe-friendly output — use with jq / scripts.
its dokploy mounts update <mount-id> --src "/data" --dst "/app/data" --json
```

#### `its dokploy mounts remove <mountId>`

Remove a mount. Permanent — use --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm removal | — |

**Examples:**

```bash
its dokploy mounts remove <mount-id> --confirm
```

---

### webhook

> Source: `src/providers/dokploy/commands/webhook.ts`

| Command | Description |
|---------|-------------|
| `its dokploy webhook check <app>` | Diff the Dokploy expected webhook against the GitHub repo's actual hooks. Catches the autoDeploy:true-but-no-webhook drift. Detects GitHub App installations (no per-repo hook needed). |
| `its dokploy webhook setup <app>` | Create the GitHub webhook on the repo wired to the Dokploy auto-deploy endpoint. Idempotent — if a hook with the same URL already exists, it's left alone. |
| `its dokploy webhook <app>` | List GitHub webhooks on the repo backing this application. Surfaces the most common fields; pass --json for raw shape. |

#### `its dokploy webhook check <app>`

Diff the Dokploy expected webhook against the GitHub repo's actual hooks. Catches the autoDeploy:true-but-no-webhook drift. Detects GitHub App installations (no per-repo hook needed).

**Examples:**

```bash
# Test GitHub webhook end-to-end
its dokploy webhook check <app-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy webhook check <app-id> --json
```

#### `its dokploy webhook setup <app>`

Create the GitHub webhook on the repo wired to the Dokploy auto-deploy endpoint. Idempotent — if a hook with the same URL already exists, it's left alone.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--secret` | `` | Webhook secret (defaults to a stable string per applicationId) | — |

**Examples:**

```bash
its dokploy webhook setup <app-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy webhook setup <app-id> --json
```

#### `its dokploy webhook <app>`

List GitHub webhooks on the repo backing this application. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its dokploy webhook list <app-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy webhook list <app-id> --json

# Re-runs every 10s — handy for dashboards or incident response.
its dokploy webhook list <app-id> --watch
```

---

### nodes

> Source: `src/providers/dokploy/commands/nodes.ts`

| Command | Description |
|---------|-------------|
| `its dokploy nodes` | List Docker Swarm nodes (host, availability, role, engine version) |
| `its dokploy nodes info <nodeId>` | Detailed info for a single swarm node (CPU, memory, labels, status) |
| `its dokploy nodes apps` | List swarm services (replicated apps) running across nodes |
| `its dokploy nodes stats` | Live container resource stats across all swarm containers (CPU%, MemUsage, BlockIO, NetIO) |

#### `its dokploy nodes`

List Docker Swarm nodes (host, availability, role, engine version).

**Examples:**

```bash
its dokploy nodes

# Pipe-friendly output — use with jq / scripts.
its dokploy nodes --json

# Re-runs every 10s — handy for dashboards or incident response.
its dokploy nodes --watch
```

#### `its dokploy nodes info <nodeId>`

Detailed info for a single swarm node (CPU, memory, labels, status).

**Examples:**

```bash
its dokploy nodes info <node-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy nodes info <node-id> --json
```

#### `its dokploy nodes apps`

List swarm services (replicated apps) running across nodes.

**Examples:**

```bash
its dokploy nodes apps

# Pipe-friendly output — use with jq / scripts.
its dokploy nodes apps --json
```

#### `its dokploy nodes stats`

Live container resource stats across all swarm containers (CPU%, MemUsage, BlockIO, NetIO).

**Examples:**

```bash
its dokploy nodes stats

# Pipe-friendly output — use with jq / scripts.
its dokploy nodes stats --json
```

---

### cluster

> Source: `src/providers/dokploy/commands/nodes.ts`

| Command | Description |
|---------|-------------|
| `its dokploy cluster join-token` | Get the `docker swarm join` command for adding a manager or worker node. Pass --role manager|worker (default worker). |

#### `its dokploy cluster join-token`

Get the `docker swarm join` command for adding a manager or worker node. Pass --role manager|worker (default worker).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--role` | `` | Node role: manager or worker | worker |

**Examples:**

```bash
its dokploy cluster join-token --role worker

its dokploy cluster join-token --role manager
```

---

### containers

> Source: `src/providers/dokploy/commands/containers.ts`

| Command | Description |
|---------|-------------|
| `its dokploy containers` | List ALL containers on the Dokploy host (not scoped to app). Surfaces the most common fields; pass --json for raw shape. |
| `its dokploy containers config <containerId>` | Get full Docker config for a container (env, mounts, network, labels) |
| `its dokploy containers start <containerId>` | Start a container by ID (not app name). Lower-level than `apps start` — operates on a single container without re-deploying. |
| `its dokploy containers stop <containerId>` | Stop a container by ID (not app name). Lower-level than `apps stop` — operates on a single container without re-deploying. |
| `its dokploy containers restart <containerId>` | Restart a container by ID (not app name). Lower-level than `apps restart` — operates on a single container without re-deploying. |
| `its dokploy containers kill <containerId>` | Kill a container by ID (not app name). Lower-level than `apps kill` — operates on a single container without re-deploying. Destructive — needs --confirm. |
| `its dokploy containers remove <containerId>` | Remove a container by ID (not app name). Lower-level than `apps stop` — operates on a single container without re-deploying. Destructive — needs --confirm. |

#### `its dokploy containers`

List ALL containers on the Dokploy host (not scoped to app). Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
# Every container on the host, not just app-scoped
its dokploy containers

# Pipe-friendly output — use with jq / scripts.
its dokploy containers --json

# Re-runs every 10s — handy for dashboards or incident response.
its dokploy containers --watch
```

#### `its dokploy containers config <containerId>`

Get full Docker config for a container (env, mounts, network, labels).

**Examples:**

```bash
its dokploy containers config <container-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy containers config <container-id> --json
```

#### `its dokploy containers start <containerId>`

Start a container by ID (not app name). Lower-level than `apps start` — operates on a single container without re-deploying.

**Examples:**

```bash
its dokploy containers start <container-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy containers start <container-id> --json
```

#### `its dokploy containers stop <containerId>`

Stop a container by ID (not app name). Lower-level than `apps stop` — operates on a single container without re-deploying.

**Examples:**

```bash
its dokploy containers stop <container-id>
```

#### `its dokploy containers restart <containerId>`

Restart a container by ID (not app name). Lower-level than `apps restart` — operates on a single container without re-deploying.

**Examples:**

```bash
its dokploy containers restart <container-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy containers restart <container-id> --json
```

#### `its dokploy containers kill <containerId>`

Kill a container by ID (not app name). Lower-level than `apps kill` — operates on a single container without re-deploying. Destructive — needs --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm this destructive action | — |

**Examples:**

```bash
its dokploy containers kill <container-id> --confirm
```

#### `its dokploy containers remove <containerId>`

Remove a container by ID (not app name). Lower-level than `apps stop` — operates on a single container without re-deploying. Destructive — needs --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm this destructive action | — |

**Examples:**

```bash
its dokploy containers remove <container-id> --confirm
```

---

### maintenance

> Source: `src/providers/dokploy/commands/maintenance.ts`

| Command | Description |
|---------|-------------|
| `its dokploy maintenance disk` | Docker disk usage on the Dokploy host (Images / Containers / Volumes / Build Cache) including reclaimable space |
| `its dokploy maintenance web` | Dokploy web-server settings (server IP, HTTPS, certificate type, host) |
| `its dokploy maintenance traefik` | Global Traefik dynamic config (Dokploy-wide, not per-app). Use `apps traefik` for a single app's routing. |
| `its dokploy maintenance clean` | Reclaim disk on the Dokploy host. --target images|volumes|containers|builder|monitoring|all (destructive — requires --confirm). Default target is `images` (safest). |
| `its dokploy maintenance time` | Server time + timezone (sanity check for cron drift / TLS errors) |

#### `its dokploy maintenance disk`

Docker disk usage on the Dokploy host (Images / Containers / Volumes / Build Cache) including reclaimable space.

**Examples:**

```bash
# Images / containers / volumes / build cache
its dokploy maintenance disk

# Pipe-friendly output — use with jq / scripts.
its dokploy maintenance disk --json
```

#### `its dokploy maintenance web`

Dokploy web-server settings (server IP, HTTPS, certificate type, host).

**Examples:**

```bash
its dokploy maintenance web

# Pipe-friendly output — use with jq / scripts.
its dokploy maintenance web --json
```

#### `its dokploy maintenance traefik`

Global Traefik dynamic config (Dokploy-wide, not per-app). Use `apps traefik` for a single app's routing.

**Examples:**

```bash
its dokploy maintenance traefik

# Pipe-friendly output — use with jq / scripts.
its dokploy maintenance traefik --json
```

#### `its dokploy maintenance clean`

Reclaim disk on the Dokploy host. --target images|volumes|containers|builder|monitoring|all (destructive — requires --confirm). Default target is `images` (safest).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--target` | `` | What to clean | images |
| `--confirm` | `` | Confirm the cleanup | — |

**Examples:**

```bash
its dokploy maintenance clean --target builder --confirm

its dokploy maintenance clean --target images --confirm
```

#### `its dokploy maintenance time`

Server time + timezone (sanity check for cron drift / TLS errors).

**Examples:**

```bash
# Sanity check for cron drift / TLS errors
its dokploy maintenance time

# Pipe-friendly output — use with jq / scripts.
its dokploy maintenance time --json
```

---

### traefik

> Source: `src/providers/dokploy/commands/maintenance.ts`

| Command | Description |
|---------|-------------|
| `its dokploy traefik logs` | Tail the dokploy-traefik container logs over SSH. Useful for debugging 502/cert issues at the proxy layer. |
| `its dokploy traefik acme` | Read the Traefik acme.json from the dokploy-traefik container — surfaces every Let's Encrypt certificate the host knows about. Read-only over SSH. |

#### `its dokploy traefik logs`

Tail the dokploy-traefik container logs over SSH. Useful for debugging 502/cert issues at the proxy layer.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--tail` | `` | Lines to tail (default 200) | 200 |

```bash
its dokploy traefik logs
```

#### `its dokploy traefik acme`

Read the Traefik acme.json from the dokploy-traefik container — surfaces every Let's Encrypt certificate the host knows about. Read-only over SSH.

```bash
its dokploy traefik acme
```

---

### github

> Source: `src/providers/dokploy/commands/github.ts`

| Command | Description |
|---------|-------------|
| `its dokploy github providers` | List GitHub App installations configured in Dokploy |
| `its dokploy github repos <githubId>` | Repos visible to a GitHub App installation. Pass the `githubId` from `github providers` (NOT the inner `gitProviderId` — different IDs). Accepts either and auto-resolves. |
| `its dokploy github branches <owner> <repo>` | Branches in a GitHub repo (uses the GitHub App, no PAT needed) |

#### `its dokploy github providers`

List GitHub App installations configured in Dokploy.

**Examples:**

```bash
its dokploy github providers

# Pipe-friendly output — use with jq / scripts.
its dokploy github providers --json
```

#### `its dokploy github repos <githubId>`

Repos visible to a GitHub App installation. Pass the `githubId` from `github providers` (NOT the inner `gitProviderId` — different IDs). Accepts either and auto-resolves.

**Examples:**

```bash
its dokploy github repos <github-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy github repos <github-id> --json
```

#### `its dokploy github branches <owner> <repo>`

Branches in a GitHub repo (uses the GitHub App, no PAT needed).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--githubId` | `` | Specific Dokploy GitHub provider ID (optional; default first match) | — |

**Examples:**

```bash
its dokploy github branches owner repo

# Pipe-friendly output — use with jq / scripts.
its dokploy github branches owner repo --json
```

---

### users

> Source: `src/providers/dokploy/commands/users.ts`

| Command | Description |
|---------|-------------|
| `its dokploy users` | List users + roles in the active Dokploy organization. Surfaces the most common fields; pass --json for raw shape. |
| `its dokploy users me` | Current user (the API token's owner) + org context + role |
| `its dokploy users invite` | Invite a user to the active Dokploy organization by email (user.invitation) |
| `its dokploy users remove <userId>` | Remove a user from the active Dokploy organization (requires --confirm) |

#### `its dokploy users`

List users + roles in the active Dokploy organization. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its dokploy users

# Pipe-friendly output — use with jq / scripts.
its dokploy users --json

# Re-runs every 10s — handy for dashboards or incident response.
its dokploy users --watch
```

#### `its dokploy users me`

Current user (the API token's owner) + org context + role.

**Examples:**

```bash
its dokploy users me

# Pipe-friendly output — use with jq / scripts.
its dokploy users me --json
```

#### `its dokploy users invite`

Invite a user to the active Dokploy organization by email (user.invitation).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--email` | `` | Invitee email address (required) | — |
| `--role` | `` | Initial role | user |

**Examples:**

```bash
its dokploy users invite --email jane.smith@example.com

# Pipe-friendly output — use with jq / scripts.
its dokploy users invite --email jane.smith@example.com --json
```

#### `its dokploy users remove <userId>`

Remove a user from the active Dokploy organization (requires --confirm).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the removal | — |

**Examples:**

```bash
its dokploy users remove <user-id> --confirm
```

---

### orgs

> Source: `src/providers/dokploy/commands/users.ts`

| Command | Description |
|---------|-------------|
| `its dokploy orgs` | Organizations the API key has access to. Surfaces the most common fields; pass --json for raw shape. |
| `its dokploy orgs active` | The currently active organization for the API key. Returns whichever record the API key currently targets. |

#### `its dokploy orgs`

Organizations the API key has access to. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its dokploy orgs

# Pipe-friendly output — use with jq / scripts.
its dokploy orgs --json

# Re-runs every 10s — handy for dashboards or incident response.
its dokploy orgs --watch
```

#### `its dokploy orgs active`

The currently active organization for the API key. Returns whichever record the API key currently targets.

**Examples:**

```bash
its dokploy orgs active

# Returns ownerId + slug — handy for scripted multi-org tooling.
its dokploy orgs active --json
```

---

### compose

> Source: `src/providers/dokploy/commands/compose.ts`

| Command | Description |
|---------|-------------|
| `its dokploy compose get <composeId>` | Get a docker-compose stack by composeId. Pass the id (or any natural identifier) as the positional arg. |
| `its dokploy compose services <composeId>` | List services declared inside a compose stack (compose.loadServices) |
| `its dokploy compose config <composeId>` | Show the resolved docker-compose YAML Dokploy will deploy (after env substitution) |
| `its dokploy compose deploy <composeId>` | Trigger a deploy on a compose stack. Trigger a fresh deploy from the wired source. |
| `its dokploy compose stop <composeId>` | Stop a compose stack. Stop the resource. Use --confirm if the action is destructive. |
| `its dokploy compose redeploy <composeId>` | Redeploy a compose stack (rebuild + deploy). Redeploys the existing container; doesn't rebuild from source. |
| `its dokploy compose delete <composeId>` | Delete a compose stack permanently (requires --confirm). Stops the stack first, then removes the record from Dokploy. |

#### `its dokploy compose get <composeId>`

Get a docker-compose stack by composeId. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its dokploy compose get <compose-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy compose get <compose-id> --json
```

#### `its dokploy compose services <composeId>`

List services declared inside a compose stack (compose.loadServices).

**Examples:**

```bash
its dokploy compose services <compose-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy compose services <compose-id> --json
```

#### `its dokploy compose config <composeId>`

Show the resolved docker-compose YAML Dokploy will deploy (after env substitution).

**Examples:**

```bash
# After env substitution
its dokploy compose config <compose-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy compose config <compose-id> --json
```

#### `its dokploy compose deploy <composeId>`

Trigger a deploy on a compose stack. Trigger a fresh deploy from the wired source.

**Examples:**

```bash
its dokploy compose deploy <compose-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy compose deploy <compose-id> --json
```

#### `its dokploy compose stop <composeId>`

Stop a compose stack. Stop the resource. Use --confirm if the action is destructive.

**Examples:**

```bash
its dokploy compose stop <compose-id>
```

#### `its dokploy compose redeploy <composeId>`

Redeploy a compose stack (rebuild + deploy). Redeploys the existing container; doesn't rebuild from source.

**Examples:**

```bash
its dokploy compose redeploy <compose-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy compose redeploy <compose-id> --json
```

#### `its dokploy compose delete <composeId>`

Delete a compose stack permanently (requires --confirm). Stops the stack first, then removes the record from Dokploy.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the delete | — |

**Examples:**

```bash
its dokploy compose delete <compose-id> --confirm
```

---

### backup

> Source: `src/providers/dokploy/commands/backup.ts`

| Command | Description |
|---------|-------------|
| `its dokploy backup create` | Create a scheduled backup config. Postgres example: its dokploy backup create --db-type postgres --db-id <postgresId> --schedule '0 2,14 * * *' --prefix prod-postgres/ --destination <destinationId> --database ccd --keep 60 |
| `its dokploy backup get <backupId>` | Get a backup config by id (schedule, destination, prefix). Pass the id (or any natural identifier) as the positional arg. |
| `its dokploy backup update <backupId>` | Update a backup config (schedule, prefix, keep count, enabled). Only provided flags change. e.g. its dokploy backup update <id> --schedule '0 1 * * *' --keep 7 |
| `its dokploy backup files <destinationId>` | List backup files present in a remote destination (S3/MinIO bucket etc.) |
| `its dokploy backup run <backupId>` | Trigger a manual backup run for a given backup config (backup.manualBackup) |
| `its dokploy backup delete <backupId>` | Delete a backup configuration (requires --confirm). Does NOT delete stored backup files on the remote destination. |

#### `its dokploy backup create`

Create a scheduled backup config. Postgres example: its dokploy backup create --db-type postgres --db-id <postgresId> --schedule '0 2,14 * * *' --prefix prod-postgres/ --destination <destinationId> --database ccd --keep 60.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--db-type` | `` | postgres | mariadb | mysql | mongo | libsql | web-server (default: postgres) | — |
| `--db-id` | `` | Parent resource id (postgresId/mysqlId/...). Omit for web-server. | — |
| `--schedule` | `` | Cron expression, e.g. '0 2,14 * * *' | — |
| `--prefix` | `` | Destination path prefix, e.g. prod-postgres/ | — |
| `--destination` | `` | Destination id (its dokploy destinations list) | — |
| `--database` | `` | Database name to dump (default: web-server uses dokploy) | — |
| `--keep` | `` | keepLatestCount — how many backups to retain | — |
| `--disabled` | `` | Create the config disabled (default: enabled) | — |

```bash
its dokploy backup create
```

#### `its dokploy backup get <backupId>`

Get a backup config by id (schedule, destination, prefix). Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its dokploy backup get <backup-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy backup get <backup-id> --json
```

#### `its dokploy backup update <backupId>`

Update a backup config (schedule, prefix, keep count, enabled). Only provided flags change. e.g. its dokploy backup update <id> --schedule '0 1 * * *' --keep 7.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--schedule` | `` | Cron expression | — |
| `--prefix` | `` | Destination path prefix | — |
| `--keep` | `` | keepLatestCount | — |
| `--enabled` | `` | Set enabled true|false | — |

```bash
its dokploy backup update <backupId>
```

#### `its dokploy backup files <destinationId>`

List backup files present in a remote destination (S3/MinIO bucket etc.).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--search` | `` | Path/prefix to list under (e.g. ccd-prod-postgres-tydlay). Empty lists the root. | — |

**Examples:**

```bash
its dokploy backup files <destination-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy backup files <destination-id> --json
```

#### `its dokploy backup run <backupId>`

Trigger a manual backup run for a given backup config (backup.manualBackup).

**Examples:**

```bash
its dokploy backup run <backup-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy backup run <backup-id> --json
```

#### `its dokploy backup delete <backupId>`

Delete a backup configuration (requires --confirm). Does NOT delete stored backup files on the remote destination.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the delete | — |

**Examples:**

```bash
# Doesn't delete stored backup files
its dokploy backup delete <backup-id> --confirm
```

---

### schedule

> Source: `src/providers/dokploy/commands/schedule.ts`

| Command | Description |
|---------|-------------|
| `its dokploy schedule <id>` | List cron schedules attached to an application, compose stack, or the server |
| `its dokploy schedule get <scheduleId>` | Get a schedule by id. Pass the id (or any natural identifier) as the positional arg. |
| `its dokploy schedule run <scheduleId>` | Trigger a schedule manually (schedule.runManually). Execute the named job/script. |
| `its dokploy schedule delete <scheduleId>` | Delete a schedule (requires --confirm). Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record. |

#### `its dokploy schedule <id>`

List cron schedules attached to an application, compose stack, or the server.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `` | Schedule type | application |

**Examples:**

```bash
its dokploy schedule <id>

# Pipe-friendly output — use with jq / scripts.
its dokploy schedule <id> --json

# Re-runs every 10s — handy for dashboards or incident response.
its dokploy schedule <id> --watch
```

#### `its dokploy schedule get <scheduleId>`

Get a schedule by id. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its dokploy schedule get <schedule-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy schedule get <schedule-id> --json
```

#### `its dokploy schedule run <scheduleId>`

Trigger a schedule manually (schedule.runManually). Execute the named job/script.

**Examples:**

```bash
its dokploy schedule run <schedule-id>

# Pipe-friendly output — use with jq / scripts.
its dokploy schedule run <schedule-id> --json
```

#### `its dokploy schedule delete <scheduleId>`

Delete a schedule (requires --confirm). Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the delete | — |

**Examples:**

```bash
its dokploy schedule delete <schedule-id> --confirm
```

---

### git

> Source: `src/providers/dokploy/commands/git.ts`

| Command | Description |
|---------|-------------|
| `its dokploy git` | List all configured git providers across GitHub Apps, GitLab OAuth apps, and Bitbucket workspaces |
| `its dokploy git setup` | Set up a new git provider. GitHub is UI-only (prints dashboard URL); GitLab/Bitbucket use credentials passed via flags. |
| `its dokploy git delete <id>` | Remove a git provider. Pass --type so the right delete endpoint is used. |

#### `its dokploy git`

List all configured git providers across GitHub Apps, GitLab OAuth apps, and Bitbucket workspaces.

**Examples:**

```bash
its dokploy git

# Pipe-friendly output — use with jq / scripts.
its dokploy git --json

# Re-runs every 10s — handy for dashboards or incident response.
its dokploy git --watch
```

#### `its dokploy git setup`

Set up a new git provider. GitHub is UI-only (prints dashboard URL); GitLab/Bitbucket use credentials passed via flags.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `` | Git provider type | — |
| `--name` | `` | Display name (any string — shows in Dokploy UI) | — |
| `--url` | `` | GitLab instance URL (default https://gitlab.com) | — |
| `--application-id` | `` | GitLab OAuth Application ID | — |
| `--secret` | `` | GitLab OAuth Application Secret | — |
| `--redirect-uri` | `` | GitLab OAuth redirect URI (must match the app config). Defaults to <DOKPLOY_URL>/api/providers/gitlab/callback. | — |
| `--group` | `` | GitLab group name (optional, scopes to a group) | — |
| `--username` | `` | Bitbucket username (for type=bitbucket) | — |
| `--app-password` | `` | Bitbucket app password — generate at https://bitbucket.org/account/settings/app-passwords/ | — |
| `--workspace` | `` | Bitbucket workspace slug (optional, scopes repo discovery) | — |
| `--dry-run` | `` | Print the planned request without sending | — |

**Examples:**

```bash
# Prompts for client id/secret + redirect URI in the terminal pane.
its dokploy git setup --type gitlab

# GitHub uses a dashboard-driven OAuth dance — see `cf token request` flow instead.
its dokploy git setup --type bitbucket
```

#### `its dokploy git delete <id>`

Remove a git provider. Pass --type so the right delete endpoint is used.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `` | Git provider type | — |
| `--confirm` | `` | Confirm the delete | — |

**Examples:**

```bash
its dokploy git delete <id> --type gitlab --confirm
```

---
