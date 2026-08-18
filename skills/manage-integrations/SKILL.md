---
name: manage-integrations
description: Connect Voice Logica integrations, enable the matching agent tool, and test with a Call ID. Use when the user wants HubSpot, Shopify, Soft1, Google, SMS, helpdesk, OAuth reconnect, private ERP tunnels, or a private PBX edge device.
---

# Manage Voice Logica integrations

Connecting a company integration is not enough. The agent still has to be allowed to use it, then you prove it on a real call.

## The 3-layer rule

1. **Connect the integration** at company level (OAuth, API key, or tunnel / edge device).
2. **Enable the agent tool** on the specific agent (`list_agent_tools` / `update_agent_tool`, or the matching shop / CRM / calendar tool).
3. **Test with a Call ID.** Read `toolCalls` on that call. Do not declare success from the dashboard toggle alone.

If layer 1 is missing, the tool will not fire. If layer 2 is missing, the integration exists but this agent cannot use it. If layer 3 fails, fetch the Call ID before editing the prompt.

There is no per-integration plan lock. Failures are usually: not connected, `needsReconnect`, wrong role, missing tunnel, or tool not enabled.

Tool returns nothing / partial / error / timeout: confirm the record exists in the source system, then field mapping and permissions. Phone lookups often fail on landline vs mobile format — enable phone normalization and try order ID as a fallback.

Never paste live API tokens, refresh tokens, or client secrets into chat. Ask the user to paste them only into the Voice Logica integration form. Some provider tokens display **once** (Shopify Admin API) — if lost, regenerate; the old token dies.

## Public catalog (self-serve)

**CRM:** HubSpot, Zoho, Salesforce. See `manage-crm`.

**E-commerce:** WooCommerce, Shopify, OpenCart, Magento, PrestaShop, Megasoft, custom shop API. See `manage-ecommerce`. PrestaShop has no default voice-agent tool — mainly workflows / MCP.

**ERP:** Soft1, Galaxy Drugstore, Galaxy ERP, Pylon, Entersoft. See `manage-erp`. Cloud URL vs On-prem SSH tunnel.

**Calendar / contacts / chat:** Google Calendar is the agent **Appointments** tab (company Google login) — not an Integrations card. Google Meet, Google Chat, and Microsoft 365 are Integrations. See `manage-calendar-google-microsoft`.

**Messaging:** SMS senders, Viber. See `manage-sms`. These are messaging channels, not Integration keys.

**Helpdesk / PM / hospitality:** Freshdesk, Zendesk, Jira, ClickUp, Trello, Airtable, Guesty, Hosthub, WebHotelier. See `manage-helpdesk-ops`. Trello has no default agent tool.

**Couriers / pay / PBX API:** ACS, Speedex, Geniki, ELTA, BOX NOW, Skroutz Last Mile, Viva, Yuboto PBX. See `manage-helpdesk-ops`. Viva is mainly a workflow payment-link node. Yuboto PBX integration (domain + API key) is **not** a SIP phone and **not** an Edge Device — phone registration stays in `manage-phones`.

## Not self-serve

Do not try to add these from the customer Add Integration UI. Tell the user they need Voice Logica to enable them:

- Odoo
- Monday.com
- WhatsApp

Splynx is admin-only. See `manage-erp`.

## OAuth (`needsReconnect`)

If an OAuth integration shows `needsReconnect` (Google, Microsoft, HubSpot, Shopify, Salesforce, and similar):

1. Disconnect the integration.
2. Reconnect and finish the provider consent screen.
3. Re-enable the agent tool if it dropped.
4. Test with a new Call ID.

Never "fix" a dead OAuth token by editing the system prompt.

## Private systems — do not swap these

Two different bridges. Using the wrong one will not work.

**Private ERP / HTTP API** (on-prem Soft1, custom HTTP, inventory behind a firewall):

- Path: **Tunnels (SSH)** — menu is **admin-only**
- The agent or workflow calls an HTTP URL that Voice Logica reaches through the SSH tunnel.

**Private PBX / SIP / RTP** (on-prem phone system, local extensions):

- Path: **Phones → Edge Devices**
- Transport: **WireGuard UDP 51820**
- Tools: `get_edge_devices`, `set_edge_device_forward`, `remove_edge_device_forward`
- Phone CRUD stays in `manage-phones`.

Do not put a private PBX on an SSH tunnel. Do not put a private HTTP ERP on an Edge Device. A clinic/pharmacy often needs **both** (Edge for calls, tunnel for ERP).

## How to work

1. List agents. Match by name. Use the real ID.
2. Identify the integration family and open that specialist skill.
3. Confirm the company connection, then the agent tool, then a Call ID.
4. After a write, say what is connected, which agent can use it, and what to test.
