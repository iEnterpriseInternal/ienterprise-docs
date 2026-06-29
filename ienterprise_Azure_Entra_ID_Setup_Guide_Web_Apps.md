# ienterprise | Azure Entra ID Setup Guide for Web Applications

## ienterprise Web Applications  
### Web Application Authentication & Authorization Configuration

**CONFIDENTIAL**  
Prepared by **ienterprise | Streamline LLC**

---

## What This Is About

This guide walks your IT team through the Azure Entra ID (formerly Azure Active Directory) configuration needed to enable single sign-on and role-based access for ienterprise web applications. ienterprise is a modular platform designed to streamline device management, support, and communication across the enterprise.

No custom code changes are required on your side. Just a series of configuration steps in the Azure Entra portal, and we handle the rest.

**Typical turnaround:** Same day once the app registration and role assignments are complete.

---

## 1. Prerequisites

1. Admin access to your organization's Azure Entra portal (entra.microsoft.com).
2. Access to the ienterprise web application environment (staging and/or production).
3. Familiarity with your company's naming conventions and Azure AD group structure.

---

## 2. ienterprise Modules Overview

ienterprise is a modular platform. The following modules may be assigned to users via Azure AD roles:

| Module | Description |
|--------|-------------|
| Site Manager | Store-centric tool empowering front-line teams to resolve device issues independently and reduce IT support volume. |
| Enterprise Manager | Centralized dashboard for real-time device health, security compliance, and operational metrics. |
| Smart Support | Consolidated diagnostics, troubleshooting, and remote actions for rapid issue resolution. |
| Device Manager | Automated bulk enrollment, asset tracking, and role-based device assignment for accurate provisioning. |
| Automation Admin | Orchestration engine for scheduling and monitoring OS updates with minimal disruption. |
| Notification Admin | Communication management for creating, scheduling, and tracking system-wide alerts. |
| iElevate | Technician-focused mobile tool for network/device validation and step-by-step installation guidance. |

---

## 3. Understanding ienterprise Roles

Each module and function in ienterprise is governed by Azure AD app roles. Assign these roles to users or groups based on their job responsibilities.

| Display Name | Description | Module | Value | State |
|--------------|-------------|--------|-------|-------|
| `IE_SST1` | Smart Support Tier 1 | Smart Support | `IE.SST1` | Enabled |
| `IE_SST2` | Smart Support Tier 2 | Smart Support | `IE.SST2` | Disabled |
| `IE_SST3` | Smart Support Tier 3 | Smart Support | `IE.SST3` | Enabled |
| `IE_SS_RETIRE` | Device retirement actions | Smart Support | `IE.SS_RETIRE` | Enabled |
| `IE_SS_ERASE` | Device erase capability | Smart Support | `IE.SS_ERASE` | Enabled |
| `IE_SS_LOST` | Lost device actions | Smart Support | `IE.SS_LOST` | Enabled |
| `NIGHTLY_JOB_ADMIN` | Nightly job administration | Automation Admin | `IE.NIGHTLY_JOB_ADMIN` | Enabled |
| `IE_DEVICE_ASSIGNMENT` | Device assignment | Device Manager | `IE.SS_DEVICE_ASSIGN` | Enabled |
| `IE_NOTIFICATION_ADMIN` | Notification administration | Notification Admin | `IE.NOTIFICATION_ADMIN` | Enabled |
| `IE_EM_DASHBOARD` | Enterprise Manager dashboard | Enterprise Manager | `IE.DASHBOARD` | Enabled |

Adjust the enabled/disabled state as needed for your deployment. Add or modify roles for additional modules as required.

---

## 4. App Registration in Azure Entra ID

1. Sign in to the Azure portal (portal.azure.com).
2. Go to **Applications > App registrations**.
3. Click **New registration**.
4. **Name:** Use a clear name (e.g., `ienterprise WebApp - <Company>`).
5. **Supported account types:** Select *Accounts in this organizational directory only (Single tenant)*.
6. **Redirect URI — Type:** Web.
   - **Staging:** `https://<company>.ienterprise.info/api/auth/callback/azure-ad`
   - **Production:** `https://<company>.ienterprise.com/api/auth/callback/azure-ad`
7. Click **Register**.

---

## 5. Configure API Permissions

1. In your app registration, navigate to **API permissions**.
2. Click **Add a permission**.
3. Select **Microsoft Graph**.
4. Choose **Delegated permissions**.
5. Add the following permissions: `openid`, `offline_access`.
6. Click **Add permissions** to confirm.

---

## 6. Set Up Authentication

1. Go to the **Authentication** section in your app registration.
2. Under **Implicit grant and hybrid flows**, enable **Access tokens** and **ID tokens**.
3. Save your changes.

---

## 7. Expose API Scopes

1. Navigate to **Expose an API**.
2. Click **Add a scope**.
3. **Scope name:** Recommended: `api`
4. **Who can consent?:** Select *Admin Only*.
5. **Admin consent display name:** Enter a descriptive name.
6. **Admin consent description:** Provide a brief description of the scope.
7. **State:** Set to *Enabled*.
8. Click **Add scope**.
9. Record the scope name for use in web integration.

---

## 8. Define App Roles

1. Go to **App roles** in your app registration.
2. Click **Create app role** for each required ienterprise role (see Section 3).
3. Set display name, description, allowed member types (Users/Groups), value, and state.
4. Click **Apply** after creating each role.

---

## 9. Assign Groups and Roles

1. In the Entra portal, navigate to **Enterprise Applications**.
2. Select your ienterprise application.
3. Go to **Users and Groups**.
4. Click **Add user/group**.
5. Assign each app role to the appropriate Azure AD group based on your support structure.
6. Click **Assign** to confirm.

### Reference Mapping Example

| App Role | Azure AD Group Mapping |
|----------|------------------------|
| `IE_SST1` | Customer Existing Tier 1 Group |
| `IE_SST2` | Customer Existing Tier 2 Group |
| `IE_SST3` | Customer Existing Tier 3 Group |
| `IE_SS_RETIRE` | Customer Existing Tier 3 Group |
| `NIGHTLY_JOB_ADMIN` | Automation/Job Admin Group |
| `IE_DEVICE_ASSIGNMENT` | Device Assignment Group |
| `IE_NOTIFICATION_ADMIN` | Notification Admins |
| `IE_EM_DASHBOARD` | Management/Reporting Group |
| `IE_SS_ERASE` | Customer Existing Tier 3 Group |
| `IE_SS_LOST` | Customer Existing Tier 3 Group |

---

## 10. Update Application Manifest

1. In your app registration, go to **Manifest**.
2. Set `"accessTokenAcceptedVersion": 2` and ensure `requestedAccessTokenVersion` is also set to `2`.
3. Save the manifest.

---

## 11. Collect Integration Credentials

Securely share the following with your ienterprise implementation team:

| Credential | Where to Find It |
|------------|------------------|
| Client ID | App registration > Overview page |
| Directory (tenant) ID | App registration > Overview page |
| Client Secret | Certificates & secrets (create new, set 365-day expiry, record immediately) |

> Store client secrets in a secure password manager. Never share via email or plaintext.

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

- **Redirect URI mismatch:** Double-check the URI in both Azure and ienterprise configuration.
- **Permission errors:** Ensure all required permissions and roles are assigned and admin consent is granted.
- **Token issues:** Confirm access and ID tokens are enabled under Authentication and the manifest version is set to `2`.

### Need Help?

Contact your ienterprise implementation team or visit Microsoft's Azure AD documentation for additional reference.

---

## 14. Additional Resources

- Azure App Registration Quickstart
- Configure permissions and consent
- Expose an API in Azure AD
- ienterprise Documentation (add your internal onboarding link)

Review this guide whenever you update your ienterprise environment or Azure Entra ID settings to ensure ongoing secure integration.

---

**Confidential | ienterprise by Streamline LLC**
