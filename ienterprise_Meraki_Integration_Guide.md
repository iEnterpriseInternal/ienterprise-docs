# ienterprise | Cisco Meraki Integration Guide

## Network Visibility for Your Mobile Ecosystem  
### Configuration & Setup Requirements

**CONFIDENTIAL**  
Prepared by **ienterprise | Streamline LLC**

---

## What This Is About

Welcome to the ienterprise Meraki integration setup. This guide walks you through everything we need to connect your Cisco Meraki network infrastructure to ienterprise, giving you real-time visibility into every device on your network — from switches and access points to iPhones, iPads, printers, and payment terminals.

The integration is read-only. We pull data from your Meraki dashboard — we never change configurations or push commands to your network. Your infrastructure stays exactly the way you set it up.

---

## What This Integration Does

Once connected, ienterprise pulls network data from your Meraki environment and maps it against your device management platform (Jamf, Intune, etc.) to give you a unified picture of your mobile ecosystem. Here's what that unlocks:

- **Network health visibility** — see switch and access point status across every store, in real time.
- **Client device tracking** — identify iPhones, iPads, Macs, printers, and payment terminals connected to each network.
- **Peripheral monitoring** — track Adyen payment devices, Epson/Brother/HP printers, and WatchTower appliances on your network.
- **Infrastructure mapping** — map switches to access points using LLDP/CDP data to understand your physical network topology.
- **Cross-platform correlation** — match network clients to MDM-managed devices for complete device accountability.

---

## What We Need from You

There are two things we need to get this integration running: API access and a few details about your network setup. Let's walk through both.

---

## 1. Meraki API Access

We need a Meraki Dashboard API key with read-only access. This key lets ienterprise pull network, device, and client information from your Meraki organization. No write permissions are required.

You have two options:

- **Option A:** Generate the API key yourself and share it with us securely.
- **Option B:** Add our service account (`streamline_network_support@streamlinedigital.com`) to your Meraki dashboard as a read-only administrator, and we'll generate the key on our end.

### How to Generate a Meraki API Key

1. Log in to the Meraki Dashboard at dashboard.meraki.com.
2. Navigate to **Organization > Settings**.
3. Ensure **Dashboard API access** is enabled.
4. Go to your profile (click your name at the top right) and select **My profile**.
5. Under **API access**, click **Generate new API key**.
6. Copy the key and share it with your Streamline onboarding contact via a secure channel.

> **IMPORTANT: Read-Only Access**
> ienterprise only requires GET (read) access to your Meraki APIs. We never modify, create, or delete any network configuration. Your infrastructure is always under your control.

---

## 2. Network Configuration Details

To map your network data correctly, we need answers to the following questions. Your network or IT team will have this information readily available.

| # | Question | Your Response |
|---|----------|---------------|
| 1 | **How is the WiFi profile configured?** <br> Do you randomize MAC addresses? If yes, we'll need an alternative method to align network clients with MDM-managed devices. | |
| 2 | **What is your SSID name?** | |
| 3 | **What port setup do you have for peripherals?** <br> e.g., printers on ports 1–10, WatchTower on port 16 | |
| 4 | **What are the relevant VLANs to pull client info from?** <br> Please include IP ranges within each VLAN. | |
| 5 | **What is the network naming convention for store networks?** <br> How do you identify which network belongs to a specific store? | |
| 6 | **What peripheral types/names are on your network?** <br> e.g., Adyen P400, AMS1, Epson/Brother/HP printers | |

---

## How the Integration Works

ienterprise connects to the Meraki Dashboard API v1 to pull four categories of data. All API calls are `GET` requests — strictly read-only. Here's the breakdown:

| Data Category | API Endpoint | What We Collect |
|---------------|--------------|-----------------|
| Networks | `/api/v1/organizations/{orgId}/networks` | List of all networks in your Meraki organization, mapped to your store locations. |
| Network Devices | `/api/v1/organizations/{orgId}/devices/statuses?networkIds[]={id}` | Status of switches and access points across each network. |
| Network Clients | `/api/v1/organizations/{orgId}/networks/{netId}/clients` | All client devices: iPhones, iPads, Macs, printers, payment terminals, and WatchTower appliances. |
| Device Topology | `/api/v1/devices/{serial}/lldpCdp` | LLDP/CDP neighbor data for mapping physical connections between switches and access points. |

### Data Flow

The integration follows a straightforward sequence:

1. **Discover networks** — ienterprise pulls the full list of networks from your Meraki organization and maps them to store locations using your naming convention.
2. **Poll device status** — for each network, we check the online/offline status of every switch and access point.
3. **Identify clients** — we pull all connected clients and classify them by type (iOS device, printer, payment terminal, etc.) using MAC address, hostname, and VLAN data.
4. **Map topology** — LLDP/CDP data tells us which access points connect to which switches, building a picture of your physical network layout.
5. **Correlate with MDM** — network client data is matched against your MDM platform (Jamf, Intune) to link network presence to managed device records. This is where MAC address configuration matters.

---

## Security & Data Handling

We take data security seriously. Here's how we handle your Meraki integration data:

- **Read-only access.** All API calls are GET requests. ienterprise cannot modify your Meraki configuration.
- **Encrypted in transit.** All communication with the Meraki API uses HTTPS/TLS encryption.
- **Secure key storage.** Your API key is stored encrypted and is never exposed in logs or user interfaces.
- **Revocable at any time.** You maintain full control. Revoke the API key or remove our service account whenever you choose, and the integration stops immediately.
- **No personal data collection.** We collect device and network metadata only — no user browsing data, personal information, or traffic content.

---

## Onboarding Checklist

Use this checklist to make sure everything is in place before we activate the integration.

- [ ] Enable Dashboard API access in Meraki Organization Settings
- [ ] Generate API key **or** add `streamline_network_support@streamlinedigital.com` as read-only admin
- [ ] Share API key securely with your Streamline onboarding contact
- [ ] Provide WiFi profile details (MAC randomization, SSID name)
- [ ] Provide peripheral port assignments (printer ports, WatchTower port, etc.)
- [ ] Provide relevant VLANs and IP ranges
- [ ] Provide network naming convention for store identification
- [ ] Provide list of peripheral types and device names on the network

---

## Next Steps

Once you've provided the API key and answered the configuration questions above, here's what happens:

1. We configure the Meraki adapter in ienterprise using your API key and network details.
2. We run a validation pass to confirm we can pull data from all expected networks and device types.
3. We map your stores, so network data appears in the correct Site Manager and Enterprise Manager views.
4. We activate monitoring and your team starts seeing network health data alongside device management data in ienterprise.
5. We review together during a short walkthrough to make sure everything looks right and your team knows where to find the data.

---

## Questions?

If anything in this guide is unclear or you need help with any of the steps, reach out to your Streamline onboarding contact or email us at:

**Contact:** streamline_network_support@streamlinedigital.com

We're here to make this as smooth as possible. Most integrations are live within 48 hours of receiving the API key and configuration details.

---

**Confidential | ienterprise by Streamline LLC**
