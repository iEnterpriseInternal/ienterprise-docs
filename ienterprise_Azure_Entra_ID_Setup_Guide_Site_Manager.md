# ienterprise | Azure Entra ID Setup Guide for Site Manager

## ienterprise Site Manager App  
### Mobile Application Authentication Configuration

**CONFIDENTIAL**  
Prepared by **ienterprise | Streamline LLC**

---

## What This Is About

This guide walks your IT team through the Azure Entra ID (formerly Azure Active Directory) configuration needed to enable single sign-on and role-based access for the ienterprise Site Manager mobile application. The Site Manager app is a store-centric tool that empowers front-line teams to resolve device issues independently and reduce IT support volume.

No custom code changes are required on your side. Just a series of configuration steps in the Azure Entra portal, and we handle the rest.

**Typical turnaround:** Same day once the app registration and role assignments are complete.

---

## 1. Prerequisites

1. Admin access to your organization's Azure Entra portal (entra.microsoft.com).
2. Access to the ienterprise Site Manager mobile application environment (staging and/or production).
3. Familiarity with your company's naming conventions and Azure AD group structure.

---

## 2. Understanding Site Manager Roles

Each function in the Site Manager app is governed by Azure AD app roles. Assign these roles to users or groups based on their job responsibilities.

| Display Name | Description | Module | Value | State |
|--------------|-------------|--------|-------|-------|
| `SM_STORE_MANAGER` | Site Manager Store Manager | SiteManager | `SM.STORE_MANAGER` | Enabled |
| `SM_STORE_REP` | Site Manager Store Representative | SiteManager | `SM.STORE_REP` | Enabled |

---

## 3. App Registration in Azure Entra ID

1. Sign in to the Azure portal (portal.azure.com).
2. In the search bar, search for **Microsoft Entra ID** and select it.
3. On the Microsoft Entra ID page, select **App registrations** on the left menu, then select **+ New registration**.
4. **Name:** Use a clear name (e.g., `SiteManager - <Company>`).
5. **Supported account types:** Select *Accounts in this organizational directory only (Single tenant)*.
6. **Redirect URI — Type:** Public client/native (mobile & desktop).
   - **Staging:** `msauth.com.streamlinedigital.site-manager://auth`
   - **Production:** `msauth.com.streamlinedigital.sitemanager://auth`
7. Click **Register**.
8. On the **Overview** page, note the **Application (client) ID** and **Directory (tenant) ID**.

---

## 4. Create Client Secret

1. Select **Certificates & secrets** on the left menu.
2. Click **+ New client secret**.
3. Provide a description and set expiry (recommended: 365 days).
4. Click **Add** and securely record the generated secret value.

> Store client secrets in a secure password manager. Never share via email or plaintext.

---

## 5. Define App Roles

1. Go to **App roles** in your app registration.
2. Click **Create app role** for each required role (see Section 2).
3. Set display name, description, allowed member types (Users/Groups), value, and state.
4. Click **Apply** after creating each role.

---

## 6. Assign Groups and Roles

1. In the Entra portal, navigate to **Enterprise Applications**.
2. Select your Site Manager application.
3. Go to **Users and Groups**.
4. Click **Add user/group**.
5. Assign each app role to the appropriate Azure AD group based on your store structure.

### Reference Mapping Example

| App Role | Azure AD Group Mapping |
|----------|------------------------|
| `SM_STORE_MANAGER` | Site Manager – Store Managers Group |
| `SM_STORE_REP` | Site Manager – Store Representatives Group |

---

## 7. Configure API Permissions

1. In your app registration, navigate to **API permissions**.
2. Click **Add a permission**.
3. Select **Microsoft Graph**.
4. Choose **Delegated permissions**.
5. Add the following permissions: `openid`, `offline_access`, `profile`.
6. Click **Add permissions** to confirm.

---

## 8. Expose API Scopes

1. Navigate to **Expose an API**.
2. Click **Add a scope**.
3. **Scope name:** Recommended: `api`
4. **Who can consent?:** Select *Admin Only*.
5. **Admin consent display name:** Enter a descriptive name.
6. **Admin consent description:** Provide a brief description of the scope.
7. **State:** Set to *Enabled*.
8. Click **Add scope**.

---

## 9. Update Application Manifest

1. In your app registration, go to **Manifest**.
2. Set `"accessTokenAcceptedVersion": 2` and ensure `requestedAccessTokenVersion` is also set to `2`.
3. Save the manifest.

---

## 10. Configure Redirect URIs

Depending on your platform and device targets, configure the following redirect URIs under **Authentication > Platform configurations**:

### iOS / macOS

| Setting | Value |
|---------|-------|
| Bundle ID | `com.streamlinedigital.site-manager` |
| Redirect URI | `msauth.com.streamlinedigital.site-manager://auth` |

### Mobile and Desktop Applications

- `https://login.microsoftonline.com/common/oauth2/nativeclient`
- `https://login.live.com/oauth20_desktop.srf`
- `msal{Client ID}://auth` (replace `{Client ID}` with your Application Client ID)

---

## 11. Collect Integration Credentials

Securely share the following with your ienterprise implementation team:

| Credential | Where to Find It |
|------------|------------------|
| Client ID | App registration > Overview page |
| Directory (tenant) ID | App registration > Overview page |
| Client Secret | Certificates & secrets (recorded during creation) |

---

## 12. Security Best Practices

- Store client secrets in a secure password manager; never share via email or plaintext.
- Rotate secrets regularly and promptly remove expired/unused credentials.
- Assign only the minimum permissions and roles required for your use case.
- Review group and role assignments periodically for compliance.
- Use admin consent for sensitive permissions and scopes.

---

## 13. Troubleshooting & Support

### Common Issues and Resolutions

- **Redirect URI mismatch:** Double-check the URI in both Azure and the Site Manager app configuration.
- **Permission errors:** Ensure all required permissions and roles are assigned and admin consent is granted.
- **Token issues:** Confirm the manifest version is updated to `2` and redirect URIs are correctly configured for mobile platforms.
- **Authentication loop on mobile:** Verify the Bundle ID and redirect URI match exactly in both Azure and the app.

### Need Help?

Contact your ienterprise implementation team or visit Microsoft's Azure AD documentation for additional reference.

---

**Confidential | ienterprise by Streamline LLC**
