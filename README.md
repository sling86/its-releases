# its CLI

[![Latest release](https://img.shields.io/github/v/release/sling86/its-releases?include_prereleases&label=release&sort=semver)](https://github.com/sling86/its-releases/releases/latest)
[![Downloads](https://img.shields.io/github/downloads/sling86/its-releases/total?label=downloads)](https://github.com/sling86/its-releases/releases)
[![Platforms](https://img.shields.io/badge/platforms-Windows%20%C2%B7%20Linux-blue)](#install)
[![License](https://img.shields.io/badge/license-proprietary-lightgrey)](./LICENSE)

A unified CLI over the SaaS and self-hosted tools admins run day-to-day — Entra ID, Intune, Tactical RMM, UniFi, Dokploy, Bitwarden, Wrike, Exchange Online, SharePoint, PeopleHR, Business Central, Power BI, Azure, Cloudflare and more. **18 providers, 570+ commands**, same command shape (`its <provider> <resource> [action]`) across all of them. Portable across any organisation — the provider list is modular, you only configure what you use.

This repository is the **distribution channel** — prebuilt binaries plus the rendered docs and changelog. The source is closed; binaries are free to use under the [proprietary licence](./LICENSE) for personal or single-organisation internal use.

## Contents

- [Install](#install)
- [Stable download URLs](#stable-download-urls)
- [Documentation](#documentation)
- [What's in each release](#whats-in-each-release)
- [Verify](#verify)
- [Licence](#licence)
- [Reporting bugs](#reporting-bugs)

## Install

### Linux (x64)

```bash
curl -fsSL https://github.com/sling86/its-releases/releases/latest/download/install.sh | bash
```

Drops `its` into `~/.local/bin`. Useful env knobs:

- `ITS_INSTALL_DIR=/opt/its/bin` — install somewhere else
- `ITS_VERSION=v0.2.23` — pin a specific version instead of `latest`

After install, run `its --completions install` to wire tab completion into your shell.

### Windows (x64)

```powershell
irm https://github.com/sling86/its-releases/releases/latest/download/install.ps1 | iex
```

Drops `its.exe` into `%LOCALAPPDATA%\Programs\its` and adds it to your user PATH. No admin rights needed. Useful env knobs (set before piping to `iex`):

- `$env:ITS_INSTALL_DIR = 'C:\tools\its'` — install somewhere else
- `$env:ITS_VERSION = 'v0.2.24'` — pin a specific version instead of `latest`

Prefer a click-through installer? Download from the [latest release](https://github.com/sling86/its-releases/releases/latest):

| Asset | When to use |
|-------|-------------|
| `ItsSetup.exe` | Inno Setup installer — adds to PATH, registers an uninstaller |
| `ItsSetup.zip` | Same payload as `ItsSetup.exe`, zipped — for environments that block `.exe` downloads |
| `its.exe` | Raw compiled binary — drop somewhere on `%PATH%` if you don't want either installer |

After install, open a new shell and run `its --completions install` for tab completion.

## Stable download URLs

These URLs always resolve to the most recent release — pin a version with `/releases/download/v0.2.23/<asset>` instead.

| Asset | URL |
|-------|-----|
| Linux tarball | `https://github.com/sling86/its-releases/releases/latest/download/its-linux-x64.tar.gz` |
| Linux raw binary | `https://github.com/sling86/its-releases/releases/latest/download/its-linux-x64` |
| Windows installer | `https://github.com/sling86/its-releases/releases/latest/download/ItsSetup.exe` |
| Windows zip | `https://github.com/sling86/its-releases/releases/latest/download/ItsSetup.zip` |
| Windows raw binary | `https://github.com/sling86/its-releases/releases/latest/download/its.exe` |
| Linux install script | `https://github.com/sling86/its-releases/releases/latest/download/install.sh` |
| Windows install script | `https://github.com/sling86/its-releases/releases/latest/download/install.ps1` |

## Documentation

The CLI is its own best documentation:

- `its --help` — top-level overview, all providers
- `its <provider>` — resources + actions for one provider
- `its <provider> <resource> help` — every flag for one command
- `its docs serve` — interactive browser UI with Cmd-K palette and runnable examples

Or read the rendered docs in this repo (auto-synced from source on every release):

- **[docs/index.md](docs/index.md)** — master contents, full resource tree, source map
- **[docs/cli.md](docs/cli.md)** — global flags, output modes, secret redaction
- **[CHANGELOG.md](CHANGELOG.md)** — versioned release notes

**Per-provider command reference:**

- [Tactical RMM](docs/rmm.md) · [Entra ID](docs/entra.md) · [Dokploy](docs/dokploy.md)
- [Bitwarden](docs/bw.md) · [SharePoint](docs/sp.md) · [UniFi Network](docs/unifi.md)
- [Wrike](docs/wrike.md) · [Azure CLI](docs/az.md) · [Exchange Online](docs/exo.md)
- [Intune](docs/intune.md) · [UniFi Protect](docs/protect.md) · [Power BI](docs/pbi.md)
- [Power Platform](docs/pa.md) · [Cloudflare](docs/cf.md) · [PeopleHR](docs/hr.md)
- [Business Central](docs/bc.md) · [ctxc memories](docs/ctxc.md) · [Docs UI](docs/docs.md)

## What's in each release

Every release page has the same structure:

1. **Quick install** — one-line installer for the platform(s) the release covers
2. **Changes** — auto-populated from the curated `CHANGELOG.md` section (see [CHANGELOG.md](CHANGELOG.md) in this repo), or a filtered `git log` between the previous tag and this one when no curated section exists. Only conventional-commit types that are user-visible appear (`feat`, `fix`, `perf`, `refactor`, `build`, `docs`, plus anything marked `BREAKING`).
3. **Assets** — the binaries listed above.

## Verify

Asset SHA-256s appear on the release page next to each download. To verify a Linux binary after install:

```bash
sha256sum ~/.local/bin/its
# Compare against the checksum on the release page.
```

## Licence

Closed source. Binaries are made available free of charge for **personal use** or **internal use within a single organisation**. Redistribution, reverse-engineering, and hosting as a service are not permitted without prior written permission.

Full text: **[LICENSE](LICENSE)**. For commercial licensing enquiries, [open an issue](https://github.com/sling86/its-releases/issues).

## Reporting bugs

[Open an issue](https://github.com/sling86/its-releases/issues) — include the output of `its --version` and the command that reproduced the problem.
