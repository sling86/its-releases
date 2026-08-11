# Intune (`intune`)

Microsoft Intune device management — managed devices, apps, platform scripts, remediations, compliance policies, ESP, Autopilot.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [devices](#devices)
- [compliance](#compliance)
- [apps](#apps)
- [scripts](#scripts)
- [remediations](#remediations)
- [policies](#policies)
- [esp](#esp)
- [autopilot](#autopilot)
- [group](#group)
- [assignments](#assignments)
- [settings](#settings)
- [intents](#intents)
- [updates](#updates)
- [appconfig](#appconfig)
- [appprotection](#appprotection)
- [doctor](#doctor)
- [graph](#graph)

## Setup

```bash
its intune setup           # Interactive wizard
its intune setup --check   # Check configuration status
its intune setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `TENANT_ID` | Microsoft Entra tenant ID (same as Entra provider) |
| `CLIENT_ID` | App registration client ID (same as Entra) |
| `CLIENT_SECRET` | App registration client secret (same as Entra) |

Intune reuses the same Graph API credentials as the Entra provider — no additional secrets needed. The app registration needs these **additional API permissions**: `DeviceManagementManagedDevices.Read.All`, `DeviceManagementApps.Read.All`, `DeviceManagementConfiguration.Read.All`, `DeviceManagementServiceConfig.Read.All`.

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/intune/client.ts` | API client methods |
| `src/providers/intune/types.ts` | TypeScript interfaces |
| `src/providers/intune/commands/` | Command definitions (split by resource) |
| `src/providers/intune/assignments.ts` | assignments |
| `src/providers/intune/definition.ts` | definition |

## Resources

### devices

> Source: `src/providers/intune/commands/devices.ts`

| Command | Description |
|---------|-------------|
| `its intune devices` | List Intune-managed devices. Surfaces the most common fields; pass --json for raw shape. |
| `its intune devices get <id>` | Get managed device details. Pass the id (or any natural identifier) as the positional arg. |
| `its intune devices search <query>` | Search devices by name, user, or serial number. Substring match across the most relevant fields; case-insensitive. |
| `its intune devices sync <id>` | Trigger a device sync. Force the device to sync with Intune. |
| `its intune devices noncompliant` | List devices failing compliance. Returns devices failing compliance checks. |

#### `its intune devices`

List Intune-managed devices. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Number of results (default 50, paginates automatically) | 50 |
| `--all` | `` | Fetch all results (overrides --top) | — |

**Examples:**

```bash
its intune devices

its intune devices --all

its intune devices --filter compliance=noncompliant
```

#### `its intune devices get <id>`

Get managed device details. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its intune devices get <serial>
```

#### `its intune devices search <query>`

Search devices by name, user, or serial number. Substring match across the most relevant fields; case-insensitive.

**Examples:**

```bash
its intune devices search "jane"
```

#### `its intune devices sync <id>`

Trigger a device sync. Force the device to sync with Intune.

**Examples:**

```bash
# Queues a check-in — the device applies pending policy on next contact
its intune devices sync 8f1c2d3e-4a5b-6c7d-8e9f-0a1b2c3d4e5f

its intune devices sync <device-id>
```

#### `its intune devices noncompliant`

List devices failing compliance. Returns devices failing compliance checks.

**Examples:**

```bash
its intune devices noncompliant
```

---

### compliance

> Source: `src/providers/intune/commands/compliance.ts`

| Command | Description |
|---------|-------------|
| `its intune compliance why <device>` | Explain why a device is non-compliant — lists every failing compliance setting (policy, setting, state, error). Resolve the device by id or name. |

#### `its intune compliance why <device>`

Explain why a device is non-compliant — lists every failing compliance setting (policy, setting, state, error). Resolve the device by id or name.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--all` | `` | Show every reported setting, not just the failing ones | — |

**Examples:**

```bash
its intune compliance why LAPTOP-042

its intune compliance why 12345678-90ab-cdef-1234-567890abcdef

its intune compliance why LAPTOP-042 --all
```

---

### apps

> Source: `src/providers/intune/commands/apps.ts`

| Command | Description |
|---------|-------------|
| `its intune apps` | List managed apps. Surfaces the most common fields; pass --json for raw shape. |
| `its intune apps get <id>` | Get app details and assignments. Pass the id (or any natural identifier) as the positional arg. |
| `its intune apps required` | List apps with required assignments (blocks ESP). Returns apps required by Intune policy. |

#### `its intune apps`

List managed apps. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Number of results (default 50, paginates automatically) | 50 |
| `--all` | `` | Fetch all results (overrides --top) | — |
| `--with-assignments` | `` | Include assignment target group IDs inline (one extra column) | — |

**Examples:**

```bash
its intune apps

# Re-runs every 10s — handy for dashboards or incident response.
its intune apps --watch
```

#### `its intune apps get <id>`

Get app details and assignments. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its intune apps get <app-id>
```

#### `its intune apps required`

List apps with required assignments (blocks ESP). Returns apps required by Intune policy.

**Examples:**

```bash
# Assignments with intent=required
its intune apps required
```

---

### scripts

> Source: `src/providers/intune/commands/scripts.ts`

| Command | Description |
|---------|-------------|
| `its intune scripts` | List platform scripts. Surfaces the most common fields; pass --json for raw shape. |
| `its intune scripts get <id>` | Get platform script details and content. Pass the id (or any natural identifier) as the positional arg. |
| `its intune scripts status <id>` | Get script run status per device. Returns current state plus any pending operations. |

#### `its intune scripts`

List platform scripts. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Number of results (default 50, paginates automatically) | 50 |
| `--all` | `` | Fetch all results (overrides --top) | — |

**Examples:**

```bash
its intune scripts

# Re-runs every 10s — handy for dashboards or incident response.
its intune scripts --watch
```

#### `its intune scripts get <id>`

Get platform script details and content. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its intune scripts get <script-id>
```

#### `its intune scripts status <id>`

Get script run status per device. Returns current state plus any pending operations.

**Examples:**

```bash
# Per-device success/fail
its intune scripts status <script-id>

# Re-runs every 10s — handy for dashboards or incident response.
its intune scripts status <script-id> --watch
```

---

### remediations

> Source: `src/providers/intune/commands/remediations.ts`

| Command | Description |
|---------|-------------|
| `its intune remediations` | List proactive remediation scripts. Surfaces the most common fields; pass --json for raw shape. |
| `its intune remediations get <id>` | Get remediation script details. Pass the id (or any natural identifier) as the positional arg. |
| `its intune remediations status <id>` | Get remediation run status per device. Returns current state plus any pending operations. |

#### `its intune remediations`

List proactive remediation scripts. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Number of results (default 50, paginates automatically) | 50 |
| `--all` | `` | Fetch all results (overrides --top) | — |

**Examples:**

```bash
its intune remediations

# Re-runs every 10s — handy for dashboards or incident response.
its intune remediations --watch
```

#### `its intune remediations get <id>`

Get remediation script details. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its intune remediations get <id>
```

#### `its intune remediations status <id>`

Get remediation run status per device. Returns current state plus any pending operations.

**Examples:**

```bash
its intune remediations status <id>

# Re-runs every 10s — handy for dashboards or incident response.
its intune remediations status <id> --watch
```

---

### policies

> Source: `src/providers/intune/commands/policies.ts`

| Command | Description |
|---------|-------------|
| `its intune policies` | List device compliance policies. Surfaces the most common fields; pass --json for raw shape. |
| `its intune policies get <id>` | Get compliance policy details. Pass the id (or any natural identifier) as the positional arg. |
| `its intune policies configs` | List device configuration profiles. Configuration profiles applied to the device. |

#### `its intune policies`

List device compliance policies. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Number of results (default 50, paginates automatically) | 50 |
| `--all` | `` | Fetch all results (overrides --top) | — |
| `--with-assignments` | `` | Include assignment target group IDs inline (one extra column) | — |

**Examples:**

```bash
its intune policies

# Re-runs every 10s — handy for dashboards or incident response.
its intune policies --watch
```

#### `its intune policies get <id>`

Get compliance policy details. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its intune policies get <policy-id>
```

#### `its intune policies configs`

List device configuration profiles. Configuration profiles applied to the device.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Number of results (default 50, paginates automatically) | 50 |
| `--all` | `` | Fetch all results (overrides --top) | — |
| `--with-assignments` | `` | Include assignment target group IDs inline (one extra column) | — |

**Examples:**

```bash
its intune policies configs
```

---

### esp

> Source: `src/providers/intune/commands/esp.ts`

| Command | Description |
|---------|-------------|
| `its intune esp` | List Enrollment Status Page profiles. Surfaces the most common fields; pass --json for raw shape. |
| `its intune esp get <id>` | Get ESP profile details and tracked apps. Pass the id (or any natural identifier) as the positional arg. |
| `its intune esp update [id]` | Update ESP profile settings (timeout, tracked apps). PATCH semantics — only the supplied fields change. |

#### `its intune esp`

List Enrollment Status Page profiles. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
# Enrolment Status Page profiles
its intune esp

# Re-runs every 10s — handy for dashboards or incident response.
its intune esp --watch
```

#### `its intune esp get <id>`

Get ESP profile details and tracked apps. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its intune esp get <profile-id>
```

#### `its intune esp update [id]`

Update ESP profile settings (timeout, tracked apps). PATCH semantics — only the supplied fields change.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--timeout` | `` | Timeout in minutes | — |
| `--track-app` | `` | Add app ID to tracked apps | — |
| `--untrack-app` | `` | Remove app ID from tracked apps | — |
| `--show-progress` | `` | Show installation progress (true/false) | — |
| `--allow-use-on-failure` | `` | Allow device use on failure (true/false) | — |

**Examples:**

```bash
# Heavy tracked apps routinely exceed the default window and time the ESP out
its intune esp update 8f1c2d3e-... --timeout 3600

its intune esp update 8f1c2d3e-... --track-app 9a8b7c6d-...

its intune esp update <profile-id> --timeout 120
```

---

### autopilot

> Source: `src/providers/intune/commands/autopilot.ts`

| Command | Description |
|---------|-------------|
| `its intune autopilot` | List Autopilot deployment profiles. Surfaces the most common fields; pass --json for raw shape. |
| `its intune autopilot devices` | List Autopilot-registered devices. Returns devices for the resource. |
| `its intune autopilot tag <serial> [tag]` | Set group tag on an Autopilot device. Set or clear a tag value. |

#### `its intune autopilot`

List Autopilot deployment profiles. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its intune autopilot

# Re-runs every 10s — handy for dashboards or incident response.
its intune autopilot --watch
```

#### `its intune autopilot devices`

List Autopilot-registered devices. Returns devices for the resource.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Number of results (default 50, paginates automatically) | 50 |
| `--all` | `` | Fetch all results (overrides --top) | — |

**Examples:**

```bash
its intune autopilot devices
```

#### `its intune autopilot tag <serial> [tag]`

Set group tag on an Autopilot device. Set or clear a tag value.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--clear` | `` | Remove the group tag | — |

**Examples:**

```bash
its intune autopilot tag ABC1234 "Finance-Laptops"

its intune autopilot tag ABC1234 --clear

its intune autopilot tag <serial> "Office-Standard"
```

---

### group

> Source: `src/providers/intune/commands/lookup.ts`

| Command | Description |
|---------|-------------|
| `its intune group find <groupId>` | Reverse lookup — list every Intune resource assigned to a group |

#### `its intune group find <groupId>`

Reverse lookup — list every Intune resource assigned to a group.

**Examples:**

```bash
its intune group find "All Devices"
```

---

### assignments

> Source: `src/providers/intune/commands/assignments.ts`

| Command | Description |
|---------|-------------|
| `its intune assignments audit` | Audit Intune assignments — per-target matrix (which configs/compliance/apps hit each group) plus orphan resources with no assignment. Group GUIDs are resolved to display names. |

#### `its intune assignments audit`

Audit Intune assignments — per-target matrix (which configs/compliance/apps hit each group) plus orphan resources with no assignment. Group GUIDs are resolved to display names.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Number of each resource type to fetch (default 100) | 100 |
| `--all` | `` | Fetch up to 1000 of each resource type (overrides --top) | — |

**Examples:**

```bash
its intune assignments audit

its intune assignments audit --all
```

---

### settings

| Command | Description |
|---------|-------------|
| `its intune settings` | List Settings Catalog policies (the modern Intune configuration surface) |
| `its intune settings get <id>` | Get a Settings Catalog policy with assignments expanded |

#### `its intune settings`

List Settings Catalog policies (the modern Intune configuration surface).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Number of results (default 50) | 50 |
| `--all` | `` | Fetch all results (overrides --top, paginates up to 1000) | — |
| `--with-assignments` | `` | Expand assignment target group IDs inline | — |

**Examples:**

```bash
# Modern Intune configuration policies
its intune settings

# Re-runs every 10s — handy for dashboards or incident response.
its intune settings --watch
```

#### `its intune settings get <id>`

Get a Settings Catalog policy with assignments expanded.

**Examples:**

```bash
its intune settings get <policy-id>
```

---

### intents

> Source: `src/providers/intune/commands/lookup.ts`

| Command | Description |
|---------|-------------|
| `its intune intents` | List Endpoint Security policy intents (firewall, ASR, BitLocker, etc.) |
| `its intune intents get <id>` | Get an Endpoint Security intent with assignments expanded |

#### `its intune intents`

List Endpoint Security policy intents (firewall, ASR, BitLocker, etc.).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Number of results (default 50) | 50 |
| `--all` | `` | Fetch all results (overrides --top, paginates up to 1000) | — |
| `--with-assignments` | `` | Expand assignment target group IDs inline | — |

**Examples:**

```bash
# Firewall, ASR, BitLocker policies
its intune intents

# Re-runs every 10s — handy for dashboards or incident response.
its intune intents --watch
```

#### `its intune intents get <id>`

Get an Endpoint Security intent with assignments expanded.

**Examples:**

```bash
its intune intents get <intent-id>
```

---

### updates

> Source: `src/providers/intune/commands/coverage.ts`

| Command | Description |
|---------|-------------|
| `its intune updates` | List Windows Update profiles. --type feature|quality|driver (default feature) |
| `its intune updates get <id>` | Get a Windows Update profile by id (auto-detects type — pass --type to disambiguate) |

#### `its intune updates`

List Windows Update profiles. --type feature|quality|driver (default feature).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `` | Profile category | feature |
| `--top` | `` | Number of results (default 50) | 50 |
| `--all` | `` | Fetch all results (overrides --top, paginates up to 1000) | — |
| `--with-assignments` | `` | Expand assignment target group IDs inline | — |

**Examples:**

```bash
its intune updates

# Re-runs every 10s — handy for dashboards or incident response.
its intune updates --watch
```

#### `its intune updates get <id>`

Get a Windows Update profile by id (auto-detects type — pass --type to disambiguate).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `` | Profile category | feature |

**Examples:**

```bash
its intune updates get <ring-id>
```

---

### appconfig

| Command | Description |
|---------|-------------|
| `its intune appconfig` | List mobile app configuration policies (per-app key/value config) |
| `its intune appconfig get <id>` | Get an app configuration policy with assignments expanded |

#### `its intune appconfig`

List mobile app configuration policies (per-app key/value config).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Number of results (default 50) | 50 |
| `--all` | `` | Fetch all results (overrides --top, paginates up to 1000) | — |
| `--with-assignments` | `` | Expand assignment target group IDs inline | — |

**Examples:**

```bash
its intune appconfig

# Re-runs every 10s — handy for dashboards or incident response.
its intune appconfig --watch
```

#### `its intune appconfig get <id>`

Get an app configuration policy with assignments expanded.

**Examples:**

```bash
its intune appconfig get <policy-id>
```

---

### appprotection

> Source: `src/providers/intune/commands/coverage.ts`

| Command | Description |
|---------|-------------|
| `its intune appprotection` | List App Protection (MAM) policies. --platform ios|android (default ios) |
| `its intune appprotection get <id>` | Get an App Protection policy by id. Pass the id (or any natural identifier) as the positional arg. |

#### `its intune appprotection`

List App Protection (MAM) policies. --platform ios|android (default ios).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--platform` | `` | Mobile OS | ios |
| `--top` | `` | Number of results (default 50) | 50 |
| `--all` | `` | Fetch all results (overrides --top, paginates up to 1000) | — |
| `--with-assignments` | `` | Expand assignment target group IDs inline | — |

**Examples:**

```bash
its intune appprotection --platform ios

its intune appprotection --platform android

# Re-runs every 10s — handy for dashboards or incident response.
its intune appprotection --platform ios --watch
```

#### `its intune appprotection get <id>`

Get an App Protection policy by id. Pass the id (or any natural identifier) as the positional arg.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--platform` | `` | Mobile OS | ios |

**Examples:**

```bash
its intune appprotection get <policy-id>
```

---

### doctor

> Source: `src/providers/intune/commands/doctor.ts`

| Command | Description |
|---------|-------------|
| `its intune doctor` | Intune health snapshot — non-compliant devices, sync staleness, unencrypted endpoints, autopilot pending count |

#### `its intune doctor`

Intune health snapshot — non-compliant devices, sync staleness, unencrypted endpoints, autopilot pending count.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--stale-hours` | `` | Threshold for device sync staleness (default 48) | 48 |
| `--top` | `` | Maximum devices to scan (default 999) | 999 |

**Examples:**

```bash
its intune doctor

# Re-runs every 10s — handy for dashboards or incident response.
its intune doctor --watch
```

---

### graph

| Command | Description |
|---------|-------------|
| `its intune graph get <path>` | Raw Graph GET — pass any /v1.0 or /beta path (use --beta for beta) |
| `its intune graph post <path>` | Raw Graph POST — pass any /v1.0 or /beta path (use --beta for beta) |
| `its intune graph patch <path>` | Raw Graph PATCH — pass any /v1.0 or /beta path (use --beta for beta) |
| `its intune graph put <path>` | Raw Graph PUT — pass any /v1.0 or /beta path (use --beta for beta) |
| `its intune graph delete <path>` | Raw Graph DELETE — pass any /v1.0 or /beta path (use --beta for beta) |

#### `its intune graph get <path>`

Raw Graph GET — pass any /v1.0 or /beta path (use --beta for beta).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--beta` | `` | Use /beta instead of /v1.0 | — |
| `--header` | `` | Extra headers as comma-separated K=V pairs (e.g. Prefer=return=minimal) | — |
| `--raw` | `` | Return the response body as raw bytes (no JSON decode). Required for binary endpoints like /content. Currently honoured by the `sp` provider. | — |
| `--out` | `` | Write the response to this file path instead of stdout. Implies --raw. | — |

**Examples:**

```bash
its intune graph get /users

its intune graph get /administrativeUnits --beta

its intune graph get /users --header ConsistencyLevel=eventual

its intune graph get "/deviceManagement/managedDevices?$top=5"
```

#### `its intune graph post <path>`

Raw Graph POST — pass any /v1.0 or /beta path (use --beta for beta).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--body` | `` | Request body — inline JSON string or @file.json to read from disk | — |
| `--beta` | `` | Use /beta instead of /v1.0 | — |
| `--header` | `` | Extra headers as comma-separated K=V pairs (e.g. Prefer=return=minimal) | — |

**Examples:**

```bash
its intune graph post /users --body '{"displayName":"Jane Smith"}'

its intune graph post /administrativeUnits --beta

its intune graph post /users --header ConsistencyLevel=eventual

its intune graph post "/deviceManagement/managedDevices/<id>/syncDevice"
```

#### `its intune graph patch <path>`

Raw Graph PATCH — pass any /v1.0 or /beta path (use --beta for beta).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--body` | `` | Request body — inline JSON string or @file.json to read from disk | — |
| `--beta` | `` | Use /beta instead of /v1.0 | — |
| `--header` | `` | Extra headers as comma-separated K=V pairs (e.g. Prefer=return=minimal) | — |

**Examples:**

```bash
its intune graph patch /users --body '{"displayName":"Jane Smith"}'

its intune graph patch /administrativeUnits --beta

its intune graph patch /users --header ConsistencyLevel=eventual

its intune graph patch "/deviceManagement/deviceCompliancePolicies/<id>" --body @./patch.json
```

#### `its intune graph put <path>`

Raw Graph PUT — pass any /v1.0 or /beta path (use --beta for beta).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--body` | `` | Request body — inline JSON string or @file.json to read from disk | — |
| `--beta` | `` | Use /beta instead of /v1.0 | — |
| `--header` | `` | Extra headers as comma-separated K=V pairs (e.g. Prefer=return=minimal) | — |

**Examples:**

```bash
its intune graph put /users --body '{"displayName":"Jane Smith"}'

its intune graph put /administrativeUnits --beta

its intune graph put /users --header ConsistencyLevel=eventual

its intune graph put "/deviceManagement/managedDevices/<id>" --body @./body.json
```

#### `its intune graph delete <path>`

Raw Graph DELETE — pass any /v1.0 or /beta path (use --beta for beta).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--beta` | `` | Use /beta instead of /v1.0 | — |
| `--header` | `` | Extra headers as comma-separated K=V pairs (e.g. Prefer=return=minimal) | — |

**Examples:**

```bash
# No confirmation prompt — the path is sent exactly as given
its intune graph delete /groups/8f1c2d3e-4a5b-6c7d-8e9f-0a1b2c3d4e5f

its intune graph delete /administrativeUnits --beta

its intune graph delete /users --header ConsistencyLevel=eventual

its intune graph delete "/deviceManagement/managedDevices/<id>"
```

---
