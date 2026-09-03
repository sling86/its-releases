# its — Global Commands

> Auto-generated from `src/core/global-commands.ts`. Do not edit — run `bun run docs` to regenerate.

Commands invoked as `its <command>`, with no provider. Every one of them accepts `--help`.

| Command | What it does |
|---------|--------------|
| [`its audit`](#its-audit) | Security audit for one person — Entra account, MFA, sign-ins, RMM health, Intune compliance |
| [`its auth`](#its-auth) | Microsoft delegated sign-in — profiles, provider mapping, token diagnostics |
| [`its config`](#its-config) | Per-provider configuration status — what is set, what is in the keychain, secrets redacted |
| [`its diff`](#its-diff) | Compare a live command result against a saved JSON snapshot |
| [`its digest`](#its-digest) | Cross-provider status digest — one snapshot of everything |
| [`its export`](#its-export) | Export configuration from every configured provider, optionally with live data |
| [`its find`](#its-find) | Search every provider at once for a device, user, IP or hostname |
| [`its health`](#its-health) | Full health picture for one machine or person across all systems |
| [`its help`](#its-help) | Show the root help — providers, global options, output modes |
| [`its info`](#its-info) | Diagnostic snapshot — version, platform, install paths, update state, keychain |
| [`its inventory`](#its-inventory) | Cross-reference devices across RMM, Intune and UniFi to find the gaps |
| [`its log`](#its-log) | Append a timestamped entry to today's vault daily note |
| [`its onboard`](#its-onboard) | New-starter readiness snapshot — read-only, creates nothing |
| [`its resume`](#its-resume) | List open ctxc backlog and resume-prompt memories, grouped by project |
| [`its secrets`](#its-secrets) | Manage credentials in the OS keychain |
| [`its setup`](#its-setup) | Interactive setup wizard — configure the CLI or one provider |
| [`its status`](#its-status) | Which providers are configured, and optionally whether they answer |
| [`its trust-cert`](#its-trust-cert) | Trust-on-first-use store for self-signed TLS certificates |
| [`its tui`](#its-tui) | Interactive browser — provider, resource, action, no flags to memorise |
| [`its update`](#its-update) | Check for a newer release, and apply it with --apply |
| [`its user`](#its-user) | Everything about one person, across every system |
| [`its version`](#its-version) | Print the bundled version and exit |
| [`its watch`](#its-watch) | Re-run a command every N seconds with change highlighting |

## its audit

Security audit for one person — Entra account, MFA, sign-ins, RMM health, Intune compliance.

```bash
its audit <username|UPN>
its audit lifecycle-drift
```

Queries Entra (account state, MFA registration, sign-ins, risky-user status), RMM (agent health) and Intune (compliance) in parallel for a single person. `lifecycle-drift` instead reconciles PeopleHR leavers against Entra `accountEnabled` and Wrike leaver tickets, flagging mismatches such as an HR leaver still enabled in Entra.

**Examples**

```bash
its audit jane.smith@example.com
its audit jane
its audit lifecycle-drift
```

## its auth

Microsoft delegated sign-in — profiles, provider mapping, token diagnostics.

**Changes state** · **Interactive** — prompts for input

```bash
its auth login [--profile <name>] [--device-code]
its auth use <profile> [--default | --provider <a,b>]
its auth logout [--profile <name>]
its auth status
its auth doctor
its auth refresh [prefix]
its auth scopes [provider]
its auth setup [provider]
```

Runs the OAuth authorisation-code flow with PKCE against the `CLIENT_ID` app and a localhost callback. The refresh token persists to `~/.its/tokens/delegated.json` and is swapped to per-resource scopes without re-login. Named profiles let e.g. `entra`/`intune`/`sp` run as an admin account while `teams`/`outlook` run as your own user, with no flags at call time. `auth doctor` is the fastest way to see which providers will run as the user and which as the service principal.

**Examples**

```bash
its auth login
its auth login --profile admin
its auth use admin --provider entra,intune,sp
its auth doctor
```

## its config

Per-provider configuration status — what is set, what is in the keychain, secrets redacted.

```bash
its config
```

One row per provider: whether it is configured, which of its environment variables are set, which of its secrets are held in the OS keychain, and whether a session is active. Secret values are never printed — the output is safe to paste into a ticket.

**Examples**

```bash
its config
its config --json
```

## its diff

Compare a live command result against a saved JSON snapshot.

```bash
its diff <provider> <resource> <snapshot.json> [--key <column>]
```

Re-runs a command and diffs the result against a previously exported snapshot, reporting added, removed and changed records. `--key` picks the column that identifies a record; without it the first column is used.

**Examples**

```bash
its diff unifi clients yesterday.json
its diff entra users baseline.json --key upn
```

## its digest

Cross-provider status digest — one snapshot of everything.

```bash
its digest
```

Fans out across every configured provider and prints a single consolidated summary. Intended as the morning check.

**Examples**

```bash
its digest
its digest --json
```

## its export

Export configuration from every configured provider, optionally with live data.

```bash
its export [--data] [--json]
```

Exports the configuration of every configured provider. `--data` additionally fetches and includes live data from each one, which is considerably slower.

**Examples**

```bash
its export --json
its export --data --json
```

## its find

Search every provider at once for a device, user, IP or hostname.

```bash
its find <query>
```

Broadcasts one query across every configured provider and returns whatever matches — an agent in RMM, a user in Entra, a client on the UniFi network, a device in Intune. The fastest way to answer 'what do we know about this string'.

**Examples**

```bash
its find WKS-9
its find jane.smith@example.com
its find 192.168.1.50
```

## its health

Full health picture for one machine or person across all systems.

```bash
its health <hostname|username>
```

Resolves the target across RMM, Intune, Entra and UniFi and reports agent status, compliance, sign-in state and network presence together.

**Examples**

```bash
its health WKS-9
its health jane.smith@example.com
```

## its help

Show the root help — providers, global options, output modes.

```bash
its help
its --help
its <provider> --help
```

`its help` prints the root index. Every provider, resource and command also accepts `--help` for its own page, including all the global commands listed here.

**Examples**

```bash
its help
its rmm --help
its entra users --help
```

## its info

Diagnostic snapshot — version, platform, install paths, update state, keychain.

```bash
its info
```

Prints the information worth pasting into a bug report: bundled version, platform and Bun version, resolved install and config paths, update-check cache state, audit-log location and whether the OS keychain is available.

**Examples**

```bash
its info
```

## its inventory

Cross-reference devices across RMM, Intune and UniFi to find the gaps.

```bash
its inventory [--unifi]
```

Builds a device matrix across RMM, Intune and optionally UniFi, and surfaces the gaps — devices present in one system but missing from another. `--unifi` also folds in currently-online UniFi clients.

**Examples**

```bash
its inventory
its inventory --unifi --json
```

## its log

Append a timestamped entry to today's vault daily note.

**Changes state**

```bash
its log <text>
```

Appends a `### HH:MM — <text>` entry under `## Sessions` in today's daily note, creating the note if it does not exist. The entry is inserted above the ctxc-auto block so the aggregate stays at the bottom. Vault path defaults to `~/obsidian/VaulT`; override with `ITS_VAULT_PATH`.

**Examples**

```bash
its log "rotated the Dokploy token"
```

## its onboard

New-starter readiness snapshot — read-only, creates nothing.

```bash
its onboard preview <email>
```

Read-only prep check for a new starter. Pulls the PeopleHR record (name, role, department, company, location, start date), reports whether an Entra account already exists, and suggests a template colleague in the same department whose groups and licences can be copied. Creates and modifies nothing.

**Examples**

```bash
its onboard preview jane.smith@example.com
```

## its resume

List open ctxc backlog and resume-prompt memories, grouped by project.

```bash
its resume [--tag <tag>...] [--project <project>]
```

Lists open ctxc memories tagged `backlog` or `resume-prompt`, grouped by project — what was left in flight. Override the tags with `--tag` (repeatable) or narrow to one project with `--project`.

**Examples**

```bash
its resume
its resume --project it-cli
```

## its secrets

Manage credentials in the OS keychain.

**Changes state**

```bash
its secrets [list]
its secrets migrate
its secrets clear
its secrets audit-repos
```

Secrets live in the OS-native credential store (Windows Credential Locker, macOS Keychain, Linux libsecret) and override any `.env` value. `list` shows which entries exist — never their values. `migrate` moves secrets out of `~/.its/.env` into the keychain. `clear` removes every entry. `audit-repos` scans local repositories for committed credentials.

**Examples**

```bash
its secrets
its secrets migrate
its secrets audit-repos
```

## its setup

Interactive setup wizard — configure the CLI or one provider.

**Changes state** · **Interactive** — prompts for input

```bash
its setup
its <provider> setup
```

`its setup` reports what is already configured and names the exact next command. `its <provider> setup` runs that provider's wizard: it checks requirements, prompts for each value, routes secrets to the keychain, writes the rest to `.env.local`, then tests the connection before finishing.

**Examples**

```bash
its setup
its rmm setup
its bw setup --vault work
```

## its status

Which providers are configured, and optionally whether they answer.

```bash
its status [--test]
```

Lists every provider with its configured state. `--test` additionally calls each provider's connection test and reports reachability and latency — slower, and it makes a live call per provider.

**Examples**

```bash
its status
its status --test
```

## its trust-cert

Trust-on-first-use store for self-signed TLS certificates.

**Changes state**

```bash
its trust-cert [list]
its trust-cert <url> [--replace] [--yes]
its trust-cert remove <host:port>
```

Pins a self-signed certificate as a trust anchor for one host. Verification stays on — a pinned certificate passes and a changed one fails, which is what makes this safe rather than a blanket bypass. Used for self-signed controllers such as UniFi. `--replace` re-pins a host that already has an entry; `--yes` accepts without the confirmation prompt, which is required in any non-TTY context since the prompt is skipped rather than read from stdin. `remove` takes the host as it is pinned, INCLUDING the port — a bare hostname is normalised to :443 and will not match an entry stored under another port. Pinning is for self-signed certificates only: it disables hostname verification, so a host that has moved to a publicly trusted certificate should have its pin removed rather than refreshed.

**Examples**

```bash
its trust-cert
its trust-cert https://unifi.example.com
its trust-cert https://unifi.example.com:8443 --replace --yes
its trust-cert remove unifi.example.com:8443
```

## its tui

Interactive browser — provider, resource, action, no flags to memorise.

**Interactive** — prompts for input

```bash
its tui
its <provider> -i
```

Full-screen browser over every command. `its <provider> -i` jumps straight in, pre-seeded at that provider's resource step. Requires a TTY.

**Examples**

```bash
its tui
its rmm -i
```

## its update

Check for a newer release, and apply it with --apply.

```bash
its update
its update --apply
its --check-updates
```

Checks the release feed and reports whether a newer version exists. The result is cached in `~/.its/update-check.json`; a background check refreshes it, so this command forces a fresh look.

The remedy follows how `its` was installed, because the two differ: a **source checkout** (`bun link`, the shim runs `src/bootstrap.ts`) updates with `git pull --ff-only && bun install`, while a **binary install** runs the platform installer. Handing a source checkout the binary installer replaces the shim with a compiled binary and silently detaches the CLI from the repo it is developed in, so the two are never confused.

`--apply` performs the update rather than printing it. On a source checkout it refuses when tracked files are uncommitted — untracked scratch is fine — then fast-forwards and reinstalls dependencies; the new version is live immediately because the shim runs source. On a binary install it runs the platform installer and waits, so the exit code is the install's own. Builds are versioned rather than overwritten — POSIX repoints a symlink, Windows renames the running `its.exe` aside — so updating from inside a running `its` is safe, and the previous couple of builds stay on disk for rollback.

**Examples**

```bash
its update
its update --apply
```

## its user

Everything about one person, across every system.

```bash
its user <upn>
```

Single-user snapshot across providers: the Entra account with its groups, licences and last sign-in; RMM agents the user is logged into; Intune devices assigned to them; and recent Wrike tickets. Runs in parallel, and silently skips providers that are not configured.

**Examples**

```bash
its user jane.smith@example.com
```

## its version

Print the bundled version and exit.

```bash
its version
its --version
```

**Examples**

```bash
its version
```

## its watch

Re-run a command every N seconds with change highlighting.

```bash
its watch <provider> <resource> [action] [--flags]
its watch <provider> <resource> --interval <seconds>
```

Re-runs a command on a loop and highlights what changed between runs. `--interval` sets the poll period in seconds (default 5).

**Examples**

```bash
its watch rmm agents --status online --interval 10
its watch entra users --filter company=candle
its watch unifi clients
```
