# its CLI — Documentation Index

> Auto-generated from command definitions. Do not edit — run `bun run docs` to regenerate.

Start here to find any command, resource, or source file in the `its` CLI.

## Navigation

| Document | Description |
|----------|-------------|
| [README](../README.md) | Quick start, examples, setup |
| [cli.md](./cli.md) | CLI reference — usage, options, output modes |
| [rmm.md](./rmm.md) | Tactical RMM — 57 commands across 16 resources |
| [entra.md](./entra.md) | Entra ID — 100 commands across 21 resources |
| [dokploy.md](./dokploy.md) | Dokploy — 113 commands across 25 resources |
| [bw.md](./bw.md) | Bitwarden — 37 commands across 10 resources |
| [sp.md](./sp.md) | SharePoint — 45 commands across 10 resources |
| [unifi.md](./unifi.md) | UniFi Network — 38 commands across 14 resources |
| [wrike.md](./wrike.md) | Wrike — 48 commands across 12 resources |
| [az.md](./az.md) | Azure CLI — 23 commands across 10 resources |
| [exo.md](./exo.md) | Exchange Online — 33 commands across 8 resources |
| [intune.md](./intune.md) | Intune — 40 commands across 15 resources |
| [protect.md](./protect.md) | UniFi Protect — 6 commands across 4 resources |
| [pbi.md](./pbi.md) | Power BI — 21 commands across 6 resources |
| [pa.md](./pa.md) | Power Platform — 11 commands across 4 resources |
| [cf.md](./cf.md) | Cloudflare — 16 commands across 5 resources |
| [hr.md](./hr.md) | PeopleHR — 8 commands across 4 resources |
| [bc.md](./bc.md) | Business Central — 5 commands across 4 resources |
| [ctxc.md](./ctxc.md) | ctxc memories — 5 commands across 1 resources |
| [docs.md](./docs.md) | Docs UI — 5 commands across 5 resources |
| [gh.md](./gh.md) | GitHub — 4 commands across 2 resources |
| [outlook.md](./outlook.md) | Outlook — 42 commands across 11 resources |

**20 providers** · **187 resources** · **657 commands**

### [Tactical RMM](./rmm.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [agents](./rmm.md#agents) | list, stale, search, get, ping, reboot, remove, run, history, notes, pending, wake, edit, refresh | `src/providers/rmm/commands/agents.ts` |
| [dashboard](./rmm.md#dashboard) | list | `src/providers/rmm/commands/dashboard.ts` |
| [clients](./rmm.md#clients) | list, create, delete | `src/providers/rmm/commands/dashboard.ts` |
| [sites](./rmm.md#sites) | list, create, delete | `src/providers/rmm/commands/dashboard.ts` |
| [processes](./rmm.md#processes) | list, top, kill | `src/providers/rmm/commands/processes.ts` |
| [services](./rmm.md#services) | list, get, control, enable, disable | `src/providers/rmm/commands/services.ts` |
| [updates](./rmm.md#updates) | list, scan, install | `src/providers/rmm/commands/updates.ts` |
| [software](./rmm.md#software) | list, search | `src/providers/rmm/commands/software.ts` |
| [alerts](./rmm.md#alerts) | list, get | `src/providers/rmm/commands/alerts.ts` |
| [scripts](./rmm.md#scripts) | list, get, run, upload-local, delete, upsert | `src/providers/rmm/commands/scripts.ts` |
| [checks](./rmm.md#checks) | list, create, delete | `src/providers/rmm/commands/checks.ts` |
| [tasks](./rmm.md#tasks) | list, create, delete | `src/providers/rmm/commands/tasks.ts` |
| [policies](./rmm.md#policies) | list, get, checks, add-check, patch-policy | `src/providers/rmm/commands/policies.ts` |
| [diagnostics](./rmm.md#diagnostics) | list | `src/providers/rmm/commands/diagnostics.ts` |
| [doctor](./rmm.md#doctor) | list | `src/providers/rmm/commands/doctor.ts` |
| [custom-fields](./rmm.md#custom-fields) | list, set | `src/providers/rmm/commands/custom-fields.ts` |

### [Entra ID](./entra.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [users](./entra.md#users) | list, update, search, get, groups, chain, licences, invite, create, enable, bootstrap-admin, stale, disable, revoke-sessions, set-password, delete, transfer, reinstate | `src/providers/entra/commands/users.ts` |
| [groups](./entra.md#groups) | list, search, get, members, create, add-member, remove-member, edit-rule, audit-rules | `src/providers/entra/commands/groups.ts` |
| [licences](./entra.md#licences) | list, assign, remove, users, unlicensed, audit, waste | `src/providers/entra/commands/licences.ts` |
| [roles](./entra.md#roles) | list, members, assign, remove, assignments | `src/providers/entra/commands/roles.ts` |
| [signin](./entra.md#signin) | list, summary, explain, suspicious | `src/providers/entra/commands/signin.ts` |
| [tap](./entra.md#tap) | create, list, revoke | `src/providers/entra/commands/tap.ts` |
| [ca](./entra.md#ca) | list, enabled, get, patch, exclude-guests, create, delete, exclude-user, unexclude-user, why-blocked, named-locations | `src/providers/entra/commands/ca.ts` |
| [authmethods](./entra.md#authmethods) | policy, get, enable, disable, patch | `src/providers/entra/commands/auth-methods.ts` |
| [consent](./entra.md#consent) | list, add-scope, remove-scope, app-roles, app-role-grant, app-role-revoke | `src/providers/entra/commands/consent.ts` |
| [xtenant](./entra.md#xtenant) | default, trust-mfa, trust-device, partners, partner-add, partner-set | `src/providers/entra/commands/xtenant.ts` |
| [audit](./entra.md#audit) | list | `src/providers/entra/commands/audit.ts` |
| [security](./entra.md#security) | risky, mfa, admin-mfa | `src/providers/entra/commands/security.ts` |
| [directory](./entra.md#directory) | org, domains, deleted, tree, summary, app-usage | `src/providers/entra/commands/directory.ts` |
| [onboarding](./entra.md#onboarding) | summary, copy-groups, convert-mailbox | `src/providers/entra/commands/onboarding.ts` |
| [offboarding](./entra.md#offboarding) | summary, run | `src/providers/entra/commands/onboarding.ts` |
| [break-glass](./entra.md#break-glass) | audit | `src/providers/entra/commands/break-glass.ts` |
| [whoami](./entra.md#whoami) | show | `src/providers/entra/commands/whoami.ts` |
| [doctor](./entra.md#doctor) | list | `src/providers/entra/commands/doctor.ts` |
| [apps](./entra.md#apps) | register, add-password | `src/providers/entra/commands/apps.ts` |
| [admin-bootstrap](./entra.md#admin-bootstrap) | run | `src/providers/entra/commands/admin-bootstrap.ts` |
| [graph](./entra.md#graph) | get, post, patch, put, delete | — |

### [Dokploy](./dokploy.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [projects](./dokploy.md#projects) | list, get, create, delete | `src/providers/dokploy/commands/projects.ts` |
| [apps](./dokploy.md#apps) | list, get, create, delete, deploy, stop, start, restart, set-source, set-build, rebuild, wait-deploy, redeploy, logs, monitoring, traefik, status, clone, shell, migrate, env-runtime, apply-env, health, doctor, bootstrap, cert-status | `src/providers/dokploy/commands/apps.ts` |
| [databases](./dokploy.md#databases) | list, url, get, create, deploy, stop, delete | `src/providers/dokploy/commands/databases.ts` |
| [deployments](./dokploy.md#deployments) | list, queue, kill | `src/providers/dokploy/commands/deployments.ts` |
| [domains](./dokploy.md#domains) | list, create, check, delete | `src/providers/dokploy/commands/domains.ts` |
| [env](./dokploy.md#env) | list, push, set, unset, pull | `src/providers/dokploy/commands/env.ts` |
| [environments](./dokploy.md#environments) | list, env, push, pull, set | `src/providers/dokploy/commands/environments.ts` |
| [registries](./dokploy.md#registries) | list | `src/providers/dokploy/commands/infrastructure.ts` |
| [destinations](./dokploy.md#destinations) | list | `src/providers/dokploy/commands/infrastructure.ts` |
| [notifications](./dokploy.md#notifications) | list | `src/providers/dokploy/commands/infrastructure.ts` |
| [dashboard](./dokploy.md#dashboard) | list | `src/providers/dokploy/commands/dashboard.ts` |
| [mounts](./dokploy.md#mounts) | list, add, update, remove | `src/providers/dokploy/commands/mounts.ts` |
| [webhook](./dokploy.md#webhook) | check, setup, list | `src/providers/dokploy/commands/webhook.ts` |
| [nodes](./dokploy.md#nodes) | list, info, apps, stats | `src/providers/dokploy/commands/nodes.ts` |
| [cluster](./dokploy.md#cluster) | join-token | `src/providers/dokploy/commands/nodes.ts` |
| [containers](./dokploy.md#containers) | list, config, start, stop, restart, kill, remove | `src/providers/dokploy/commands/containers.ts` |
| [maintenance](./dokploy.md#maintenance) | disk, web, traefik, clean, time | `src/providers/dokploy/commands/maintenance.ts` |
| [traefik](./dokploy.md#traefik) | logs, acme | `src/providers/dokploy/commands/maintenance.ts` |
| [github](./dokploy.md#github) | providers, repos, branches | `src/providers/dokploy/commands/github.ts` |
| [users](./dokploy.md#users) | list, me, invite, remove | `src/providers/dokploy/commands/users.ts` |
| [orgs](./dokploy.md#orgs) | list, active | `src/providers/dokploy/commands/users.ts` |
| [compose](./dokploy.md#compose) | get, services, config, deploy, stop, redeploy, delete | `src/providers/dokploy/commands/compose.ts` |
| [backup](./dokploy.md#backup) | create, get, update, files, run, delete | `src/providers/dokploy/commands/backup.ts` |
| [schedule](./dokploy.md#schedule) | list, get, run, delete | `src/providers/dokploy/commands/schedule.ts` |
| [git](./dokploy.md#git) | list, setup, delete | `src/providers/dokploy/commands/git.ts` |

### [Bitwarden](./bw.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [items](./bw.md#items) | list, search, get, totp, trash, recent, favourites, create, update, move, delete, restore, purge | `src/providers/bw/commands.ts` |
| [folders](./bw.md#folders) | list, get, summary, create, delete | `src/providers/bw/commands.ts` |
| [password](./bw.md#password) | list | `src/providers/bw/commands.ts` |
| [profile](./bw.md#profile) | list | `src/providers/bw/commands.ts` |
| [dashboard](./bw.md#dashboard) | list | `src/providers/bw/commands.ts` |
| [pin](./bw.md#pin) | reset | `src/providers/bw/commands.ts` |
| [session](./bw.md#session) | unlock, lock, list | `src/providers/bw/commands.ts` |
| [vaults](./bw.md#vaults) | list, create, delete | `src/providers/bw/commands.ts` |
| [audit](./bw.md#audit) | list, weak, reused, exposed, duplicates, unfiled, cleanup, vault-report | `src/providers/bw/commands.ts` |
| [doctor](./bw.md#doctor) | list | `src/providers/bw/commands.ts` |

### [SharePoint](./sp.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [sites](./sp.md#sites) | list, get, search, root, subsites, structure | `src/providers/sp/commands/sites.ts` |
| [drives](./sp.md#drives) | list, root, folder, get | `src/providers/sp/commands/drives.ts` |
| [lists](./sp.md#lists) | list, get, columns, items, create-item, update-item, delete-item | `src/providers/sp/commands/lists.ts` |
| [files](./sp.md#files) | download, upload, folder, delete, share, move, checkout, checkin, versions, restore | `src/providers/sp/commands/files.ts` |
| [search](./sp.md#search) | list | `src/providers/sp/commands/search.ts` |
| [permissions](./sp.md#permissions) | list, item, share, grant-app, remove | `src/providers/sp/commands/permissions.ts` |
| [groups](./sp.md#groups) | list, members, add-member, remove-member | `src/providers/sp/commands/groups.ts` |
| [pages](./sp.md#pages) | list, get | `src/providers/sp/commands/pages.ts` |
| [dashboard](./sp.md#dashboard) | list | `src/providers/sp/commands/dashboard.ts` |
| [graph](./sp.md#graph) | get, post, patch, put, delete | — |

### [UniFi Network](./unifi.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [sites](./unifi.md#sites) | list, health, sysinfo | `src/providers/unifi/commands/sites.ts` |
| [devices](./unifi.md#devices) | list, get, restart, locate, upgrade, provision, power-cycle, leds, poe | `src/providers/unifi/commands/devices.ts` |
| [clients](./unifi.md#clients) | list, get, search, block, unblock, reconnect, offline | `src/providers/unifi/commands/clients.ts` |
| [guests](./unifi.md#guests) | authorise, unauthorise | `src/providers/unifi/commands/clients.ts` |
| [networks](./unifi.md#networks) | list | `src/providers/unifi/commands/networks.ts` |
| [wlans](./unifi.md#wlans) | list, toggle, password | `src/providers/unifi/commands/networks.ts` |
| [firewall](./unifi.md#firewall) | list, groups | `src/providers/unifi/commands/networks.ts` |
| [ports](./unifi.md#ports) | list | `src/providers/unifi/commands/networks.ts` |
| [routes](./unifi.md#routes) | list | `src/providers/unifi/commands/networks.ts` |
| [events](./unifi.md#events) | list | `src/providers/unifi/commands/events.ts` |
| [alarms](./unifi.md#alarms) | list, count, archive | `src/providers/unifi/commands/events.ts` |
| [rogue](./unifi.md#rogue) | list | `src/providers/unifi/commands/events.ts` |
| [vouchers](./unifi.md#vouchers) | list, create, revoke | `src/providers/unifi/commands/vouchers.ts` |
| [dashboard](./unifi.md#dashboard) | list | `src/providers/unifi/commands/dashboard.ts` |

### [Wrike](./wrike.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [tickets](./wrike.md#tickets) | list, stats, active, mine, get, search, create, set-due, assign, update-title, update-importance, update-status, update-description, add-comment, update-comment, delete-comment, narrative, attachments, download, attach | `src/providers/wrike/commands/tickets.ts` |
| [tasks](./wrike.md#tasks) | list, search, get, create, set-due, update-title, update-importance, update-status, update-description, add-comment, attach | `src/providers/wrike/commands/tasks.ts` |
| [projects](./wrike.md#projects) | list, search, tasks | `src/providers/wrike/commands/tasks.ts` |
| [contacts](./wrike.md#contacts) | list, search, find, get | `src/providers/wrike/commands/contacts.ts` |
| [spaces](./wrike.md#spaces) | list | `src/providers/wrike/commands/spaces.ts` |
| [folders](./wrike.md#folders) | list | `src/providers/wrike/commands/spaces.ts` |
| [workflows](./wrike.md#workflows) | list | `src/providers/wrike/commands/workflows.ts` |
| [custom-fields](./wrike.md#custom-fields) | list | `src/providers/wrike/commands/workflows.ts` |
| [item-types](./wrike.md#item-types) | list | `src/providers/wrike/commands/workflows.ts` |
| [onboarding](./wrike.md#onboarding) | get | `src/providers/wrike/commands/onboarding.ts` |
| [leavers](./wrike.md#leavers) | list, complete, get | `src/providers/wrike/commands/leavers.ts` |
| [dashboard](./wrike.md#dashboard) | list | `src/providers/wrike/commands/dashboard.ts` |

### [Azure CLI](./az.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [account](./az.md#account) | list, get, set | `src/providers/az/commands/account.ts` |
| [groups](./az.md#groups) | list | `src/providers/az/commands/resources.ts` |
| [resources](./az.md#resources) | list, get | `src/providers/az/commands/resources.ts` |
| [vm](./az.md#vm) | list, get, start, stop, restart, deallocate | `src/providers/az/commands/vm.ts` |
| [storage](./az.md#storage) | list | `src/providers/az/commands/storage.ts` |
| [keyvault](./az.md#keyvault) | list, secrets | `src/providers/az/commands/keyvault.ts` |
| [nsg](./az.md#nsg) | list, get | `src/providers/az/commands/network.ts` |
| [vnet](./az.md#vnet) | list, get | `src/providers/az/commands/network.ts` |
| [webapp](./az.md#webapp) | list, get, restart | `src/providers/az/commands/webapp.ts` |
| [cost](./az.md#cost) | summary | `src/providers/az/commands/cost.ts` |

### [Exchange Online](./exo.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [groups](./exo.md#groups) | list, get, members, create, delete, add-member, remove-member | `src/providers/exo/commands/groups.ts` |
| [mailboxes](./exo.md#mailboxes) | list, get, stats, create, permissions, add-permission, remove-permission, forwarding, user-access, set-forwarding, set-type, set-visibility | `src/providers/exo/commands/mailboxes.ts` |
| [rules](./exo.md#rules) | list | `src/providers/exo/commands/rules.ts` |
| [domains](./exo.md#domains) | list | `src/providers/exo/commands/domains.ts` |
| [trace](./exo.md#trace) | list, detail, historical, historical-status | `src/providers/exo/commands/trace.ts` |
| [autoreply](./exo.md#autoreply) | get, enable, disable | `src/providers/exo/commands/autoreply.ts` |
| [recipients](./exo.md#recipients) | search, send-as, add-send-as, remove-send-as | `src/providers/exo/commands/recipients.ts` |
| [forwarding](./exo.md#forwarding) | check | `src/providers/exo/commands/forwarding.ts` |

### [Intune](./intune.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [devices](./intune.md#devices) | list, get, search, sync, noncompliant | `src/providers/intune/commands/devices.ts` |
| [apps](./intune.md#apps) | list, get, required | `src/providers/intune/commands/apps.ts` |
| [scripts](./intune.md#scripts) | list, get, status | `src/providers/intune/commands/scripts.ts` |
| [remediations](./intune.md#remediations) | list, get, status | `src/providers/intune/commands/remediations.ts` |
| [policies](./intune.md#policies) | list, get, configs | `src/providers/intune/commands/policies.ts` |
| [esp](./intune.md#esp) | list, get, update | `src/providers/intune/commands/esp.ts` |
| [autopilot](./intune.md#autopilot) | list, devices, tag | `src/providers/intune/commands/autopilot.ts` |
| [group](./intune.md#group) | find | `src/providers/intune/commands/lookup.ts` |
| [settings](./intune.md#settings) | list, get | — |
| [intents](./intune.md#intents) | list, get | `src/providers/intune/commands/lookup.ts` |
| [updates](./intune.md#updates) | list, get | `src/providers/intune/commands/coverage.ts` |
| [appconfig](./intune.md#appconfig) | list, get | — |
| [appprotection](./intune.md#appprotection) | list, get | `src/providers/intune/commands/coverage.ts` |
| [doctor](./intune.md#doctor) | list | `src/providers/intune/commands/doctor.ts` |
| [graph](./intune.md#graph) | get, post, patch, put, delete | — |

### [UniFi Protect](./protect.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [cameras](./protect.md#cameras) | list, get, offline | `src/providers/protect/commands/cameras.ts` |
| [nvr](./protect.md#nvr) | list | `src/providers/protect/commands/nvr.ts` |
| [events](./protect.md#events) | list | `src/providers/protect/commands/events.ts` |
| [dashboard](./protect.md#dashboard) | list | `src/providers/protect/commands/dashboard.ts` |

### [Power BI](./pbi.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [workspaces](./pbi.md#workspaces) | list, members, add-user, update-user, remove-user | `src/providers/pbi/commands/workspaces.ts` |
| [reports](./pbi.md#reports) | list, search, url | `src/providers/pbi/commands/reports.ts` |
| [apps](./pbi.md#apps) | list | `src/providers/pbi/commands/apps.ts` |
| [licenses](./pbi.md#licenses) | list | `src/providers/pbi/commands/licenses.ts` |
| [activity](./pbi.md#activity) | list | `src/providers/pbi/commands/activity.ts` |
| [my](./pbi.md#my) | login, logout, whoami, workspaces, reports, datasets, add-workspace-user, update-workspace-user, remove-workspace-user, refresh | `src/providers/pbi/commands/my.ts` |

### [Power Platform](./pa.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [environments](./pa.md#environments) | list, get | `src/providers/pa/commands/environments.ts` |
| [flows](./pa.md#flows) | list, get, stop, start, delete, set-owner, runs | `src/providers/pa/commands/flows.ts` |
| [apps](./pa.md#apps) | list | `src/providers/pa/commands/apps.ts` |
| [connections](./pa.md#connections) | list | `src/providers/pa/commands/connections.ts` |

### [Cloudflare](./cf.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [accounts](./cf.md#accounts) | list | `src/providers/cf/commands/accounts.ts` |
| [zones](./cf.md#zones) | list, get, purge | `src/providers/cf/commands/zones.ts` |
| [dns](./cf.md#dns) | list, get, create, update, delete | `src/providers/cf/commands/dns.ts` |
| [tunnels](./cf.md#tunnels) | list, get, connections, delete, routes | `src/providers/cf/commands/tunnels.ts` |
| [token](./cf.md#token) | url, request | `src/providers/cf/commands/token.ts` |

### [PeopleHR](./hr.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [drift](./hr.md#drift) | detect | `src/providers/hr/commands.ts` |
| [employees](./hr.md#employees) | list, search, get | `src/providers/hr/commands.ts` |
| [starters](./hr.md#starters) | list, recent | `src/providers/hr/commands.ts` |
| [leavers](./hr.md#leavers) | list, recent | `src/providers/hr/commands.ts` |

### [Business Central](./bc.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [companies](./bc.md#companies) | list, get | `src/providers/bc/commands.ts` |
| [query](./bc.md#query) | get | `src/providers/bc/commands.ts` |
| [record](./bc.md#record) | get | `src/providers/bc/commands.ts` |
| [health](./bc.md#health) | get | `src/providers/bc/commands.ts` |

### [ctxc memories](./ctxc.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [memories](./ctxc.md#memories) | search, recall, get, list, stats | `src/providers/ctxc/commands.ts` |

### [Docs UI](./docs.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [build](./docs.md#build) | list | `src/providers/docs/commands.ts` |
| [serve](./docs.md#serve) | list | `src/providers/docs/commands.ts` |
| [open](./docs.md#open) | list | `src/providers/docs/commands.ts` |
| [search](./docs.md#search) | list | `src/providers/docs/commands.ts` |
| [show](./docs.md#show) | list | `src/providers/docs/commands.ts` |

### [GitHub](./gh.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [branch-protect](./gh.md#branch-protect) | apply, show | — |
| [webhook](./gh.md#webhook) | setup, list | — |

### [Outlook](./outlook.md)

| Resource | Actions | Source |
|----------|---------|--------|
| [mail](./outlook.md#mail) | list, get, headers, search, thread, move, copy, read, unread, flag, categorise, delete, send | `src/providers/outlook/commands/mail.ts` |
| [drafts](./outlook.md#drafts) | create, reply, forward, update, send, list | `src/providers/outlook/commands/drafts.ts` |
| [folders](./outlook.md#folders) | list, get, create | `src/providers/outlook/commands/folders.ts` |
| [attachments](./outlook.md#attachments) | list, get, add, delete | `src/providers/outlook/commands/attachments.ts` |
| [events](./outlook.md#events) | list, get, create, update, delete, respond, availability | `src/providers/outlook/commands/events.ts` |
| [settings](./outlook.md#settings) | get | `src/providers/outlook/commands/settings.ts` |
| [autoreply](./outlook.md#autoreply) | get, set | `src/providers/outlook/commands/settings.ts` |
| [categories](./outlook.md#categories) | list | `src/providers/outlook/commands/settings.ts` |
| [rules](./outlook.md#rules) | list, create, delete | `src/providers/outlook/commands/rules.ts` |
| [contacts](./outlook.md#contacts) | search | `src/providers/outlook/commands/contacts.ts` |
| [triage](./outlook.md#triage) | list | `src/providers/outlook/commands/triage.ts` |

## Key Source Files

| File | Purpose |
|------|---------|
| `src/index.ts` | Entry point, provider registration |
| `src/cli.ts` | Argument parser, action whitelist |
| `src/core/types.ts` | `CommandDef`, `ParsedArgs`, `OutputConfig` |
| `src/core/http.ts` | HTTP client with retry/backoff |
| `src/core/auth.ts` | Graph API token caching |
| `src/core/output.ts` | Result formatting (human/AI/JSON) |
| `src/core/cache.ts` | Response caching |
| `src/core/errors.ts` | Error types |
| `src/core/secrets.ts` | PIN-encrypted secret store |
| `src/core/setup/` | Interactive setup wizard system |
| `src/providers/base.ts` | `BaseProvider` contract |
| `src/providers/registry.ts` | Provider factory registry |
| `src/providers/rmm/client.ts` | Tactical RMM API client |
| `src/providers/entra/client.ts` | Entra ID API client |
| `src/providers/dokploy/client.ts` | Dokploy API client |
| `src/providers/bw/client.ts` | Bitwarden API client |
| `src/providers/sp/client.ts` | SharePoint API client |
| `src/providers/unifi/client.ts` | UniFi Network API client |
| `src/providers/wrike/client.ts` | Wrike API client |
| `src/providers/az/client.ts` | Azure CLI API client |
| `src/providers/exo/client.ts` | Exchange Online API client |
| `src/providers/intune/client.ts` | Intune API client |
| `src/providers/protect/client.ts` | UniFi Protect API client |
| `src/providers/pbi/client.ts` | Power BI API client |
| `src/providers/pa/client.ts` | Power Platform API client |
| `src/providers/cf/client.ts` | Cloudflare API client |
| `src/providers/hr/client.ts` | PeopleHR API client |
| `src/providers/bc/client.ts` | Business Central API client |
| `src/providers/ctxc/client.ts` | ctxc memories API client |
| `src/providers/docs/client.ts` | Docs UI API client |
| `src/providers/gh/client.ts` | GitHub API client |
| `src/providers/outlook/client.ts` | Outlook API client |

## Command Tree

```
its
├── rmm
│   ├── agents
│   │   ├── (list)
│   │   ├── stale
│   │   ├── search <query>
│   │   ├── get <agent>
│   │   ├── ping <agent>
│   │   ├── reboot <agent>
│   │   ├── remove <agent>
│   │   ├── run <agent>
│   │   ├── history <agent>
│   │   ├── notes <agent>
│   │   ├── pending <agent>
│   │   ├── wake <agent>
│   │   ├── edit <agent>
│   │   └── refresh <agent>
│   ├── dashboard (list)
│   ├── clients
│   │   ├── (list)
│   │   ├── create
│   │   └── delete <client>
│   ├── sites
│   │   ├── (list)
│   │   ├── create
│   │   └── delete <site_id>
│   ├── processes
│   │   ├── (list) <agent>
│   │   ├── top <agent>
│   │   └── kill <agent_id>
│   ├── services
│   │   ├── (list) <agent>
│   │   ├── get <agent>
│   │   ├── control <agent>
│   │   ├── enable <agent>
│   │   └── disable <agent>
│   ├── updates
│   │   ├── (list) <agent>
│   │   ├── scan <agent>
│   │   └── install <agent_id>
│   ├── software
│   │   ├── (list) <agent>
│   │   └── search <query>
│   ├── alerts
│   │   ├── (list)
│   │   └── get <alert_id>
│   ├── scripts
│   │   ├── (list)
│   │   ├── get <script_id>
│   │   ├── run <agent>
│   │   ├── upload-local <agent> <path>
│   │   ├── delete <script_id>
│   │   └── upsert <name> <path>
│   ├── checks
│   │   ├── (list) <agent>
│   │   ├── create <agent>
│   │   └── delete [agent_id]
│   ├── tasks
│   │   ├── (list) <agent>
│   │   ├── create <agent>
│   │   └── delete
│   ├── policies
│   │   ├── (list)
│   │   ├── get <policy_id>
│   │   ├── checks <policy_id>
│   │   ├── add-check <policy_id>
│   │   └── patch-policy <policy_id>
│   ├── diagnostics (list) <agent>
│   ├── doctor (list)
│   └── custom-fields
│       ├── (list)
│       └── set <agent> <field> <value>
├── entra
│   ├── users
│   │   ├── (list)
│   │   ├── update <id>
│   │   ├── search <query>
│   │   ├── get <id>
│   │   ├── groups <id>
│   │   ├── chain <id>
│   │   ├── licences <id>
│   │   ├── invite
│   │   ├── create
│   │   ├── enable <id>
│   │   ├── bootstrap-admin <id>
│   │   ├── stale
│   │   ├── disable <id>
│   │   ├── revoke-sessions <id>
│   │   ├── set-password <id>
│   │   ├── delete <id>
│   │   ├── transfer <id>
│   │   └── reinstate <id>
│   ├── groups
│   │   ├── (list)
│   │   ├── search <query>
│   │   ├── get <group_id>
│   │   ├── members <group_id>
│   │   ├── create
│   │   ├── add-member <group_id>
│   │   ├── remove-member <group_id>
│   │   ├── edit-rule <group_id>
│   │   └── audit-rules [group_id]
│   ├── licences
│   │   ├── (list)
│   │   ├── assign <user_id>
│   │   ├── remove <user_id>
│   │   ├── users <sku_id>
│   │   ├── unlicensed
│   │   ├── audit
│   │   └── waste
│   ├── roles
│   │   ├── (list)
│   │   ├── members <role_id>
│   │   ├── assign <user_id>
│   │   ├── remove <user_id>
│   │   └── assignments
│   ├── signin
│   │   ├── (list)
│   │   ├── summary <user_id>
│   │   ├── explain <id>
│   │   └── suspicious
│   ├── tap
│   │   ├── create <user_id>
│   │   ├── (list) <user_id>
│   │   └── revoke <user_id> <method_id>
│   ├── ca
│   │   ├── (list)
│   │   ├── enabled
│   │   ├── get <id_or_name>
│   │   ├── patch <id_or_name>
│   │   ├── exclude-guests <id_or_name>
│   │   ├── create
│   │   ├── delete <id_or_name>
│   │   ├── exclude-user <id_or_name> <user>
│   │   ├── unexclude-user <id_or_name> <user>
│   │   ├── why-blocked <user>
│   │   └── named-locations
│   ├── authmethods
│   │   ├── policy
│   │   ├── get <method>
│   │   ├── enable <method>
│   │   ├── disable <method>
│   │   └── patch <method>
│   ├── consent
│   │   ├── (list) <client_app_id>
│   │   ├── add-scope <client_app_id> <scopes>
│   │   ├── remove-scope <client_app_id> <scopes>
│   │   ├── app-roles <client_app_id>
│   │   ├── app-role-grant <client_app_id> <role>
│   │   └── app-role-revoke <client_app_id> <assignment_id>
│   ├── xtenant
│   │   ├── default
│   │   ├── trust-mfa <on_off>
│   │   ├── trust-device <on_off>
│   │   ├── partners
│   │   ├── partner-add <tenant_id>
│   │   └── partner-set <tenant_id>
│   ├── audit (list)
│   ├── security
│   │   ├── risky
│   │   ├── mfa <user_id>
│   │   └── admin-mfa
│   ├── directory
│   │   ├── org
│   │   ├── domains
│   │   ├── deleted
│   │   ├── tree <user_id>
│   │   ├── summary
│   │   └── app-usage
│   ├── onboarding
│   │   ├── summary <user_id>
│   │   ├── copy-groups
│   │   └── convert-mailbox <user_id>
│   ├── offboarding
│   │   ├── summary <user_id>
│   │   └── run <user_id>
│   ├── break-glass audit
│   ├── whoami show
│   ├── doctor (list)
│   ├── apps
│   │   ├── register <name>
│   │   └── add-password <app>
│   ├── admin-bootstrap run <user_id>
│   └── graph
│       ├── get <path>
│       ├── post <path>
│       ├── patch <path>
│       ├── put <path>
│       └── delete <path>
├── dokploy
│   ├── projects
│   │   ├── (list)
│   │   ├── get <projectId>
│   │   ├── create
│   │   └── delete <projectId>
│   ├── apps
│   │   ├── (list)
│   │   ├── get <applicationId>
│   │   ├── create
│   │   ├── delete <applicationId>
│   │   ├── deploy <app>
│   │   ├── stop <applicationId>
│   │   ├── start <applicationId>
│   │   ├── restart <applicationId>
│   │   ├── set-source <app>
│   │   ├── set-build <app>
│   │   ├── rebuild <app>
│   │   ├── wait-deploy <app>
│   │   ├── redeploy <applicationId>
│   │   ├── logs <app>
│   │   ├── monitoring <app>
│   │   ├── traefik <app>
│   │   ├── status <app>
│   │   ├── clone <source> <newName>
│   │   ├── shell <app> [cmd]
│   │   ├── migrate <app>
│   │   ├── env-runtime <app>
│   │   ├── apply-env <app>
│   │   ├── health
│   │   ├── doctor <app>
│   │   ├── bootstrap <name>
│   │   └── cert-status <app>
│   ├── databases
│   │   ├── (list)
│   │   ├── url <id>
│   │   ├── get
│   │   ├── create
│   │   ├── deploy <databaseId>
│   │   ├── stop <databaseId>
│   │   └── delete <databaseId>
│   ├── deployments
│   │   ├── (list) <applicationId>
│   │   ├── queue
│   │   └── kill <deploymentId>
│   ├── domains
│   │   ├── (list) <applicationId>
│   │   ├── create <applicationId>
│   │   ├── check [host]
│   │   └── delete <domainId>
│   ├── env
│   │   ├── (list) <applicationId>
│   │   ├── push <applicationId>
│   │   ├── set <applicationId> <pairs>
│   │   ├── unset <applicationId> <keys>
│   │   └── pull <applicationId>
│   ├── environments
│   │   ├── (list) <projectId>
│   │   ├── env <environmentId>
│   │   ├── push <environmentId>
│   │   ├── pull <environmentId>
│   │   └── set <environmentId> <pairs>
│   ├── registries (list)
│   ├── destinations (list)
│   ├── notifications (list)
│   ├── dashboard (list)
│   ├── mounts
│   │   ├── (list) <app>
│   │   ├── add <app>
│   │   ├── update <mountId>
│   │   └── remove <mountId>
│   ├── webhook
│   │   ├── check <app>
│   │   ├── setup <app>
│   │   └── (list) <app>
│   ├── nodes
│   │   ├── (list)
│   │   ├── info <nodeId>
│   │   ├── apps
│   │   └── stats
│   ├── cluster join-token
│   ├── containers
│   │   ├── (list)
│   │   ├── config <containerId>
│   │   ├── start <containerId>
│   │   ├── stop <containerId>
│   │   ├── restart <containerId>
│   │   ├── kill <containerId>
│   │   └── remove <containerId>
│   ├── maintenance
│   │   ├── disk
│   │   ├── web
│   │   ├── traefik
│   │   ├── clean
│   │   └── time
│   ├── traefik
│   │   ├── logs
│   │   └── acme
│   ├── github
│   │   ├── providers
│   │   ├── repos <githubId>
│   │   └── branches <owner> <repo>
│   ├── users
│   │   ├── (list)
│   │   ├── me
│   │   ├── invite
│   │   └── remove <userId>
│   ├── orgs
│   │   ├── (list)
│   │   └── active
│   ├── compose
│   │   ├── get <composeId>
│   │   ├── services <composeId>
│   │   ├── config <composeId>
│   │   ├── deploy <composeId>
│   │   ├── stop <composeId>
│   │   ├── redeploy <composeId>
│   │   └── delete <composeId>
│   ├── backup
│   │   ├── create
│   │   ├── get <backupId>
│   │   ├── update <backupId>
│   │   ├── files <destinationId>
│   │   ├── run <backupId>
│   │   └── delete <backupId>
│   ├── schedule
│   │   ├── (list) <id>
│   │   ├── get <scheduleId>
│   │   ├── run <scheduleId>
│   │   └── delete <scheduleId>
│   └── git
│       ├── (list)
│       ├── setup
│       └── delete <id>
├── bw
│   ├── items
│   │   ├── (list)
│   │   ├── search <query>
│   │   ├── get <id>
│   │   ├── totp <query>
│   │   ├── trash
│   │   ├── recent
│   │   ├── favourites
│   │   ├── create <name>
│   │   ├── update <id>
│   │   ├── move <id>
│   │   ├── delete <id>
│   │   ├── restore <id>
│   │   └── purge <id>
│   ├── folders
│   │   ├── (list)
│   │   ├── get <name>
│   │   ├── summary
│   │   ├── create <name>
│   │   └── delete <name>
│   ├── password (list) <query>
│   ├── profile (list)
│   ├── dashboard (list)
│   ├── pin reset
│   ├── session
│   │   ├── unlock
│   │   ├── lock
│   │   └── (list)
│   ├── vaults
│   │   ├── (list)
│   │   ├── create <name>
│   │   └── delete <name>
│   ├── audit
│   │   ├── (list)
│   │   ├── weak
│   │   ├── reused
│   │   ├── exposed
│   │   ├── duplicates
│   │   ├── unfiled
│   │   ├── cleanup
│   │   └── vault-report
│   └── doctor (list)
├── sp
│   ├── sites
│   │   ├── (list)
│   │   ├── get <siteId>
│   │   ├── search <query>
│   │   ├── root
│   │   ├── subsites <siteId>
│   │   └── structure <siteId>
│   ├── drives
│   │   ├── (list) <siteId>
│   │   ├── root <siteId>
│   │   ├── folder <siteId>
│   │   └── get <siteId>
│   ├── lists
│   │   ├── (list) <siteId>
│   │   ├── get <siteId>
│   │   ├── columns <siteId>
│   │   ├── items <siteId>
│   │   ├── create-item <siteId>
│   │   ├── update-item <siteId>
│   │   └── delete-item <siteId>
│   ├── files
│   │   ├── download
│   │   ├── upload <siteId>
│   │   ├── folder <siteId>
│   │   ├── delete <siteId>
│   │   ├── share <siteId>
│   │   ├── move <siteId>
│   │   ├── checkout <siteId>
│   │   ├── checkin <siteId>
│   │   ├── versions <siteId>
│   │   └── restore <siteId>
│   ├── search (list) <query>
│   ├── permissions
│   │   ├── (list) <siteId>
│   │   ├── item <siteId>
│   │   ├── share <siteId>
│   │   ├── grant-app <siteId>
│   │   └── remove <siteId>
│   ├── groups
│   │   ├── (list) <site>
│   │   ├── members <site> <group>
│   │   ├── add-member <site> <group> <principal>
│   │   └── remove-member <site> <group> <principal>
│   ├── pages
│   │   ├── (list) <siteId>
│   │   └── get <siteId>
│   ├── dashboard (list)
│   └── graph
│       ├── get <path>
│       ├── post <path>
│       ├── patch <path>
│       ├── put <path>
│       └── delete <path>
├── unifi
│   ├── sites
│   │   ├── (list)
│   │   ├── health
│   │   └── sysinfo
│   ├── devices
│   │   ├── (list)
│   │   ├── get <mac>
│   │   ├── restart <mac>
│   │   ├── locate <mac>
│   │   ├── upgrade <mac>
│   │   ├── provision <mac>
│   │   ├── power-cycle
│   │   ├── leds
│   │   └── poe
│   ├── clients
│   │   ├── (list)
│   │   ├── get <mac>
│   │   ├── search <query>
│   │   ├── block <mac>
│   │   ├── unblock <mac>
│   │   ├── reconnect <mac>
│   │   └── offline
│   ├── guests
│   │   ├── authorise <mac>
│   │   └── unauthorise <mac>
│   ├── networks (list)
│   ├── wlans
│   │   ├── (list)
│   │   ├── toggle <wlan_id>
│   │   └── password <wlan_id>
│   ├── firewall
│   │   ├── (list)
│   │   └── groups
│   ├── ports (list)
│   ├── routes (list)
│   ├── events (list)
│   ├── alarms
│   │   ├── (list)
│   │   ├── count
│   │   └── archive
│   ├── rogue (list)
│   ├── vouchers
│   │   ├── (list)
│   │   ├── create
│   │   └── revoke <id>
│   └── dashboard (list)
├── wrike
│   ├── tickets
│   │   ├── (list)
│   │   ├── stats
│   │   ├── active
│   │   ├── mine
│   │   ├── get <idOrPermalink>
│   │   ├── search <query>
│   │   ├── create
│   │   ├── set-due <taskId> <dueDate>
│   │   ├── assign <taskId> <userId>
│   │   ├── update-title <taskId> <title>
│   │   ├── update-importance <taskId> <importance>
│   │   ├── update-status <taskId> <status>
│   │   ├── update-description <taskId> [text]
│   │   ├── add-comment <taskId> [text]
│   │   ├── update-comment <commentId> [text]
│   │   ├── delete-comment <commentId>
│   │   ├── narrative [taskId]
│   │   ├── attachments <taskId>
│   │   ├── download <taskId> [attachmentId]
│   │   └── attach <taskId> <filePath>
│   ├── tasks
│   │   ├── (list) <folder>
│   │   ├── search <query>
│   │   ├── get <idOrPermalink>
│   │   ├── create <folder>
│   │   ├── set-due <taskId> <dueDate>
│   │   ├── update-title <taskId> <title>
│   │   ├── update-importance <taskId> <importance>
│   │   ├── update-status <taskId> <status>
│   │   ├── update-description <taskId> [text]
│   │   ├── add-comment <taskId> [text]
│   │   └── attach <taskId> <filePath>
│   ├── projects
│   │   ├── (list)
│   │   ├── search <query>
│   │   └── tasks <project>
│   ├── contacts
│   │   ├── (list)
│   │   ├── search <query>
│   │   ├── find [query]
│   │   └── get <user_id>
│   ├── spaces (list)
│   ├── folders (list) <space>
│   ├── workflows (list)
│   ├── custom-fields (list)
│   ├── item-types (list)
│   ├── onboarding get <permalink>
│   ├── leavers
│   │   ├── (list)
│   │   ├── complete <idOrPermalink>
│   │   └── get <idOrPermalink>
│   └── dashboard (list)
├── az
│   ├── account
│   │   ├── (list)
│   │   ├── get
│   │   └── set <subscription>
│   ├── groups (list)
│   ├── resources
│   │   ├── (list)
│   │   └── get <id>
│   ├── vm
│   │   ├── (list)
│   │   ├── get <name>
│   │   ├── start <name>
│   │   ├── stop <name>
│   │   ├── restart <name>
│   │   └── deallocate <name>
│   ├── storage (list)
│   ├── keyvault
│   │   ├── (list)
│   │   └── secrets <vault>
│   ├── nsg
│   │   ├── (list)
│   │   └── get <name>
│   ├── vnet
│   │   ├── (list)
│   │   └── get <name>
│   ├── webapp
│   │   ├── (list)
│   │   ├── get <name>
│   │   └── restart <name>
│   └── cost summary
├── exo
│   ├── groups
│   │   ├── (list)
│   │   ├── get <group>
│   │   ├── members <group>
│   │   ├── create <name>
│   │   ├── delete <group>
│   │   ├── add-member <group> <member>
│   │   └── remove-member <group> <member>
│   ├── mailboxes
│   │   ├── (list)
│   │   ├── get <mailbox>
│   │   ├── stats <mailbox>
│   │   ├── create <name> <alias>
│   │   ├── permissions <mailbox>
│   │   ├── add-permission <mailbox> <user>
│   │   ├── remove-permission <mailbox> <user>
│   │   ├── forwarding <mailbox>
│   │   ├── user-access <user>
│   │   ├── set-forwarding <mailbox> <target>
│   │   ├── set-type <mailbox> <type>
│   │   └── set-visibility <mailbox>
│   ├── rules (list)
│   ├── domains (list)
│   ├── trace
│   │   ├── (list)
│   │   ├── detail <trace-id> <recipient>
│   │   ├── historical
│   │   └── historical-status <job-id>
│   ├── autoreply
│   │   ├── get <mailbox>
│   │   ├── enable <mailbox>
│   │   └── disable <mailbox>
│   ├── recipients
│   │   ├── search <query>
│   │   ├── send-as <identity>
│   │   ├── add-send-as <identity> <trustee>
│   │   └── remove-send-as <identity> <trustee>
│   └── forwarding check [upn]
├── intune
│   ├── devices
│   │   ├── (list)
│   │   ├── get <id>
│   │   ├── search <query>
│   │   ├── sync <id>
│   │   └── noncompliant
│   ├── apps
│   │   ├── (list)
│   │   ├── get <id>
│   │   └── required
│   ├── scripts
│   │   ├── (list)
│   │   ├── get <id>
│   │   └── status <id>
│   ├── remediations
│   │   ├── (list)
│   │   ├── get <id>
│   │   └── status <id>
│   ├── policies
│   │   ├── (list)
│   │   ├── get <id>
│   │   └── configs
│   ├── esp
│   │   ├── (list)
│   │   ├── get <id>
│   │   └── update [id]
│   ├── autopilot
│   │   ├── (list)
│   │   ├── devices
│   │   └── tag <serial> [tag]
│   ├── group find <groupId>
│   ├── settings
│   │   ├── (list)
│   │   └── get <id>
│   ├── intents
│   │   ├── (list)
│   │   └── get <id>
│   ├── updates
│   │   ├── (list)
│   │   └── get <id>
│   ├── appconfig
│   │   ├── (list)
│   │   └── get <id>
│   ├── appprotection
│   │   ├── (list)
│   │   └── get <id>
│   ├── doctor (list)
│   └── graph
│       ├── get <path>
│       ├── post <path>
│       ├── patch <path>
│       ├── put <path>
│       └── delete <path>
├── protect
│   ├── cameras
│   │   ├── (list)
│   │   ├── get <id>
│   │   └── offline
│   ├── nvr (list)
│   ├── events (list)
│   └── dashboard (list)
├── pbi
│   ├── workspaces
│   │   ├── (list)
│   │   ├── members <workspace_id>
│   │   ├── add-user <workspace_id>
│   │   ├── update-user <workspace_id>
│   │   └── remove-user <workspace_id>
│   ├── reports
│   │   ├── (list)
│   │   ├── search <query>
│   │   └── url <report_id>
│   ├── apps (list)
│   ├── licenses (list)
│   ├── activity (list)
│   └── my
│       ├── login
│       ├── logout
│       ├── whoami
│       ├── workspaces
│       ├── reports
│       ├── datasets
│       ├── add-workspace-user <workspace_id>
│       ├── update-workspace-user <workspace_id>
│       ├── remove-workspace-user <workspace_id>
│       └── refresh <dataset_id>
├── pa
│   ├── environments
│   │   ├── (list)
│   │   └── get <environment_id>
│   ├── flows
│   │   ├── (list)
│   │   ├── get <flow_id>
│   │   ├── stop <flow_id>
│   │   ├── start <flow_id>
│   │   ├── delete <flow_id>
│   │   ├── set-owner <flow_id>
│   │   └── runs <flow_id>
│   ├── apps (list)
│   └── connections (list)
├── cf
│   ├── accounts (list)
│   ├── zones
│   │   ├── (list)
│   │   ├── get <zone>
│   │   └── purge <zone>
│   ├── dns
│   │   ├── (list)
│   │   ├── get <record_id>
│   │   ├── create
│   │   ├── update <record_id>
│   │   └── delete <record_id>
│   ├── tunnels
│   │   ├── (list)
│   │   ├── get <tunnel>
│   │   ├── connections <tunnel>
│   │   ├── delete <tunnel>
│   │   └── routes <tunnel>
│   └── token
│       ├── url
│       └── request
├── hr
│   ├── drift detect
│   ├── employees
│   │   ├── (list)
│   │   ├── search <query>
│   │   └── get <email>
│   ├── starters
│   │   ├── (list)
│   │   └── recent
│   └── leavers
│       ├── (list)
│       └── recent
├── bc
│   ├── companies
│   │   ├── (list)
│   │   └── get <nameOrId>
│   ├── query get <entity>
│   ├── record get <entity> <id>
│   └── health get
├── ctxc
│   └── memories
│       ├── search <query>
│       ├── recall
│       ├── get <id>
│       ├── (list)
│       └── stats
├── docs
│   ├── build (list)
│   ├── serve (list)
│   ├── open (list) <topic>
│   ├── search (list) <query>
│   └── show (list) <topic>
├── gh
│   ├── branch-protect
│   │   ├── apply <repo>
│   │   └── show <repo>
│   └── webhook
│       ├── setup <repo> <url>
│       └── (list) <repo>
└── outlook
    ├── mail
    │   ├── (list)
    │   ├── get <message_id>
    │   ├── headers <message_id>
    │   ├── search <query>
    │   ├── thread <conversation_id>
    │   ├── move <message_id> <folder_id>
    │   ├── copy <message_id> <folder_id>
    │   ├── read <message_id>
    │   ├── unread <message_id>
    │   ├── flag <message_id>
    │   ├── categorise <message_id> <categories>
    │   ├── delete [message_id]
    │   └── send
    ├── drafts
    │   ├── create
    │   ├── reply <message_id>
    │   ├── forward <message_id>
    │   ├── update <draft_id>
    │   ├── send <draft_id>
    │   └── (list)
    ├── folders
    │   ├── (list)
    │   ├── get <folder_id>
    │   └── create <name>
    ├── attachments
    │   ├── (list) <message_id>
    │   ├── get <message_id> [attachment_id]
    │   ├── add <message_id>
    │   └── delete <message_id> [attachment_id]
    ├── events
    │   ├── (list)
    │   ├── get <event_id>
    │   ├── create
    │   ├── update <event_id>
    │   ├── delete <event_id>
    │   ├── respond <event_id> <response>
    │   └── availability <schedules>
    ├── settings get
    ├── autoreply
    │   ├── get
    │   └── set
    ├── categories (list)
    ├── rules
    │   ├── (list)
    │   ├── create
    │   └── delete <rule_id>
    ├── contacts search <query>
    └── triage (list)
```

## Source Tree

```
src/
├── core/
│   ├── setup/
│   │   ├── checker.ts
│   │   ├── env-writer.ts
│   │   ├── prompt.ts
│   │   ├── types.ts
│   │   └── wizard.ts
│   ├── audit.ts
│   ├── auth-scopes.ts
│   ├── auth.ts
│   ├── cache.ts
│   ├── clipboard.ts
│   ├── completions.ts
│   ├── delegated-auth.ts
│   ├── errors.ts
│   ├── graph-passthrough.ts
│   ├── help.ts
│   ├── http.ts
│   ├── keychain.ts
│   ├── logger.ts
│   ├── long-args.ts
│   ├── output.ts
│   ├── parallel.ts
│   ├── secrets-audit.ts
│   ├── secrets.ts
│   ├── session.ts
│   ├── shell-dry-run.ts
│   ├── trusted-certs.ts
│   ├── types.ts
│   └── updates.ts
├── help-ui/
│   ├── web/
│   │   ├── fonts/
│   │   │   ├── InterVariable.woff2
│   │   │   └── JetBrainsMono.woff2
│   │   ├── lib/
│   │   │   ├── cmd-parse.ts
│   │   │   ├── CommandCard.svelte
│   │   │   ├── ExampleForm.svelte
│   │   │   ├── JsonTree.svelte
│   │   │   ├── Palette.svelte
│   │   │   ├── Recent.svelte
│   │   │   ├── ResultPanel.svelte
│   │   │   ├── run-client.ts
│   │   │   ├── scratch.ts
│   │   │   ├── ScratchPanel.svelte
│   │   │   ├── signature.ts
│   │   │   ├── state.svelte.ts
│   │   │   ├── TerminalDock.svelte
│   │   │   ├── TerminalPane.svelte
│   │   │   ├── ThemePicker.svelte
│   │   │   ├── Tree.svelte
│   │   │   └── types.ts
│   │   ├── styles/
│   │   │   ├── app.css
│   │   │   └── theme.css
│   │   ├── App.svelte
│   │   ├── index.html
│   │   ├── main.ts
│   │   └── tsconfig.json
│   ├── embedded-assets.ts
│   ├── examples.ts
│   ├── interactive-detect.ts
│   ├── registry-snapshot.ts
│   ├── run-result.ts
│   ├── search-engine.ts
│   ├── server.ts
│   ├── topic-renderer.ts
│   └── ws-bridge.ts
├── providers/
│   ├── az/
│   │   ├── commands/
│   │   │   ├── account.ts
│   │   │   ├── cost.ts
│   │   │   ├── index.ts
│   │   │   ├── keyvault.ts
│   │   │   ├── network.ts
│   │   │   ├── resources.ts
│   │   │   ├── storage.ts
│   │   │   ├── vm.ts
│   │   │   └── webapp.ts
│   │   ├── client.ts
│   │   ├── definition.ts
│   │   └── types.ts
│   ├── bc/
│   │   ├── client.ts
│   │   ├── commands.ts
│   │   ├── definition.ts
│   │   └── types.ts
│   ├── bw/
│   │   ├── audit.ts
│   │   ├── client.ts
│   │   ├── commands.ts
│   │   ├── crypto.ts
│   │   ├── definition.ts
│   │   ├── doctor.ts
│   │   └── types.ts
│   ├── cf/
│   │   ├── commands/
│   │   │   ├── accounts.ts
│   │   │   ├── dns.ts
│   │   │   ├── index.ts
│   │   │   ├── token.ts
│   │   │   ├── tunnels.ts
│   │   │   └── zones.ts
│   │   ├── client.ts
│   │   ├── definition.ts
│   │   └── types.ts
│   ├── ctxc/
│   │   ├── client.ts
│   │   ├── commands.ts
│   │   └── definition.ts
│   ├── docs/
│   │   ├── build-helpers.ts
│   │   ├── client.ts
│   │   ├── commands.ts
│   │   └── definition.ts
│   ├── dokploy/
│   │   ├── commands/
│   │   │   ├── apps.ts
│   │   │   ├── backup.ts
│   │   │   ├── compose.ts
│   │   │   ├── containers.ts
│   │   │   ├── dashboard.ts
│   │   │   ├── databases.ts
│   │   │   ├── deployments.ts
│   │   │   ├── domains.ts
│   │   │   ├── env.ts
│   │   │   ├── environments.ts
│   │   │   ├── git.ts
│   │   │   ├── github.ts
│   │   │   ├── index.ts
│   │   │   ├── infrastructure.ts
│   │   │   ├── maintenance.ts
│   │   │   ├── mounts.ts
│   │   │   ├── nodes.ts
│   │   │   ├── projects.ts
│   │   │   ├── schedule.ts
│   │   │   ├── users.ts
│   │   │   └── webhook.ts
│   │   ├── client.ts
│   │   ├── definition.ts
│   │   ├── resolve.ts
│   │   ├── runtime.ts
│   │   ├── ssh.ts
│   │   └── types.ts
│   ├── entra/
│   │   ├── commands/
│   │   │   ├── admin-bootstrap.ts
│   │   │   ├── apps.ts
│   │   │   ├── audit.ts
│   │   │   ├── auth-methods.ts
│   │   │   ├── break-glass.ts
│   │   │   ├── ca.ts
│   │   │   ├── consent.ts
│   │   │   ├── directory.ts
│   │   │   ├── doctor.ts
│   │   │   ├── groups.ts
│   │   │   ├── index.ts
│   │   │   ├── licences.ts
│   │   │   ├── onboarding.ts
│   │   │   ├── roles.ts
│   │   │   ├── security.ts
│   │   │   ├── signin.ts
│   │   │   ├── tap.ts
│   │   │   ├── users.ts
│   │   │   ├── whoami.ts
│   │   │   └── xtenant.ts
│   │   ├── auth-methods.ts
│   │   ├── client.ts
│   │   ├── definition.ts
│   │   ├── sku-names.ts
│   │   └── types.ts
│   ├── exo/
│   │   ├── commands/
│   │   │   ├── autoreply.ts
│   │   │   ├── domains.ts
│   │   │   ├── forwarding.ts
│   │   │   ├── groups.ts
│   │   │   ├── index.ts
│   │   │   ├── mailboxes.ts
│   │   │   ├── recipients.ts
│   │   │   ├── rules.ts
│   │   │   └── trace.ts
│   │   ├── client.ts
│   │   ├── definition.ts
│   │   ├── resolve.ts
│   │   └── types.ts
│   ├── gh/
│   │   ├── commands/
│   │   │   └── index.ts
│   │   ├── client.ts
│   │   └── definition.ts
│   ├── hr/
│   │   ├── client.ts
│   │   ├── commands.ts
│   │   ├── definition.ts
│   │   ├── drift.ts
│   │   └── types.ts
│   ├── intune/
│   │   ├── commands/
│   │   │   ├── apps.ts
│   │   │   ├── autopilot.ts
│   │   │   ├── coverage.ts
│   │   │   ├── devices.ts
│   │   │   ├── doctor.ts
│   │   │   ├── esp.ts
│   │   │   ├── index.ts
│   │   │   ├── lookup.ts
│   │   │   ├── policies.ts
│   │   │   ├── remediations.ts
│   │   │   └── scripts.ts
│   │   ├── client.ts
│   │   ├── definition.ts
│   │   └── types.ts
│   ├── outlook/
│   │   ├── commands/
│   │   │   ├── attachments.ts
│   │   │   ├── contacts.ts
│   │   │   ├── drafts.ts
│   │   │   ├── events.ts
│   │   │   ├── folders.ts
│   │   │   ├── index.ts
│   │   │   ├── mail.ts
│   │   │   ├── rules.ts
│   │   │   ├── settings.ts
│   │   │   └── triage.ts
│   │   ├── client.ts
│   │   ├── definition.ts
│   │   ├── helpers.ts
│   │   └── types.ts
│   ├── pa/
│   │   ├── commands/
│   │   │   ├── apps.ts
│   │   │   ├── connections.ts
│   │   │   ├── environments.ts
│   │   │   ├── flows.ts
│   │   │   └── index.ts
│   │   ├── client.ts
│   │   ├── definition.ts
│   │   └── types.ts
│   ├── pbi/
│   │   ├── commands/
│   │   │   ├── activity.ts
│   │   │   ├── apps.ts
│   │   │   ├── index.ts
│   │   │   ├── licenses.ts
│   │   │   ├── my.ts
│   │   │   ├── reports.ts
│   │   │   └── workspaces.ts
│   │   ├── client.ts
│   │   ├── definition.ts
│   │   ├── my-auth.ts
│   │   ├── my-client.ts
│   │   └── types.ts
│   ├── protect/
│   │   ├── commands/
│   │   │   ├── cameras.ts
│   │   │   ├── dashboard.ts
│   │   │   ├── events.ts
│   │   │   ├── index.ts
│   │   │   └── nvr.ts
│   │   ├── client.ts
│   │   ├── definition.ts
│   │   └── types.ts
│   ├── rmm/
│   │   ├── commands/
│   │   │   ├── agents.ts
│   │   │   ├── alerts.ts
│   │   │   ├── checks.ts
│   │   │   ├── custom-fields.ts
│   │   │   ├── dashboard.ts
│   │   │   ├── diagnostics.ts
│   │   │   ├── doctor.ts
│   │   │   ├── index.ts
│   │   │   ├── policies.ts
│   │   │   ├── processes.ts
│   │   │   ├── scripts.ts
│   │   │   ├── services.ts
│   │   │   ├── software.ts
│   │   │   ├── tasks.ts
│   │   │   └── updates.ts
│   │   ├── client.ts
│   │   ├── definition.ts
│   │   ├── resolve.ts
│   │   ├── types.ts
│   │   └── wrap-ps.ts
│   ├── sp/
│   │   ├── commands/
│   │   │   ├── dashboard.ts
│   │   │   ├── drives.ts
│   │   │   ├── files.ts
│   │   │   ├── groups.ts
│   │   │   ├── index.ts
│   │   │   ├── lists.ts
│   │   │   ├── pages.ts
│   │   │   ├── permissions.ts
│   │   │   ├── search.ts
│   │   │   └── sites.ts
│   │   ├── client.ts
│   │   ├── definition.ts
│   │   └── types.ts
│   ├── unifi/
│   │   ├── commands/
│   │   │   ├── clients.ts
│   │   │   ├── dashboard.ts
│   │   │   ├── devices.ts
│   │   │   ├── events.ts
│   │   │   ├── index.ts
│   │   │   ├── networks.ts
│   │   │   ├── sites.ts
│   │   │   └── vouchers.ts
│   │   ├── client.ts
│   │   ├── definition.ts
│   │   └── types.ts
│   ├── wrike/
│   │   ├── commands/
│   │   │   ├── comment-body.ts
│   │   │   ├── contacts.ts
│   │   │   ├── dashboard.ts
│   │   │   ├── index.ts
│   │   │   ├── leavers.ts
│   │   │   ├── onboarding.ts
│   │   │   ├── spaces.ts
│   │   │   ├── tasks.ts
│   │   │   ├── tickets.ts
│   │   │   └── workflows.ts
│   │   ├── client.ts
│   │   ├── definition.ts
│   │   └── types.ts
│   ├── actions.ts
│   ├── base.ts
│   ├── definitions.ts
│   └── registry.ts
├── cli.ts
├── config.ts
├── index.ts
├── tui-history.ts
└── tui.ts
```
