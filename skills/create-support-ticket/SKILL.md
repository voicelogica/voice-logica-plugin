---
name: create-support-ticket
description: Create a Voice Logica support ticket in the Voice Logica app when the user has a problem they cannot fix, needs human follow-up, or asks to open a ticket. Use after a short diagnosis, and attach Call IDs and agent IDs.
---

# Create a Voice Logica support ticket

File it in the **Voice Logica app** (Contact Support) through MCP. These are not Freshdesk / Zendesk tickets (`manage-helpdesk-ops`) and not mailbox email.

Confirm the MCP is connected (Connect / OAuth to https://app.voicelogica.ai). Use an API key only if the client has no Connect flow.

## When to file

- The user asks to open a ticket, escalate, or get a human.
- After one honest diagnosis pass the issue is still broken, or it needs a human (carrier, number porting, billing, account access).
- SIP REFER declined, a recording that should exist is missing, or the agent spoke JSON / tool names / brackets.

Do **not** file a ticket for a setting change you can do in this chat.

## Required fields

Collect before creating. Do not invent Call IDs.

1. Short subject (what failed — not "help")
2. Expected vs actual
3. Agent name and ID
4. Call ID(s) if it is a call, transfer, or recording issue
5. When it started / whether it used to work
6. What you already tried

Also useful when relevant: destination type (mobile / landline / extension), whether the destination rang, DID / public number, and whether the window had **zero Call IDs** (traffic never reached Voice Logica).

Write the body in the user language. Put IDs in as plain text.

## MCP tools (do not invent others)

- `create_ticket`
- `list_tickets` / `get_ticket`
- `list_ticket_mentions` / `reply_to_ticket` / `change_ticket_status`
- `close_all_tickets` — admin only; do not use unless the user asked

## How to create

1. `list_tickets` first. If an open ticket already covers this, `reply_to_ticket` instead of a duplicate.
2. `create_ticket` with the subject and body above. Always include Call IDs and agent IDs in the body when you have them.
3. Return the ticket ID and a one-line summary of what you filed.
4. If create fails or no ticket tool is available, say so and give the Support portal: https://support.voicelogica.ai — tell them to file there with the same Call IDs.

## After create

Do not keep editing the agent unless the user asks. Offer to add a Call ID or recording if they have one.
