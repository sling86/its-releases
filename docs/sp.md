# SharePoint (`sp`)

SharePoint Online — sites, document libraries, lists, files, search, permissions, pages.

[Index](./index.md) · [CLI Reference](./cli.md) · [README](../README.md)
Other providers: [rmm](./rmm.md) · [entra](./entra.md) · [dokploy](./dokploy.md) · [bw](./bw.md) · [unifi](./unifi.md) · [wrike](./wrike.md) · [az](./az.md) · [exo](./exo.md) · [intune](./intune.md) · [protect](./protect.md) · [pbi](./pbi.md) · [pa](./pa.md) · [cf](./cf.md) · [hr](./hr.md) · [bc](./bc.md) · [ctxc](./ctxc.md) · [docs](./docs.md) · [gh](./gh.md) · [outlook](./outlook.md) · [m365](./m365.md) · [teams](./teams.md)

## Contents

- [Setup](#setup)
- [sites](#sites)
- [drives](#drives)
- [lists](#lists)
- [files](#files)
- [search](#search)
- [permissions](#permissions)
- [groups](#groups)
- [recycle-bin](#recycle-bin)
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
| `src/providers/sp/find-group.ts` | find group |

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
| `its sp sites storage [siteId]` | Storage usage per site and drive (used/total). Defaults to all sites (up to 100); pass one or more site ids to scope. |

#### `its sp sites`

List all SharePoint sites. Surfaces the most common fields; pass --json for raw shape.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--all` | `` | Include personal (OneDrive) sites | — |

**Examples:**

```bash
its sp sites

# Re-runs every 10s — handy for dashboards or incident response.
its sp sites --watch
```

#### `its sp sites get <siteId>`

Get site details by ID. Pass the id (or any natural identifier) as the positional arg.

**Examples:**

```bash
its sp sites get <site-id>
```

#### `its sp sites search <query>`

Search sites by name. Substring match across the most relevant fields; case-insensitive.

**Examples:**

```bash
its sp sites search "marketing"
```

#### `its sp sites root`

Get the root site. Returns the document library root.

**Examples:**

```bash
its sp sites root
```

#### `its sp sites subsites <siteId>`

List child sites. Returns child sites of the given site.

**Examples:**

```bash
its sp sites subsites <site-id>
```

#### `its sp sites structure <siteId>`

Get site structure (drives, lists, subsites). Walks the site hierarchy + drives.

**Examples:**

```bash
# Lists, drives, subsites in one view
its sp sites structure <site-id>
```

#### `its sp sites storage [siteId]`

Storage usage per site and drive (used/total). Defaults to all sites (up to 100); pass one or more site ids to scope.

```bash
its sp sites storage [siteId]
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
| `its sp drives recent <siteId>` | Recently modified items in a drive over the last N days (delta query). Requires --drive. |

#### `its sp drives <siteId>`

List document libraries on a site. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its sp drives <site-id>

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
its sp drives get <site-id> --drive <drive-id> --item <item-id>
```

#### `its sp drives recent <siteId>`

Recently modified items in a drive over the last N days (delta query). Requires --drive.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--drive` | `` | Drive ID (from `its sp drives <siteId>`) | — |
| `--days` | `` | Look-back window in days | 7 |

```bash
its sp drives recent <siteId>
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
its sp lists create-item example.sharepoint.com,1a2b3c4d,5e6f7a8b --list "IT Assets" --fields '{"Title":"Dell 5540","Serial":"ABC123"}'

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
its sp lists update-item example.sharepoint.com,1a2b3c4d,5e6f7a8b --list "IT Assets" --item 42 --fields '{"Status":"Retired"}'

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
its sp lists delete-item example.sharepoint.com,1a2b,3c4d --list "IT Assets" --item 42 --confirm

its sp lists delete-item <site-id> --list <list-id> --item <item-id> --confirm
```

---

### files

> Source: `src/providers/sp/commands/files.ts`

| Command | Description |
|---------|-------------|
| `its sp files download` | Download a drive item to disk (--out) or pipe binary-safe to stdout. Resolves the pre-signed @microsoft.graph.downloadUrl from item metadata and fetches that — `sp graph get .../content` corrupts binary on the UTF-8 path. |
| `its sp files upload <siteId>` | Upload a text file. Stream a local file to the resource. |
| `its sp files folder <siteId>` | Create a folder under a parent item. |
| `its sp files delete <siteId>` | Delete a file or folder (moves to recycle bin). Permanent — use --confirm. Audit trail (if the upstream supports it) keeps the deletion record. |
| `its sp files share <siteId>` | Create a sharing link for a file/folder (Graph createLink) and return its URL. --type view|edit, --scope organisation|anonymous (anonymous may be tenant-blocked). |
| `its sp files move <siteId>` | Move or rename a file. Move an item between folders (reversible). |
| `its sp files checkout <siteId>` | Check out a file for editing. Locks the item against concurrent edits. |
| `its sp files checkin <siteId>` | Check in a file. Releases the lock after editing. |
| `its sp files versions <siteId>` | List file version history. Returns version history for a file. |
| `its sp files restore <siteId>` | Restore a file to a previous version. Restore a soft-deleted item from trash. |

#### `its sp files download`

Download a drive item to disk (--out) or pipe binary-safe to stdout. Resolves the pre-signed @microsoft.graph.downloadUrl from item metadata and fetches that — `sp graph get .../content` corrupts binary on the UTF-8 path.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--site` | `` | Site ID (with --drive --item|--path) | — |
| `--user` | `` | User UPN — operate on /users/<upn>/drive | — |
| `--drive` | `` | Drive ID (with --item or --path) | — |
| `--item` | `` | Drive item ID | — |
| `--path` | `` | Item path relative to drive root (e.g. /Folder/file.docx) | — |
| `--url` | `` | Pre-signed @microsoft.graph.downloadUrl (skips metadata lookup) | — |
| `--out` | `` | Local file path to write to. Omit to pipe to stdout. | — |

**Examples:**

```bash
its sp files download --user jane.smith@example.com --item 01Q3JEFHMUOTAVKHPGWNBJPEDKM376OQH6 --out out.docx

its sp files download --site <siteId> --drive <driveId> --path "/Folder/file.pdf" --out file.pdf

its sp files download --url "https://.../download" --out file.bin

its sp files download --user jane.smith@example.com --item <id> | sha256sum
```

#### `its sp files upload <siteId>`

Upload a text file. Stream a local file to the resource.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--drive` | `` | Drive ID | — |
| `--path` | `` | Parent path (default /) | / |
| `--name` | `` | File name | — |
| `--content` | `` | Text content to upload | — |
| `--content-file` | `` | Read --content from a local UTF-8 file (use for content > ~15KB — Windows command-line cap) | — |

**Examples:**

```bash
its sp files upload example.sharepoint.com,1a2b3c4d,5e6f7a8b --drive b!xY7 --path /Policies --content-file ./policy.md

its sp files upload <site-id> --drive <drive-id> --path "Shared Documents" --name report.pdf --content-file ./report.pdf
```

#### `its sp files folder <siteId>`

Create a folder under a parent item.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--drive` | `` | Drive ID | — |
| `--parent` | `` | Parent item ID | — |
| `--name` | `` | Folder name | — |

**Examples:**

```bash
its sp files folder example.sharepoint.com,1a2b3c4d,5e6f7a8b --drive b!xY7 --parent root --name "2026 Audits"

its sp files folder <site-id> --drive <drive-id> --parent <parent-id> --name "New Folder"
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
# Goes to the site recycle bin — restore with `its sp recycle-bin restore`
its sp files delete example.sharepoint.com,1a2b,3c4d --drive b!xY7 --item 01Q3JEFH --confirm

its sp files delete <site-id> --drive <drive-id> --item <item-id> --confirm
```

#### `its sp files share <siteId>`

Create a sharing link for a file/folder (Graph createLink) and return its URL. --type view|edit, --scope organisation|anonymous (anonymous may be tenant-blocked).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--drive` | `` | Drive ID | — |
| `--item` | `` | Item ID | — |
| `--type` | `` | Link type | view |
| `--scope` | `` | Link scope | organization |

**Examples:**

```bash
its sp files share example.sharepoint.com,1a2b,3c4d --drive b!xY7 --item 01Q3JEFH --type view --scope organization

# Creates a link anyone with the URL can open — audit these with `its sp audit sharing-links`
its sp files share example.sharepoint.com,1a2b,3c4d --drive b!xY7 --item 01Q3JEFH --type view --scope anonymous
```

#### `its sp files move <siteId>`

Move or rename a file. Move an item between folders (reversible).

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--drive` | `` | Drive ID | — |
| `--item` | `` | Item ID | — |
| `--name` | `` | New file name | — |
| `--parent` | `` | New parent folder ID | — |

**Examples:**

```bash
its sp files move example.sharepoint.com,1a2b3c4d,5e6f7a8b --drive b!xY7 --item 01Q3JEFH --name "policy-v2.docx"

its sp files move example.sharepoint.com,1a2b3c4d,5e6f7a8b --drive b!xY7 --item 01Q3JEFH --parent 01ARCHIVE

its sp files move <site-id> --drive <drive-id> --item <item-id> --name "new.docx" --parent <parent-id>
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
its sp files checkout example.sharepoint.com,1a2b3c4d,5e6f7a8b --drive b!xY7 --item 01Q3JEFH

its sp files checkout <site-id> --drive <drive-id> --item <item-id>
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
its sp files checkin example.sharepoint.com,1a2b3c4d,5e6f7a8b --drive b!xY7 --item 01Q3JEFH --comment "Updated section 4"

its sp files checkin <site-id> --drive <drive-id> --item <item-id> --comment "v2"
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
its sp files restore example.sharepoint.com,1a2b3c4d,5e6f7a8b --drive b!xY7 --item 01Q3JEFH --version 3.0 --confirm

its sp files restore <site-id> --item <item-id> --version <version-id>
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

# Re-runs every 10s — handy for dashboards or incident response.
its sp search "quarterly report" --watch
```

---

### permissions

> Source: `src/providers/sp/commands/permissions.ts`

| Command | Description |
|---------|-------------|
| `its sp permissions find-group <groupId>` | Reverse-lookup: which sites grant an Entra group access, directly or nested inside a site's Owners/Members group. Run this before retiring a security group. Reports sites it could not read rather than counting them as clear — absence of hits only proves the group is unused if every site was readable. |
| `its sp permissions <siteId>` | List app-level site permissions. Surfaces the most common fields; pass --json for raw shape. |
| `its sp permissions item <siteId>` | List sharing permissions on a file or folder. Single record detail. |
| `its sp permissions share <siteId>` | Create a sharing link. Creates a sharing link / direct grant. |
| `its sp permissions grant-app <siteId>` | Grant the it-cli app (or another app via --app) a Sites.Selected role on one site. Useful for bootstrapping the role needed by `its sp groups *`. Requires Sites.FullControl.All on the CALLING credentials — typically via a separate admin app (SP_ADMIN_CLIENT_ID/SP_ADMIN_CLIENT_SECRET) or a one-off elevation. |
| `its sp permissions remove <siteId>` | Remove a sharing permission. Permanent — use --confirm. |

#### `its sp permissions find-group <groupId>`

Reverse-lookup: which sites grant an Entra group access, directly or nested inside a site's Owners/Members group. Run this before retiring a security group. Reports sites it could not read rather than counting them as clear — absence of hits only proves the group is unused if every site was readable.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--top` | `` | Maximum sites to scan (default 200) | — |
| `--concurrency` | `` | Sites scanned in parallel (default 6) | — |

**Examples:**

```bash
its sp permissions find-group 462e4d2a-1f3c-4b8e-9d21-7a5e0c9b1234

its sp permissions find-group <groupId> --top 50 --concurrency 10
```

#### `its sp permissions <siteId>`

List app-level site permissions. Surfaces the most common fields; pass --json for raw shape.

**Examples:**

```bash
its sp permissions <site-id>

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
its sp permissions share example.sharepoint.com,1a2b,3c4d --drive b!xY7 --item 01Q3JEFH --type view --scope users

its sp permissions share <site-id> --item <item-id> --type view --scope organization
```

#### `its sp permissions grant-app <siteId>`

Grant the it-cli app (or another app via --app) a Sites.Selected role on one site. Useful for bootstrapping the role needed by `its sp groups *`. Requires Sites.FullControl.All on the CALLING credentials — typically via a separate admin app (SP_ADMIN_CLIENT_ID/SP_ADMIN_CLIENT_SECRET) or a one-off elevation.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--role` | `` | Role to grant: read, write, or fullcontrol | fullcontrol |
| `--app` | `` | Client (object) id of the app to grant. Defaults to the current SP_CLIENT_ID / CLIENT_ID. | — |
| `--name` | `` | Display name to store with the grant. Defaults to 'its-cli'. | its-cli |

**Examples:**

```bash
# Sites.Selected grant — scopes an app to this one site instead of the whole tenant
its sp permissions grant-app example.sharepoint.com,1a2b,3c4d --app 8f1c2d3e-4a5b-6c7d-8e9f-0a1b2c3d4e5f --role write --name "its CLI"
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
its sp permissions remove example.sharepoint.com,1a2b3c4d,5e6f7a8b --drive b!xY7 --item 01Q3JEFH --permission aTowIzQuZg --confirm

its sp permissions remove <site-id> --drive <drive-id> --item <item-id> --permission <permission-id> --confirm
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

**Examples:**

```bash
its sp groups add-member example.sharepoint.com,1a2b3c4d,5e6f7a8b "IT Owners" jane.smith@example.com

its sp groups add-member example.sharepoint.com,1a2b3c4d,5e6f7a8b "IT Owners" 8f1c2d3e-4a5b-6c7d-8e9f-0a1b2c3d4e5f
```

#### `its sp groups remove-member <site> <group> <principal>`

Remove a member from an SP site group. Destructive — use --confirm.

**Flags:**

| Flag | Alias | Description | Default |
|------|-------|-------------|---------|
| `--confirm` | `` | Confirm removal | — |

**Examples:**

```bash
its sp groups remove-member example.sharepoint.com,1a2b3c4d,5e6f7a8b "IT Owners" jane.smith@example.com --confirm
```

---

### recycle-bin

> Source: `src/providers/sp/commands/recycle-bin.ts`

| Command | Description |
|---------|-------------|
| `its sp recycle-bin <site>` | List the site recycle bin. Reads classic SP REST (/_api/web/RecycleBin) — Graph's /drives surface has no recycle-bin sub-resource. Pass the site id or web URL as the positional arg. |

#### `its sp recycle-bin <site>`

List the site recycle bin. Reads classic SP REST (/_api/web/RecycleBin) — Graph's /drives surface has no recycle-bin sub-resource. Pass the site id or web URL as the positional arg.

**Examples:**

```bash
its sp recycle-bin list <siteId>

its sp recycle-bin list https://example.sharepoint.com/sites/IT
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
| `--raw` | `` | Return the response body as raw bytes (no JSON decode). Required for binary endpoints like /content. Currently honoured by the `sp` provider. | — |
| `--out` | `` | Write the response to this file path instead of stdout. Implies --raw. | — |

**Examples:**

```bash
its sp graph get /users

its sp graph get /administrativeUnits --beta

its sp graph get /users --header ConsistencyLevel=eventual

its sp graph get "/sites/<site-id>/lists"
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
its sp graph post /users --body '{"displayName":"Jane Smith"}'

its sp graph post /administrativeUnits --beta

its sp graph post /users --header ConsistencyLevel=eventual

its sp graph post "/sites/<site-id>/lists" --body @./new-list.json
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
its sp graph patch /users --body '{"displayName":"Jane Smith"}'

its sp graph patch /administrativeUnits --beta

its sp graph patch /users --header ConsistencyLevel=eventual

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
its sp graph put /users --body '{"displayName":"Jane Smith"}'

its sp graph put /administrativeUnits --beta

its sp graph put /users --header ConsistencyLevel=eventual

its sp graph put "/sites/<site-id>/drive/items/<item-id>/content" --body @./file.bin
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
# No confirmation prompt — the path is sent exactly as given
its sp graph delete /groups/8f1c2d3e-4a5b-6c7d-8e9f-0a1b2c3d4e5f

its sp graph delete /administrativeUnits --beta

its sp graph delete /users --header ConsistencyLevel=eventual

its sp graph delete "/sites/<site-id>/lists/<list-id>"
```

---
