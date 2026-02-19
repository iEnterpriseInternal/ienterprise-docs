# ienterprise | Network Access Configuration Guide

## Endpoint Allowlisting  
### Firewall & Proxy Configuration Requirements

**CONFIDENTIAL**  
Prepared by **ienterprise | Streamline LLC**

---

## What This Is About

This guide walks network and security teams through the firewall and proxy configuration required to connect to the ienterprise platform.

The objective is simple: **allow outbound HTTPS access to three specific endpoints** so the platform operates without interruption.

- No inbound ports required  
- No agents to install  
- Minimal allowlist changes  

**Typical turnaround:** Same day once rules are applied and propagated.

---

## 1. Required Endpoints

Allowlist the following Fully Qualified Domain Names (FQDNs) for outbound HTTPS traffic.

| Endpoint (FQDN) | Port | Purpose |
|-----------------|------|---------|
| ralphlauren.ienterprise.com | 443 | Production environment |
| ralphlauren.staging.ienterprise.info | 443 | Staging / UAT environment |
| sitemanager.blob.core.windows.net | 443 | Azure Blob Storage for static assets |

**Protocol:** HTTPS (TLS 1.2 or higher)  
**Direction:** Outbound only  
**Inbound access:** Not required

---

## 2. Configuration Steps

### Firewall Rules

1. Log in to your firewall management console (Palo Alto, Fortinet, Cisco ASA, etc.).  
2. Navigate to outbound security or URL filtering policies.  
3. Create or update allow rules for each FQDN listed above.  
4. Set destination port to **443 (TCP)**.  
5. Apply rules to all relevant source zones and user groups.  
6. Commit and push the configuration.

### Web Proxy / SSL Inspection

If using a forward proxy or SSL inspection solution (Zscaler, Bluecoat, etc.):

- Add all three FQDNs to the proxy allowlist  
- Bypass SSL inspection for these domains  
- Ensure no custom CA certificate is injected into the TLS handshake  

### DNS Resolution

- Confirm internal DNS resolvers can resolve all three FQDNs  
- If using DNS filtering (Umbrella, Infoblox), add these domains to the allow policy  

---

## 3. Verification

Run the following tests from a corporate-network-connected machine:

| Test | Expected Result |
|-----|-----------------|
| Browse to https://ralphlauren.ienterprise.com | Page loads successfully |
| Browse to https://ralphlauren.staging.ienterprise.info | Page loads successfully |
| nslookup sitemanager.blob.core.windows.net | Resolves to valid Azure IP |

If a test fails, review firewall or proxy logs for blocked requests.

---

## 4. Troubleshooting

- **Connection timeout:** Verify firewall rules are committed and scoped correctly  
- **SSL or certificate error:** SSL inspection may be intercepting traffic; bypass inspection  
- **DNS resolution failure:** Confirm domains are not blocked by DNS security filtering  
- **Broken or partial page load:** Azure Blob Storage endpoint may be blocked  

---

## 5. Support

If issues persist, contact the ienterprise support team with:

- The unreachable FQDN  
- Error message or screenshot  
- Relevant firewall or proxy log entries  

---

**Confidential | ienterprise by Streamline LLC**
