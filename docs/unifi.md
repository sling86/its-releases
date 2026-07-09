# UniFi Network (`unifi`)

UniFi Network controller — devices, clients, WLANs, networks, firewall, events, alarms, vouchers.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [sites](#sites)
- [devices](#devices)
- [clients](#clients)
- [guests](#guests)
- [networks](#networks)
- [wlans](#wlans)
- [firewall](#firewall)
- [routes](#routes)
- [portforwards](#portforwards)
- [port-forwards](#port-forwards)
- [ports](#ports)
- [events](#events)
- [alarms](#alarms)
- [rogue](#rogue)
- [vouchers](#vouchers)
- [dashboard](#dashboard)
- [audit](#audit)

## Setup

```bash
its unifi setup           # Interactive wizard
its unifi setup --check   # Check configuration status
its unifi setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `UNIFI_HOST` | UniFi controller URL (e.g. `https://unifi.example.com:8443`). Also accepts `UNIFI_URL`. |
| `UNIFI_USERNAME` | UniFi controller username |
| `UNIFI_PASSWORD` | UniFi controller password |
| `UNIFI_SITE` | Default site name (defaults to `default`) |

Most commands accept `--site <name>` to override the default site. Use `its unifi sites` to list available sites and their site codes.

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/unifi/client.ts` | API client methods |
| `src/providers/unifi/types.ts` | TypeScript interfaces |
| `src/providers/unifi/commands/` | Command definitions (split by resource) |
| `src/providers/unifi/audit.ts` | audit |
| `src/providers/unifi/definition.ts` | definition |

## Resources

### sites

> Source: `src/providers/unifi/commands/sites.ts`

| Command | Description |
|---------|-------------|
| `its unifi sites` | List all UniFi sites. Surfaces the most common fields; pass --json for raw shape. |
| `its unifi sites health` | Site health — WAN/WLAN/LAN subsystem status. Live health check across the resource's dependencies. |
| `its unifi sites sysinfo` | Controller version, uptime, and IP addresses. Returns the controller version + uptime. |

#### `its unifi sites`

List all UniFi sites. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi sites

# Pipe-friendly output — use with jq / scripts.
its unifi sites --json

# Re-runs every 10s — handy for dashboards or incident response.
its unifi sites --watch
```

#### `its unifi sites health`

Site health — WAN/WLAN/LAN subsystem status. Live health check across the resource's dependencies.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
# WAN/LAN/WLAN status per site
its unifi sites health

# Useful as a watchdog feed for site-level alerts.
its unifi sites health --json

# Re-runs every 10s — handy for dashboards or incident response.
its unifi sites health --watch
```

#### `its unifi sites sysinfo`

Controller version, uptime, and IP addresses. Returns the controller version + uptime.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi sites sysinfo

# Pipe-friendly output — use with jq / scripts.
its unifi sites sysinfo --json
```

---

### devices

> Source: `src/providers/unifi/commands/devices.ts`

| Command | Description |
|---------|-------------|
| `its unifi devices` | List all UniFi devices with name, MAC, IP, type, state, uptime, clients |
| `its unifi devices get <mac>` | Get device detail by MAC address. Pass the id (or any natural identifier) as the positional arg. |
| `its unifi devices restart <mac>` | Restart a device by MAC address. Stop + start in one call. |
| `its unifi devices locate <mac>` | Toggle locate LED on a device. Flash the device's locate LED. |
| `its unifi devices upgrade <mac>` | Trigger firmware upgrade on a device. Trigger a firmware upgrade. |
| `its unifi devices provision <mac>` | Force re-provision a device. Force the device to re-pull its provisioning. |
| `its unifi devices power-cycle` | PoE power cycle a switch port. Cycle PoE on a port. --confirm required. |
| `its unifi devices leds` | Toggle all site LEDs on or off. Toggle the controller's status LEDs. |
| `its unifi devices poe` | PoE power summary for all switches. Returns the PoE budget + per-port consumption. |

#### `its unifi devices`

List all UniFi devices with name, MAC, IP, type, state, uptime, clients.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `` | Filter by type (ap|switch|gateway|all) | — |
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi devices

its unifi devices --site "Office"

# Re-runs every 10s — handy for dashboards or incident response.
its unifi devices --watch
```

#### `its unifi devices get <mac>`

Get device detail by MAC address. Pass the id (or any natural identifier) as the positional arg.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi devices get <mac>

# Pipe-friendly output — use with jq / scripts.
its unifi devices get <mac> --json
```

#### `its unifi devices restart <mac>`

Restart a device by MAC address. Stop + start in one call.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the restart | — |
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi devices restart <mac> --confirm

# Pipe-friendly output — use with jq / scripts.
its unifi devices restart <mac> --confirm --json
```

#### `its unifi devices locate <mac>`

Toggle locate LED on a device. Flash the device's locate LED.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--enable` | `` | Enable locate LED (default true, pass --no-enable to disable) | true |
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi devices locate <mac>

# Pipe-friendly output — use with jq / scripts.
its unifi devices locate <mac> --json
```

#### `its unifi devices upgrade <mac>`

Trigger firmware upgrade on a device. Trigger a firmware upgrade.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the upgrade | — |
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi devices upgrade <mac> --confirm

# Pipe-friendly output — use with jq / scripts.
its unifi devices upgrade <mac> --confirm --json
```

#### `its unifi devices provision <mac>`

Force re-provision a device. Force the device to re-pull its provisioning.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi devices provision <mac>

# Pipe-friendly output — use with jq / scripts.
its unifi devices provision <mac> --json
```

#### `its unifi devices power-cycle`

PoE power cycle a switch port. Cycle PoE on a port. --confirm required.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--mac` | `` | Switch MAC address | — |
| `--port` | `` | Port index to power cycle | — |
| `--confirm` | `` | Confirm the power cycle | — |
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi devices power-cycle --mac <mac> --port 5

# Pipe-friendly output — use with jq / scripts.
its unifi devices power-cycle --mac <mac> --port 5 --json
```

#### `its unifi devices leds`

Toggle all site LEDs on or off. Toggle the controller's status LEDs.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--enable` | `` | Enable LEDs (true/false) | true |
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi devices leds off

# Pipe-friendly output — use with jq / scripts.
its unifi devices leds off --json
```

#### `its unifi devices poe`

PoE power summary for all switches. Returns the PoE budget + per-port consumption.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
# Power budget + per-port draw
its unifi devices poe

# Pipe-friendly output — use with jq / scripts.
its unifi devices poe --json
```

---

### clients

> Source: `src/providers/unifi/commands/clients.ts`

| Command | Description |
|---------|-------------|
| `its unifi clients` | List online clients. Surfaces the most common fields; pass --json for raw shape. |
| `its unifi clients get <mac>` | Get client detail by MAC address. Pass the id (or any natural identifier) as the positional arg. |
| `its unifi clients search <query>` | Search all known clients by name, hostname, IP, or MAC. Substring match across the most relevant fields; case-insensitive. |
| `its unifi clients block <mac>` | Block a client by MAC address. Blocks a client. Reversible via `unblock`. |
| `its unifi clients unblock <mac>` | Unblock a client by MAC address. Re-allows a previously blocked client. |
| `its unifi clients reconnect <mac>` | Force reconnect a client. Force a client to disassociate + re-auth. |
| `its unifi clients offline` | List recently disconnected clients. Returns recently-disconnected clients. |

#### `its unifi clients`

List online clients. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--filter` | `` | Filter by connection type (wired|wireless|all) | all |
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi clients

its unifi clients --filter wireless

its unifi clients --filter wired
```

#### `its unifi clients get <mac>`

Get client detail by MAC address. Pass the id (or any natural identifier) as the positional arg.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi clients get <mac>

# Pipe-friendly output — use with jq / scripts.
its unifi clients get <mac> --json
```

#### `its unifi clients search <query>`

Search all known clients by name, hostname, IP, or MAC. Substring match across the most relevant fields; case-insensitive.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi clients search "jane"

# Pipe-friendly output — use with jq / scripts.
its unifi clients search "jane" --json
```

#### `its unifi clients block <mac>`

Block a client by MAC address. Blocks a client. Reversible via `unblock`.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the block | — |
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi clients block <mac>

# Pipe-friendly output — use with jq / scripts.
its unifi clients block <mac> --json
```

#### `its unifi clients unblock <mac>`

Unblock a client by MAC address. Re-allows a previously blocked client.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi clients unblock <mac>

# Pipe-friendly output — use with jq / scripts.
its unifi clients unblock <mac> --json
```

#### `its unifi clients reconnect <mac>`

Force reconnect a client. Force a client to disassociate + re-auth.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi clients reconnect <mac>

# Pipe-friendly output — use with jq / scripts.
its unifi clients reconnect <mac> --json
```

#### `its unifi clients offline`

List recently disconnected clients. Returns recently-disconnected clients.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--hours` | `` | Look-back period in hours (default 24) | 24 |
| `--site` | `` | Site name override | — |

**Examples:**

```bash
# Recently-disconnected clients
its unifi clients offline

# Pipe-friendly output — use with jq / scripts.
its unifi clients offline --json
```

---

### guests

> Source: `src/providers/unifi/commands/clients.ts`

| Command | Description |
|---------|-------------|
| `its unifi guests authorise <mac>` | Authorise a guest WiFi client. Permits a guest network device. |
| `its unifi guests unauthorise <mac>` | Revoke guest WiFi authorisation. Revokes a guest authorisation. |

#### `its unifi guests authorise <mac>`

Authorise a guest WiFi client. Permits a guest network device.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--minutes` | `` | Duration in minutes | — |
| `--up` | `` | Upload speed limit (Kbps) | — |
| `--down` | `` | Download speed limit (Kbps) | — |
| `--megabytes` | `` | Data transfer limit (MB) | — |
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi guests authorise <mac> --minutes 1440

# Pipe-friendly output — use with jq / scripts.
its unifi guests authorise <mac> --minutes 1440 --json
```

#### `its unifi guests unauthorise <mac>`

Revoke guest WiFi authorisation. Revokes a guest authorisation.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi guests unauthorise <mac>

# Pipe-friendly output — use with jq / scripts.
its unifi guests unauthorise <mac> --json
```

---

### networks

> Source: `src/providers/unifi/commands/networks.ts`

| Command | Description |
|---------|-------------|
| `its unifi networks` | List networks and VLANs. Surfaces the most common fields; pass --json for raw shape. |

#### `its unifi networks`

List networks and VLANs. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi networks

# Pipe-friendly output — use with jq / scripts.
its unifi networks --json

# Re-runs every 10s — handy for dashboards or incident response.
its unifi networks --watch
```

---

### wlans

> Source: `src/providers/unifi/commands/networks.ts`

| Command | Description |
|---------|-------------|
| `its unifi wlans` | List WiFi SSIDs. Surfaces the most common fields; pass --json for raw shape. |
| `its unifi wlans toggle <wlan_id>` | Enable or disable a WiFi SSID. Toggle a boolean state; idempotent. |
| `its unifi wlans password <wlan_id>` | Update WiFi password for an SSID. Rotate a PSK / passphrase. Disconnects every client on the SSID — use --confirm. |

#### `its unifi wlans`

List WiFi SSIDs. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi wlans

# Pipe-friendly output — use with jq / scripts.
its unifi wlans --json

# Re-runs every 10s — handy for dashboards or incident response.
its unifi wlans --watch
```

#### `its unifi wlans toggle <wlan_id>`

Enable or disable a WiFi SSID. Toggle a boolean state; idempotent.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--enable` | `` | Enable (true) or disable (false) the SSID | true |
| `--site` | `` | Site name override | — |

**Examples:**

```bash
# Toggles whatever the current enabled state is.
its unifi wlans toggle <wlan-id>

# Explicit on; --disable forces off.
its unifi wlans toggle <wlan-id> --enable
```

#### `its unifi wlans password <wlan_id>`

Update WiFi password for an SSID. Rotate a PSK / passphrase. Disconnects every client on the SSID — use --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--passphrase` | `` | New WiFi passphrase | — |
| `--site` | `` | Site name override | — |
| `--confirm` | `` | Confirm the PSK rotation | — |

**Examples:**

```bash
its unifi wlans password <wlan-id> --passphrase "new-password" --confirm

# Pipe-friendly output — use with jq / scripts.
its unifi wlans password <wlan-id> --passphrase "new-password" --confirm --json
```

---

### firewall

> Source: `src/providers/unifi/commands/networks.ts`

| Command | Description |
|---------|-------------|
| `its unifi firewall` | List firewall rules. Surfaces the most common fields; pass --json for raw shape. |
| `its unifi firewall groups` | List firewall groups. List groups for a resource. |

#### `its unifi firewall`

List firewall rules. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi firewall

# Pipe-friendly output — use with jq / scripts.
its unifi firewall --json

# Re-runs every 10s — handy for dashboards or incident response.
its unifi firewall --watch
```

#### `its unifi firewall groups`

List firewall groups. List groups for a resource.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi firewall groups

# Pipe-friendly output — use with jq / scripts.
its unifi firewall groups --json
```

---

### routes

> Source: `src/providers/unifi/commands/networks.ts`

| Command | Description |
|---------|-------------|
| `its unifi routes` | List static routes. Surfaces the most common fields; pass --json for raw shape. |

#### `its unifi routes`

List static routes. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi routes

# Pipe-friendly output — use with jq / scripts.
its unifi routes --json

# Re-runs every 10s — handy for dashboards or incident response.
its unifi routes --watch
```

---

### portforwards

> Source: `src/providers/unifi/commands/portforwards.ts`

| Command | Description |
|---------|-------------|
| `its unifi portforwards` | List every WAN port-forward rule — your inbound attack surface. Columns: WAN port → forward IP:port, protocol and source restriction ("any" = open to the whole internet). Pass --json for the raw shape. |
| `its unifi portforwards toggle <id>` | Enable or disable a WAN port-forward rule — close an internet-exposed forward. Pass --enable or --disable. |

#### `its unifi portforwards`

List every WAN port-forward rule — your inbound attack surface. Columns: WAN port → forward IP:port, protocol and source restriction ("any" = open to the whole internet). Pass --json for the raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi portforwards

# Same command under the hyphenated resource name.
its unifi port-forwards

# Scope to a specific site.
its unifi portforwards --site t7kq3dcp

# Raw rule shape including WAN interface and logging.
its unifi portforwards --json
```

#### `its unifi portforwards toggle <id>`

Enable or disable a WAN port-forward rule — close an internet-exposed forward. Pass --enable or --disable.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--enable` | `` | Enable the rule | — |
| `--disable` | `` | Disable the rule (close the forward) | — |
| `--site` | `` | Site name override | — |
| `--confirm` | `` | Confirm the change | — |

**Examples:**

```bash
its unifi portforwards toggle <id> --disable --confirm
```

---

### port-forwards

> Source: `src/providers/unifi/commands/portforwards.ts`

| Command | Description |
|---------|-------------|
| `its unifi port-forwards` | List every WAN port-forward rule — your inbound attack surface. Columns: WAN port → forward IP:port, protocol and source restriction ("any" = open to the whole internet). Pass --json for the raw shape. |
| `its unifi port-forwards toggle <id>` | Enable or disable a WAN port-forward rule — close an internet-exposed forward. Pass --enable or --disable. |

#### `its unifi port-forwards`

List every WAN port-forward rule — your inbound attack surface. Columns: WAN port → forward IP:port, protocol and source restriction ("any" = open to the whole internet). Pass --json for the raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi portforwards

# Same command under the hyphenated resource name.
its unifi port-forwards

# Scope to a specific site.
its unifi portforwards --site t7kq3dcp

# Raw rule shape including WAN interface and logging.
its unifi portforwards --json
```

#### `its unifi port-forwards toggle <id>`

Enable or disable a WAN port-forward rule — close an internet-exposed forward. Pass --enable or --disable.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--enable` | `` | Enable the rule | — |
| `--disable` | `` | Disable the rule (close the forward) | — |
| `--site` | `` | Site name override | — |
| `--confirm` | `` | Confirm the change | — |

**Examples:**

```bash
its unifi portforwards toggle <id> --disable --confirm
```

---

### ports

> Source: `src/providers/unifi/commands/portforwards.ts`

| Command | Description |
|---------|-------------|
| `its unifi ports` | List every WAN port-forward rule — your inbound attack surface. Columns: WAN port → forward IP:port, protocol and source restriction ("any" = open to the whole internet). Pass --json for the raw shape. |

#### `its unifi ports`

List every WAN port-forward rule — your inbound attack surface. Columns: WAN port → forward IP:port, protocol and source restriction ("any" = open to the whole internet). Pass --json for the raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi portforwards

# Same command under the hyphenated resource name.
its unifi port-forwards

# Scope to a specific site.
its unifi portforwards --site t7kq3dcp

# Raw rule shape including WAN interface and logging.
its unifi portforwards --json

its unifi ports

# Pipe-friendly output — use with jq / scripts.
its unifi ports --json

# Re-runs every 10s — handy for dashboards or incident response.
its unifi ports --watch
```

---

### events

> Source: `src/providers/unifi/commands/events.ts`

| Command | Description |
|---------|-------------|
| `its unifi events` | List recent events. Surfaces the most common fields; pass --json for raw shape. |

#### `its unifi events`

List recent events. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--hours` | `` | Look-back period in hours (default 24) | 24 |
| `--limit` | `` | Maximum number of events (default 50) | 50 |
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi events --hours 1

# Pipe-friendly output — use with jq / scripts.
its unifi events --hours 1 --json

# Re-runs every 10s — handy for dashboards or incident response.
its unifi events --hours 1 --watch
```

---

### alarms

> Source: `src/providers/unifi/commands/events.ts`

| Command | Description |
|---------|-------------|
| `its unifi alarms` | List alarms with archived status. Surfaces the most common fields; pass --json for raw shape. |
| `its unifi alarms count` | Count active (non-archived) alarms. Returns a single number — cheap for thresholds. |
| `its unifi alarms archive` | Archive all alarms. Archive (soft-delete) the record. Reversible. |

#### `its unifi alarms`

List alarms with archived status. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi alarms

# Pipe-friendly output — use with jq / scripts.
its unifi alarms --json

# Re-runs every 10s — handy for dashboards or incident response.
its unifi alarms --watch
```

#### `its unifi alarms count`

Count active (non-archived) alarms. Returns a single number — cheap for thresholds.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi alarms count

# Pipe-friendly output — use with jq / scripts.
its unifi alarms count --json
```

#### `its unifi alarms archive`

Archive all alarms. Archive (soft-delete) the record. Reversible.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm archiving all alarms | — |
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi alarms archive --confirm

# Pipe-friendly output — use with jq / scripts.
its unifi alarms archive --confirm --json
```

---

### rogue

> Source: `src/providers/unifi/commands/events.ts`

| Command | Description |
|---------|-------------|
| `its unifi rogue` | List detected rogue access points. Surfaces the most common fields; pass --json for raw shape. |

#### `its unifi rogue`

List detected rogue access points. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--hours` | `` | Look-back period in hours (default 24) | 24 |
| `--site` | `` | Site name override | — |

**Examples:**

```bash
# Detected rogue access points
its unifi rogue

# Pipe-friendly output — use with jq / scripts.
its unifi rogue --json

# Re-runs every 10s — handy for dashboards or incident response.
its unifi rogue --watch
```

---

### vouchers

> Source: `src/providers/unifi/commands/vouchers.ts`

| Command | Description |
|---------|-------------|
| `its unifi vouchers` | List guest WiFi vouchers. Surfaces the most common fields; pass --json for raw shape. |
| `its unifi vouchers create` | Create guest WiFi vouchers. Idempotent on duplicate names — use update/edit to mutate an existing record. |
| `its unifi vouchers revoke <id>` | Revoke/delete a guest voucher. Reverse an assignment. --confirm where required. |

#### `its unifi vouchers`

List guest WiFi vouchers. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi vouchers

# Pipe-friendly output — use with jq / scripts.
its unifi vouchers --json

# Re-runs every 10s — handy for dashboards or incident response.
its unifi vouchers --watch
```

#### `its unifi vouchers create`

Create guest WiFi vouchers. Idempotent on duplicate names — use update/edit to mutate an existing record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--minutes` | `` | Voucher duration in minutes | — |
| `--count` | `` | Number of vouchers to create (default 1) | 1 |
| `--quota` | `` | Number of uses per voucher (0 = unlimited, default 0) | 0 |
| `--note` | `` | Note to attach to the vouchers | — |
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi vouchers create --minutes 1440 --count 5

# Pipe-friendly output — use with jq / scripts.
its unifi vouchers create --minutes 1440 --count 5 --json
```

#### `its unifi vouchers revoke <id>`

Revoke/delete a guest voucher. Reverse an assignment. --confirm where required.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm the revocation | — |
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi vouchers revoke <voucher-id> --confirm

# Pipe-friendly output — use with jq / scripts.
its unifi vouchers revoke <voucher-id> --confirm --json
```

---

### dashboard

> Source: `src/providers/unifi/commands/dashboard.ts`

| Command | Description |
|---------|-------------|
| `its unifi dashboard` | Comprehensive site overview — health, devices, clients, alarms |

#### `its unifi dashboard`

Comprehensive site overview — health, devices, clients, alarms.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

**Examples:**

```bash
its unifi dashboard

# Pipe-friendly output — use with jq / scripts.
its unifi dashboard --json

# Re-runs every 10s — handy for dashboards or incident response.
its unifi dashboard --watch
```

---

### audit

> Source: `src/providers/unifi/commands/audit.ts`

| Command | Description |
|---------|-------------|
| `its unifi audit` | Audit RF and firewall/VLAN posture — co-channel overlap, unisolated VLANs, insecure WiFi defaults |

#### `its unifi audit`

Audit RF and firewall/VLAN posture — co-channel overlap, unisolated VLANs, insecure WiFi defaults.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site name override | — |

```bash
its unifi audit
```

---
