# iEnterprise | ServiceNow Integration Guide

## Native Ticketing Integration  
### Configuration & Setup Requirements

**CONFIDENTIAL**  
Prepared by **iEnterprise | Streamline Digital**

---

## What This Is About

This guide walks you through connecting iEnterprise to your ServiceNow instance. The goal is simple: enable store teams to create and track support tickets from **Site Manager** without logging into ServiceNow directly.

We require a small set of details from your team. Once provided, we handle the rest.

- No custom development inside ServiceNow  
- No workflow changes  
- Clean, secure connection between systems  

**Typical setup time:** 3–5 business days after credentials and mappings are received.

---

## 1. Connection & Authentication

To establish secure communication with your ServiceNow instance, we need the following:

### Required Details

- **Instance Base URL**  
  Example: `https://yourcompany.service-now.com`

- **API Username**  
  Dedicated integration user account

- **API Password**  
  Password for the integration account

- **Required Roles**  
  - Incident create/read access  
  - Attachment access  

### Recommendation

Use a **non-human service account**, not a personal login. A dedicated API user aligned with enterprise security policies avoids disruptions caused by password changes or employee transitions.

---

## 2. API Access Requirements

The integration uses standard ServiceNow REST APIs only.

### Required Permissions

| Action | Table | Access Level |
|------|------|-------------|
| Create incidents | incident | Create |
| Read incidents | incident | Read |
| Upload attachments | sys_attachment | Create |

No workflow updates. No scripting. Existing ServiceNow behavior remains unchanged.

---

## 3. Channel Configuration

### Required Change in ServiceNow

Add a new value to the **Channel** dropdown field:

| Field | New Value |
|------|-----------|
| Channel | Site Manager |

### Why This Matters

ServiceNow reporting relies heavily on ticket origin channels such as Phone, Email, or Portal.

Without a dedicated **Site Manager** channel:
- Tickets appear as generic API traffic
- Adoption metrics are lost
- ROI cannot be measured accurately

This is a single dropdown value change. It does not affect routing, automation, or workflows.

---

## 4. Category & Subcategory Mapping

Each ServiceNow instance uses unique Category and Subcategory values.

### What We Need
- Exact Category and Subcategory values from your ServiceNow configuration

### Why It Matters
- Tickets align with IT classification standards
- No misclassified incidents
- No manual cleanup

---

## 5. Ticket Status Mapping

Store teams expect real-time ticket status updates.

Provide status names and internal integer IDs.

| Status Name | Internal Value |
|------------|----------------|
| New | |
| In Progress | |
| On Hold | |
| Resolved | |
| Closed | |

---

## 6. Store / Location Field Mapping

Retail tickets must be tied to a physical location.

- Store Number field name
- Field type (standard location or custom)

---

## What Happens Next

1. Configure integration  
2. Validate API connectivity  
3. Test ticket lifecycle  
4. Confirm production readiness  

**Timeline:** 3–5 business days

---

## Quick Checklist

- ServiceNow instance URL  
- API user credentials  
- Channel value added  
- Category & Subcategory values  
- Status IDs  
- Store/location mapping  

---

**Confidential | iEnterprise by Streamline Digital**
