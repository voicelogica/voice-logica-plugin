---
name: manage-crm
description: Connect and configure HubSpot, Zoho, or Salesforce on a Voice Logica agent. Use when the user wants CRM contact lookup, property updates, call activities, or a HubSpot agent tool.
---

# Manage Voice Logica CRM

Self-serve CRMs: **HubSpot**, **Zoho**, **Salesforce**.

Monday.com is **not** self-serve. Do not try to add it from the customer UI. Point the user to Voice Logica if they need Monday.com.

## 3-layer rule

1. Connect the CRM integration on the company (OAuth).
2. Enable the CRM tool on the agent.
3. Test with a Call ID and read `toolCalls`.

If OAuth shows `needsReconnect`, disconnect and reconnect. Never edit the prompt to paper over a dead token. Never paste live API tokens into chat.

## HubSpot agent tool

The HubSpot agent tool (`hubspot` in `list_agent_tools` / `update_agent_tool`) has three field groups. Enable only what the call flow needs:

- `readContactProperties` — lookup / read contact fields during the call
- `updateContactProperties` — write fields the caller actually provided
- `createCallActivity` — log the call on the HubSpot contact

A connected HubSpot account with the tool off will not run. A tool with no properties selected will run and do nothing useful.

After enabling, confirm the property names match HubSpot (internal names, not labels). Then test: find a contact, update one field, check the Call ID.

## Zoho and Salesforce

Same 3-layer rule. Connect OAuth, enable the agent tool, test with a Call ID.

Do not invent object or field names. Read the live tool config first. Change only the fields the user asked for.

## Common jobs

**"The agent did not update the CRM."**
Get a Call ID. Check `toolCalls` for the CRM tool. If it never fired, check layer 2 and the prompt instruction to use it. If it fired and failed, check OAuth / `needsReconnect` and the property names. Do not start by rewriting the prompt.

**"Log every call in HubSpot."**
Enable `createCallActivity`. Confirm the integration is connected. Test one call and verify the activity on the contact.

**"Which CRM is this agent using?"**
List agent tools and the company integrations. Say what is connected vs what this agent can use.

## After a change

Say which CRM, which agent, and which tool fields are on. Ask for a test Call ID.
