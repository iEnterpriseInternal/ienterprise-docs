# ienterprise | Azure Entra ID Setup Guide for Site Manager

## Mobile Application Authentication Configuration

**CONFIDENTIAL**  
Prepared by **ienterprise | Streamline LLC**

---

## What This Is About

This guide walks IT teams through Azure Entra ID configuration for the ienterprise Site Manager mobile application.

The Site Manager app enables store teams to manage device issues independently using secure SSO.

---

## 1. Prerequisites

- Admin access to Azure Entra portal  
- Access to Site Manager app environments  
- Familiarity with Azure AD groups  

---

## 2. Site Manager Roles

- SM_STORE_MANAGER  
- SM_STORE_REP  

Assign roles based on store responsibilities.

---

## 3. App Registration

1. Sign in to portal.azure.com  
2. Go to Microsoft Entra ID > App registrations  
3. New registration  
4. Name: `SiteManager - <Company>`  
5. Account type: Single tenant  
6. Redirect URI (Public client):
   - Staging: msauth.com.streamlinedigital.site-manager://auth  
   - Production: msauth.com.streamlinedigital.sitemanager://auth  

---

## 4. Client Secret

Create a client secret with 365-day expiry and store securely.

---

## 5. App Roles

Create roles:
- SM_STORE_MANAGER  
- SM_STORE_REP  

Assign roles to Azure AD groups.

---

## 6. API Permissions

Microsoft Graph delegated permissions:
- openid  
- offline_access  
- profile  

---

## 7. Expose API Scope

- Scope name: api  
- Admin consent only  
- Enabled  

---

## 8. Manifest Update

Set:
- accessTokenAcceptedVersion: 2  

---

## 9. Redirect URI Configuration

Configure under Authentication:
- iOS/macOS bundle redirect  
- Native client fallback URLs  

---

## 10. Integration Credentials

Share securely:
- Client ID  
- Tenant ID  
- Client Secret  

---

## 11. Security Best Practices

- Rotate secrets  
- Limit role assignments  
- Review access regularly  

---

**Confidential | ienterprise by Streamline LLC**
