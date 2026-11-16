# Purview (`purview`)

Microsoft Purview compliance, read-only — retention policies and their rules (locations, action, duration) and retention labels with their publish and auto-apply policies.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [sp](./sp.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [attendance](./attendance.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md) · [yodeck](./yodeck.md)

## Contents

- [Setup](#setup)
- [retention](#retention)
- [labels](#labels)

## Setup

```bash
its purview setup           # Interactive wizard
its purview setup --check   # Check configuration status
its purview setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `EXO_APP_ID` | App registration used for the certificate connection (shared with `exo`) |
| `EXO_ORGANIZATION` | Tenant `.onmicrosoft.com` domain (shared with `exo`) |
| `EXO_CERTIFICATE_PFX` | Path to the .pfx (shared with `exo`) |
| `EXO_CERTIFICATE_PASSWORD` | Password for the .pfx, if it has one |

Needs **PowerShell 7** and the **ExchangeOnlineManagement** module. Connects with `Connect-IPPSSession` and the `exo` certificate settings; there is no signed-in route. The app must hold `Exchange.ManageAsApp` **and** a compliance directory role (Compliance Administrator or Compliance Data Administrator). Without the role the cmdlets fail with an access error. Nothing here writes.

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/purview/client.ts` | API client methods |
| `src/providers/purview/commands/` | Command definitions (split by resource) |
| `src/providers/purview/definition.ts` | definition |
| `src/providers/purview/parse.ts` | parse |

## Resources

### retention

| Command | Description |
|---------|-------------|
| `its purview retention` | List retention policies and their rules: where they apply (OneDrive, SharePoint, Exchange, Teams), the action and the duration. Read-only. |

#### `its purview retention`

List retention policies and their rules: where they apply (OneDrive, SharePoint, Exchange, Teams), the action and the duration. Read-only.

**Examples:**

```bash
# Would a retention policy keep OneDrive files we expire?
its purview retention list --ai
```

---

### labels

| Command | Description |
|---------|-------------|
| `its purview labels` | List retention labels, with the policies that publish them and the ones that apply them automatically (and their query). Read-only. |

#### `its purview labels`

List retention labels, with the policies that publish them and the ones that apply them automatically (and their query). Read-only.

```bash
its purview labels
```

---
