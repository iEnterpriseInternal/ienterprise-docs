# ienterprise | Microsoft Intune Integration Guide

## Device Management Integration  
### Configuration & Setup Requirements

**CONFIDENTIAL**  
Prepared by **ienterprise | Streamline LLC**

---

## What This Is About

This guide walks you through connecting ienterprise to your Microsoft Intune environment via the Microsoft Graph API. The goal is straightforward: give your IT teams and store managers real-time visibility into managed devices, applications, and configuration compliance directly from ienterprise's **Device Manager** and **Site Manager** — without switching between consoles.

We need a small set of details from your team. Once we have them, we handle the rest.

- No custom development inside Intune or Azure  
- No changes to your existing policies  
- A clean, read-only connection between ienterprise and your Intune tenant through Microsoft Graph  

**Typical setup time:** 3–5 business days after we receive your Azure AD app registration details and confirm API permissions.

---

## 1. Connection & Authentication

Intune doesn't use basic username/password authentication the way some legacy systems do. Everything flows through the Microsoft Graph API, which means authentication goes through Azure Active Directory using OAuth 2.0.

Here's what we need from your Azure AD tenant:

### Required Details

- **Azure AD Tenant ID**  
  Your directory (tenant) ID from the Azure Portal
- **Application (Client) ID**  
  The App Registration client ID created for ienterprise
- **Client Secret or Certificate**  
  A client secret or certificate for the app registration
- **Graph API Base URL**  
  `https://graph.microsoft.com` (standard for all tenants)

### Recommendation

Create a **dedicated App Registration** specifically for the ienterprise integration. Don't reuse an existing one that has broader permissions — a dedicated registration keeps the permission scope tight and makes it easy to audit or revoke access without affecting other integrations.

---

## 2. API Permission Requirements

The integration uses the Microsoft Graph API exclusively. We need **Application-type** permissions (not Delegated), because ienterprise connects as a background service without a signed-in user. The App Registration needs:

| Permission | Type | Purpose |
|-----------|------|---------|
| `DeviceManagementManagedDevices.Read.All` | Application | Read managed device inventory and details |
| `DeviceManagementApps.Read.All` | Application | Read detected applications on devices |
| `DeviceManagementConfiguration.Read.All` | Application | Read device configuration profiles and compliance |
| `Device.Read.All` | Application | Read Azure AD registered device records |

**Important:** All permissions require **Admin Consent** from a Global Administrator or Intune Administrator in your tenant. Without admin consent, the API calls return `403` errors regardless of the app registration setup.

---

## 3. API Endpoints & Data Flow

Here's exactly what ienterprise connects to and why. All calls are read-only `GET` requests against Microsoft Graph.

### Managed Devices

The core endpoint. It returns every device enrolled in Intune — device name, OS, compliance state, last sync time, and more.

| Purpose | Endpoint |
|--------|----------|
| Device list | `GET v1.0/deviceManagement/managedDevices` |
| Device details | `GET v1.0/deviceManagement/managedDevices/{deviceId}` |
| Filter by OS | `$filter=operatingSystem eq 'ios'` |
| Beta (expanded) | `GET beta/deviceManagement/managedDevices({id})?$expand=detectedApps,users` |

**A note on properties:** Device name, store number, and device type depend on extended attributes being populated in Intune. The remaining properties (OS version, compliance state, encryption status, last sync, etc.) are present in the standard API response.

### Detected Applications

ienterprise pulls application data to show which apps are installed across your fleet. This powers the app compliance views in Device Manager.

| Purpose | Endpoint |
|--------|----------|
| All applications | `GET v1.0/deviceManagement/detectedApps` |
| Devices by app | `GET v1.0/deviceManagement/detectedApps/{appId}/managedDevices` |

**v1.0 limitation:** You can't get apps for a specific device directly in v1.0 — the navigation goes the other direction, from detected app to its devices. The Beta endpoint supports `$expand` on the device details call to include `detectedApps`, which ienterprise uses when available.

### Device Configuration

Configuration profiles tell ienterprise whether devices are set up correctly — Wi-Fi profiles, VPN settings, restrictions, and compliance baselines.

| Purpose | Endpoint |
|--------|----------|
| All configurations | `GET v1.0/deviceManagement/deviceConfigurations` |
| Configuration details | `GET v1.0/deviceManagement/deviceConfigurations/{configId}` |
| Device statuses | `GET v1.0/deviceManagement/deviceConfigurations/{configId}/deviceStatuses` |

---

## 4. Device Property Mapping

Every Intune environment uses extended attributes differently. Some organizations store the store number in a device naming convention; others use Azure AD extension attributes or Intune categories. We need to understand your setup so ienterprise displays the right data in the right place.

Send us the mapping for these key fields:

| ienterprise Field | Your Intune Source |
|-------------------|--------------------|
| Device Name | e.g., `deviceName` from managedDevice |
| Store / Location Number | e.g., Azure AD extension attribute, device category, or naming convention |
| Device Type / Role | e.g., POS device, shared iPad, kiosk — how do you distinguish these? |
| User Assignment | e.g., `primaryUser` from Intune, or Azure AD user mapping |

This mapping is what lets a store manager open Site Manager, tap a location, and immediately see every device assigned to that store with its current status. Without it, devices show up as a flat, unsorted list.

---

## 5. Operating System & Compliance Filtering

Most ienterprise deployments focus on iOS devices in retail environments, but Intune manages everything — iOS, Android, Windows, macOS. We need to know which platforms matter to you so we can filter the data and keep ienterprise dashboards clean and relevant.

| What We Need | Details |
|--------------|---------|
| Operating systems to include | e.g., iOS only, iOS + Windows, all platforms |
| Compliance policies | Which compliance policies should trigger alerts in ienterprise? |
| Sync frequency threshold | How many hours without a check-in should flag a device as "stale"? |

The `$filter` parameter on the `managedDevices` endpoint handles platform filtering at the API level, so we only pull what's relevant. This keeps API calls efficient and your dashboards focused.

---

## 6. Beta vs. v1.0 API Considerations

Microsoft Graph offers two API versions for Intune data: the stable **v1.0** and the **Beta**. Here's the trade-off and why it matters for your deployment:

| Capability | v1.0 (Stable) | Beta |
|-----------|---------------|------|
| Device list & details | Supported | Supported |
| Detected apps per device | Not supported (navigate from app) | Supported via `$expand` |
| Device health scripts | Not supported | Supported via `$expand` |
| User relationship data | Limited | Full support via `$expand` |
| Stability guarantee | Production-grade, no breaking changes | May change without notice |

ienterprise defaults to **v1.0** for production stability but can leverage Beta endpoints for richer data where your team is comfortable with the trade-off. We'll discuss this during setup and configure accordingly.

---

## What Happens Next

Once you send us the details above, here's the sequence:

| Step | What We Do | What You'll See |
|------|-----------|-----------------|
| 1 | Register & configure the integration | We set up the Graph API connection using your app registration details |
| 2 | Validate API connectivity | We confirm ienterprise can authenticate and pull device data from your tenant |
| 3 | Map device properties | Devices appear in Device Manager and Site Manager with correct store assignments |
| 4 | Test end-to-end data flow | We verify device details, app data, and compliance status render correctly |
| 5 | Confirm production readiness | You get a green light to go live |

**Timeline:** 3–5 business days from when we receive your Azure AD app registration details and property mappings. Most integrations go live in under a week.

---

## Quick Checklist

Here's everything we need, all in one place:

- [ ] **Azure AD Tenant ID** — Your IT / Azure Admin  
- [ ] **App Registration (Client ID + Secret)** — Your IT / Azure Admin  
- [ ] **Grant Admin Consent on API permissions** — Global or Intune Admin  
- [ ] **Device property mapping (store number, type)** — Your IT team  
- [ ] **Operating system filter preferences** — Your IT team  
- [ ] **Compliance policy alert preferences** — Your IT team  
- [ ] **Beta vs. v1.0 API preference** — Your IT team  

---

## Contact

**Questions?** Reach out to your ienterprise account team. We're happy to hop on a call and walk through any of this together.

**Contact:** support@ienterprise.com

---

**Confidential | ienterprise by Streamline LLC**
