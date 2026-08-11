# Tactical RMM (`rmm`)

Tactical RMM endpoint management — agents, alerts, software, services, updates, scripts, checks, tasks, policies.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [agents](#agents)
- [dashboard](#dashboard)
- [clients](#clients)
- [sites](#sites)
- [processes](#processes)
- [services](#services)
- [updates](#updates)
- [software](#software)
- [alerts](#alerts)
- [scripts](#scripts)
- [checks](#checks)
- [tasks](#tasks)
- [policies](#policies)
- [diagnostics](#diagnostics)
- [doctor](#doctor)
- [custom-fields](#custom-fields)

## Setup

```bash
its rmm setup           # Interactive wizard
its rmm setup --check   # Check configuration status
its rmm setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `TACTICAL_URL` | Tactical RMM API address; managed-worker setup offers the hosted service by default (not the dashboard address) |
| `TACTICAL_API_KEY` | Personal API key from RMM dashboard > Settings > Global Settings > API Keys. Select your own Tactical user; the key inherits that user's role. |

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/rmm/client.ts` | API client methods |
| `src/providers/rmm/types.ts` | TypeScript interfaces |
| `src/providers/rmm/commands/` | Command definitions (split by resource) |
| `src/providers/rmm/definition.ts` | definition |
| `src/providers/rmm/resolve.ts` | resolve |
| `src/providers/rmm/wrap-ps.ts` | wrap ps |

## Resources

### agents

> Source: `src/providers/rmm/commands/agents.ts`

| Command | Description |
|---------|-------------|
| `its rmm agents` | List all RMM agents with status, hostname, OS, site. Surfaces the most common fields; pass --json for raw shape. |
| `its rmm agents stale` | List agents overdue for 7+ days (stale/abandoned). Sorted oldest-first so the worst cases surface first. Use --days to tune the threshold. |
| `its rmm agents search <query>` | Fuzzy substring match across hostname, client, site, and logged-in username. Case-insensitive — useful when you only know part of a name. |
| `its rmm agents get <agent>` | Full agent detail — hardware, OS, last-seen, public IP, disks, version, maintenance state. Accepts hostname, username, or agent UUID. |
| `its rmm agents ping <agent>` | Live connectivity check — synchronously waits for the agent to respond. Returns last-seen + current online state. |
| `its rmm agents reboot <agent>` | Reboot the agent's host OS. Destructive — needs --confirm. Returns immediately; the host may take 1-3 minutes to come back. |
| `its rmm agents remove <agent>` | Permanently delete the agent record from RMM. Doesn't uninstall the local agent service — pair with the RMM uninstall script if the device is being decommissioned. |
| `its rmm agents prune` | Bulk-remove stale agent records last seen before a threshold (and never-seen agents). Destructive — previews the target list by default and needs --confirm to delete. Doesn't uninstall the local agent service. Pair with `agents stale` to inspect first. |
| `its rmm agents run <agent>` | Execute a one-shot shell command on the target agent. Returns stdout + stderr + exit code. Use --shell powershell|cmd|bash; default timeout is 30s. |
| `its rmm agents history <agent>` | Command + script execution history for one agent. Shows time, type, exit status, who ran it, and the truncated payload. |
| `its rmm agents notes <agent>` | Free-form notes attached to the agent — common workflow is documenting incident actions or device peculiarities. |
| `its rmm agents pending <agent>` | Queued reboots, script runs, and update installs that the agent hasn't picked up yet. Useful when a command seems to have stalled. |
| `its rmm agents wake <agent>` | Send a magic packet via another online agent on the same LAN. Doesn't work across VLANs / VPN — agent must be on a network with a reachable RMM peer. |
| `its rmm agents edit <agent>` | Reassign an agent's client/site or update description (PUT /agents/{id}/) |
| `its rmm agents refresh <agent>` | Trigger a WMI/sysinfo refresh on an agent — repopulates hardware fields (serial, model, etc.) |
| `its rmm agents eventlog <agent>` | Read a Windows event log (Application, System, or Security) from an agent over the last N days. |

#### `its rmm agents`

List all RMM agents with status, hostname, OS, site. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--status` | `-s` | Filter by status | — |
| `--type` | `-t` | Filter by type | — |
| `--client` | `-c` | Filter by client name | — |
| `--site` | `` | Filter by site name | — |
| `--rebooted-since` | `` | Only agents whose last boot is more recent than this — ISO timestamp or relative span (7d, 24h, 30m) | — |

**Examples:**

```bash
# Every agent, human-readable table
its rmm agents

# Only offline or overdue agents
its rmm agents --status offline

# Filter by site (case-insensitive substring)
its rmm agents --site "fernhurst"

# Agents that booted within the last 24 hours
its rmm agents --rebooted-since 24h

# Agents booted on or after a given date
its rmm agents --rebooted-since 2026-06-14

its rmm agents --ai | ai "which agents are overdue?"
```

#### `its rmm agents stale`

List agents overdue for 7+ days (stale/abandoned). Sorted oldest-first so the worst cases surface first. Use --days to tune the threshold.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--days` | `` | Minimum days since last seen (default 7) | 7 |

**Examples:**

```bash
# Agents that haven't checked in for 14+ days
its rmm agents stale

its rmm agents stale --days 30
```

#### `its rmm agents search <query>`

Fuzzy substring match across hostname, client, site, and logged-in username. Case-insensitive — useful when you only know part of a name.

**Examples:**

```bash
its rmm agents search "OFFICE-PC"

its rmm agents search "jane.smith"
```

#### `its rmm agents get <agent>`

Full agent detail — hardware, OS, last-seen, public IP, disks, version, maintenance state. Accepts hostname, username, or agent UUID.

**Examples:**

```bash
its rmm agents get OFFICE-PC-01

its rmm agents get jane.smith

its rmm agents get 12345678-1234-1234-1234-123456789abc
```

#### `its rmm agents ping <agent>`

Live connectivity check — synchronously waits for the agent to respond. Returns last-seen + current online state.

**Examples:**

```bash
its rmm agents ping OFFICE-PC-01

# Resolves agent by logged-in user with interactive disambiguation.
its rmm agents ping aaron.lock
```

#### `its rmm agents reboot <agent>`

Reboot the agent's host OS. Destructive — needs --confirm. Returns immediately; the host may take 1-3 minutes to come back.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the reboot | — |

**Examples:**

```bash
# Returns immediately — the host may take a few minutes to come back
its rmm agents reboot WKS-9 --confirm

# Requires --confirm. UI surfaces a destructive-action dialog before sending.
its rmm agents reboot OFFICE-PC-01 --confirm

# UUID accepted in place of hostname.
its rmm agents reboot c3e5f1a7-... --confirm
```

#### `its rmm agents remove <agent>`

Permanently delete the agent record from RMM. Doesn't uninstall the local agent service — pair with the RMM uninstall script if the device is being decommissioned.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the removal | — |

**Examples:**

```bash
# Deletes the RMM record only — the agent service stays installed on the machine
its rmm agents remove WKS-9 --confirm

# Destructive — needs --confirm. Agent record is gone permanently.
its rmm agents remove OFFICE-PC-01 --confirm
```

#### `its rmm agents prune`

Bulk-remove stale agent records last seen before a threshold (and never-seen agents). Destructive — previews the target list by default and needs --confirm to delete. Doesn't uninstall the local agent service. Pair with `agents stale` to inspect first.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--older-than` | `` | Age threshold — days (30) or a span (30d, 12h, 2w). Agents last seen before this are pruned. | — |
| `--confirm` | `` | Actually delete the matched agent records | — |

**Examples:**

```bash
# List agents last seen 90+ days ago (no delete without --confirm)
its rmm agents prune --older-than 90d

# Permanently remove those stale agent records
its rmm agents prune --older-than 90d --confirm

its rmm agents prune --older-than 6w --confirm
```

#### `its rmm agents run <agent>`

Execute a one-shot shell command on the target agent. Returns stdout + stderr + exit code. Use --shell powershell|cmd|bash; default timeout is 30s.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--shell` | `` | Shell type | powershell |
| `--cmd` | `` | Command to execute (use --file for multi-line scripts) | — |
| `--file` | `` | Path to a local script file (overrides --cmd). PowerShell multi-line scripts auto-wrapped via base64 + temp file. | — |
| `--raw` | `` | Disable auto-wrapping for multi-line PowerShell (sends as-is) | — |
| `--timeout` | `` | Timeout in seconds | 30 |
| `--as-user` | `` | Run in the active user's session instead of SYSTEM (requires a signed-in desktop user) | — |

**Examples:**

```bash
its rmm agents run WKS-9 --shell powershell --cmd "Get-Service spooler"

# Raise --timeout above the agent-side runtime — the HTTP read timeout is derived from it
its rmm agents run WKS-9 --shell powershell --cmd "gpupdate /force" --timeout 300

# Runs in the logged-on user's session instead of as SYSTEM. Fails if nobody is signed in
its rmm agents run WKS-9 --shell powershell --cmd "whoami" --as-user

its rmm agents run WKS-9 --shell powershell --file ./fix-printer.ps1

its rmm agents run OFFICE-PC-01 --shell powershell --cmd "Get-Process"

its rmm agents run OFFICE-PC-01 --shell cmd --cmd "ipconfig /all"

its rmm agents run LINUX-WEB-01 --shell bash --cmd "uname -a && uptime"
```

#### `its rmm agents history <agent>`

Command + script execution history for one agent. Shows time, type, exit status, who ran it, and the truncated payload.

**Examples:**

```bash
its rmm agents history OFFICE-PC-01
```

#### `its rmm agents notes <agent>`

Free-form notes attached to the agent — common workflow is documenting incident actions or device peculiarities.

**Examples:**

```bash
its rmm agents notes OFFICE-PC-01
```

#### `its rmm agents pending <agent>`

Queued reboots, script runs, and update installs that the agent hasn't picked up yet. Useful when a command seems to have stalled.

**Examples:**

```bash
its rmm agents pending OFFICE-PC-01
```

#### `its rmm agents wake <agent>`

Send a magic packet via another online agent on the same LAN. Doesn't work across VLANs / VPN — agent must be on a network with a reachable RMM peer.

**Examples:**

```bash
# Sends the magic packet via another online agent on the same LAN — will not cross VLANs
its rmm agents wake WKS-9

its rmm agents wake OFFICE-PC-01

# No --site flag — duplicate hostnames auto-resolve to the online/most-recent match, or pass the UUID directly.
its rmm agents wake c3e5f1a7-...
```

#### `its rmm agents edit <agent>`

Reassign an agent's client/site or update description (PUT /agents/{id}/).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--client` | `` | Client name or ID to move the agent into | — |
| `--site` | `` | Site name or ID to move the agent into (must belong to --client if both given) | — |
| `--description` | `` | New description text | — |

**Examples:**

```bash
its rmm agents edit WKS-9 --client "Acme" --site "Head Office"

its rmm agents edit WKS-9 --description "Finance workstation"

its rmm agents edit OFFICE-PC-01 --site <site-id>

its rmm agents edit OFFICE-PC-01 --description "Marketing — Jane's desk"
```

#### `its rmm agents refresh <agent>`

Trigger a WMI/sysinfo refresh on an agent — repopulates hardware fields (serial, model, etc.).

**Examples:**

```bash
# Repopulates hardware fields such as serial and model after a WMI cache goes stale
its rmm agents refresh WKS-9

# Repopulates hardware fields (serial, BIOS, etc.)
its rmm agents refresh OFFICE-PC-01
```

#### `its rmm agents eventlog <agent>`

Read a Windows event log (Application, System, or Security) from an agent over the last N days.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `` | Log type | Application |
| `--days` | `` | Look-back window in days | 7 |

```bash
its rmm agents eventlog <agent>
```

---

### dashboard

> Source: `src/providers/rmm/commands/dashboard.ts`

| Command | Description |
|---------|-------------|
| `its rmm dashboard` | Single-screen fleet health view — online/offline/overdue counts, server vs workstation, maintenance mode, reboot-pending, failing checks. Cheap to call; suitable for --watch. |

#### `its rmm dashboard`

Single-screen fleet health view — online/offline/overdue counts, server vs workstation, maintenance mode, reboot-pending, failing checks. Cheap to call; suitable for --watch.

**Examples:**

```bash
# Counts by site, status, OS
its rmm dashboard

# Re-runs every 10s — handy for incident-response screens.
its rmm dashboard --watch
```

---

### clients

> Source: `src/providers/rmm/commands/dashboard.ts`

| Command | Description |
|---------|-------------|
| `its rmm clients` | Top-level RMM client list with each client's sites flattened into one row. Use IDs from here as --client filter values elsewhere. |
| `its rmm clients create` | Create a client (POST /clients/). TRMM also creates its first site in the same call — pass --site for its name (default 'Default'). Idempotent — skips if a client of that name already exists. |
| `its rmm clients delete <client>` | Delete a client by ID or name (DELETE /clients/{id}/). --confirm required. If the client still has agents, pass --move-to-site <siteId> to reassign them first (TRMM refuses otherwise). |

#### `its rmm clients`

Top-level RMM client list with each client's sites flattened into one row. Use IDs from here as --client filter values elsewhere.

**Examples:**

```bash
its rmm clients

# Re-runs every 10s — handy for dashboards or incident response.
its rmm clients --watch
```

#### `its rmm clients create`

Create a client (POST /clients/). TRMM also creates its first site in the same call — pass --site for its name (default 'Default'). Idempotent — skips if a client of that name already exists.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | New client name | — |
| `--site` | `` | Name of the client's initial site (default 'Default') | Default |

**Examples:**

```bash
# TRMM requires an initial site — a client cannot exist without one
its rmm clients create --name "Acme" --site "Head Office"
```

#### `its rmm clients delete <client>`

Delete a client by ID or name (DELETE /clients/{id}/). --confirm required. If the client still has agents, pass --move-to-site <siteId> to reassign them first (TRMM refuses otherwise).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm deletion | — |
| `--move-to-site` | `` | Site ID to move this client's agents to before deleting | — |

**Examples:**

```bash
# Without --confirm, prints what would be deleted and stops
its rmm clients delete Acme

its rmm clients delete Acme --confirm

# Required when the client still has agents — TRMM refuses the delete otherwise
its rmm clients delete Acme --move-to-site 12 --confirm
```

---

### sites

> Source: `src/providers/rmm/commands/dashboard.ts`

| Command | Description |
|---------|-------------|
| `its rmm sites` | All RMM sites across every client, with per-site agent_count. Site IDs feed --site filters on agent commands. |
| `its rmm sites create` | Create a site under a client (POST /clients/sites/). Idempotent — skips if a site of that name already exists for the client. |
| `its rmm sites delete <site_id>` | Delete a site by ID (DELETE /clients/sites/{id}/). Fails if the site still has agents. --confirm required. |

#### `its rmm sites`

All RMM sites across every client, with per-site agent_count. Site IDs feed --site filters on agent commands.

**Examples:**

```bash
its rmm sites

# Only sites belonging to a named client (global --filter col=val — no dedicated --client flag).
its rmm sites --filter "client=Head Office"

# Re-runs every 10s — handy for dashboards or incident response.
its rmm sites --watch
```

#### `its rmm sites create`

Create a site under a client (POST /clients/sites/). Idempotent — skips if a site of that name already exists for the client.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--client` | `` | Client name or ID | — |
| `--name` | `` | New site name | — |

**Examples:**

```bash
its rmm sites create --client "Acme" --name "Warehouse"
```

#### `its rmm sites delete <site_id>`

Delete a site by ID (DELETE /clients/sites/{id}/). Fails if the site still has agents. --confirm required.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm deletion | — |

**Examples:**

```bash
its rmm sites delete 12 --confirm

# Lists every site with its ID — the site must have no agents left
its rmm sites
```

---

### processes

> Source: `src/providers/rmm/commands/processes.ts`

| Command | Description |
|---------|-------------|
| `its rmm processes <agent>` | Live process snapshot from the agent — sorted by CPU%, top 50. Use `processes top` instead when you only want the heavy hitters. |
| `its rmm processes top <agent>` | Top processes by CPU usage (live snapshot via PowerShell) |
| `its rmm processes kill <agent>` | Terminate a process by PID. Destructive — needs --confirm. Use `processes list` first to find the PID. |

#### `its rmm processes <agent>`

Live process snapshot from the agent — sorted by CPU%, top 50. Use `processes top` instead when you only want the heavy hitters.

**Examples:**

```bash
its rmm processes OFFICE-PC-01

its rmm processes OFFICE-PC-01 --filter name=chrome

# Re-runs every 10s — handy for dashboards or incident response.
its rmm processes OFFICE-PC-01 --watch
```

#### `its rmm processes top <agent>`

Top processes by CPU usage (live snapshot via PowerShell).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--count` | `` | Number of processes to show | 10 |

**Examples:**

```bash
its rmm processes top WKS-9 --count 10

# Top processes by CPU + memory
its rmm processes top OFFICE-PC-01
```

#### `its rmm processes kill <agent>`

Terminate a process by PID. Destructive — needs --confirm. Use `processes list` first to find the PID.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--pid` | `` | Process ID to kill | — |
| `--confirm` | `` | Confirm the kill | — |

**Examples:**

```bash
its rmm processes kill WKS-9 --pid 4821 --confirm

its rmm processes list WKS-9

its rmm processes kill OFFICE-PC-01 --pid 1234 --confirm

# No --name on kill — find the PID first, then its rmm processes kill <agent> --pid <pid> --confirm.
its rmm processes OFFICE-PC-01 --filter name=notepad
```

---

### services

> Source: `src/providers/rmm/commands/services.ts`

| Command | Description |
|---------|-------------|
| `its rmm services <agent>` | Windows / Linux service inventory for one agent. Filter with --status running|stopped|paused. |
| `its rmm services get <agent>` | Service detail — startup type, dependencies, current state, last exit code. |
| `its rmm services control <agent>` | Control a service (start/stop/restart). Stop requires --confirm. |
| `its rmm services enable <agent>` | Set startup type to Automatic and start the service if stopped. Idempotent. |
| `its rmm services disable <agent>` | Set a service startup type to Disabled (requires --confirm) |

#### `its rmm services <agent>`

Windows / Linux service inventory for one agent. Filter with --status running|stopped|paused.

**Examples:**

```bash
its rmm services OFFICE-PC-01

its rmm services OFFICE-PC-01 --filter status=running

# Quickly spot services that should be running.
its rmm services OFFICE-PC-01 --filter status=stopped
```

#### `its rmm services get <agent>`

Service detail — startup type, dependencies, current state, last exit code.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | Service name | — |

**Examples:**

```bash
its rmm services get OFFICE-PC-01 --name spooler
```

#### `its rmm services control <agent>`

Control a service (start/stop/restart). Stop requires --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | Service name | — |
| `--action` | `` | Action to perform | — |
| `--confirm` | `` | Required for stop action | — |

**Examples:**

```bash
its rmm services control WKS-9 --name spooler --action restart

its rmm services control WKS-9 --name spooler --action stop --confirm

its rmm services control OFFICE-PC-01 --name spooler --action restart

its rmm services control OFFICE-PC-01 --name spooler --action start
```

#### `its rmm services enable <agent>`

Set startup type to Automatic and start the service if stopped. Idempotent.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | Service name | — |

**Examples:**

```bash
# Sets startup to Automatic and starts it if stopped
its rmm services enable WKS-9 --name spooler

its rmm services enable OFFICE-PC-01 --name spooler
```

#### `its rmm services disable <agent>`

Set a service startup type to Disabled (requires --confirm).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | Service name | — |
| `--confirm` | `` | Confirm the disable | — |

**Examples:**

```bash
its rmm services disable WKS-9 --name spooler --confirm

its rmm services disable OFFICE-PC-01 --name spooler --confirm
```

---

### updates

> Source: `src/providers/rmm/commands/updates.ts`

| Command | Description |
|---------|-------------|
| `its rmm updates report` | Fleet Windows-Update compliance — fans out across agents and lists those with pending updates, most-critical-then-most-behind first, with separate critical/important columns. Narrow with --client/--site; --all also lists fully-patched agents. Pipe --csv/--json to export. One API call per agent, so it's slower than a single-agent query. |
| `its rmm updates <agent>` | Windows Update queue — pending, installed, failed. Shows KB, severity, install state, and the per-KB approval action (approve/ignore/nothing) — only `approve` updates get installed. |
| `its rmm updates scan [agent]` | Force a re-scan of Microsoft Update. Single agent, or fan out across a client/site/all-online to refresh the pending list before an `updates report`. |
| `its rmm updates install [agent]` | Install APPROVED pending Windows updates (TRMM installs only updates with action=approve — approve them first with `its rmm updates approve`). Single agent, or fan out across a client/site/all-online — both require --confirm (fan-out previews the targets without it). Single-agent install no-ops with a hint if nothing is approved. Reboot-after-install is controlled by the agent's patch policy (see `its rmm policies patch-policy`), not this command. |
| `its rmm updates approve <agent>` | Approve pending Windows updates so `updates install` will install them — TRMM installs ONLY approved updates, so an unapproved box is a no-op install. Target one --kb KB5034441 or --all-pending (the latter needs --confirm). |
| `its rmm updates defer <agent>` | Defer (ignore) pending Windows updates so install skips them. Target one --kb KB5034441 or --all-pending (the latter needs --confirm). Re-approve later with `updates approve`. |

#### `its rmm updates report`

Fleet Windows-Update compliance — fans out across agents and lists those with pending updates, most-critical-then-most-behind first, with separate critical/important columns. Narrow with --client/--site; --all also lists fully-patched agents. Pipe --csv/--json to export. One API call per agent, so it's slower than a single-agent query.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--client` | `` | Only agents in this client (substring) | — |
| `--site` | `` | Only agents in this site (substring) | — |
| `--all` | `` | Include fully-patched agents in the table | — |

**Examples:**

```bash
# Every agent with pending Windows updates, most-behind first
its rmm updates report

its rmm updates report --client "Candle Retail"

# Also list fully-patched agents
its rmm updates report --all
```

#### `its rmm updates <agent>`

Windows Update queue — pending, installed, failed. Shows KB, severity, install state, and the per-KB approval action (approve/ignore/nothing) — only `approve` updates get installed.

**Examples:**

```bash
# Windows Update queue
its rmm updates OFFICE-PC-01

# Re-runs every 10s — handy for dashboards or incident response.
its rmm updates OFFICE-PC-01 --watch
```

#### `its rmm updates scan [agent]`

Force a re-scan of Microsoft Update. Single agent, or fan out across a client/site/all-online to refresh the pending list before an `updates report`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--all-online` | `` | Fan out to every online agent | — |
| `--client` | `` | Fan out to online agents in this client (substring) | — |
| `--site` | `` | Fan out to online agents in this site (substring) | — |
| `--policy` | `` | Fan out to online agents under this automation policy id | — |
| `--confirm` | `` | Required to execute a fan-out (preview without it) | — |

**Examples:**

```bash
its rmm updates scan OFFICE-PC

# List the online agents a fan-out scan would hit
its rmm updates scan --client "Candle Retail"

its rmm updates scan --client "Candle Retail" --confirm

its rmm updates scan OFFICE-PC-01
```

#### `its rmm updates install [agent]`

Install APPROVED pending Windows updates (TRMM installs only updates with action=approve — approve them first with `its rmm updates approve`). Single agent, or fan out across a client/site/all-online — both require --confirm (fan-out previews the targets without it). Single-agent install no-ops with a hint if nothing is approved. Reboot-after-install is controlled by the agent's patch policy (see `its rmm policies patch-policy`), not this command.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--all-online` | `` | Fan out to every online agent | — |
| `--client` | `` | Fan out to online agents in this client (substring) | — |
| `--site` | `` | Fan out to online agents in this site (substring) | — |
| `--policy` | `` | Fan out to online agents under this automation policy id | — |
| `--confirm` | `` | Required to execute a fan-out (preview without it) | — |

**Examples:**

```bash
its rmm updates install OFFICE-PC --confirm

# List the online agents a fan-out install would hit (no install without --confirm)
its rmm updates install --client "Candle Retail"

its rmm updates install --client "Candle Retail" --confirm

its rmm updates install OFFICE-PC-01 --confirm
```

#### `its rmm updates approve <agent>`

Approve pending Windows updates so `updates install` will install them — TRMM installs ONLY approved updates, so an unapproved box is a no-op install. Target one --kb KB5034441 or --all-pending (the latter needs --confirm).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--kb` | `` | KB to act on (e.g. KB5034441 or 5034441) | — |
| `--all-pending` | `` | Act on every not-yet-installed update | — |
| `--confirm` | `` | Required for --all-pending | — |

**Examples:**

```bash
its rmm updates approve OFFICE-PC --kb KB5034441

its rmm updates approve OFFICE-PC --all-pending --confirm
```

#### `its rmm updates defer <agent>`

Defer (ignore) pending Windows updates so install skips them. Target one --kb KB5034441 or --all-pending (the latter needs --confirm). Re-approve later with `updates approve`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--kb` | `` | KB to act on (e.g. KB5034441 or 5034441) | — |
| `--all-pending` | `` | Act on every not-yet-installed update | — |
| `--confirm` | `` | Required for --all-pending | — |

**Examples:**

```bash
its rmm updates defer OFFICE-PC --kb KB5034441

its rmm updates defer OFFICE-PC --all-pending --confirm
```

---

### software

> Source: `src/providers/rmm/commands/software.ts`

| Command | Description |
|---------|-------------|
| `its rmm software <agent>` | Full installed-software inventory for one agent. Includes publisher, version, install date. Useful for compliance reports. |
| `its rmm software search <query>` | Fleet-wide title hunt — given a publisher or product name, find every agent that has it installed. Useful for incident response (e.g. find every box running a vulnerable version). |

#### `its rmm software <agent>`

Full installed-software inventory for one agent. Includes publisher, version, install date. Useful for compliance reports.

**Examples:**

```bash
its rmm software OFFICE-PC-01

# Re-runs every 10s — handy for dashboards or incident response.
its rmm software OFFICE-PC-01 --watch
```

#### `its rmm software search <query>`

Fleet-wide title hunt — given a publisher or product name, find every agent that has it installed. Useful for incident response (e.g. find every box running a vulnerable version).

**Examples:**

```bash
# Find every agent with Chrome installed
its rmm software search "chrome"

# Catches any title from that publisher.
its rmm software search "Adobe"
```

---

### alerts

> Source: `src/providers/rmm/commands/alerts.ts`

| Command | Description |
|---------|-------------|
| `its rmm alerts` | Active and resolved alerts across the fleet. Filter by --severity info|warning|error or --resolved. |
| `its rmm alerts get <alert_id>` | Alert detail — full message, source check, snooze state, agent it fired on. |
| `its rmm alerts resolve <alert_id>` | Mark an alert resolved (clears it from the active dashboard). Needs --confirm. |
| `its rmm alerts snooze <alert_id>` | Suppress an alert for N days (default 1). Needs --confirm. |
| `its rmm alerts unsnooze <alert_id>` | Lift a snooze early so the alert can fire again. Needs --confirm. |

#### `its rmm alerts`

Active and resolved alerts across the fleet. Filter by --severity info|warning|error or --resolved.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--severity` | `` | Filter by severity (info/warning/error) | — |
| `--resolved` | `` | Show resolved alerts | false |

**Examples:**

```bash
# Unresolved alerts are shown by default.
its rmm alerts

# Include resolved alerts (hidden by default).
its rmm alerts --resolved

# Severity: information | warning | error.
its rmm alerts --severity error
```

#### `its rmm alerts get <alert_id>`

Alert detail — full message, source check, snooze state, agent it fired on.

**Examples:**

```bash
its rmm alerts get <alert-id>
```

#### `its rmm alerts resolve <alert_id>`

Mark an alert resolved (clears it from the active dashboard). Needs --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the resolve | — |

**Examples:**

```bash
its rmm alerts resolve 4821 --confirm
```

#### `its rmm alerts snooze <alert_id>`

Suppress an alert for N days (default 1). Needs --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--days` | `` | Days to snooze for | 1 |
| `--confirm` | `` | Confirm the snooze | — |

**Examples:**

```bash
its rmm alerts snooze 4821 --days 7 --confirm
```

#### `its rmm alerts unsnooze <alert_id>`

Lift a snooze early so the alert can fire again. Needs --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the unsnooze | — |

**Examples:**

```bash
its rmm alerts unsnooze 4821 --confirm
```

---

### scripts

> Source: `src/providers/rmm/commands/scripts.ts`

| Command | Description |
|---------|-------------|
| `its rmm scripts` | All saved automation scripts in the RMM script library. Returns name, shell, category, default timeout. |
| `its rmm scripts get <script_id>` | Script detail — body, default args, category, hash, last edited timestamp. |
| `its rmm scripts run [agent]` | Execute a saved RMM script on a target agent, or fan it out across a fleet with --all-online/--client/--site/--policy (online agents only). Streams stdout/stderr back; use --timeout for long-running jobs (default 120s). Fan-out previews the target list and needs --confirm to execute. |
| `its rmm scripts upload-local <agent> <path>` | Upload a local .ps1/.sh/.py script to TRMM, run it on the agent, capture output, and delete the script afterwards. Use for ad-hoc local maintenance scripts. Pass --keep to leave the script registered. |
| `its rmm scripts delete <script_id>` | Permanently remove a script from the library. Destructive — needs --confirm. |
| `its rmm scripts upsert <name> <path>` | Idempotently push a local script to TRMM by name — creates if missing, updates if present (PUT). Works around the TRMM POST /scripts/ quirk where the response is a plain string, not the created object — we re-list to resolve the new ID. |

#### `its rmm scripts`

All saved automation scripts in the RMM script library. Returns name, shell, category, default timeout.

**Examples:**

```bash
its rmm scripts

# Re-runs every 10s — handy for dashboards or incident response.
its rmm scripts --watch
```

#### `its rmm scripts get <script_id>`

Script detail — body, default args, category, hash, last edited timestamp.

**Examples:**

```bash
its rmm scripts get <script-id>
```

#### `its rmm scripts run [agent]`

Execute a saved RMM script on a target agent, or fan it out across a fleet with --all-online/--client/--site/--policy (online agents only). Streams stdout/stderr back; use --timeout for long-running jobs (default 120s). Fan-out previews the target list and needs --confirm to execute.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--script` | `` | Script ID | — |
| `--args` | `` | Script arguments (comma-separated) | — |
| `--timeout` | `` | Timeout in seconds | 120 |
| `--raw` | `` | Print the script's raw stdout (and stderr) directly, instead of JSON-wrapped output with escaped \r\n (single-agent only) | — |
| `--all-online` | `` | Fan out to every online agent | — |
| `--client` | `` | Fan out to online agents in this client (substring) | — |
| `--site` | `` | Fan out to online agents in this site (substring) | — |
| `--policy` | `` | Fan out to online agents under this automation policy id | — |
| `--confirm` | `` | Required to execute a fan-out run (preview without it) | — |
| `--as-user` | `` | Run in the logged-on user's session instead of as SYSTEM. Needed to exercise per-user scripts; fails if nobody is signed in. | — |

**Examples:**

```bash
# Run saved script 12 on a single agent
its rmm scripts run OFFICE-PC --script 12

# Run a per-user script in the signed-in user's session
its rmm scripts run OFFICE-PC --script 435 --as-user

# List the online agents a fan-out would hit (no run without --confirm)
its rmm scripts run --all-online --script 12

# Execute across every online agent
its rmm scripts run --all-online --script 12 --confirm

# Fan out to one client's online agents
its rmm scripts run --client "Candle Retail" --script 12 --confirm

its rmm scripts run --site "Fernhurst" --script 12 --confirm

its rmm scripts run OFFICE-PC-01 --script "Restart Print Spooler"

# Default agent timeout is 90s — raise it for slow scripts.
its rmm scripts run OFFICE-PC-01 --script "Long Audit" --timeout 600
```

#### `its rmm scripts upload-local <agent> <path>`

Upload a local .ps1/.sh/.py script to TRMM, run it on the agent, capture output, and delete the script afterwards. Use for ad-hoc local maintenance scripts. Pass --keep to leave the script registered.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--shell` | `` | Script shell (defaults to auto-detect from extension: ps1=powershell, sh=shell, py=python, nu=nushell, ts=deno) | — |
| `--args` | `` | Script arguments (comma-separated) | — |
| `--timeout` | `` | Timeout in seconds (default 120) | 120 |
| `--keep` | `` | Leave the uploaded script registered in TRMM after execution | — |
| `--category` | `` | Category for the uploaded script (default 'Ad-hoc') | Ad-hoc |
| `--as-user` | `` | Run in the logged-on user's session instead of as SYSTEM. Fails if nobody is signed in. | — |

**Examples:**

```bash
# Uploads, runs, captures output, then deletes the script from TRMM — pass --keep to leave it
its rmm scripts upload-local WKS-9 ./Fix-Printer.ps1 --timeout 300

its rmm scripts upload-local WKS-9 ./Fix-Profile.ps1 --as-user

its rmm scripts upload-local ./fix-printers.ps1

its rmm scripts upload-local ./linux-housekeeping.sh --shell bash
```

#### `its rmm scripts delete <script_id>`

Permanently remove a script from the library. Destructive — needs --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the deletion | — |

**Examples:**

```bash
its rmm scripts delete 57 --confirm

# Lists the script library with IDs
its rmm scripts list

its rmm scripts delete <script-id> --confirm
```

#### `its rmm scripts upsert <name> <path>`

Idempotently push a local script to TRMM by name — creates if missing, updates if present (PUT). Works around the TRMM POST /scripts/ quirk where the response is a plain string, not the created object — we re-list to resolve the new ID.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--shell` | `` | Shell: powershell, cmd, shell, python, nushell, deno (auto from extension) | — |
| `--category` | `` | Category (default 'Ad-hoc') | Ad-hoc |
| `--description` | `` | Script description (overwritten on update) | — |
| `--timeout` | `` | Default timeout seconds (default 120) | 120 |
| `--args` | `` | Default args, comma-separated | — |
| `--run-as-user` | `` | Run in the logged-on user's session instead of as SYSTEM. Needed for anything touching %USERPROFILE%, HKCU or the user PATH — as SYSTEM those hit the SYSTEM profile. | — |

**Examples:**

```bash
its rmm scripts upsert 'Check Git Installed' C:/Scripts/Check-Git.ps1 --category 'Compliance Checks' --timeout 30

# Anything installing into a profile must run as the user, not SYSTEM
its rmm scripts upsert 'Configure User Profile' ./Set-UserProfile.ps1 --run-as-user

# Creates if missing, updates body if name exists
its rmm scripts upsert "Restart Spooler" ./fix-spooler.ps1
```

---

### checks

> Source: `src/providers/rmm/commands/checks.ts`

| Command | Description |
|---------|-------------|
| `its rmm checks <agent>` | Scheduled health checks attached to one agent. Includes last-run result + cadence. |
| `its rmm checks failing` | Fleet-wide failing-checks sweep — fans out across online agents and surfaces every check currently failing, worst-first. The fast 'what's red across the estate right now' view. Narrow with --client/--site. |
| `its rmm checks results <agent>` | Drill into one check's live result on an agent — status, last run, fail count, return code, and captured stdout/stderr. Pass --check <id> (find it via `its rmm checks <agent>`). |
| `its rmm checks run <agent>` | Force all of an agent's checks to run now (POST /checks/<agent>/run/) instead of waiting for the next scheduled cycle. Returns immediately; fresh results land on the agent's next check-in. |
| `its rmm checks create <agent>` | Attach a check to an agent. Use --type to pick: diskspace/cpuload/memory/ping/winsvc/script (defaults to script when --script is given). Tune alerting with --severity, --fails, --interval. |
| `its rmm checks edit [agent]` | Retune an existing check without delete+recreate — change interval, severity, fail-count, or thresholds. PUT is partial, so only the flags you pass change. Works on POLICY checks too: they are edited by --check id, so the agent is optional (find policy check ids via `its rmm policies checks <id>`). |
| `its rmm checks delete [agent_id]` | Remove a scheduled check from an agent. Destructive — needs --confirm. |

#### `its rmm checks <agent>`

Scheduled health checks attached to one agent. Includes last-run result + cadence.

**Examples:**

```bash
its rmm checks OFFICE-PC-01

# Re-runs every 10s — handy for dashboards or incident response.
its rmm checks OFFICE-PC-01 --watch
```

#### `its rmm checks failing`

Fleet-wide failing-checks sweep — fans out across online agents and surfaces every check currently failing, worst-first. The fast 'what's red across the estate right now' view. Narrow with --client/--site.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--client` | `` | Only agents in this client (substring) | — |
| `--site` | `` | Only agents in this site (substring) | — |

**Examples:**

```bash
# Every failing check across all online agents, worst-first
its rmm checks failing

its rmm checks failing --client "Candle Retail"
```

#### `its rmm checks results <agent>`

Drill into one check's live result on an agent — status, last run, fail count, return code, and captured stdout/stderr. Pass --check <id> (find it via `its rmm checks <agent>`).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--check` | `` | Check ID to inspect | — |

**Examples:**

```bash
# Status, last run, retcode + stdout/stderr for one check
its rmm checks results OFFICE-PC --check 7
```

#### `its rmm checks run <agent>`

Force all of an agent's checks to run now (POST /checks/<agent>/run/) instead of waiting for the next scheduled cycle. Returns immediately; fresh results land on the agent's next check-in.

**Examples:**

```bash
# Re-evaluate all checks immediately (e.g. after fixing a script-check body)
its rmm checks run OFFICE-PC
```

#### `its rmm checks create <agent>`

Attach a check to an agent. Use --type to pick: diskspace/cpuload/memory/ping/winsvc/script (defaults to script when --script is given). Tune alerting with --severity, --fails, --interval.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `` | Check type | — |
| `--severity` | `` | Alert severity (default error) | — |
| `--fails` | `` | Failures before alert (default 1) | — |
| `--interval` | `` | Run interval seconds, 0 = inherit (default 0) | — |
| `--disk` | `` | diskspace: drive (e.g. C:) | — |
| `--error` | `` | diskspace/cpuload/memory: error threshold % | — |
| `--warning` | `` | diskspace/cpuload/memory: warning threshold % | — |
| `--ip` | `` | ping: host/IP to ping | — |
| `--service` | `` | winsvc: Windows service name | — |
| `--restart-if-stopped` | `` | winsvc: restart the service if found stopped | — |
| `--pass-if-pending` | `` | winsvc: pass when start is pending | — |
| `--pass-if-missing` | `` | winsvc: pass when the service doesn't exist | — |
| `--script` | `` | script: script ID | — |
| `--timeout` | `` | script: timeout seconds (default 120) | — |
| `--args` | `` | script: arguments (comma-separated) | — |

**Examples:**

```bash
# Alert when C: free space drops below thresholds (%)
its rmm checks create OFFICE-PC --type diskspace --disk C: --error 10 --warning 25

its rmm checks create OFFICE-PC --type cpuload --error 90 --warning 75

its rmm checks create SRV-01 --type winsvc --service Spooler --restart-if-stopped

its rmm checks create SRV-01 --type ping --ip 10.10.0.1

its rmm checks create OFFICE-PC --script 42 --timeout 300

its rmm checks create OFFICE-PC-01 --script <script-id> --interval 600
```

#### `its rmm checks edit [agent]`

Retune an existing check without delete+recreate — change interval, severity, fail-count, or thresholds. PUT is partial, so only the flags you pass change. Works on POLICY checks too: they are edited by --check id, so the agent is optional (find policy check ids via `its rmm policies checks <id>`).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--check` | `` | Check ID to edit | — |
| `--severity` | `` | Alert severity | — |
| `--fails` | `` | Failures before alert | — |
| `--interval` | `` | Run interval seconds | — |
| `--error` | `` | Error threshold % (diskspace/cpuload/memory) | — |
| `--warning` | `` | Warning threshold % (diskspace/cpuload/memory) | — |
| `--timeout` | `` | Script check: timeout seconds | — |
| `--args` | `` | Script check: comma-separated script args (replaces existing) | — |

**Examples:**

```bash
# Change a check's thresholds (find the id via `its rmm checks <agent>`)
its rmm checks edit OFFICE-PC --check 7 --warning 60 --error 80

its rmm checks edit OFFICE-PC --check 7 --severity warning --fails 3

# Change a script check's timeout + args
its rmm checks edit OFFICE-PC --check 7 --timeout 300 --args -Verbose,-Force

# Promote a policy check from info to warning once its remediation is wired
its rmm checks edit --check 41 --severity warning
```

#### `its rmm checks delete [agent_id]`

Remove a scheduled check from an agent. Destructive — needs --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--check` | `` | Check ID to delete | — |
| `--confirm` | `` | Confirm the deletion | — |

**Examples:**

```bash
its rmm checks delete --check 418 --confirm

# Lists that agent's checks with their IDs
its rmm checks list WKS-9

its rmm checks delete <check-id> --confirm
```

---

### tasks

> Source: `src/providers/rmm/commands/tasks.ts`

| Command | Description |
|---------|-------------|
| `its rmm tasks [agent]` | Recurring task schedule attached to one agent — script + interval + next-run. Pass --policy <id> instead of an agent to list a policy's tasks, including the check-failure remediations that fire fleet-wide. |
| `its rmm tasks create [agent]` | Create an automated task that runs a script. Target an agent, or --policy <id> to apply across every agent under a policy. Default is a manual task (run on demand); --daily-time HH:MM makes it scheduled (+ --weekdays to limit days); --on-check-failure <checkId> makes it a remediation that fires whenever that check fails. |
| `its rmm tasks edit` | Enable or disable an existing task without delete+recreate. Recreating a policy checkfailure task would lose its assigned_check wiring, so this is the only safe way to arm or disarm a remediation. Also retunes name, severity and run-asap. |
| `its rmm tasks delete` | Remove a scheduled task from an agent. Destructive — needs --confirm. |

#### `its rmm tasks [agent]`

Recurring task schedule attached to one agent — script + interval + next-run. Pass --policy <id> instead of an agent to list a policy's tasks, including the check-failure remediations that fire fleet-wide.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--policy` | `` | List tasks attached to this automation policy instead of an agent | — |

**Examples:**

```bash
its rmm tasks OFFICE-PC

# Verify the check-failure remediations attached to a policy
its rmm tasks list --policy 4

its rmm tasks OFFICE-PC-01

# Re-runs every 10s — handy for dashboards or incident response.
its rmm tasks OFFICE-PC-01 --watch
```

#### `its rmm tasks create [agent]`

Create an automated task that runs a script. Target an agent, or --policy <id> to apply across every agent under a policy. Default is a manual task (run on demand); --daily-time HH:MM makes it scheduled (+ --weekdays to limit days); --on-check-failure <checkId> makes it a remediation that fires whenever that check fails.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--script` | `` | Script ID to run | — |
| `--policy` | `` | Attach to this automation policy instead of a single agent — applies to every agent under it | — |
| `--on-check-failure` | `` | Check ID this task remediates. Makes it a checkfailure task: it runs whenever that check fails, rather than on a schedule. This is the check-then-fix pattern. | — |
| `--severity` | `` | Alert severity for the task itself (default info) | — |
| `--disabled` | `` | Create the task switched off. Use when arming a policy-wide remediation would cause a thundering herd — create it disabled, then enable once you've seen what the check actually reports. | — |
| `--name` | `` | Task name (default: derived from the script ID) | — |
| `--args` | `` | Script arguments (comma-separated, e.g. -Mode,Notify) | — |
| `--timeout` | `` | Per-run timeout in seconds (default 90) | 90 |
| `--daily-time` | `` | Run daily at HH:MM (24h) — turns this into a scheduled task | — |
| `--weekdays` | `` | With --daily-time: restrict to these days (mon,tue,wed,thu,fri,sat,sun); default every day | — |
| `--run-asap` | `` | Run as soon as possible if a scheduled run was missed | — |

**Examples:**

```bash
# The house remediation pattern: when check 41 fails on any agent under policy 4, run script 433 there
its rmm tasks create --policy 4 --on-check-failure 41 --script 433 --name 'Auto-install dev runtimes' --timeout 1800

its rmm tasks create --policy 4 --script 252 --daily-time 06:30

its rmm tasks create OFFICE-PC --script 12 --name 'Clear cache'

its rmm tasks create <agent-id> --name "Weekly reboot" --script <script-id> --daily-time "03:00" --weekdays sun
```

#### `its rmm tasks edit`

Enable or disable an existing task without delete+recreate. Recreating a policy checkfailure task would lose its assigned_check wiring, so this is the only safe way to arm or disarm a remediation. Also retunes name, severity and run-asap.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--task` | `` | Task ID to edit | — |
| `--enable` | `` | Switch the task on | — |
| `--disable` | `` | Switch the task off. Use to disarm a policy-wide remediation, or to retire a superseded deployer without deleting it. | — |
| `--name` | `` | Rename the task | — |
| `--severity` | `` | Alert severity | — |
| `--run-asap` | `` | Run as soon as possible after a missed schedule | — |

**Examples:**

```bash
# Turn on a checkfailure task created with --disabled
its rmm tasks edit --task 243 --enable

# Stop an old task without deleting it, so it stays auditable
its rmm tasks edit --task 15 --disable
```

#### `its rmm tasks delete`

Remove a scheduled task from an agent. Destructive — needs --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--task` | `` | Task ID to delete | — |
| `--confirm` | `` | Confirm the deletion | — |

**Examples:**

```bash
its rmm tasks delete --task 92 --confirm

# Lists that agent's automated tasks with their IDs
its rmm tasks list WKS-9

its rmm tasks delete --task <task-id> --confirm
```

---

### policies

> Source: `src/providers/rmm/commands/policies.ts`

| Command | Description |
|---------|-------------|
| `its rmm policies` | RMM automation policies — packages of checks + scheduled tasks applied across clients/sites. |
| `its rmm policies get <policy_id>` | Policy detail — included checks, tasks, target agents/sites. |
| `its rmm policies checks <policy_id>` | List checks attached to a policy (uses the asymmetric `/automation/policies/<id>/checks/` GET route — see ctxc 588) |
| `its rmm policies add-check <policy_id>` | Add a check to a policy (applies to every agent under it). --type: diskspace/cpuload/memory/ping/winsvc/script (defaults to script when --script is given). Uses POST /checks/ with `policy` set and `agent` OMITTED — including agent:null returns 404 because the route resolver hits the agent path first (ctxc 588). |
| `its rmm policies patch-policy <policy_id>` | Edit a policy's Windows Update schedule + per-severity approvals (WinUpdatePolicy). Partial update — only the flags you pass change. --confirm required: applies to EVERY agent under the policy. |

#### `its rmm policies`

RMM automation policies — packages of checks + scheduled tasks applied across clients/sites.

**Examples:**

```bash
its rmm policies

# Re-runs every 10s — handy for dashboards or incident response.
its rmm policies --watch
```

#### `its rmm policies get <policy_id>`

Policy detail — included checks, tasks, target agents/sites.

**Examples:**

```bash
its rmm policies get <policy-id>
```

#### `its rmm policies checks <policy_id>`

List checks attached to a policy (uses the asymmetric `/automation/policies/<id>/checks/` GET route — see ctxc 588).

**Examples:**

```bash
its rmm policies checks <policy-id>
```

#### `its rmm policies add-check <policy_id>`

Add a check to a policy (applies to every agent under it). --type: diskspace/cpuload/memory/ping/winsvc/script (defaults to script when --script is given). Uses POST /checks/ with `policy` set and `agent` OMITTED — including agent:null returns 404 because the route resolver hits the agent path first (ctxc 588).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `` | Check type | — |
| `--severity` | `` | Alert severity (default error; script branch defaults warning) | — |
| `--fails` | `` | Failures before alert (default 1) | — |
| `--interval` | `` | Run interval seconds (script defaults 86400 daily, others 0=inherit) | — |
| `--disk` | `` | diskspace: drive (e.g. C:) | — |
| `--error` | `` | diskspace/cpuload/memory: error threshold % | — |
| `--warning` | `` | diskspace/cpuload/memory: warning threshold % | — |
| `--ip` | `` | ping: host/IP | — |
| `--service` | `` | winsvc: Windows service name | — |
| `--restart-if-stopped` | `` | winsvc: restart if stopped | — |
| `--pass-if-pending` | `` | winsvc: pass when start pending | — |
| `--pass-if-missing` | `` | winsvc: pass when service absent | — |
| `--script` | `` | script: Script ID | — |
| `--timeout` | `` | script: timeout seconds (default 90) | — |

**Examples:**

```bash
# Add a disk-space check to every agent under policy 4
its rmm policies add-check 4 --type diskspace --disk C: --error 10 --warning 25

its rmm policies add-check 4 --type winsvc --service Spooler --restart-if-stopped

its rmm policies add-check 4 --script 42

its rmm policies add-check <policy-id> --script <script-id>
```

#### `its rmm policies patch-policy <policy_id>`

Edit a policy's Windows Update schedule + per-severity approvals (WinUpdatePolicy). Partial update — only the flags you pass change. --confirm required: applies to EVERY agent under the policy.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--run-time-hour` | `` | Install hour 0–23 (hour-only — no minutes) | — |
| `--frequency` | `` | Schedule frequency (daily = daily/weekly + --days) | — |
| `--days` | `` | Weekdays for daily/weekly schedule, e.g. mon,wed,fri (run_time_days) | — |
| `--day-of-month` | `` | Day of month 1–31 for monthly schedule (run_time_day) | — |
| `--reboot` | `` | Reboot after install | — |
| `--critical` | `` | Critical updates | — |
| `--important` | `` | Important updates | — |
| `--moderate` | `` | Moderate updates | — |
| `--low` | `` | Low updates | — |
| `--other` | `` | Other updates | — |
| `--confirm` | `` | Apply — affects every agent under the policy | — |

**Examples:**

```bash
its rmm policies patch-policy 7 --run-time-hour 3 --frequency daily --critical approve --important approve --confirm
```

---

### diagnostics

> Source: `src/providers/rmm/commands/diagnostics.ts`

| Command | Description |
|---------|-------------|
| `its rmm diagnostics <agent>` | System health snapshot — CPU, RAM, disk, power plan, uptime, top processes |

#### `its rmm diagnostics <agent>`

System health snapshot — CPU, RAM, disk, power plan, uptime, top processes.

**Examples:**

```bash
# CPU, RAM, disk, power plan, uptime and top processes in one call
its rmm diagnostics list WKS-9

# Full health snapshot — CPU, disk, services, last check-in
its rmm diagnostics OFFICE-PC-01

# Re-runs every 10s — handy for dashboards or incident response.
its rmm diagnostics OFFICE-PC-01 --watch
```

---

### doctor

> Source: `src/providers/rmm/commands/doctor.ts`

| Command | Description |
|---------|-------------|
| `its rmm doctor` | Tactical RMM health snapshot — agent status, failing checks, active alerts, sites with no online agents |

#### `its rmm doctor`

Tactical RMM health snapshot — agent status, failing checks, active alerts, sites with no online agents.

**Examples:**

```bash
# Verify RMM connectivity + auth
its rmm doctor

# Re-runs every 10s — handy for dashboards or incident response.
its rmm doctor --watch
```

---

### custom-fields

> Source: `src/providers/rmm/commands/custom-fields.ts`

| Command | Description |
|---------|-------------|
| `its rmm custom-fields` | List TRMM custom field definitions across all models (agent / client / site). |
| `its rmm custom-fields set <agent> <field> <value>` | Set a custom field value on an agent. Accepts the field by numeric id OR by exact name (looked up against /core/customfields/). Writes both `value` and the typed column TRMM actually reads. |

#### `its rmm custom-fields`

List TRMM custom field definitions across all models (agent / client / site).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--model` | `` | Filter by model | — |

```bash
its rmm custom-fields
```

#### `its rmm custom-fields set <agent> <field> <value>`

Set a custom field value on an agent. Accepts the field by numeric id OR by exact name (looked up against /core/customfields/). Writes both `value` and the typed column TRMM actually reads.

**Examples:**

```bash
its rmm custom-fields set WKS-9 "RustDesk ID" 123456789

its rmm custom-fields set WKS-9 4 123456789
```

---
