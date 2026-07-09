# Cloudflare (`cf`)

Cloudflare API v4 — zones (domains), DNS records, Cloudflare Tunnels (cloudflared), accounts.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [accounts](#accounts)
- [zones](#zones)
- [dns](#dns)
- [tunnels](#tunnels)
- [token](#token)

## Setup

```bash
its cf setup           # Interactive wizard
its cf setup --check   # Check configuration status
its cf setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `CLOUDFLARE_API_TOKEN` | Cloudflare API token (Bearer auth) |
| `CLOUDFLARE_ACCOUNT_ID` | Default account id for tunnel commands (optional) |

Auth is a single API token created at https://dash.cloudflare.com/profile/api-tokens. Recommended scopes:

- **Zone.Zone:Read** — list/get zones
- **Zone.DNS:Edit** — DNS create/update/delete
- **Account.Cloudflare Tunnel:Read** — tunnel inventory
- **Account.Account Settings:Read** — `accounts list`

For a read-only audit token use the same minus DNS:Edit. Token zone resources can be scoped to specific zones if you don't want tenant-wide.

`tunnels *` commands need an account id — set `CLOUDFLARE_ACCOUNT_ID` in `~/.its/.env` or pass `--account <id>` per call. Run `its cf accounts` once to find it.

Zone resolver: every command that takes a zone accepts either the 32-char zone id or the domain name (e.g. `--zone contractcandles.com`).

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/cf/client.ts` | API client methods |
| `src/providers/cf/types.ts` | TypeScript interfaces |
| `src/providers/cf/commands/` | Command definitions (split by resource) |
| `src/providers/cf/definition.ts` | definition |

## Resources

### accounts

> Source: `src/providers/cf/commands/accounts.ts`

| Command | Description |
|---------|-------------|
| `its cf accounts` | List Cloudflare accounts the token can see. Surfaces the most common fields; pass --json for raw shape. |

#### `its cf accounts`

List Cloudflare accounts the token can see. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its cf accounts

# Pipe-friendly output — use with jq / scripts.
its cf accounts --json

# Re-runs every 10s — handy for dashboards or incident response.
its cf accounts --watch
```

---

### zones

> Source: `src/providers/cf/commands/zones.ts`

| Command | Description |
|---------|-------------|
| `its cf zones` | List Cloudflare zones (domains). Surfaces the most common fields; pass --json for raw shape. |
| `its cf zones get <zone>` | Show zone details (accepts domain name or zone id). Pass the id (or any natural identifier) as the positional arg. |
| `its cf zones purge <zone>` | Purge zone cache (everything, or specific URLs with --file) |

#### `its cf zones`

List Cloudflare zones (domains). Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | Filter by exact domain name | — |

**Examples:**

```bash
its cf zones

its cf zones --name example.com

its cf zones

its cf zones --name example.com

# Re-runs every 10s — handy for dashboards or incident response.
its cf zones --watch
```

#### `its cf zones get <zone>`

Show zone details (accepts domain name or zone id). Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its cf zones get example.com

# Pipe-friendly output — use with jq / scripts.
its cf zones get example.com --json
```

#### `its cf zones purge <zone>`

Purge zone cache (everything, or specific URLs with --file).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--file` | `` | URL to purge (repeat for multiple). Omit to purge everything. | — |
| `--confirm` | `` | Confirm purging the entire zone cache (not needed with --file) | — |

**Examples:**

```bash
its cf zones purge example.com --confirm
```

---

### dns

> Source: `src/providers/cf/commands/dns.ts`

| Command | Description |
|---------|-------------|
| `its cf dns` | List DNS records for a zone. Surfaces the most common fields; pass --json for raw shape. |
| `its cf dns get <record_id>` | Show a single DNS record. Pass the id (or any natural identifier) as the positional arg. |
| `its cf dns create` | Create a DNS record. Idempotent on duplicate names — use update/edit to mutate an existing record. |
| `its cf dns update <record_id>` | Patch fields on an existing DNS record. PATCH semantics — only the supplied fields change. |
| `its cf dns delete <record_id>` | Delete a DNS record. Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record. |

#### `its cf dns`

List DNS records for a zone. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--zone` | `-z` | Zone domain or id (required) | — |
| `--type` | `` | Filter by record type | — |
| `--name` | `` | Filter by record name (e.g. www.example.com) | — |

**Examples:**

```bash
its cf dns --zone example.com

its cf dns --zone example.com --type A

its cf dns --zone example.com --name www

its cf dns --zone example.com

its cf dns --zone example.com --type A

# Re-runs every 10s — handy for dashboards or incident response.
its cf dns --zone example.com --watch
```

#### `its cf dns get <record_id>`

Show a single DNS record. Pass the id (or any natural identifier) as the positional arg.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--zone` | `-z` | Zone domain or id (required) | — |

**Examples:**

```bash
its cf dns get <record-id> --zone example.com

# Pipe-friendly output — use with jq / scripts.
its cf dns get <record-id> --zone example.com --json
```

#### `its cf dns create`

Create a DNS record. Idempotent on duplicate names — use update/edit to mutate an existing record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--zone` | `-z` | Zone domain or id | — |
| `--type` | `` | Record type | — |
| `--name` | `` | Record name (e.g. www or www.example.com) | — |
| `--content` | `` | Record content/value | — |
| `--ttl` | `` | TTL in seconds (1=auto, default 1) | 1 |
| `--proxied` | `` | Proxy through Cloudflare | false |
| `--priority` | `` | Priority (MX records) | — |
| `--comment` | `` | Record comment | — |

**Examples:**

```bash
its cf dns create --zone example.com --type A --name www --content 1.2.3.4

its cf dns create --zone example.com --type CNAME --name app --content app.example.dokploy.com --proxied false
```

#### `its cf dns update <record_id>`

Patch fields on an existing DNS record. PATCH semantics — only the supplied fields change.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--zone` | `-z` | Zone domain or id | — |
| `--type` | `` | New type | — |
| `--name` | `` | New name | — |
| `--content` | `` | New content | — |
| `--ttl` | `` | New TTL (1=auto) | — |
| `--proxied` | `` | Proxy through Cloudflare | — |
| `--priority` | `` | New priority | — |
| `--comment` | `` | New comment | — |

**Examples:**

```bash
its cf dns update <record-id> --zone example.com --content 5.6.7.8

# Pipe-friendly output — use with jq / scripts.
its cf dns update <record-id> --zone example.com --content 5.6.7.8 --json
```

#### `its cf dns delete <record_id>`

Delete a DNS record. Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--zone` | `-z` | Zone domain or id | — |
| `--confirm` | `` | Confirm deletion | — |

**Examples:**

```bash
its cf dns delete <record-id> --zone example.com --confirm
```

---

### tunnels

> Source: `src/providers/cf/commands/tunnels.ts`

| Command | Description |
|---------|-------------|
| `its cf tunnels` | List Cloudflare tunnels (cloudflared) for the account. Surfaces the most common fields; pass --json for raw shape. |
| `its cf tunnels get <tunnel>` | Show tunnel details (accepts name or id). Pass the id (or any natural identifier) as the positional arg. |
| `its cf tunnels connections <tunnel>` | List active cloudflared connections for a tunnel. Returns API connections for the resource. |
| `its cf tunnels delete <tunnel>` | Delete a Cloudflare tunnel (destructive — prompts for confirmation unless --yes). Use --cascade to drop active connections first. |
| `its cf tunnels routes <tunnel>` | List public hostname ingress rules for a tunnel. Routes defined in the configuration. |

#### `its cf tunnels`

List Cloudflare tunnels (cloudflared) for the account. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--account` | `` | Account id (defaults to CLOUDFLARE_ACCOUNT_ID) | — |

**Examples:**

```bash
its cf tunnels

# Pipe-friendly output — use with jq / scripts.
its cf tunnels --json

# Re-runs every 10s — handy for dashboards or incident response.
its cf tunnels --watch
```

#### `its cf tunnels get <tunnel>`

Show tunnel details (accepts name or id). Pass the id (or any natural identifier) as the positional arg.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--account` | `` | Account id | — |

**Examples:**

```bash
its cf tunnels get <tunnel-id>

# Pipe-friendly output — use with jq / scripts.
its cf tunnels get <tunnel-id> --json
```

#### `its cf tunnels connections <tunnel>`

List active cloudflared connections for a tunnel. Returns API connections for the resource.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--account` | `` | Account id | — |

**Examples:**

```bash
its cf tunnels connections <tunnel-id>

# Pipe-friendly output — use with jq / scripts.
its cf tunnels connections <tunnel-id> --json
```

#### `its cf tunnels delete <tunnel>`

Delete a Cloudflare tunnel (destructive — prompts for confirmation unless --yes). Use --cascade to drop active connections first.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--account` | `` | Account id | — |
| `--cascade` | `` | Force-delete even if connections are still active | false |
| `--yes` | `-y` | Skip the confirmation prompt | false |

**Examples:**

```bash
its cf tunnels delete <tunnel-id> --yes
```

#### `its cf tunnels routes <tunnel>`

List public hostname ingress rules for a tunnel. Routes defined in the configuration.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--account` | `` | Account id | — |

**Examples:**

```bash
its cf tunnels routes <tunnel-id>

# Pipe-friendly output — use with jq / scripts.
its cf tunnels routes <tunnel-id> --json
```

---

### token

> Source: `src/providers/cf/commands/token.ts`

| Command | Description |
|---------|-------------|
| `its cf token url` | Print a Cloudflare dashboard URL pre-filled with the scopes the `cf` provider needs (Zone:Read, DNS:Edit, Tunnel:Read, Account:Read) |
| `its cf token request` | Open the prefilled Cloudflare token page in the browser, then prompt for the new token and save it (keychain + .env). Captures the account id automatically. |

#### `its cf token url`

Print a Cloudflare dashboard URL pre-filled with the scopes the `cf` provider needs (Zone:Read, DNS:Edit, Tunnel:Read, Account:Read).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | Token name shown on the Cloudflare dashboard | its CLI |

**Examples:**

```bash
# Print the Cloudflare dashboard URL with the right scopes
its cf token url

# Pipe-friendly output — use with jq / scripts.
its cf token url --json
```

#### `its cf token request`

Open the prefilled Cloudflare token page in the browser, then prompt for the new token and save it (keychain + .env). Captures the account id automatically.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--name` | `` | Token name shown on the Cloudflare dashboard | its CLI |
| `--no-open` | `` | Just print the URL, don't launch the browser | false |

**Examples:**

```bash
# Opens the Cloudflare dashboard, prompts for the new token, saves to keychain + .env
its cf token request

# Print the prefilled URL and prompt — useful on remote shells
its cf token request --no-open
```

---
