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

1. Connect the ERP integration on the company. Choose **Cloud URL** (public HTTPS) or **On-prem tunnel**.
2. Enable the matching agent tool (`list_agent_tools`). Documented names: `soft1_erp`, `galaxy_drugstore`, `galaxy_erp`, `pylon_erp`, `entersoft_one`. Read the live list — do not invent.
3. Test with a Call ID and read `toolCalls`.

Never paste live ERP API tokens into chat.

## Cloud vs tunnel

If the ERP is already on public HTTPS, use Cloud URL — no tunnel.

If it is only on a private LAN (`192.168.x.x` with no public HTTPS), use **Tunnels (SSH)**:

1. Admin opens **AI → Tunnels** (menu is **admin-only**).
2. Create tunnel: name, local target host:port.
3. Site IT installs the tunnel agent (Windows service / Linux systemd). Outbound SSH to the gateway.
4. Integration → provider → mode **On-prem tunnel** → pick that tunnel → Test / Sync.
5. Enable the agent ERP tool.

This is not the path for a private PBX. SIP / RTP goes to **Phones → Edge Devices** (WireGuard UDP 51820). See `manage-integrations` and `manage-phones`. Do not swap them. A pharmacy/clinic often needs **both**.

Tunnel checklist: status up, site service running, outbound SSH allowed, local host:port correct, integration mode = tunnel + correct tunnel selected, Test/Sync succeeds. Timeouts with tunnel up usually mean the ERP itself is slow.

## Per-ERP fields (company connection)

- **Soft1:** username / password / `appId` (often default **2001**). Wizard: discover company/branch. Failures: inactive series, missing SODTYPE permissions, tunnel required, auth fail.
- **Galaxy Drugstore:** URL or tunnel to LAN host:port (e.g. `192.168.x.x:8080`), user/pass, company, warehouses. Distinct from Galaxy ERP.
- **Galaxy ERP:** `/api/glx/*` session. URL/tunnel + user/pass (+ optional company).
- **Pylon:** Connectivity API URL/tunnel + apiCode / application / DB / user / pass + entity codes.
- **Entersoft ONE:** URL/tunnel, userId / password / branch / lang (+ optional subscription).

## Common jobs

**"Which ERP is connected?"**
List company integrations and the agent's tools. Say what is connected vs what this agent can use.

**"On-prem Soft1 / custom HTTP."**
Confirm a tunnel exists, the URL is the tunneled host (not a public guess), then enable the tool and test with a Call ID.

**"The agent invented stock / invoice data."**
Get a Call ID. If the ERP tool never ran, enable it and add a hard prompt rule: never invent ERP success. If it ran and failed, fix the connection or tunnel, not the model.

## After a change

Say which ERP, cloud vs tunnel, and which agent can use it. Ask for a test Call ID.
