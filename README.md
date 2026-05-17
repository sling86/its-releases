# its CLI — releases

Distribution channel for the [its CLI](https://github.com/sling86/it-cli) (source is private).

This repo holds **only** release binaries — see the [Releases page](https://github.com/sling86/its-releases/releases) for downloads.

## Install (Linux)

```bash
curl -fsSL https://github.com/sling86/its-releases/releases/latest/download/install.sh | bash
```

Drops `its` in `~/.local/bin`. Override the install dir with `ITS_INSTALL_DIR=…` and pin a version with `ITS_VERSION=v0.2.11`.

## Install (Windows)

Download the latest installer from [Releases](https://github.com/sling86/its-releases/releases/latest):

- `ItsSetup.exe` — Inno Setup installer
- `ItsSetup.zip` — same payload, zipped
- `its.exe` — raw compiled binary (drop on `%PATH%`)

## Manual download (any platform)

Stable per-platform URLs that always point at the newest release:

| Asset | URL |
| --- | --- |
| Linux tarball | `https://github.com/sling86/its-releases/releases/latest/download/its-linux-x64.tar.gz` |
| Linux raw binary | `https://github.com/sling86/its-releases/releases/latest/download/its-linux-x64` |
| Windows installer | `https://github.com/sling86/its-releases/releases/latest/download/ItsSetup.exe` |
| Windows zip | `https://github.com/sling86/its-releases/releases/latest/download/ItsSetup.zip` |
| Windows raw binary | `https://github.com/sling86/its-releases/releases/latest/download/its.exe` |
| Install script | `https://github.com/sling86/its-releases/releases/latest/download/install.sh` |
