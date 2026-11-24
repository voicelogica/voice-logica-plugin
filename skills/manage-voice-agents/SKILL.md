---
name: manage-voice-agents
description: 'Start here for any Voice Logica request, including "Help me set up a voice agent for my business". If the Voice Logica tools (get_onboarding_flow, get_agents) are not in this chat, it is not connected yet, so the whole reply is how to connect (the website alone cannot connect the chat). In ChatGPT, Plugins, Voice Logica, MCP servers, Connect, then a new chat. In Claude, Customize, Connectors, Voice Logica, Connect. Existing account, sign in with it; new, Email me a code, then Create account & connect. Once connected, call get_onboarding_flow for setup, or route to the specialist skill for agents, phones, transfers, integrations, calls, billing or tickets.'
---

# Manage Voice Logica agents

Use the Voice Logica MCP. Do not invent agent IDs, phone numbers, or API fields.

## Before anything else

1. Confirm the MCP is connected. If the Voice Logica tools are missing, the connection is not signed in yet, and only the app can start that sign-in, so tell the user where to press Connect: ChatGPT: Plugins, Voice Logica, MCP servers, Connect on voice-logica, then a new chat; Claude: Customize, Connectors, Voice Logica, Connect; Claude Code: type /mcp, pick voice-logica and sign in; Cursor: Settings, Tools & MCP, voice-logica. In the window that opens, someone who already has a Voice Logica account signs in with it (email and password, Google or Microsoft); someone new presses "New here or no password? Email me a code", enters the code and finishes with "Create account & connect". Tell them both, since you do not know yet which one they are. That Connect button is the only way in: signing in on app.voicelogica.ai by itself leaves the chat unconnected. Use an API key only if the client has no Connect flow.
2. List current agents first when the user names an agent. Match by name, then use the real ID.
3. Open the specialist skill for the job. Do not do a deep edit, diagnosis, or integration change from this router.

## Getting started

When the user asks for help setting up a voice agent for their business, to set up their phone agent or get started, or has just connected, call `get_onboarding_flow` and follow its next step: the setup questions one at a time, a test call (`call_me`, `test_in_browser` or the agent's dial-in number; `wait_for_test_call` reports how it went), then how it sounded. Record each step with `record_onboarding_step`. Use `edit-voice-agents` for the build itself. The page to give the user is https://docs.voicelogica.ai/getting-started/set-up-from-chatgpt-or-claude/.

## Route to the right skill

- Create / build / go-live a new agent: `edit-voice-agents`
- Edit settings, prompts, languages, transfers, knowledge: `edit-voice-agents`
- Prompt over ~20k / knowledge files / router+specialist: `edit-voice-agents`
- Procedures (topic steps loaded only when needed: returns, account questions, complaints, bookings): `edit-voice-agents`
- Call went badly / change how the agent behaves: `fix-agent-behavior`
- Diagnose a failed call, missing tool, or "is it the model?": `troubleshoot-voice-agents`
- DIDs, SIP trunks, inbound numbers, edge devices, registration, one-way audio: `manage-phones`
- Connect any integration (OAuth, API, tunnels, edge): `manage-integrations`
- HubSpot / Zoho / Salesforce: `manage-crm`
- Woo, Shopify, and other shops: `manage-ecommerce`
- Soft1, Galaxy, Pylon, Entersoft: `manage-erp`
- Google / Microsoft calendar, contacts, Chat: `manage-calendar-google-microsoft`
- SMS / Viber senders: `manage-sms`
- Freshdesk, Zendesk, Jira, couriers, payments: `manage-helpdesk-ops`
- Workflows, triggers, nodes: `manage-workflows`
- Call logs, outbound campaigns, test scenarios: `manage-calls-campaigns`
- Instructions for AI left on calls (after-call notes), by hand or on a schedule: `act-on-call-instructions`
- Companies, plans, operating hours: `manage-company-billing`
- File a ticket in the Voice Logica app (Contact Support) with Call IDs: `create-support-ticket` — fallback https://docs.voicelogica.ai/support/
- Full MCP tool catalog: `use-voice-logica-mcp`

Transfers live on the agent (Transfer Connection + transfer prompt). Phone registrations live in `manage-phones`. Do not mix them. Attended / warm often fails on PBXs that decline SIP REFER — `edit-voice-agents` picks blind vs attended.

## Style

Speak in the user language. Keep replies short. After a write, confirm what changed.
