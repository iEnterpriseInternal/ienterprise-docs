# ienterprise | Omnissa Integration Guide

## Device Management Integration  
### Configuration & Setup Requirements

**CONFIDENTIAL**  
Prepared by **ienterprise | Streamline LLC**

---

## A Quick Note on Naming

**Omnissa** is the current name of the platform your team may still know as **VMware Workspace ONE UEM** or **AirWatch**. The console, the REST API, and the endpoints are unchanged from the AirWatch / Workspace ONE era — only the company branding changed. This guide says "Omnissa Workspace ONE UEM"; use whichever name matches your console login.

---

## What This Is About

This guide walks you through connecting ienterprise to your Omnissa Workspace ONE UEM environment. The goal is straightforward: give your IT teams and store managers real-time visibility into managed devices, applications, and compliance directly from ienterprise's **Device Manager** and **Site Manager** — and let store staff run everyday device actions without opening the UEM console.

Everything you need to do happens inside your own console. There's a short list of details to send back to us at the end, and then we handle the rest.

- No custom development inside Omnissa
- No changes to your existing device policies or profiles
- A dedicated, tightly-scoped API client that you can audit or revoke at any time

**Typical setup time:** 3–5 business days after we receive your OAuth credentials and the details in Section 9.

---

## What We Need from You — The Short Version

If you read nothing else, these are the five things that unblock the integration:

| # | What We Need | Why It Matters |
|---|--------------|----------------|
| 1 | An **OAuth 2.0 client** (Client ID + Secret) and the regional token URL | This is how ienterprise authenticates. See Section 2. |
| 2 | An API role that includes **Apps (Read)**, **Groups (Read)**, and **Compliance Policy (Read)** | Without these, the device page shows no installed apps, no group membership, and no compliance state. See Section 3. |
| 3 | The **Organization Group name and numeric Group ID** the client is scoped to | Determines which devices ienterprise can see, and how devices map to stores. |
| 4 | Your **Server Throttling** and **Daily Quota** values, plus whether another integration shares that Organization Group | Quota is enforced per Organization Group, not per client. See Section 5. |
| 5 | How **store identity** is modelled in your tenant — one Organization Group per store, or a custom attribute? | Changes how we build your store hierarchy. See Section 7. |

Everything below is the detail behind these five asks.

---

## 1. Before You Start

Confirm the following before you begin:

- You have an active **Omnissa Workspace ONE UEM** tenant, and the devices you want in ienterprise are already **enrolled** under the Organization Group you plan to scope this integration to.
- You have console access with **System Administrator** (or equivalent) privileges at the **top-level (Customer) Organization Group**. OAuth Client Management is only available from the top of the Organization Group hierarchy — not from a child group.
- Your Workspace ONE UEM environment is **version 2001 or later** (required for OAuth client credentials).
- You know the **number of enrolled devices** in that Organization Group and its children.
- You know your tenant **region** — NA, EMEA, or APAC.

---

## 2. Create an OAuth 2.0 Client

ienterprise connects to your Workspace ONE UEM REST API using an **OAuth 2.0 client credentials** grant — a Client ID and Secret exchanged for a bearer token. No separate REST API key is required.

**Where to find it:** OAuth Client Management is *not* under System → Advanced → API. It lives in its own area of the console and generally requires being at the top-level Customer Organization Group to see it.

### Steps

1. Go to **Groups & Settings → Configurations**, type `OAuth` into the search box, and select **OAuth Client Management**.
2. Click **Add**.
3. Fill in the **Register New Client** fields:

   | Field | What to Enter |
   |-------|---------------|
   | **Name** | e.g. `ienterprise-mdm-integration` |
   | **Description** | Free text — we recommend noting who created it and when |
   | **Organization Group** | **Required.** Must contain every device ienterprise should manage (or be a parent of those groups) |
   | **Role** | The role you create in Section 3 |
   | **Status** | Enabled |

4. Click **Save**.

> **Important:** The **Client Secret is shown only once.** Copy both the Client ID and Secret immediately and store them securely. If the secret is lost, the client has to be deleted and recreated — there is no way to retrieve it again.

### Your Regional Token Endpoint

Note the OAuth token endpoint that matches your tenant's region:

| Region | Token Endpoint |
|--------|----------------|
| NA | `https://na.uemauth.workspaceone.com/connect/token` |
| EMEA | `https://emea.uemauth.workspaceone.com/connect/token` |
| APAC | `https://apac.uemauth.workspaceone.com/connect/token` |

> The older `*.uemauth.vmwservices.com` hosts are legacy and being retired. Use the `workspaceone.com` hosts above.

### While You're in the Console

Go to **Groups & Settings → All Settings → System → Advanced → API → REST API** and open the **Usage** tab. Note the **Server Throttling** (per-minute) and **Daily Quota** values — we need them for Section 5.

---

## 3. Create the API Role & Permissions

Rather than assigning a built-in full-access role, create a dedicated role scoped to only what ienterprise needs.

1. Go to **Accounts → Administrators → Admin Roles → Add Role**.
2. Name it **ienterprise Integration API Role** (or similar) and add a description.
3. Under **Categories → API → REST**, set the **Read** and **Edit** boxes as shown below.
4. Assign this role to the OAuth client you created in Section 2.

### 3.1 REST API Permissions

| Resource | Read | Edit | Why ienterprise Needs It |
|----------|------|------|--------------------------|
| **Devices** | ✅ | ✅ | Device search and details, plus commands — sync, lock, unlock, restart, lost mode, wipe |
| **Custom Attributes** | ✅ | ✅ | Read and write device attributes such as check-in/check-out state, device type, and store number |
| **Apps** | ✅ | — | Installed-app inventory per device and the managed-app catalog. Powers the Applications panel and the app-out-of-date check |
| **Groups** | ✅ | — | Smart Group membership and the Organization Group hierarchy used to map devices to stores |
| **Compliance Policy** | ✅ | — | Per-device compliance state, shown on the device page |
| **Profiles** | ✅ | — | Reports profile assignment and compliance status. Read only — we never write profiles |
| **Users** | ✅ | — | Resolves the user or account associated with a device. Read only — we never write user records |
| **Products** | — | — | Not used by this integration |
| **REST Enterprise Integration** | — | — | Only needed if you'd prefer ienterprise to manage Event Notification subscriptions by API instead of in the console (Section 6) |

> **Why Apps, Groups, and Compliance Policy matter more than they look.** The single bulk endpoint ienterprise relies on returns app data, group membership, and compliance state **inline in one response**. With those three grants in place, we read your whole fleet in a handful of calls per cycle. Without them, the same data would require one call *per device* — which no API quota can absorb. These are read-only grants and add no write surface.

### 3.2 Device Command Sub-Permissions

**The top-level Devices Read/Edit checkboxes are not enough on their own.** Click **Details** on the **Devices** row to reveal a per-command list under **Device Details**. Each command needs its own checkbox:

| ienterprise Action | Required Sub-Permission (under Device Details) |
|--------------------|-----------------------------------------------|
| Restart | **General → Device Remote Reboot** (Edit) |
| Unlock (clear passcode) | **Passcode** (Edit) |
| Enable / disable Lost Mode | **Find Device** (Edit) |
| Retire / un-enrol a device | **Enterprise Wipe** (Edit) |

> **A decision for you on Retire.** The Retire action appears in the ienterprise device page and in Site Manager. Without the **Enterprise Wipe** sub-permission, it fails with a permission error the moment a store user presses it. If your policy is that ienterprise operators should *not* be able to retire devices, leave it unchecked and tell us — we'll hide the action rather than let it fail.

If any other device action returns a permission error during testing, check for a matching named sub-permission under this same **Device Details** list. The top-level Edit checkbox does not cover individual commands.

---

## 4. What ienterprise Connects To

All device inventory is read through **one bulk endpoint**. Per-device calls happen only when a person opens a specific device on screen. Use this table to sanity-check your role once it's built.

| ienterprise Capability | Endpoint | Method | Governing Permission |
|------------------------|----------|--------|----------------------|
| **Bulk device inventory** (reporting, dashboards, store device lists) | `/API/mdm/devices/extensivesearch` | GET | Devices, Custom Attributes, Groups, Compliance Policy (Read) |
| Single device record | `/API/mdm/devices/{id}` | GET | Devices (Read) |
| Installed apps on a device | `/API/mdm/devices/{uuid}/apps/search` | GET | Apps (Read) |
| Managed app catalog | `/API/mam/apps/search` | GET | Apps (Read) |
| Configuration profiles on a device | `/API/mdm/devices/{id}/profiles` | GET | Profiles (Read) |
| Certificates on a device (expiry check) | `/API/mdm/devices/{id}/certificates` | GET | Devices (Read) |
| Smart Group membership and definitions | `/API/mdm/devices/{id}/smartgroups`, `/API/mdm/smartgroups/search` | GET | Groups (Read) |
| Compliance state for a device | `/API/mdm/devices/{id}/compliance` | GET | Compliance Policy (Read) |
| Organization Group hierarchy (device → store) | `/API/system/groups/search` | GET | Groups (Read) |
| Device event log (history panel) | `/API/mdm/devices/{id}/eventlog` | GET | Devices (Read) |
| Check-in / check-out and attribute writes | `/API/mdm/devices/serialnumber/{serial}/customattributes` | POST | Custom Attributes (Edit) |
| Commands — sync, query, lock, restart, shutdown, wipe | `/API/mdm/devices/{id}/commands?command={cmd}` | POST | Devices (Edit) + the matching sub-permission |
| Lost Mode enable / disable | `/API/mdm/devices/{uuid}/lostmode/{true\|false}` | PUT | Devices (Edit) → Find Device |

You can confirm the exact endpoint contract for your own environment at `https://<your-tenant-host>/API/help`, which is generated from your tenant's actual API version.

---

## 5. API Throttling & Daily Quota

This section matters more than it might appear, because it's the one thing that can make a correctly-permissioned integration fail in production. **API quota in Workspace ONE UEM is enforced per Organization Group, not per client** — so every integration pointed at the same group draws from one shared ceiling, and hitting the daily quota is a hard mid-day stop, not a slowdown.

### How ienterprise Uses Your Quota

ienterprise keeps a refreshed copy of device inventory to power reporting, dashboards, and store device lookups. We read it in bulk, sequentially, with backoff — never as a burst against your throttle.

| | Value |
|---|---|
| Refresh cadence | Every **30 minutes** |
| Page size | 500 devices per call |
| Calls per day at 20,000 devices | ~1,900 |
| Calls per day at 30,000 devices | ~2,900 |
| Share of the default 50,000/day quota | **4–6%** |

Default tenant limits are **5,000 calls/minute** (Server Throttling) and **50,000 calls per 24 hours** (Daily Quota), both per Organization Group.

The trade-off of a 30-minute cadence is that device records may be up to 30 minutes old. Section 6 is how we close that gap without polling harder.

### Our Commitments

1. Bulk inventory comes from **one endpoint** — device details, custom attributes, group membership, compliance state, and user data in a single response.
2. **Per-device calls are for interactive use only.** When someone opens a device on screen, we fetch that device's apps, certificates, profiles, and event log on demand — a handful of calls per human action, never per device per cycle.
3. App and Smart Group catalogs are read on their own low cadence (hourly or daily), not per device.
4. We page sequentially with backoff and will not parallelise into a burst.

### What We Need from You

- Your current **Server Throttling** and **Daily Quota** values on the scoped Organization Group.
- Your **enrolled device count** in that group, and whether it's expected to grow materially.
- **Whether any other integration draws on the same Organization Group's quota.** Our share is small, but it isn't the only claim on the ceiling.
- If the Daily Quota is at the 50,000 default, we'd like a **raise to 150,000/day** for this group. We don't need it for the bulk feed — it's headroom for interactive peaks, command traffic, and shared-quota risk. If a raise isn't possible, that's workable: tell us the ceiling and we'll hold the cadence and rate-limit interactive traffic to fit.
- Whether you'd prefer the OAuth client **scoped to a narrower Organization Group** — store-deployed iOS devices only, for example. Narrowing scope reduces both call volume and the data we hold, and is the cleanest lever if quota is tight.

---

## 6. Event Notifications (Webhooks) — Recommended

Workspace ONE UEM can push device change events to ienterprise instead of waiting for our next poll. This closes the staleness gap from Section 5 — device state becomes near-real-time instead of up to 30 minutes old, with no extra polling load. It's optional; the integration works on polling alone.

Setup is entirely in your console and doesn't affect the API role:

1. Go to **Groups & Settings → All Settings → System → Advanced → API → Event Notifications**.
2. Select **Add Rule**.
3. Enter the **Target Name** and the **Target URL** your ienterprise contact provides.
4. Set **Format** to **JSON**, then use **Test Connection** to confirm reachability.
5. Subscribe to the device lifecycle events — enrolment, un-enrolment, device attribute or compliance change, and device deleted.
6. Note the **Inherit vs Override** choice: to apply the rule to a child Organization Group rather than the whole hierarchy, choose **Override** on that group.

We'll supply the target URL and its authentication details.

---

## 7. Store Identity Mapping

ienterprise is a store-operations tool — nearly every screen is scoped to a store, and device lists, tickets, and Site Manager permissions all key off a **store identifier** on the device.

Workspace ONE UEM has two plausible homes for that, and which one your tenant uses changes what we read:

| Your Model | What It Means | What We Read | What It Needs |
|------------|---------------|--------------|---------------|
| **One Organization Group per store** | Each store is its own group under a shared parent | The group hierarchy, plus each device's `LocationGroupId` | **Groups (Read)**, and the OAuth client scoped to the **parent** group so it can see every store beneath it. Store name and address come from the group record. |
| **Custom attribute on the device** | Store number or name held in a per-device custom attribute | The custom attributes already returned by the bulk feed | **Custom Attributes (Read)** — already covered. We build the store list from the distinct attribute values across devices. |
| **Both** | Groups for hierarchy, attributes for detail (address, region, area) | Both of the above | Both grants. This is the most common arrangement. |

**Please tell us which model applies.** If custom attributes are involved, send us the **exact attribute names as spelled in the console** — they are case- and space-sensitive.

For reference, the store fields ienterprise consumes are: store number, store name, country, region, area, street address, city, state/province, and postal code.

We also need the attribute name to use for **check-in / check-out state**. Our default is a single attribute named `checkin_checkout` in application group `iEnterprise` — if you'd prefer a different name or group, tell us now. It's a configuration value on our side, not a code change.

---

## 8. Network Requirements

ienterprise calls your Workspace ONE UEM tenant **outbound**. No inbound firewall changes are required on your side. Make sure the following are reachable:

| Endpoint | Port | Purpose |
|----------|------|---------|
| `<your-tenant>.awmdm.com` (or your console's API host) | 443 (HTTPS) | All Workspace ONE UEM REST API calls |
| Your regional token host, e.g. `na.uemauth.workspaceone.com` | 443 (HTTPS) | OAuth 2.0 token endpoint |

If you adopt Event Notifications (Section 6), your tenant will additionally make **outbound** calls to the ienterprise target URL over 443. That's the only direction change — we never require an inbound path into your network.

If your tenant restricts API access by source IP, tell us and we'll provide the egress addresses of our integration service.

---

## 9. What to Send Us

Once the steps above are complete, share the following with your ienterprise implementation contact. **Use a secure transfer method — not plain-text email.**

| # | Item | Where to Find It | Required |
|---|------|------------------|----------|
| 1 | **API base URL** | Your console host, e.g. `https://<your-tenant>.awmdm.com/API` — verify at `https://<your-tenant-host>/API/help` | Yes |
| 2 | **Client ID** and **Client Secret** | Section 2 — copy the secret at creation, it's shown once | Yes |
| 3 | **Tenant region** (NA / EMEA / APAC) and matching **token URL** | Section 2 | Yes |
| 4 | **Organization Group name and numeric Group ID** | Groups & Settings → Groups → Organization Groups → Details | Yes |
| 5 | **Server Throttling** and **Daily Quota** values on that group | System → Advanced → API → REST API → Usage tab | Yes |
| 6 | **Enrolled device count** in that group and its children | Devices → List View | Yes |
| 7 | **Store identity model** and, if applicable, exact **custom attribute names** | Section 7 | Yes |
| 8 | Confirmation the role includes **Apps / Groups / Compliance Policy (Read)** — and **Enterprise Wipe**, if Retire is in scope | Section 3 | Yes |
| 9 | Workspace ONE UEM **environment / console version** | Console footer or About page | Helpful |
| 10 | Name and contact of a console admin we can reach during connectivity testing | — | Helpful |

---

## 10. What You Get at Go-Live

Assuming the access in this guide is granted:

| Capability | At Go-Live |
|------------|------------|
| Device inventory, search, and device detail page | Yes |
| Device commands — sync, query, lock, unlock, restart, shutdown, wipe | Yes |
| Lost Mode enable / disable | Yes |
| Installed apps, certificates, profiles, and group membership on the device page | Yes — requires the Apps and Groups grants |
| Certificate-expiry and app-version diagnostics | Yes — requires the Apps grant |
| Device event log / history panel | Yes |
| Check-in / check-out via custom attribute | Yes — requires Custom Attributes (Edit) |
| Store-scoped device lists and Site Manager store views | Yes — depends on Section 7 being resolved |
| Bulk inventory into ienterprise reporting and dashboards | Yes — 30-minute refresh |
| Retire / Enterprise Wipe from the portal | Only if the Enterprise Wipe sub-permission is granted |
| macOS and Windows endpoints alongside iOS | Follow-on phase |
| Event-driven (webhook) inventory updates | Once Section 6 is configured |

---

## What Happens Next

Once you send us the details in Section 9, here's the sequence:

| Step | What We Do | What You'll See |
|------|-----------|-----------------|
| 1 | Configure the integration | We set up the OAuth connection using your Client ID and Secret |
| 2 | Validate API connectivity | We confirm ienterprise can authenticate and read device data from your Organization Group |
| 3 | Map store identity | Devices appear in Device Manager and Site Manager under the correct stores |
| 4 | Test end-to-end | We verify device details, apps, compliance status, and device commands all work |
| 5 | Confirm production readiness | You get a green light to go live |

**Timeline:** 3–5 business days from when we receive your credentials and store mapping. Most integrations go live in under a week.

---

## Quick Checklist

Everything you need to do, in one place:

- [ ] **Create the OAuth 2.0 client** and copy the Client ID + Secret — Your UEM Admin
- [ ] **Note your regional token URL** (NA / EMEA / APAC) — Your UEM Admin
- [ ] **Create the API role** with Devices, Custom Attributes, Apps, Groups, Compliance Policy, Profiles, Users — Your UEM Admin
- [ ] **Check the device command sub-permissions** under Devices → Details — Your UEM Admin
- [ ] **Decide on Retire / Enterprise Wipe** — grant it or tell us to hide the action — Your IT Policy Owner
- [ ] **Assign the role to the OAuth client** — Your UEM Admin
- [ ] **Record Server Throttling and Daily Quota** values, and any shared-quota integrations — Your UEM Admin
- [ ] **Confirm the Organization Group name, Group ID, and device count** — Your UEM Admin
- [ ] **Tell us your store identity model** and exact custom attribute names — Your IT team
- [ ] **Set up Event Notifications** (optional, recommended) — Your UEM Admin
- [ ] **Send everything in Section 9 securely** — Your IT team

---

## Contact

**Questions?** Reach out to your ienterprise account team. We're happy to hop on a call and walk through any of this together.

**Contact:** support@ienterprise.com

---

**Confidential | ienterprise by Streamline LLC**
