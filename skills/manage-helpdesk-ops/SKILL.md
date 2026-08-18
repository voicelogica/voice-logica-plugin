---
name: manage-helpdesk-ops
description: Connect Freshdesk, Zendesk, Jira, ClickUp, Trello, Airtable, Guesty, Hosthub, WebHotelier, couriers, Viva, or Yuboto, and create courier returns. Use when the user wants helpdesk tickets, boards, bookings, payments, or shipping on an agent — not Voice Logica support tickets.
---

# Manage helpdesk and ops integrations

This skill is for **customer** helpdesk, boards, bookings, payments, and couriers.

Voice Logica **app** support tickets are a different product. Those use `create_ticket` / `list_tickets` / `reply_to_ticket`. Open `create-support-ticket` for those. Do not file a Voice Logica ticket when the user asked to create a Freshdesk or Zendesk ticket.

## Public integrations

- **Helpdesk:** Freshdesk, Zendesk — one `helpdesk` agent tool covers Freshdesk **or** Zendesk. Keep one path active.
- **Work trackers:** Jira (`jira`), ClickUp (`clickup`), Trello (no default agent tool — workflows / MCP), Airtable (`airtable`)
- **Hospitality:** Guesty (`guesty`), Hosthub (`hosthub`), WebHotelier (`webhotelier`)
- **Payments / messaging ops:** Viva (mainly workflow payment-link; demo vs live + optional sourceCode), Yuboto
- **Couriers:** ACS, Speedex, Geniki, ELTA, BOX NOW, Skroutz Last Mile — agent tool `courier` plus MCP `courier_create_return`

Read `list_agent_tools` for the live names. Do not invent MCP connect helpers that are not in `use-voice-logica-mcp`.

## MCP tools

- `courier_create_return` — create a courier return shipment

Other helpdesk / board / payment actions go through the connected integration and the agent or workflow (`apiCall`, `emailConfig`). Email mailboxes (`list_mailboxes`, `list_emails`, `get_email`, `send_email`) are account email, not a Freshdesk ticket.

## Company connection fields

- **Freshdesk:** subdomain + API key
- **Zendesk:** subdomain + agent email + API token
- **Jira:** site URL, email, API token; optional default project/issue type
- **ClickUp:** personal access token + default list; optional contacts list / phone field
- **Trello:** authorize + member token (optional own API key)
- **Airtable:** PAT → pick base/table → **map a phone field** or lookup fails (scopes: records read/write, schema bases read)
- **Guesty:** client id/secret (+ optional base URL)
- **Hosthub:** API key (+ optional baseUrl)
- **WebHotelier:** username / password / propertyCode
- **ACS:** apiKey, companyId/password, userId/password, lang
- **Geniki:** username / password / applicationKey
- **BOX NOW:** client id + secret
- **Skroutz Last Mile:** merchant API token
- **Speedex / ELTA:** enable only (public tracking — no secrets)
- **Viva:** client id/secret, env live/demo, optional sourceCode
- **Yuboto PBX:** domain + API key under Integrations — **not** a SIP phone and **not** an Edge Device (`manage-phones`)

Never paste live tokens into chat. Confirm property / site / subdomain codes before blaming the prompt.

## 3-layer rule

1. Connect the integration on the company.
2. Enable the agent tool or the workflow that calls it.
3. Test with a Call ID (or a workflow test) and read `toolCalls`.

If OAuth shows `needsReconnect`, disconnect and reconnect.

## Common jobs

**"Create a Freshdesk / Zendesk ticket from the call."**
Confirm the helpdesk integration is connected and the agent or `onCallEnd` workflow writes the ticket. Test with a Call ID. This is not `create_ticket`.

**"Create a courier return."**
Use `courier_create_return` with the real order / tracking fields the user provides. Do not invent tracking numbers. Ask which courier.

**"Push the call into Jira / ClickUp / Trello / Airtable."**
Connect that integration, then wire a workflow node or agent tool. Confirm on a test item, not only from the agent saying it did. Do not promise a Trello voice tool unless `list_agent_tools` shows one.

**"Guesty / Hosthub / WebHotelier booking."**
Same 3-layer rule. Confirm property/site codes.

**"Viva payment link."**
Workflow payment-link node. Check live vs demo and sourceCode. Delivery of the link is often SMS/Viber (`manage-sms`).

## After a change

Say which ops system, which agent or workflow, and what to test. If they actually wanted Voice Logica support, switch to `create-support-ticket`.
