---
name: manage-erp
description: Connect Soft1, Galaxy Drugstore, Galaxy ERP, Pylon, or Entersoft, and route private HTTP ERPs through SSH tunnels. Use when the user wants ERP lookup, stock, invoices, or an on-prem HTTP API on a voice agent.
---

# Manage Voice Logica ERP

Self-serve ERPs: **Soft1**, **Galaxy Drugstore**, **Galaxy ERP**, **Pylon**, **Entersoft**.

## Not in the customer Add UI

- **Odoo** is not self-serve. Do not try to add it from the Add Integration UI.
- **Splynx** is admin-only. A normal user cannot enable it. Say so and stop.

## 3-layer rule

1. Connect the ERP integration on the company.
2. Enable the matching agent tool or workflow `apiCall`.
3. Test with a Call ID and read `toolCalls`.

Never paste live ERP API tokens into chat.

## Private HTTP ERPs

If the ERP is only reachable on a private network, use **Tunnels (SSH)**. The agent or workflow then calls an HTTP URL through that tunnel.

This is not the path for a private PBX. SIP / RTP goes to **Phones → Edge Devices** (WireGuard UDP 51820). See `manage-integrations` and `manage-phones`. Do not swap them.

## Common jobs

**"Which ERP is connected?"**
List company integrations and the agent's tools. Say what is connected vs what this agent can use.

**"On-prem Soft1 / custom HTTP."**
Confirm a tunnel exists, the URL is the tunneled host (not a public guess), then enable the tool and test with a Call ID.

**"The agent invented stock / invoice data."**
Get a Call ID. If the ERP tool never ran, enable it and add a hard prompt rule: never invent ERP success. If it ran and failed, fix the connection or tunnel, not the model.

## After a change

Say which ERP, tunnel or not, and which agent can use it. Ask for a test Call ID.
