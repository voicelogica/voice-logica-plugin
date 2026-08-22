---
name: manage-voice-agents
description: Route Voice Logica work to the right specialist skill after confirming MCP is connected and listing agents. Use when the user wants to create or configure agents, phones, transfers, integrations, calls, billing, tickets, or is unsure which Voice Logica skill to use.
---

# Manage Voice Logica agents

Use the Voice Logica MCP. Do not invent agent IDs, phone numbers, or API fields.

## Before anything else

1. Confirm the MCP is connected. If tools fail because MCP is not connected, tell the user to Connect Voice Logica MCP (OAuth to https://app.voicelogica.ai). Use an API key only if the client has no Connect flow.
2. List current agents first when the user names an agent. Match by name, then use the real ID.
3. Open the specialist skill for the job. Do not do a deep edit, diagnosis, or integration change from this router.

## Route to the right skill

- Create / build / go-live a new agent: `edit-voice-agents`
- Edit settings, prompts, languages, transfers, knowledge: `edit-voice-agents`
- Prompt over ~20k / knowledge files / router+specialist: `edit-voice-agents`
- Call went badly / change how the agent behaves: `fix-agent-behavior`
- Diagnose a failed call, missing tool, or "is it the model?": `troubleshoot-voice-agents`
- DIDs, SIP phones, inbound numbers, edge devices, registration, one-way audio: `manage-phones`
- Connect any integration (OAuth, API, tunnels, edge): `manage-integrations`
- HubSpot / Zoho / Salesforce: `manage-crm`
- Woo, Shopify, and other shops: `manage-ecommerce`
- Soft1, Galaxy, Pylon, Entersoft: `manage-erp`
- Google / Microsoft calendar, contacts, Chat: `manage-calendar-google-microsoft`
- SMS / Viber senders: `manage-sms`
- Freshdesk, Zendesk, Jira, couriers, payments: `manage-helpdesk-ops`
- Workflows, triggers, nodes: `manage-workflows`
- Call logs, outbound campaigns, test scenarios: `manage-calls-campaigns`
- Companies, plans, operating hours: `manage-company-billing`
- File a ticket in the Voice Logica app (Contact Support) with Call IDs: `create-support-ticket` — portal fallback https://support.voicelogica.ai
- Full MCP tool catalog: `use-voice-logica-mcp`

Transfers live on the agent (Transfer Connection + transfer prompt). Phone registrations live in `manage-phones`. Do not mix them. Attended / warm often fails on PBXs that decline SIP REFER — `edit-voice-agents` picks blind vs attended.

## Style

Speak in the user language. Keep replies short. After a write, confirm what changed.
