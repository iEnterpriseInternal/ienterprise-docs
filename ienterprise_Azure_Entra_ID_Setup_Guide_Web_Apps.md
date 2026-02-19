# ienterprise | Azure Entra ID Setup Guide for Web Applications

## Web Application Authentication & Authorization Configuration

**CONFIDENTIAL**  
Prepared by **ienterprise | Streamline LLC**

---

## What This Is About

This guide walks IT teams through Azure Entra ID configuration required to enable single sign-on (SSO) and role-based access for ienterprise web applications.

No custom code changes are required. Configuration is completed entirely within the Azure Entra portal.

**Typical turnaround:** Same day after app registration and role assignment.

---

## 1. Prerequisites

- Admin access to Azure Entra portal (entra.microsoft.com)  
- Access to ienterprise web environments (staging and/or production)  
- Familiarity with Azure AD group structure and naming conventions  

---

## 2. ienterprise Modules Overview

ienterprise is a modular platform. Access is controlled via Azure AD app roles.

- Site Manager  
- Enterprise Manager  
- Smart Support  
- Device Manager  
- Automation Admin  
- Notification Admin  
- iElevate  

---

## 3. App Registration

1. Sign in to portal.azure.com  
2. Go to Applications > App registrations  
3. Select New registration  
4. Name: `ienterprise WebApp - <Company>`  
5. Account type: Single tenant  
6. Redirect URI (Web):
   - Staging: https://<company>.ienterprise.info/api/auth/callback/azure-ad  
   - Production: https://<company>.ienterprise.com/api/auth/callback/azure-ad  
7. Click Register  

---

## 4. API Permissions

- Microsoft Graph  
- Delegated permissions:
  - openid  
  - offline_access  

Grant admin consent.

---

## 5. Authentication Settings

Enable:
- Access tokens  
- ID tokens  

---

## 6. Expose API Scope

- Scope name: api  
- Consent: Admin only  
- State: Enabled  

---

## 7. App Roles

Create app roles for required ienterprise functions (Smart Support tiers, Device Manager, Automation Admin, etc.).  
Assign roles to Azure AD groups based on responsibilities.

---

## 8. Manifest Update

Set:
- accessTokenAcceptedVersion: 2  
- requestedAccessTokenVersion: 2  

---

## 9. Integration Credentials

Share securely:
- Client ID  
- Tenant ID  
- Client Secret  

---

## 10. Security Best Practices

- Store secrets securely  
- Rotate credentials regularly  
- Assign least-privilege roles  
- Review access periodically  

---

**Confidential | ienterprise by Streamline LLC**
