# SharePoint (`sp`)

SharePoint Online — sites, document libraries, lists, files, search, permissions, pages.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md)

## Contents

- [Setup](#setup)
- [sites](#sites)
- [drives](#drives)
- [lists](#lists)
- [files](#files)
- [search](#search)
- [permissions](#permissions)
- [groups](#groups)
- [pages](#pages)
- [dashboard](#dashboard)
- [graph](#graph)

## Setup

```bash
its sp setup           # Interactive wizard
its sp setup --check   # Check configuration status
its sp setup --reset   # Re-run setup (overwrite config)
```

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `TENANT_ID` | Microsoft Entra tenant ID (same as Entra provider) |
| `SP_CLIENT_ID` | SharePoint app registration client ID (needs Sites.Read.All). Falls back to CLIENT_ID. |
| `SP_CLIENT_SECRET` | SharePoint app registration client secret. Falls back to CLIENT_SECRET. |

SharePoint can use dedicated credentials (`SP_CLIENT_ID`/`SP_CLIENT_SECRET`) or fall back to shared Entra credentials. The app needs `Sites.Read.All` for reads and `Sites.ReadWrite.All` for writes.

### Source Files

| File | Purpose |
|------|---------|
| `src/providers/sp/client.ts` | API client methods |
| `src/providers/sp/types.ts` | TypeScript interfaces |
| `src/providers/sp/commands/` | Command definitions (split by resource) |
| `src/providers/sp/definition.ts` | definition |

## Resources

### sites

> Source: `src/providers/sp/commands/sites.ts`

| Command | Description |
|---------|-------------|
| `its sp sites` | List all SharePoint sites. Surfaces the most common fields; pass --json for raw shape. |
| `its sp sites get <siteId>` | Get site details by ID. Pass the id (or any natural identifier) as the positional arg. |
| `its sp sites search <query>` | Search sites by name. Substring match across the most relevant fields; case-insensitive. |
| `its sp sites root` | Get the root site. Returns the document library root. |
| `its sp sites subsites <siteId>` | List child sites. Returns child sites of the given site. |
| `its sp sites structure <siteId>` | Get site structure (drives, lists, subsites). Walks the site hierarchy + drives. |

#### `its sp sites`

List all SharePoint sites. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--all` | `` | Include personal (OneDrive) sites | — |

**Examples:**

```bash
its sp sites

# Pipe-friendly output — use with jq / scripts.
its sp sites --json

# Re-runs every 10s — handy for dashboards or incident response.
its sp sites --watch
```

#### `its sp sites get <siteId>`

Get site details by ID. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its sp sites get <site-id>

# Pipe-friendly output — use with jq / scripts.
its sp sites get <site-id> --json
```

#### `its sp sites search <query>`

Search sites by name. Substring match across the most relevant fields; case-insensitive.

**Examples:**

```bash
its sp sites search "marketing"

# Pipe-friendly output — use with jq / scripts.
its sp sites search "marketing" --json
```

#### `its sp sites root`

Get the root site. Returns the document library root.

**Examples:**

```bash
its sp sites root

# Pipe-friendly output — use with jq / scripts.
its sp sites root --json
```

#### `its sp sites subsites <siteId>`

List child sites. Returns child sites of the given site.

**Examples:**

```bash
its sp sites subsites <site-id>

# Pipe-friendly output — use with jq / scripts.
its sp sites subsites <site-id> --json
```

#### `its sp sites structure <siteId>`

Get site structure (drives, lists, subsites). Walks the site hierarchy + drives.

**Examples:**

```bash
# Lists, drives, subsites in one view
its sp sites structure <site-id>

# Pipe-friendly output — use with jq / scripts.
its sp sites structure <site-id> --json
```

---

### drives

> Source: `src/providers/sp/commands/drives.ts`

| Command | Description |
|---------|-------------|
| `its sp drives <siteId>` | List document libraries on a site. Surfaces the most common fields; pass --json for raw shape. |
| `its sp drives root <siteId>` | List files at document library root. Returns the document library root. |
| `its sp drives folder <siteId>` | List folder contents. Returns the contents of a folder by path. |
| `its sp drives get <siteId>` | Get file or folder details. Pass the id (or any natural identifier) as the positional arg. |

#### `its sp drives <siteId>`

List document libraries on a site. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its sp drives <site-id>

# Pipe-friendly output — use with jq / scripts.
its sp drives <site-id> --json

# Re-runs every 10s — handy for dashboards or incident response.
its sp drives <site-id> --watch
```

#### `its sp drives root <siteId>`

List files at document library root. Returns the document library root.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--drive` | `` | Drive ID | — |
| `--top` | `` | Number of items to return | 50 |
| `--all` | `` | Fetch all results (overrides --top) | — |

**Examples:**

```bash
its sp drives root <site-id>

# Pipe-friendly output — use with jq / scripts.
its sp drives root <site-id> --json
```

#### `its sp drives folder <siteId>`

List folder contents. Returns the contents of a folder by path.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--drive` | `` | Drive ID | — |
| `--path` | `` | Item ID or /path to folder | — |

**Examples:**

```bash
its sp drives folder <site-id> --path "Shared Documents/Marketing"

# Pipe-friendly output — use with jq / scripts.
its sp drives folder <site-id> --path "Shared Documents/Marketing" --json
```

#### `its sp drives get <siteId>`

Get file or folder details. Pass the id (or any natural identifier) as the positional arg.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--drive` | `` | Drive ID | — |
| `--item` | `` | Item ID | — |

**Examples:**

```bash
its sp drives get <site-id> --path "Shared Documents/report.pdf"

# Pipe-friendly output — use with jq / scripts.
its sp drives get <site-id> --path "Shared Documents/report.pdf" --json
```

---

### lists

> Source: `src/providers/sp/commands/lists.ts`

| Command | Description |
|---------|-------------|
| `its sp lists <siteId>` | List all lists on a site. Surfaces the most common fields; pass --json for raw shape. |
| `its sp lists get <siteId>` | Get list details. Pass the id (or any natural identifier) as the positional arg. |
| `its sp lists columns <siteId>` | Get column definitions for a list. Returns column definitions for a list. |
| `its sp lists items <siteId>` | List items from a list. Returns rows of a list, with column values. |
| `its sp lists create-item <siteId>` | Create a list item. Idempotent on natural-key collision; use update-item to mutate. |
| `its sp lists update-item <siteId>` | Update a list item. PATCH — only supplied fields change. |
| `its sp lists delete-item <siteId>` | Delete a list item. Permanent — use --confirm. |

#### `its sp lists <siteId>`

List all lists on a site. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--all` | `` | Include hidden lists | — |

**Examples:**

```bash
its sp lists <site-id>

# Pipe-friendly output — use with jq / scripts.
its sp lists <site-id> --json

# Re-runs every 10s — handy for dashboards or incident response.
its sp lists <site-id> --watch
```

#### `its sp lists get <siteId>`

Get list details. Pass the id (or any natural identifier) as the positional arg.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--list` | `` | List ID | — |

**Examples:**

```bash
its sp lists get <site-id> --list <list-id>

# Pipe-friendly output — use with jq / scripts.
its sp lists get <site-id> --list <list-id> --json
```

#### `its sp lists columns <siteId>`

Get column definitions for a list. Returns column definitions for a list.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--list` | `` | List ID | — |
| `--all` | `` | Include hidden columns | — |

**Examples:**

```bash
its sp lists columns <site-id> --list <list-id>

# Pipe-friendly output — use with jq / scripts.
its sp lists columns <site-id> --list <list-id> --json
```

#### `its sp lists items <siteId>`

List items from a list. Returns rows of a list, with column values.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--list` | `` | List ID | — |
| `--top` | `` | Number of items to return | 100 |
| `--filter` | `` | OData filter expression | — |
| `--orderby` | `` | OData orderBy expression | — |

**Examples:**

```bash
its sp lists items <site-id> --list <list-id>

# Pipe-friendly output — use with jq / scripts.
its sp lists items <site-id> --list <list-id> --json
```

#### `its sp lists create-item <siteId>`

Create a list item. Idempotent on natural-key collision; use update-item to mutate.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--list` | `` | List ID | — |
| `--fields` | `` | JSON string of field values | — |

**Examples:**

```bash
its sp lists create-item <site-id> --list <list-id> --json '{"Title":"New row"}'

# Set arbitrary list columns via the inline JSON payload.
its sp lists create-item <site-id> --list <list-id> --json '{"Title":"New ticket","Priority":"High","Assignee":"jane.smith@example.com"}'
```

#### `its sp lists update-item <siteId>`

Update a list item. PATCH — only supplied fields change.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--list` | `` | List ID | — |
| `--item` | `` | Item ID | — |
| `--fields` | `` | JSON string of field values to update | — |

**Examples:**

```bash
its sp lists update-item <site-id> --list <list-id> --item <item-id> --json '{"Title":"Updated"}'

# PATCH semantics — only the supplied fields are changed.
its sp lists update-item <site-id> --list <list-id> --item <item-id> --json '{"Status":"Closed","ResolvedBy":"jane.smith@example.com"}'
```

#### `its sp lists delete-item <siteId>`

Delete a list item. Permanent — use --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--list` | `` | List ID | — |
| `--item` | `` | Item ID | — |
| `--confirm` | `` | Confirm deletion | — |

**Examples:**

```bash
its sp lists delete-item <site-id> --list <list-id> --item <item-id> --confirm

# Pipe-friendly output — use with jq / scripts.
its sp lists delete-item <site-id> --list <list-id> --item <item-id> --confirm --json
```

---

### files

> Source: `src/providers/sp/commands/files.ts`

| Command | Description |
|---------|-------------|
| `its sp files upload <siteId>` | Upload a text file. Stream a local file to the resource. |
| `its sp files folder <siteId>` | Create a folder. Returns the contents of a folder by path. |
| `its sp files delete <siteId>` | Delete a file or folder (moves to recycle bin). Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record. |
| `its sp files move <siteId>` | Move or rename a file. Move an item between folders. --confirm required. |
| `its sp files checkout <siteId>` | Check out a file for editing. Locks the item against concurrent edits. |
| `its sp files checkin <siteId>` | Check in a file. Releases the lock after editing. |
| `its sp files versions <siteId>` | List file version history. Returns version history for a file. |
| `its sp files restore <siteId>` | Restore a file to a previous version. Restore a soft-deleted item from trash. |

#### `its sp files upload <siteId>`

Upload a text file. Stream a local file to the resource.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--drive` | `` | Drive ID | — |
| `--path` | `` | Parent path (default /) | / |
| `--name` | `` | File name | — |
| `--content` | `` | Text content to upload | — |

**Examples:**

```bash
its sp files upload <site-id> --path "Shared Documents/report.pdf" --file ./report.pdf

# Pipe-friendly output — use with jq / scripts.
its sp files upload <site-id> --path "Shared Documents/report.pdf" --file ./report.pdf --json
```

#### `its sp files folder <siteId>`

Create a folder. Returns the contents of a folder by path.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--drive` | `` | Drive ID | — |
| `--parent` | `` | Parent item ID | — |
| `--name` | `` | Folder name | — |

**Examples:**

```bash
its sp files folder <site-id> --path "Shared Documents"

# Pipe-friendly output — use with jq / scripts.
its sp files folder <site-id> --path "Shared Documents" --json
```

#### `its sp files delete <siteId>`

Delete a file or folder (moves to recycle bin). Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--drive` | `` | Drive ID | — |
| `--item` | `` | Item ID | — |
| `--confirm` | `` | Confirm deletion | — |

**Examples:**

```bash
its sp files delete <site-id> --path "Shared Documents/old.docx" --confirm
```

#### `its sp files move <siteId>`

Move or rename a file. Move an item between folders. --confirm required.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--drive` | `` | Drive ID | — |
| `--item` | `` | Item ID | — |
| `--name` | `` | New file name | — |
| `--parent` | `` | New parent folder ID | — |

**Examples:**

```bash
its sp files move <site-id> --from "Shared Documents/old.docx" --to "Shared Documents/new.docx"

# Pipe-friendly output — use with jq / scripts.
its sp files move <site-id> --from "Shared Documents/old.docx" --to "Shared Documents/new.docx" --json
```

#### `its sp files checkout <siteId>`

Check out a file for editing. Locks the item against concurrent edits.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--drive` | `` | Drive ID | — |
| `--item` | `` | Item ID | — |

**Examples:**

```bash
its sp files checkout <site-id> --path "Shared Documents/draft.docx"

# Pipe-friendly output — use with jq / scripts.
its sp files checkout <site-id> --path "Shared Documents/draft.docx" --json
```

#### `its sp files checkin <siteId>`

Check in a file. Releases the lock after editing.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--drive` | `` | Drive ID | — |
| `--item` | `` | Item ID | — |
| `--comment` | `` | Check-in comment |  |
| `--type` | `` | Check-in type: minor, major, or overwrite | major |

**Examples:**

```bash
its sp files checkin <site-id> --path "Shared Documents/draft.docx" --comment "v2"

# Pipe-friendly output — use with jq / scripts.
its sp files checkin <site-id> --path "Shared Documents/draft.docx" --comment "v2" --json
```

#### `its sp files versions <siteId>`

List file version history. Returns version history for a file.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--drive` | `` | Drive ID | — |
| `--item` | `` | Item ID | — |

**Examples:**

```bash
its sp files versions --item <item-id>

# Pipe-friendly output — use with jq / scripts.
its sp files versions --item <item-id> --json
```

#### `its sp files restore <siteId>`

Restore a file to a previous version. Restore a soft-deleted item from trash.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--drive` | `` | Drive ID | — |
| `--item` | `` | Item ID | — |
| `--version` | `` | Version ID to restore | — |
| `--confirm` | `` | Confirm restore | — |

**Examples:**

```bash
its sp files restore <site-id> --item <item-id> --version <version-id>

# Pipe-friendly output — use with jq / scripts.
its sp files restore <site-id> --item <item-id> --version <version-id> --json
```

---

### search

> Source: `src/providers/sp/commands/search.ts`

| Command | Description |
|---------|-------------|
| `its sp search <query>` | Search across SharePoint. Surfaces the most common fields; pass --json for raw shape. |

#### `its sp search <query>`

Search across SharePoint. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--type` | `` | Entity type: driveItem, listItem, list, or site | driveItem |
| `--top` | `` | Maximum results to return | 25 |

**Examples:**

```bash
its sp search "quarterly report"

# Pipe-friendly output — use with jq / scripts.
its sp search "quarterly report" --json

# Re-runs every 10s — handy for dashboards or incident response.
its sp search "quarterly report" --watch
```

---

### permissions

> Source: `src/providers/sp/commands/permissions.ts`

| Command | Description |
|---------|-------------|
| `its sp permissions <siteId>` | List app-level site permissions. Surfaces the most common fields; pass --json for raw shape. |
| `its sp permissions item <siteId>` | List sharing permissions on a file or folder. Single record detail. |
| `its sp permissions share <siteId>` | Create a sharing link. Creates a sharing link / direct grant. |
| `its sp permissions grant-app <siteId>` | Grant the it-cli app (or another app via --app) a Sites.Selected role on one site. Useful for bootstrapping the role needed by `its sp groups *`. Requires Sites.FullControl.All on the CALLING credentials — typically via a separate admin app (SP_ADMIN_CLIENT_ID/SP_ADMIN_CLIENT_SECRET) or a one-off elevation. |
| `its sp permissions remove <siteId>` | Remove a sharing permission. Permanent — use --confirm. |

#### `its sp permissions <siteId>`

List app-level site permissions. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its sp permissions <site-id>

# Pipe-friendly output — use with jq / scripts.
its sp permissions <site-id> --json

# Re-runs every 10s — handy for dashboards or incident response.
its sp permissions <site-id> --watch
```

#### `its sp permissions item <siteId>`

List sharing permissions on a file or folder. Single record detail.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--drive` | `` | Drive ID | — |
| `--item` | `` | Item ID | — |

**Examples:**

```bash
its sp permissions item <site-id> --item <item-id>

# Pipe-friendly output — use with jq / scripts.
its sp permissions item <site-id> --item <item-id> --json
```

#### `its sp permissions share <siteId>`

Create a sharing link. Creates a sharing link / direct grant.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--drive` | `` | Drive ID | — |
| `--item` | `` | Item ID | — |
| `--type` | `` | Link type: view, edit, or embed | view |
| `--scope` | `` | Link scope: anonymous or organization | organization |

**Examples:**

```bash
its sp permissions share --item <item-id> --user jane.smith@example.com --role read

# Pipe-friendly output — use with jq / scripts.
its sp permissions share --item <item-id> --user jane.smith@example.com --role read --json
```

#### `its sp permissions grant-app <siteId>`

Grant the it-cli app (or another app via --app) a Sites.Selected role on one site. Useful for bootstrapping the role needed by `its sp groups *`. Requires Sites.FullControl.All on the CALLING credentials — typically via a separate admin app (SP_ADMIN_CLIENT_ID/SP_ADMIN_CLIENT_SECRET) or a one-off elevation.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--role` | `` | Role to grant: read, write, or fullcontrol | fullcontrol |
| `--app` | `` | Client (object) id of the app to grant. Defaults to the current SP_CLIENT_ID / CLIENT_ID. | — |
| `--name` | `` | Display name to store with the grant. Defaults to 'its-cli'. | its-cli |

```bash
its sp permissions grant-app <siteId>
```

#### `its sp permissions remove <siteId>`

Remove a sharing permission. Permanent — use --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--drive` | `` | Drive ID | — |
| `--item` | `` | Item ID | — |
| `--permission` | `` | Permission ID to remove | — |
| `--confirm` | `` | Confirm removal | — |

**Examples:**

```bash
its sp permissions remove <site-id> --item <item-id> --perm <perm-id> --confirm
```

---

### groups

> Source: `src/providers/sp/commands/groups.ts`

| Command | Description |
|---------|-------------|
| `its sp groups <site>` | List SharePoint site groups (Owners/Members/Visitors + custom) on a site. Pass a site URL or Graph site id. |
| `its sp groups members <site> <group>` | List members of an SP site group. Accept group id or title. |
| `its sp groups add-member <site> <group> <principal>` | Add a UPN, Entra security-group object id, or pre-formed claim LoginName to an SP site group. Idempotent. |
| `its sp groups remove-member <site> <group> <principal>` | Remove a member from an SP site group. Destructive — use --confirm. |

#### `its sp groups <site>`

List SharePoint site groups (Owners/Members/Visitors + custom) on a site. Pass a site URL or Graph site id.

```bash
its sp groups <site>
```

#### `its sp groups members <site> <group>`

List members of an SP site group. Accept group id or title.

```bash
its sp groups members <site> <group>
```

#### `its sp groups add-member <site> <group> <principal>`

Add a UPN, Entra security-group object id, or pre-formed claim LoginName to an SP site group. Idempotent.

```bash
its sp groups add-member <site> <group> <principal>
```

#### `its sp groups remove-member <site> <group> <principal>`

Remove a member from an SP site group. Destructive — use --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm removal | — |

```bash
its sp groups remove-member <site> <group> <principal>
```

---

### pages

> Source: `src/providers/sp/commands/pages.ts`

| Command | Description |
|---------|-------------|
| `its sp pages <siteId>` | List modern pages on a site. Surfaces the most common fields; pass --json for raw shape. |
| `its sp pages get <siteId>` | Get page details. Pass the id (or any natural identifier) as the positional arg. |

#### `its sp pages <siteId>`

List modern pages on a site. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its sp pages <site-id>

# Pipe-friendly output — use with jq / scripts.
its sp pages <site-id> --json

# Re-runs every 10s — handy for dashboards or incident response.
its sp pages <site-id> --watch
```

#### `its sp pages get <siteId>`

Get page details. Pass the id (or any natural identifier) as the positional arg.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--page` | `` | Page ID | — |

**Examples:**

```bash
its sp pages get <site-id> --page <page-id>

# Pipe-friendly output — use with jq / scripts.
its sp pages get <site-id> --page <page-id> --json
```

---

### dashboard

> Source: `src/providers/sp/commands/dashboard.ts`

| Command | Description |
|---------|-------------|
| `its sp dashboard` | Comprehensive SharePoint overview. Surfaces the most common fields; pass --json for raw shape. |

#### `its sp dashboard`

Comprehensive SharePoint overview. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
# Sites, storage, recent activity
its sp dashboard

# Pipe-friendly output — use with jq / scripts.
its sp dashboard --json

# Re-runs every 10s — handy for dashboards or incident response.
its sp dashboard --watch
```

---

### graph

| Command | Description |
|---------|-------------|
| `its sp graph get <path>` | Raw Graph GET — pass any /v1.0 or /beta path (use --beta for beta) |
| `its sp graph post <path>` | Raw Graph POST — pass any /v1.0 or /beta path (use --beta for beta) |
| `its sp graph patch <path>` | Raw Graph PATCH — pass any /v1.0 or /beta path (use --beta for beta) |
| `its sp graph put <path>` | Raw Graph PUT — pass any /v1.0 or /beta path (use --beta for beta) |
| `its sp graph delete <path>` | Raw Graph DELETE — pass any /v1.0 or /beta path (use --beta for beta) |

#### `its sp graph get <path>`

Raw Graph GET — pass any /v1.0 or /beta path (use --beta for beta).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--beta` | `` | Use /beta instead of /v1.0 | — |
| `--header` | `` | Extra headers as comma-separated K=V pairs (e.g. Prefer=return=minimal) | — |

**Examples:**

```bash
its sp graph get "/sites/<site-id>/lists"

# Pipe-friendly output — use with jq / scripts.
its sp graph get "/sites/<site-id>/lists" --json
```

#### `its sp graph post <path>`

Raw Graph POST — pass any /v1.0 or /beta path (use --beta for beta).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--body` | `` | Request body — inline JSON string or @file.json to read from disk | — |
| `--beta` | `` | Use /beta instead of /v1.0 | — |
| `--header` | `` | Extra headers as comma-separated K=V pairs (e.g. Prefer=return=minimal) | — |

**Examples:**

```bash
its sp graph post "/sites/<site-id>/lists" --body @./new-list.json

# Pipe-friendly output — use with jq / scripts.
its sp graph post "/sites/<site-id>/lists" --body @./new-list.json --json
```

#### `its sp graph patch <path>`

Raw Graph PATCH — pass any /v1.0 or /beta path (use --beta for beta).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--body` | `` | Request body — inline JSON string or @file.json to read from disk | — |
| `--beta` | `` | Use /beta instead of /v1.0 | — |
| `--header` | `` | Extra headers as comma-separated K=V pairs (e.g. Prefer=return=minimal) | — |

**Examples:**

```bash
its sp graph patch "/sites/<site-id>/lists/<list-id>" --body '{"displayName":"Renamed"}'

# Pipe-friendly output — use with jq / scripts.
its sp graph patch "/sites/<site-id>/lists/<list-id>" --body  --json
```

#### `its sp graph put <path>`

Raw Graph PUT — pass any /v1.0 or /beta path (use --beta for beta).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--body` | `` | Request body — inline JSON string or @file.json to read from disk | — |
| `--beta` | `` | Use /beta instead of /v1.0 | — |
| `--header` | `` | Extra headers as comma-separated K=V pairs (e.g. Prefer=return=minimal) | — |

**Examples:**

```bash
its sp graph put "/sites/<site-id>/drive/items/<item-id>/content" --body @./file.bin

# Pipe-friendly output — use with jq / scripts.
its sp graph put "/sites/<site-id>/drive/items/<item-id>/content" --body @./file.bin --json
```

#### `its sp graph delete <path>`

Raw Graph DELETE — pass any /v1.0 or /beta path (use --beta for beta).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--beta` | `` | Use /beta instead of /v1.0 | — |
| `--header` | `` | Extra headers as comma-separated K=V pairs (e.g. Prefer=return=minimal) | — |

**Examples:**

```bash
its sp graph delete "/sites/<site-id>/lists/<list-id>" --confirm
```

---
