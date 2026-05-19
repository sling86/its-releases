# its CLI — releases

[![Latest release](https://img.shields.io/github/v/release/sling86/its-releases?include_prereleases&label=release&sort=semver)](https://github.com/sling86/its-releases/releases/latest)
[![Downloads](https://img.shields.io/github/downloads/sling86/its-releases/total?label=downloads)](https://github.com/sling86/its-releases/releases)
[![Platforms](https://img.shields.io/badge/platforms-Windows%20%C2%B7%20Linux-blue)](#install)
[![Source](https://img.shields.io/badge/source-sling86%2Fit--cli-181717?logo=github)](https://github.com/sling86/it-cli)
[![License: MIT](https://img.shields.io/badge/license-MIT-green)](https://github.com/sling86/it-cli/blob/main/LICENSE)

Distribution channel for the [its CLI](https://github.com/sling86/it-cli) — a unified shell over Entra ID, Intune, Tactical RMM, UniFi, Dokploy, Bitwarden, Wrike, Exchange Online, SharePoint, PeopleHR, Business Central, Power BI, Azure, Cloudflare and more. **18 providers, 570+ commands**, same command shape across all of them.

This repository holds **only the prebuilt binaries** — source, docs, and issue tracking live at [`sling86/it-cli`](https://github.com/sling86/it-cli).

## Contents

- [Install](#install)
- [Stable download URLs](#stable-download-urls)
- [What's in each release](#whats-in-each-release)
- [Verify](#verify)
- [Where to read the docs](#where-to-read-the-docs)
- [Reporting bugs](#reporting-bugs)

## Install

### Linux (x64)

```bash
curl -fsSL https://github.com/sling86/its-releases/releases/latest/download/install.sh | bash
```

Drops `its` into `~/.local/bin`. Useful env knobs:

- `ITS_INSTALL_DIR=/opt/its/bin` — install somewhere else
- `ITS_VERSION=v0.2.21` — pin a specific version instead of `latest`

After install, run `its --completions install` to wire tab completion into your shell.

### Windows (x64)

Download from the [latest release](https://github.com/sling86/its-releases/releases/latest) and run:

| Asset | When to use |
|-------|-------------|
| `ItsSetup.exe` | Inno Setup installer — recommended (adds to PATH, registers uninstaller) |
| `ItsSetup.zip` | Same payload, zipped — for environments that block `.exe` downloads |
| `its.exe` | Raw compiled binary — drop somewhere on `%PATH%` if you don't want an installer |

Then in a new shell: `its --completions install` for tab completion.

## Stable download URLs

These URLs always resolve to the most recent release — pin a version with `/releases/download/v0.2.21/<asset>` instead.

| Asset | URL |
|-------|-----|
| Linux tarball | `https://github.com/sling86/its-releases/releases/latest/download/its-linux-x64.tar.gz` |
| Linux raw binary | `https://github.com/sling86/its-releases/releases/latest/download/its-linux-x64` |
| Windows installer | `https://github.com/sling86/its-releases/releases/latest/download/ItsSetup.exe` |
| Windows zip | `https://github.com/sling86/its-releases/releases/latest/download/ItsSetup.zip` |
| Windows raw binary | `https://github.com/sling86/its-releases/releases/latest/download/its.exe` |
| Install script | `https://github.com/sling86/its-releases/releases/latest/download/install.sh` |

## What's in each release

Every release page has the same structure:

1. **Quick install** — one-line installer for the platform(s) the release covers
2. **Changes** — auto-populated from the curated `CHANGELOG.md` section in [`sling86/it-cli`](https://github.com/sling86/it-cli/blob/main/CHANGELOG.md), or a filtered `git log` between the previous tag and this one when no curated section exists. Only conventional-commit types that are user-visible appear (`feat`, `fix`, `perf`, `refactor`, `build`, `docs`, plus anything marked `BREAKING`).
3. **Assets** — the binaries listed above.

For the full per-commit history, see [`it-cli`'s commit log](https://github.com/sling86/it-cli/commits/main).

## Verify

Asset SHA-256s appear on the release page next to each download. To verify a Linux binary after install:

```bash
sha256sum ~/.local/bin/its
# Compare against the checksum on the release page.
```

## Where to read the docs

The CLI is its own best documentation:

- `its --help` — top-level overview, all providers
- `its <provider>` — resources + actions for one provider
- `its <provider> <resource> help` — every flag for one command
- `its docs serve` — interactive browser UI with Cmd-K palette and runnable examples

Or read the rendered docs on GitHub:

- [README](https://github.com/sling86/it-cli/blob/main/README.md) — quick start, providers table, secret redaction
- [docs/index.md](https://github.com/sling86/it-cli/blob/main/docs/index.md) — full resource tree and source map
- [docs/cli.md](https://github.com/sling86/it-cli/blob/main/docs/cli.md) — global flags and output modes
- [CHANGELOG.md](https://github.com/sling86/it-cli/blob/main/CHANGELOG.md) — versioned changes

## Reporting bugs

Open issues against [`sling86/it-cli`](https://github.com/sling86/it-cli/issues), not this repo — this side is binary distribution only.
