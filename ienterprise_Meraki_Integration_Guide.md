# ienterprise | Cisco Meraki Integration Guide

## Network Visibility for Your Mobile Ecosystem  
### Configuration & Setup Requirements

**CONFIDENTIAL**  
Prepared by **ienterprise | Streamline LLC**

---

## What This Is About

Welcome to the ienterprise Meraki integration setup. This guide walks you through everything required to connect your Cisco Meraki network infrastructure to ienterprise, providing real-time visibility into devices across your network, including switches, access points, iPhones, iPads, printers, and payment terminals.

This integration is read-only. ienterprise pulls data from your Meraki Dashboard and never changes configurations or pushes commands. Your network remains fully under your control.

---

## What This Integration Does

Once connected, ienterprise pulls network data from Meraki and correlates it with your device management platform (Jamf, Intune, etc.) to provide a unified operational view.

Key capabilities include:

- Network health visibility across stores  
- Client device tracking (iOS, macOS, printers, payment terminals)  
- Peripheral monitoring (Adyen, Epson/Brother/HP, WatchTower)  
- Infrastructure mapping using LLDP/CDP  
- Cross-platform correlation with MDM systems  

---

## What We Need from You

Two inputs are required to activate the integration:

1. Meraki API access  
2. Network configuration details  

---

## 1. Meraki API Access

ienterprise requires a Meraki Dashboard API key with read-only access.

### Access Options

- Generate and share an API key securely  
- Add `streamline_network_support@streamlinedigital.com` as a read-only administrator

### How to Generate a Meraki API Key

1. Log in to dashboard.meraki.com  
2. Go to Organization > Settings  
3. Enable Dashboard API access  
4. Open My Profile  
5. Generate a new API key  
6. Share securely with Streamline  

Only GET (read-only) permissions are required.

---

## 2. Network Configuration Details

| # | Question |
|---|---------|
| 1 | Is MAC randomization enabled on WiFi? |
| 2 | SSID name |
| 3 | Peripheral port assignments |
| 4 | VLANs and IP ranges |
| 5 | Network naming convention |
| 6 | Peripheral device types |

---

## How the Integration Works

ienterprise connects to the Meraki Dashboard API v1 using read-only endpoints.

### Data Collected

| Category | Endpoint | Description |
|--------|----------|-------------|
| Networks | /organizations/{orgId}/networks | Store network list |
| Devices | /devices/statuses | Switch/AP status |
| Clients | /networks/{netId}/clients | Connected clients |
| Topology | /devices/{serial}/lldpCdp | Physical topology |

---

## Security & Data Handling

- Read-only access  
- Encrypted communication  
- Secure key storage  
- Key revocable at any time  
- No personal data collected  

---

## Next Steps

1. Configure integration  
2. Validate connectivity  
3. Map stores  
4. Activate monitoring  
5. Review with your team  

Most integrations are live within 48 hours.

---

## Contact

Email: streamline_network_support@streamlinedigital.com

---

**Confidential | ienterprise by Streamline LLC**
