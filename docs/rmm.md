# Tactical RMM (`rmm`)

Tactical RMM endpoint management — agents, alerts, software, services, updates, scripts, checks, tasks, policies.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md)

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
| `TACTICAL_URL` | Tactical RMM API URL (e.g. `https://api.rmm.example.com`) |
| `TACTICAL_API_KEY` | API key from RMM dashboard > Settings > Global Settings > API Keys |

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
| `its rmm agents run <agent>` | Execute a one-shot shell command on the target agent. Returns stdout + stderr + exit code. Use --shell powershell|cmd|bash; default timeout is 90s. |
| `its rmm agents history <agent>` | Command + script execution history for one agent. Shows time, type, exit status, who ran it, and the truncated payload. |
| `its rmm agents notes <agent>` | Free-form notes attached to the agent — common workflow is documenting incident actions or device peculiarities. |
| `its rmm agents pending <agent>` | Queued reboots, script runs, and update installs that the agent hasn't picked up yet. Useful when a command seems to have stalled. |
| `its rmm agents wake <agent>` | Send a magic packet via another online agent on the same LAN. Doesn't work across VLANs / VPN — agent must be on a network with a reachable RMM peer. |
| `its rmm agents edit <agent>` | Reassign an agent's client/site or update description (PUT /agents/{id}/) |
| `its rmm agents refresh <agent>` | Trigger a WMI/sysinfo refresh on an agent — repopulates hardware fields (serial, model, etc.) |

#### `its rmm agents`

List all RMM agents with status, hostname, OS, site. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--status` | `-s` | Filter by status | — |
| `--type` | `-t` | Filter by type | — |
| `--client` | `-c` | Filter by client name | — |
| `--site` | `` | Filter by site name | — |

**Examples:**

```bash
# Every agent, human-readable table
its rmm agents

# Only offline or overdue agents
its rmm agents --status offline

# Filter by site (case-insensitive substring)
its rmm agents --site "fernhurst"

# Full JSON for scripting
its rmm agents --json

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

# Pipe-friendly output for cron / jq.
its rmm agents stale --json
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
# Destructive — needs --confirm. Agent record is gone permanently.
its rmm agents remove OFFICE-PC-01 --confirm
```

#### `its rmm agents run <agent>`

Execute a one-shot shell command on the target agent. Returns stdout + stderr + exit code. Use --shell powershell|cmd|bash; default timeout is 90s.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--shell` | `` | Shell type | powershell |
| `--cmd` | `` | Command to execute (use --file for multi-line scripts) | — |
| `--file` | `` | Path to a local script file (overrides --cmd). PowerShell multi-line scripts auto-wrapped via base64 + temp file. | — |
| `--raw` | `` | Disable auto-wrapping for multi-line PowerShell (sends as-is) | — |
| `--timeout` | `` | Timeout in seconds | 30 |

**Examples:**

```bash
its rmm agents run OFFICE-PC-01 --shell powershell --cmd "Get-Process"

its rmm agents run OFFICE-PC-01 --shell cmd --cmd "ipconfig /all"

its rmm agents run LINUX-WEB-01 --shell bash --cmd "uname -a && uptime"
```

#### `its rmm agents history <agent>`

Command + script execution history for one agent. Shows time, type, exit status, who ran it, and the truncated payload.

**Examples:**

```bash
its rmm agents history OFFICE-PC-01

its rmm agents history OFFICE-PC-01 --days 7
```

#### `its rmm agents notes <agent>`

Free-form notes attached to the agent — common workflow is documenting incident actions or device peculiarities.

**Examples:**

```bash
its rmm agents notes OFFICE-PC-01

its rmm agents notes OFFICE-PC-01 --json
```

#### `its rmm agents pending <agent>`

Queued reboots, script runs, and update installs that the agent hasn't picked up yet. Useful when a command seems to have stalled.

**Examples:**

```bash
its rmm agents pending OFFICE-PC-01

# Queue depth + action types for monitoring.
its rmm agents pending OFFICE-PC-01 --json
```

#### `its rmm agents wake <agent>`

Send a magic packet via another online agent on the same LAN. Doesn't work across VLANs / VPN — agent must be on a network with a reachable RMM peer.

**Examples:**

```bash
its rmm agents wake OFFICE-PC-01

# Disambiguates when the hostname exists at multiple sites.
its rmm agents wake OFFICE-PC-01 --site "Head Office"
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
its rmm agents edit OFFICE-PC-01 --site <site-id>

its rmm agents edit OFFICE-PC-01 --description "Marketing — Jane's desk"
```

#### `its rmm agents refresh <agent>`

Trigger a WMI/sysinfo refresh on an agent — repopulates hardware fields (serial, model, etc.).

**Examples:**

```bash
# Repopulates hardware fields (serial, BIOS, etc.)
its rmm agents refresh OFFICE-PC-01

# Pipe-friendly output — use with jq / scripts.
its rmm agents refresh OFFICE-PC-01 --json
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

its rmm dashboard --json

# Counts by site, status, OS — single-screen fleet health view.
its rmm dashboard

# Re-runs every 10s — handy for incident-response screens.
its rmm dashboard --watch

# Re-runs every 10s — handy for dashboards or incident response.
its rmm dashboard --watch
```

---

### clients

> Source: `src/providers/rmm/commands/dashboard.ts`

| Command | Description |
|---------|-------------|
| `its rmm clients` | Top-level RMM client list with each client's sites flattened into one row. Use IDs from here as --client filter values elsewhere. |

#### `its rmm clients`

Top-level RMM client list with each client's sites flattened into one row. Use IDs from here as --client filter values elsewhere.

**Examples:**

```bash
its rmm clients

its rmm clients --json

# Re-runs every 10s — handy for dashboards or incident response.
its rmm clients --watch
```

---

### sites

> Source: `src/providers/rmm/commands/dashboard.ts`

| Command | Description |
|---------|-------------|
| `its rmm sites` | All RMM sites across every client. Site IDs feed --site filters on agent commands. |

#### `its rmm sites`

All RMM sites across every client. Site IDs feed --site filters on agent commands.

**Examples:**

```bash
its rmm sites

# Only sites belonging to a named client.
its rmm sites --client "Head Office"

# Re-runs every 10s — handy for dashboards or incident response.
its rmm sites --watch
```

---

### processes

> Source: `src/providers/rmm/commands/processes.ts`

| Command | Description |
|---------|-------------|
| `its rmm processes <agent>` | Live process snapshot from the agent — sorted by CPU%, top 50. Use `processes top` instead when you only want the heavy hitters. |
| `its rmm processes top <agent>` | Top processes by CPU usage (live snapshot via PowerShell) |
| `its rmm processes kill <agent_id>` | Terminate a process by PID. Destructive — needs --confirm. Use `processes list` first to find the PID. |

#### `its rmm processes <agent>`

Live process snapshot from the agent — sorted by CPU%, top 50. Use `processes top` instead when you only want the heavy hitters.

**Examples:**

```bash
its rmm processes OFFICE-PC-01

its rmm processes OFFICE-PC-01 --name "chrome"

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
# Top processes by CPU + memory
its rmm processes top OFFICE-PC-01

# Snapshot for trend analysis.
its rmm processes top OFFICE-PC-01 --json
```

#### `its rmm processes kill <agent_id>`

Terminate a process by PID. Destructive — needs --confirm. Use `processes list` first to find the PID.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--pid` | `` | Process ID to kill | — |
| `--confirm` | `` | Confirm the kill | — |

**Examples:**

```bash
its rmm processes kill OFFICE-PC-01 --pid 1234 --confirm

its rmm processes kill OFFICE-PC-01 --name "notepad" --confirm
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

its rmm services OFFICE-PC-01 --status running

# Quickly spot services that should be running.
its rmm services OFFICE-PC-01 --status stopped
```

#### `its rmm services get <agent>`

Service detail — startup type, dependencies, current state, last exit code.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | Service name | — |

**Examples:**

```bash
its rmm services get OFFICE-PC-01 --service spooler

# Pipe-friendly output — use with jq / scripts.
its rmm services get OFFICE-PC-01 --service spooler --json
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
its rmm services control OFFICE-PC-01 --service spooler --action restart

its rmm services control OFFICE-PC-01 --service spooler --action start
```

#### `its rmm services enable <agent>`

Set startup type to Automatic and start the service if stopped. Idempotent.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | Service name | — |

**Examples:**

```bash
its rmm services enable OFFICE-PC-01 --service spooler

# Pipe-friendly output — use with jq / scripts.
its rmm services enable OFFICE-PC-01 --service spooler --json
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
its rmm services disable OFFICE-PC-01 --service spooler --confirm

# Pipe-friendly output — use with jq / scripts.
its rmm services disable OFFICE-PC-01 --service spooler --confirm --json
```

---

### updates

> Source: `src/providers/rmm/commands/updates.ts`

| Command | Description |
|---------|-------------|
| `its rmm updates <agent>` | Windows Update queue — pending, installed, failed. Returns KB number + classification + size. |
| `its rmm updates scan <agent>` | Force the agent to re-scan Microsoft Update. Use after a long offline period or to refresh the pending list. |
| `its rmm updates install <agent_id>` | Install pending Windows updates on an agent (requires --confirm) |

#### `its rmm updates <agent>`

Windows Update queue — pending, installed, failed. Returns KB number + classification + size.

**Examples:**

```bash
# Windows Update queue
its rmm updates OFFICE-PC-01

its rmm updates OFFICE-PC-01 --json

# Re-runs every 10s — handy for dashboards or incident response.
its rmm updates OFFICE-PC-01 --watch
```

#### `its rmm updates scan <agent>`

Force the agent to re-scan Microsoft Update. Use after a long offline period or to refresh the pending list.

**Examples:**

```bash
its rmm updates scan OFFICE-PC-01

# Useful in scheduled checks.
its rmm updates scan OFFICE-PC-01 --json
```

#### `its rmm updates install <agent_id>`

Install pending Windows updates on an agent (requires --confirm).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the installation | — |

**Examples:**

```bash
its rmm updates install OFFICE-PC-01 --confirm

# Reboots the agent automatically when updates finish.
its rmm updates install OFFICE-PC-01 --confirm --reboot
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

# Full software inventory — feed to compliance reports.
its rmm software OFFICE-PC-01 --json

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

#### `its rmm alerts`

Active and resolved alerts across the fleet. Filter by --severity info|warning|error or --resolved.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--severity` | `` | Filter by severity (info/warning/error) | — |
| `--resolved` | `` | Show resolved alerts | false |

**Examples:**

```bash
its rmm alerts --status active

its rmm alerts --client "Head Office"

# Severity: information | warning | error.
its rmm alerts --severity error
```

#### `its rmm alerts get <alert_id>`

Alert detail — full message, source check, snooze state, agent it fired on.

**Examples:**

```bash
its rmm alerts get <alert-id>

# Pipe-friendly output — use with jq / scripts.
its rmm alerts get <alert-id> --json
```

---

### scripts

> Source: `src/providers/rmm/commands/scripts.ts`

| Command | Description |
|---------|-------------|
| `its rmm scripts` | All saved automation scripts in the RMM script library. Returns name, shell, category, default timeout. |
| `its rmm scripts get <script_id>` | Script detail — body, default args, category, hash, last edited timestamp. |
| `its rmm scripts run <agent>` | Execute a saved RMM script on the target agent. Streams stdout/stderr back; use --timeout for long-running jobs (default 120s). |
| `its rmm scripts upload-local <agent> <path>` | Upload a local .ps1/.sh/.py script to TRMM, run it on the agent, capture output, and delete the script afterwards. Use for ad-hoc D:/it-scripts/... invocations (Fix-DymoPrinter, Invoke-CorruptionCheck, Get-ShutdownDiagnostics). Pass --keep to leave the script registered. |
| `its rmm scripts delete <script_id>` | Permanently remove a script from the library. Destructive — needs --confirm. |
| `its rmm scripts upsert <name> <path>` | Idempotently push a local script to TRMM by name — creates if missing, updates if present (PUT). Works around the TRMM POST /scripts/ quirk where the response is a plain string, not the created object — we re-list to resolve the new ID. |

#### `its rmm scripts`

All saved automation scripts in the RMM script library. Returns name, shell, category, default timeout.

**Examples:**

```bash
its rmm scripts

# Useful when scripting against the registry.
its rmm scripts --json

# Re-runs every 10s — handy for dashboards or incident response.
its rmm scripts --watch
```

#### `its rmm scripts get <script_id>`

Script detail — body, default args, category, hash, last edited timestamp.

**Examples:**

```bash
its rmm scripts get <script-id>

# Pipe-friendly output — use with jq / scripts.
its rmm scripts get <script-id> --json
```

#### `its rmm scripts run <agent>`

Execute a saved RMM script on the target agent. Streams stdout/stderr back; use --timeout for long-running jobs (default 120s).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--script` | `` | Script ID | — |
| `--args` | `` | Script arguments (comma-separated) | — |
| `--timeout` | `` | Timeout in seconds | 120 |

**Examples:**

```bash
its rmm scripts run OFFICE-PC-01 --script "Restart Print Spooler"

# Default agent timeout is 90s — raise it for slow scripts.
its rmm scripts run OFFICE-PC-01 --script "Long Audit" --timeout 600
```

#### `its rmm scripts upload-local <agent> <path>`

Upload a local .ps1/.sh/.py script to TRMM, run it on the agent, capture output, and delete the script afterwards. Use for ad-hoc D:/it-scripts/... invocations (Fix-DymoPrinter, Invoke-CorruptionCheck, Get-ShutdownDiagnostics). Pass --keep to leave the script registered.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--shell` | `` | Script shell (defaults to auto-detect from extension: ps1=powershell, sh=shell, py=python, nu=nushell, ts=deno) | — |
| `--args` | `` | Script arguments (comma-separated) | — |
| `--timeout` | `` | Timeout in seconds (default 120) | 120 |
| `--keep` | `` | Leave the uploaded script registered in TRMM after execution | — |
| `--category` | `` | Category for the uploaded script (default 'Ad-hoc') | Ad-hoc |

**Examples:**

```bash
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

**Examples:**

```bash
# Creates if missing, updates body if name exists
its rmm scripts upsert "Restart Spooler" ./fix-spooler.ps1

# Pipe-friendly output — use with jq / scripts.
its rmm scripts upsert "Restart Spooler" ./fix-spooler.ps1 --json
```

---

### checks

> Source: `src/providers/rmm/commands/checks.ts`

| Command | Description |
|---------|-------------|
| `its rmm checks <agent>` | Scheduled health checks attached to one agent. Includes last-run result + cadence. |
| `its rmm checks create <agent>` | Attach a script-based check to an agent. Set --interval, --severity, --fail-count to tune alerting. |
| `its rmm checks delete [agent_id]` | Remove a scheduled check from an agent. Destructive — needs --confirm. |

#### `its rmm checks <agent>`

Scheduled health checks attached to one agent. Includes last-run result + cadence.

**Examples:**

```bash
its rmm checks OFFICE-PC-01

its rmm checks OFFICE-PC-01 --json

# Re-runs every 10s — handy for dashboards or incident response.
its rmm checks OFFICE-PC-01 --watch
```

#### `its rmm checks create <agent>`

Attach a script-based check to an agent. Set --interval, --severity, --fail-count to tune alerting.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--script` | `` | Script ID for the check | — |

**Examples:**

```bash
its rmm checks create OFFICE-PC-01 --script <script-id> --interval 600

# Pipe-friendly output — use with jq / scripts.
its rmm checks create OFFICE-PC-01 --script <script-id> --interval 600 --json
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
its rmm checks delete <check-id> --confirm
```

---

### tasks

> Source: `src/providers/rmm/commands/tasks.ts`

| Command | Description |
|---------|-------------|
| `its rmm tasks <agent>` | Recurring task schedule attached to one agent — script + interval + next-run. |
| `its rmm tasks create <agent_id>` | Schedule a script to run on an agent at a fixed cadence. Use --interval seconds; default is daily. |
| `its rmm tasks delete` | Remove a scheduled task from an agent. Destructive — needs --confirm. |

#### `its rmm tasks <agent>`

Recurring task schedule attached to one agent — script + interval + next-run.

**Examples:**

```bash
its rmm tasks OFFICE-PC-01

its rmm tasks OFFICE-PC-01 --json

# Re-runs every 10s — handy for dashboards or incident response.
its rmm tasks OFFICE-PC-01 --watch
```

#### `its rmm tasks create <agent_id>`

Schedule a script to run on an agent at a fixed cadence. Use --interval seconds; default is daily.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--script` | `` | Script ID for the task | — |

**Examples:**

```bash
its rmm tasks create <agent-id> --name "Weekly reboot" --script <script-id> --cron "0 3 * * 0"

# Pipe-friendly output — use with jq / scripts.
its rmm tasks create <agent-id> --name "Weekly reboot" --script <script-id> --cron "0 3 * * 0" --json
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
| `its rmm policies add-check <policy_id>` | Add a script-check to a policy. Uses POST /checks/ with `policy` set and `agent` OMITTED — including agent:null returns 404 because the route resolver hits the agent path first (ctxc 588). |

#### `its rmm policies`

RMM automation policies — packages of checks + scheduled tasks applied across clients/sites.

**Examples:**

```bash
its rmm policies

# Useful for diffing policy assignments between sites.
its rmm policies --json

# Re-runs every 10s — handy for dashboards or incident response.
its rmm policies --watch
```

#### `its rmm policies get <policy_id>`

Policy detail — included checks, tasks, target agents/sites.

**Examples:**

```bash
its rmm policies get <policy-id>

# Pipe-friendly output — use with jq / scripts.
its rmm policies get <policy-id> --json
```

#### `its rmm policies checks <policy_id>`

List checks attached to a policy (uses the asymmetric `/automation/policies/<id>/checks/` GET route — see ctxc 588).

**Examples:**

```bash
its rmm policies checks <policy-id>

# Pipe-friendly output — use with jq / scripts.
its rmm policies checks <policy-id> --json
```

#### `its rmm policies add-check <policy_id>`

Add a script-check to a policy. Uses POST /checks/ with `policy` set and `agent` OMITTED — including agent:null returns 404 because the route resolver hits the agent path first (ctxc 588).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--script` | `` | Script ID | — |
| `--timeout` | `` | Timeout seconds (default 90) | 90 |
| `--interval` | `` | Run interval seconds (default 86400 — daily) | 86400 |
| `--severity` | `` | Alert severity: info | warning | error | warning |
| `--fails-before-alert` | `` | Consecutive failures before alerting (default 1) | 1 |

**Examples:**

```bash
its rmm policies add-check <policy-id> --script <script-id>

# Pipe-friendly output — use with jq / scripts.
its rmm policies add-check <policy-id> --script <script-id> --json
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
# Full health snapshot — CPU, disk, services, last check-in
its rmm diagnostics OFFICE-PC-01

its rmm diagnostics OFFICE-PC-01 --json

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

# Use in CI to verify the box can still reach the RMM API.
its rmm doctor --json

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

```bash
its rmm custom-fields set <agent> <field> <value>
```

---
