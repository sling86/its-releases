# `its-cli-admin` scopes

Hand-written, not generated. It explains the `its-cli-admin` entry in
`manifest/entra-apps.jsonc`: which delegated scope each write command needs and
why. Edit it by hand when a write command changes.

## The idea

- Two sign-in apps, both public clients: no secret, no app roles, delegated
  scopes only. Nothing in either can run unattended.
- **`its-cli`**: read-only Graph, plus the signed-in person's own mail,
  calendar, Teams and files. For normal accounts.
- **`its-cli-admin`**: everything the write commands need. For admin accounts
  only.
- A delegated token is capped by the person's own roles. A scope here does
  nothing for someone who lacks the Entra or Exchange role behind it, so the
  Tier 0 scopes are safe to declare as long as the app is locked to admins.
- Lock it down: **Assignment required = Yes**, assign only the group of admin
  accounts, and put a sign-in rule on the app that needs phishing-resistant MFA
  (report-only first, then enforce).
- Sign in with `its auth login --profile admin --client-id <app id>`, then
  `its auth use admin --provider entra,intune,sp,exo,bc,pa,pbi`.

## Checked against

- Scope names were checked against the live scope list of each resource (Graph,
  Office 365 Exchange Online, Dynamics 365 Business Central, Power BI Service,
  Microsoft Flow Service), read-only. `its entra apps plan` resolves every name.
- Least-privilege choices come from the permissions table on each Microsoft
  Graph API page on Microsoft Learn.
- Resource IDs: Exchange Online `00000002-0000-0ff1-ce00-000000000000`,
  Business Central `996def3d-b36c-4153-8607-a6fd3c01b89f`. Both match the
  manifest aliases `exchange` and `bc`.

## Scope table: Microsoft Graph

| Command | Endpoint | Scope |
| --- | --- | --- |
| `entra users create\|update\|delete\|enable\|disable\|transfer\|reinstate\|set-photo`, `entra offboarding run`, `entra onboarding copy-groups` | `/users`, `/users/{id}`, `/users/{id}/manager/$ref`, `/users/{id}/photo/$value` | `User.ReadWrite.All` |
| `entra users invite` | `/invitations` | `User.Invite.All` |
| `entra users revoke-sessions` | `/users/{id}/revokeSignInSessions` | `User.ReadWrite.All` (narrower `User.RevokeSessions.All` is redundant here) |
| `entra users set-password`, `entra users bootstrap-admin`, `entra admin-bootstrap run` | `PATCH /users/{id}` with `passwordProfile` | `User.ReadWrite.All`. Resetting a privileged admin's password also needs `Directory.AccessAsUser.All`, which is **not** on its-cli-admin: do that in the Entra portal. |
| `entra licences assign\|remove` | `/users/{id}/assignLicense` | `User.ReadWrite.All` (narrower `LicenseAssignment.ReadWrite.All` is redundant here) |
| `entra groups create\|owners\|add-owner\|remove-owner\|add-member\|remove-member\|edit-rule\|make-dynamic` | `/groups`, `/groups/{id}/members/$ref`, `/groups/{id}/owners/$ref` | `Group.ReadWrite.All` |
| `entra roles assign\|remove` | `/roleManagement/directory/roleAssignments` | `RoleManagement.ReadWrite.Directory` |
| `entra pim activate\|deactivate\|assign\|unassign` | `/roleManagement/directory/roleAssignmentScheduleRequests`, `/roleManagement/directory/roleEligibilityScheduleRequests` | `RoleManagement.ReadWrite.Directory` (narrower `RoleAssignmentSchedule.ReadWrite.Directory` / `RoleEligibilitySchedule.ReadWrite.Directory` are covered by it) |
| `entra pim set` | `/policies/roleManagementPolicies/{id}/rules/{rule}` | `RoleManagementPolicy.ReadWrite.Directory` |
| `entra ca create\|patch\|delete\|exclude-user\|unexclude-user\|exclude-guests\|report-only` | `/identity/conditionalAccess/policies` | `Policy.ReadWrite.ConditionalAccess` (also needs `Policy.Read.All` and `Application.ReadWrite.All` to read the referenced apps) |
| `entra ca create-named-location` | `/identity/conditionalAccess/namedLocations` | `Policy.ReadWrite.ConditionalAccess` |
| `entra authmethods user\|remove`, `entra tap create\|revoke` | `/users/{id}/authentication/methods`, `.../temporaryAccessPassMethods` | `UserAuthenticationMethod.ReadWrite.All` |
| `entra authmethods enable\|disable\|patch` | `/policies/authenticationMethodsPolicy/authenticationMethodConfigurations/{id}` | `Policy.ReadWrite.AuthenticationMethod` |
| `entra xtenant trust-mfa\|trust-device\|partner-add\|partner-set` | `/policies/crossTenantAccessPolicy/default`, `.../partners` | `Policy.ReadWrite.CrossTenantAccess` |
| `entra apps register\|add-password\|rotate\|apply` (app and owner writes) | `/applications`, `/applications/{id}/addPassword`, `.../removePassword`, `.../owners/$ref` | `Application.ReadWrite.All` |
| `entra apps apply`, `entra consent app-role-grant\|app-role-revoke` | `/servicePrincipals/{id}/appRoleAssignments`, `.../appRoleAssignedTo` | `AppRoleAssignment.ReadWrite.All` |
| `entra apps apply`, `entra consent add-scope\|remove-scope` | `/oauth2PermissionGrants` | `DelegatedPermissionGrant.ReadWrite.All` |
| `entra devices delete` | `DELETE /devices/{id}` | Needs `Directory.AccessAsUser.All` (Learn lists nothing narrower for delegated). **Left off its-cli-admin on purpose:** it acts as the signed-in person across the whole directory. Delete devices in the Intune or Entra portal. |
| `entra sync provision` | `/servicePrincipals/{id}/synchronization/jobs/{id}/provisionOnDemand` | `Synchronization.ReadWrite.All` |
| `entra graph post\|patch\|put\|delete` (and the same on `intune`, `sp`) | any Graph path | whatever the path needs; none added for this |
| Reads the above lean on | `/auditLogs`, `/identityProtection`, `/domains`, `/organization`, `/policies`, directory objects | `AuditLog.Read.All`, `IdentityRiskEvent.Read.All`, `IdentityRiskyUser.Read.All`, `Domain.Read.All`, `Organization.Read.All`, `Policy.Read.All`, `Directory.Read.All` |

## Scope table: Intune (Graph)

| Command | Endpoint | Scope |
| --- | --- | --- |
| `intune devices sync\|rename\|set-primary-user\|primary-user` | `/deviceManagement/managedDevices/{id}/syncDevice`, `.../setDeviceName`, `.../users/$ref` | `DeviceManagementManagedDevices.ReadWrite.All` |
| `intune devices wipe\|retire\|rotate-bitlocker` | `/deviceManagement/managedDevices/{id}/wipe`, `.../retire`, `.../rotateBitLockerKeys` | `DeviceManagementManagedDevices.PrivilegedOperations.All` |
| `intune devices recovery-keys` | `/informationProtection/bitlocker/recoveryKeys/{id}` (key value) | `BitlockerKey.Read.All` |
| `intune win32-apps upload`, `intune apps assign` | `/deviceAppManagement/mobileApps`, `.../assignments` | `DeviceManagementApps.ReadWrite.All` |
| `intune app-configs create\|assign`, `intune update-rings create\|update\|assign`, `intune enrolment set-restriction` | `/deviceAppManagement/mobileAppConfigurations`, `/deviceManagement/deviceConfigurations`, `/deviceManagement/deviceEnrollmentConfigurations`, `/deviceManagement/windows*UpdateProfiles` | `DeviceManagementConfiguration.ReadWrite.All` |
| `intune autopilot import\|deregister\|sync\|tag`, `intune esp update`, `intune android qr` | `/deviceManagement/importedWindowsAutopilotDeviceIdentities`, `.../windowsAutopilotDeviceIdentities`, `.../windowsAutopilotSettings/sync`, `.../androidDeviceOwnerEnrollmentProfiles/{id}/createToken` | `DeviceManagementServiceConfig.ReadWrite.All` |

## Scope table: SharePoint, OneDrive, Teams, Outlook

These are the signed-in person's own, plus shared where the command takes a
shared mailbox.

**Mail, calendar, chat, presence and OneNote scopes are on `its-cli` only, not
on `its-cli-admin`.** `outlook` and `teams` map to the person's normal profile
(`its auth use me --provider outlook,teams`), and admin accounts have no
mailbox. Keeping them off the admin app means an admin token can't read or send
from any mailbox, shared ones included. The `outlook` and `teams` rows below say
what those commands need on `its-cli`. SharePoint, OneDrive and Teams-admin
scopes stay on both.

| Command | Endpoint | Scope |
| --- | --- | --- |
| `sp lists create-item\|update-item\|delete-item`, `sp files upload\|delete\|move\|checkout\|checkin\|restore\|share` | `/sites/{id}/lists/{id}/items`, `/sites/{id}/drives/{id}/items/{id}/...` | `Sites.ReadWrite.All`, `Files.ReadWrite.All` |
| `sp permissions share\|links\|grant-app\|remove\|break-inheritance\|restore-inheritance` | `/sites/{id}/permissions`, `.../items/{id}/permissions` | `Sites.FullControl.All` |
| `sp` calls that need a token for the SharePoint host itself | `https://<tenant>.sharepoint.com` | SharePoint resource: `AllSites.Manage`, `AllSites.FullControl` (kept as on `its-cli`) |
| `outlook mail move\|copy\|read\|flag\|categorise\|delete\|draft\|send`, `outlook drafts ...`, `outlook folders ...`, `outlook attachments ...` | `/me/messages`, `/me/mailFolders`, `/me/sendMail` | `Mail.ReadWrite`, `Mail.Send` |
| the same on a shared mailbox (`OUTLOOK_DEFAULT_USER`) | `/users/{mailbox}/...` | `Mail.ReadWrite.Shared`, `Mail.Send.Shared` |
| `outlook events create\|update\|delete\|respond` | `/me/events` | `Calendars.ReadWrite`, `Calendars.ReadWrite.Shared` |
| `outlook autoreply set`, `outlook categories ...`, `outlook rules ...` | `/me/mailboxSettings`, `/me/outlook/masterCategories`, `/me/mailFolders/inbox/messageRules` | `MailboxSettings.ReadWrite`, `Mail.ReadWrite` |
| `teams chats send\|edit-message\|react\|unreact\|delete-message` | `/chats/{id}/messages`, `.../setReaction`, `.../softDelete` | `ChatMessage.Send`, `Chat.ReadWrite` |
| `teams teams create` | `POST /teams` | `Team.Create` |
| `teams teams delete` | `DELETE /groups/{id}` | `Group.ReadWrite.All` |
| `teams files ...`, `teams chats images` | `/me/drive/items/{id}/invite`, `/shares/{id}/driveItem` | `Files.ReadWrite.All` |
| presence and joined teams (reads) | `/me/presence`, `/me/joinedTeams` | `Presence.Read`, `Team.ReadBasic.All` |
| `m365health` (all reads) | `/admin/serviceAnnouncement/healthOverviews`, `.../issues`, `.../messages` | `ServiceHealth.Read.All`, `ServiceMessage.Read.All` |
| `sp onenote ...` (reads) | `/sites/{id}/onenote`, `/me/onenote` | `Notes.Read.All` |

## Scope table: Power BI, Power Automate, Power Apps

| Command | Endpoint | Scope |
| --- | --- | --- |
| `pbi workspaces add-user\|update-user\|remove-user`, `pbi my add-workspace-user\|update-workspace-user\|remove-workspace-user` | `/groups/{id}/users` | Power BI Service `Workspace.ReadWrite.All` |
| `pbi my refresh\|refresh-schedule` | `/groups/{id}/datasets/{id}/refreshes` | Power BI Service `Dataset.ReadWrite.All` |
| `pbi scan run` | `/admin/workspaces/getInfo`, `.../scanResult/{id}` | Power BI Service `Tenant.Read.All`. **A guess:** Learn lists it for the admin scanner API but I have not run it. |
| `pbi` reads | `/apps`, `/reports`, `/datasets`, `/groups` | Power BI Service `App.Read.All`, `Report.Read.All`, `Dataset.Read.All`, `Workspace.Read.All` |
| `pa flows stop\|start\|delete\|set-owner` | `.../scopes/admin/environments/{id}/flows/{id}/stop`, `.../start`, `.../modifyPermissions` | Microsoft Flow Service `Flows.Manage.All` (`Flows.Read.All` for the reads) |
| `pa` app and connection reads | `.../scopes/admin/environments/{id}/apps`, `.../connections` | PowerApps Service `User` |

## Scope table: Exchange and Business Central

| Command | Endpoint | Scope |
| --- | --- | --- |
| every `its exo` write (mailboxes, groups, rules, transport rules, DKIM, rooms, sharing policies, quarantine, allow/block, spam) | `Connect-ExchangeOnline -AccessToken` against `https://outlook.office365.com` | Office 365 Exchange Online `Exchange.Manage`. Exchange RBAC then follows the person's own role groups, so someone with no Exchange role gets refused by Exchange. |
| `its bc record create\|update\|delete\|call`, `bc users ...` | Business Central API and Automation API | Dynamics 365 Business Central `Financials.ReadWrite.All`. BC applies the person's own permission sets. |
| `its bc environments copy\|delete\|operations`, `bc apps install\|uninstall\|update\|upload\|scheduled\|operations` | Business Central Admin Centre API | Dynamics 365 Business Central `user_impersonation`. **A guess:** it is the pair the BC MCP app already carries; I have not run an Admin Centre call as a delegated user. |

Exchange also publishes `Exchange.ManageV2`, which belongs to the newer Exchange
Online Admin API. `its exo` uses PowerShell, which takes `Exchange.Manage`, so
V2 is not declared.

## Not covered (by design)

- Nothing here is an app role, so no unattended write is possible from this app.
- `its entra apps apply` grants admin consent for all users on every declared
  scope. That is safe only because **Assignment required** limits who can sign
  in to the app at all.
- `rmm`, `unifi`, `dokploy`, `bw`, `wrike`, `cf` and the rest do not use Microsoft
  delegated scopes and are not part of this app.

## Proposed `its-cli` scopes after the move

**A proposal only.** The live `its-cli` entry is unchanged. Trim it once every
admin has moved to `its-cli-admin`: removing a scope an admin still relies on
breaks their commands.

Everything here is a read, or the person's own mail, calendar, Teams or files.
Every `*.ReadWrite.All`, `*.ManageAsApp`, `RoleManagement.*`, `Policy.ReadWrite.*`
and `Directory.AccessAsUser.All` goes.

| Scope | Why it stays |
| --- | --- |
| `offline_access` | Refresh token, so sign-in lasts. |
| `Calendars.ReadWrite`, `Mail.ReadWrite`, `Mail.Send`, `MailboxSettings.ReadWrite` | The person's own mail and calendar (`its outlook`). |
| `Calendars.ReadWrite.Shared`, `Mail.ReadWrite.Shared`, `Mail.Send.Shared` | Shared mailboxes the person already has access to. |
| `Chat.ReadWrite`, `ChatMessage.Send`, `Presence.Read`, `Team.ReadBasic.All` | The person's own Teams chats and presence (`its teams`). |
| `Files.ReadWrite` | The person's own OneDrive. |
| `Notes.Read.All` | OneNote reads. |
| `Directory.Read.All`, `Domain.Read.All`, `Organization.Read.All` | Directory lookups every read command starts with. |
| `User.Read.All`, `Group.Read.All` | Replace `User.ReadWrite.All` and `Group.ReadWrite.All` for user and group reads. |
| `Application.Read.All`, `RoleManagement.Read.Directory` | Replace the write versions for `apps` and `roles` reads. |
| `AuditLog.Read.All`, `IdentityRiskEvent.Read.All`, `IdentityRiskyUser.Read.All` | Sign-in, audit and risk reads. |
| `Policy.Read.All`, `UserAuthenticationMethod.Read.All` | Conditional Access, policy and auth-method reads. |
| `BitlockerKey.ReadBasic.All` | Recovery key metadata (not the key value). |
| `DeviceManagementApps.Read.All`, `DeviceManagementConfiguration.Read.All`, `DeviceManagementManagedDevices.Read.All`, `DeviceManagementScripts.Read.All`, `DeviceManagementServiceConfig.Read.All` | Intune reads. |
| `Sites.Read.All`, `Files.Read.All` | SharePoint site and file reads. |
| `ServiceHealth.Read.All`, `ServiceMessage.Read.All` | `its m365health`. |
| SharePoint `AllSites.Read` | Reads that need a token for the SharePoint host. |
| Power BI `App.Read.All`, `Dataset.Read.All`, `Report.Read.All`, `Workspace.Read.All` | Power BI reads. |
| Flow `User`, `Flows.Read.All`, PowerApps `User` | Power Automate and Power Apps reads. |

Not carried by `its-cli` at all: Exchange and Business Central. Anyone who needs
`its exo` or `its bc` signs in with `its-cli-admin`, or gets a dedicated app.

**Guesses in this proposal:** the exact set of read scopes the read commands
need was not traced command by command (several, like `User.Read.All` and
`Sites.Read.All`, are not on the live app today). Check by running the read
commands as a normal account against a trimmed test app before applying.
