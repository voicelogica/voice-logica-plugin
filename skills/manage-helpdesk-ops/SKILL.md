---
name: manage-helpdesk-ops
description: Connect Freshdesk, Zendesk, Jira, ClickUp, Trello, Airtable, Guesty, couriers, Viva, or Yuboto, and create courier returns. Use when the user wants helpdesk tickets, boards, bookings, payments, or shipping on an agent — not Voice Logica support tickets.
---

# Manage helpdesk and ops integrations

This skill is for **customer** helpdesk, boards, bookings, payments, and couriers.

Voice Logica **app** support tickets are a different product. Those use `create_ticket` / `list_tickets` / `reply_to_ticket`. Open `create-support-ticket` for those. Do not file a Voice Logica ticket when the user asked to create a Freshdesk or Zendesk ticket.

## Public integrations

- **Helpdesk:** Freshdesk, Zendesk
- **Work trackers:** Jira, ClickUp, Trello, Airtable
- **Hospitality:** Guesty
- **Payments / messaging ops:** Viva, Yuboto
- **Couriers:** courier integrations plus `courier_create_return`

## MCP tools

- `courier_create_return` — create a courier return shipment

Other helpdesk / board / payment actions go through the connected integration and the agent or workflow (`apiCall`, `emailConfig`). Email mailboxes (`list_mailboxes`, `list_emails`, `get_email`, `send_email`) are account email, not a Freshdesk ticket.

## 3-layer rule

1. Connect the integration on the company.
2. Enable the agent tool or the workflow that calls it.
3. Test with a Call ID (or a workflow test) and read `toolCalls`.

If OAuth shows `needsReconnect`, disconnect and reconnect. Never paste live API tokens into chat.

## Common jobs

**"Create a Freshdesk / Zendesk ticket from the call."**
Confirm the helpdesk integration is connected and the agent or `onCallEnd` workflow writes the ticket. Test with a Call ID. This is not `create_ticket`.

**"Create a courier return."**
Use `courier_create_return` with the real order / tracking fields the user provides. Do not invent tracking numbers.

**"Push the call into Jira / ClickUp / Trello / Airtable."**
Connect that integration, then wire a workflow node or agent tool. Confirm on a test item, not only from the agent saying it did.

**"Guesty booking / Viva payment / Yuboto message."**
Same 3-layer rule. Read the live connection first. Change only what the user asked for.

## After a change

Say which ops system, which agent or workflow, and what to test. If they actually wanted Voice Logica support, switch to `create-support-ticket`.
