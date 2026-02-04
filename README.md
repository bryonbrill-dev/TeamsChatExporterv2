# Microsoft Teams Chat Exporter

This repository exports Microsoft Teams chat history to HTML using the Microsoft Graph API and a PowerShell script.

## Requirements

### System requirements

- **PowerShell 7+** (the script uses `#Requires -Version 7.0`).
- Network access to:
  - `https://login.microsoftonline.com`
  - `https://graph.microsoft.com`
- An account in the target Microsoft 365 tenant with access to the chats you want to export.

### Microsoft Graph (enterprise) setup

You must register an Azure AD (Entra ID) application with delegated permissions for the Microsoft Graph API. The script uses the OAuth 2.0 **device code flow**, so **no client secret** is required.

1. **Create the app registration**
   - Entra ID (Azure AD) portal → **App registrations** → **New registration**.
   - Name it (e.g., `TeamsChatExport`).
   - Supported account type: typically **Single tenant**.
   - Redirect URI: leave blank (device code flow does not require one).

2. **Collect IDs**
   - From the app registration overview, copy:
     - **Application (client) ID** → used as `-clientId`.
     - **Directory (tenant) ID** → used as `-tenantId`.

3. **Add Microsoft Graph delegated permissions**
   - **API permissions** → **Add a permission** → **Microsoft Graph** → **Delegated permissions**.
   - Add:
     - `Chat.Read`
     - `User.Read`
     - `User.ReadBasic.All`
   - (The script also requests `offline_access` during authentication for refresh tokens.)

4. **Grant admin consent (recommended)**
   - In **API permissions**, click **Grant admin consent** for the tenant.
   - This avoids repeated consent prompts for users.

5. **Confirm device code flow**
   - No special setting is needed in the app registration for device code flow. Users will authenticate via a device code prompt in the terminal.

## Running the script

```powershell
# From the repo root
pwsh ./Get-MicrosoftTeamsChat.ps1 \
  -ExportFolder "D:\ExportedHTML" \
  -clientId "<Application (client) ID>" \
  -tenantId "<Directory (tenant) ID>" \
  -domain "contoso.com"
```

### Optional filters

You can limit exported messages by date/time:

```powershell
pwsh ./Get-MicrosoftTeamsChat.ps1 \
  -ExportFolder "D:\ExportedHTML" \
  -clientId "<Application (client) ID>" \
  -tenantId "<Directory (tenant) ID>" \
  -domain "contoso.com" \
  -From "2024-01-01" \
  -To "2024-02-01"
```

### Authentication flow

When you run the script:

1. It prints a device code and login URL.
2. Visit the URL, enter the code, and sign in with a user account from the target tenant.
3. The script will then access Microsoft Graph and export chats to the `-ExportFolder` location.

## Notes and troubleshooting

- **Permissions**: If you see access errors, confirm the signed-in user has access to the target chats and that admin consent has been granted for the app registration.
- **`invalid_client` / missing `client_secret`**: Ensure the app registration is configured as a **public client**. In Entra ID → App registration → **Authentication**, enable **Allow public client flows** (device code requires this). The script uses device code flow and does **not** use a client secret.
- **Tenant/domain mismatch**: The `-domain` parameter is used to build user UPNs for profile photos. Use your tenant's email domain (e.g., `contoso.com`).
- **Firewalls/proxies**: Ensure outbound HTTPS to `login.microsoftonline.com` and `graph.microsoft.com` is allowed.
- **PowerShell execution policy**: On first run you may see: `Get-MicrosoftTeamsChat.ps1 is not digitally signed. You cannot run this script on the current system.` This means the local execution policy blocks unsigned scripts. See [about_Execution_Policies](https://go.microsoft.com/fwlink/?LinkID=135170) for details. Common fixes include running the script with a bypass for the current session (`pwsh -ExecutionPolicy Bypass -File ./Get-MicrosoftTeamsChat.ps1 ...`), or changing the policy for your user (`Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned`) if permitted by your organization.
