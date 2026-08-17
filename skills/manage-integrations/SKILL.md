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

## Public catalog (self-serve)

**CRM:** HubSpot, Zoho, Salesforce. See `manage-crm`.

**E-commerce:** WooCommerce, Shopify, OpenCart, Magento, PrestaShop, Megasoft, custom shop API. See `manage-ecommerce`.

**ERP:** Soft1, Galaxy Drugstore, Galaxy ERP, Pylon, Entersoft. See `manage-erp`.

**Calendar / contacts / chat:** Google Calendar, Google Contacts, Google Chat, Microsoft / Outlook calendar and contacts. See `manage-calendar-google-microsoft`.

**Messaging:** SMS senders, Viber. See `manage-sms`.

**Helpdesk and ops:** Freshdesk, Zendesk, Jira, ClickUp, Trello, Airtable, Guesty, couriers, Viva, Yuboto. See `manage-helpdesk-ops`.

## Not self-serve

Do not try to add these from the customer Add Integration UI. Tell the user they need Voice Logica to enable them:

- Odoo
- Monday.com
- WhatsApp

Splynx is admin-only. See `manage-erp`.

## OAuth (`needsReconnect`)

If an OAuth integration shows `needsReconnect` (Google, Microsoft, HubSpot, and similar):

1. Disconnect the integration.
2. Reconnect and finish the provider consent screen.
3. Re-enable the agent tool if it dropped.
4. Test with a new Call ID.

Never "fix" a dead OAuth token by editing the system prompt. Never paste live API tokens, refresh tokens, or client secrets into chat. Ask the user to paste them only into the Voice Logica integration form, then confirm the connection status from MCP.

## Private systems — do not swap these

Two different bridges. Using the wrong one will not work.

**Private ERP / HTTP API** (on-prem Soft1, custom HTTP, inventory behind a firewall):

- Path: **Tunnels (SSH)**
- The agent or workflow calls an HTTP URL that Voice Logica reaches through the SSH tunnel.

**Private PBX / SIP / RTP** (on-prem phone system, local extensions):

- Path: **Phones → Edge Devices**
- Transport: **WireGuard UDP 51820**
- Tools: `get_edge_devices`, `set_edge_device_forward`, `remove_edge_device_forward`
- Phone CRUD stays in `manage-phones`.

Do not put a private PBX on an SSH tunnel. Do not put a private HTTP ERP on an Edge Device.

## How to work

1. List agents. Match by name. Use the real ID.
2. Identify the integration family and open that specialist skill.
3. Confirm the company connection, then the agent tool, then a Call ID.
4. After a write, say what is connected, which agent can use it, and what to test.
