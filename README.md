# Voice Logica plugin

One plugin folder for Cursor, Claude Code, ChatGPT, and Codex. Users install it, add a Voice Logica API key, and can manage AI voice agents, phones, integrations, workflows, calls, and billing from chat.

This is not the Voice Logica backend. It wraps the public MCP package @voicelogica/mcp-server.

## What users can do

- List, create, and edit AI voice agents (prompts, greetings, languages, voices, versions)
- Change transfer, end-call, analysis, notification, memory, and appointment settings
- Turn agent tools on or off (SMS, appointments, e-shop, HubSpot, silence, IVR, and more)
- Register VoIP phones, assign numbers, and manage edge-device forwards
- Connect CRM, e-commerce, ERP, calendar, SMS, and helpdesk integrations, then test them on a Call ID
- Book on Google or Microsoft calendars, manage contacts, and use Google Chat
- Send SMS or Viber and manage verified senders
- Build workflows (call start/end, keywords, email, API) and assign them to agents
- Review calls, SIP logs, outbound campaigns, and scheduled calls
- Generate and run test scenarios (treat scenario "succeeded" as a judge, not live truth)
- Read plans, add-ons, payment method, and subscription; update operating hours
- File and follow Voice Logica app support tickets
- Work across companies they belong to (switch / request access)

The full MCP tool catalog, grouped by job, lives in `skills/use-voice-logica-mcp`.

Example prompts: list my agents, update the Support Bot greeting, connect HubSpot and test it, show calls from the last 7 days, book a calendar slot, send an SMS from the verified sender.

## Company

Voice Logica. Website: https://voicelogica.ai. App: https://app.voicelogica.ai. Support: support@voicelogica.ai.

## Get an API key

1. Sign in at https://app.voicelogica.ai
2. Open Settings, then API Keys
3. Create a key and paste it when the plugin asks

Never commit a real key into this repo.

## Local test

### Cursor

Copy this folder to %USERPROFILE%\.cursor\plugins\local\voice-logica, reload Cursor, set the API key, then ask it to list your agents.

### Claude Code

    claude plugin validate .
    claude --plugin-dir .

Claude prompts for the API key. Then try /voice-logica:manage-voice-agents or ask it to list your agents.

### ChatGPT / Codex

Codex and ChatGPT desktop can load this folder as a local plugin. ChatGPT on the web wants a hosted MCP URL, not npx. The current server is stdio-only. For a public ChatGPT listing, expose the MCP as HTTPS later.

## Publish

Same folder, three submissions. Listing on one store does not list you on the others.

- Cursor: push a public Git repo, then https://cursor.com/marketplace/publish
- Claude community: run claude plugin validate ., then https://platform.claude.com/plugins/submit
- Official Claude marketplace is invite-only. https://clau.de/plugin-directory-submission is a short link to the docs, not a separate form.
- ChatGPT / Codex: https://developers.openai.com/plugins/deploy/submission Prefer a public HTTP MCP for ChatGPT web.

## Layout

    plugin.json                 Agent Plugins (portable)
    mcp.json                    Cursor + Agent Plugins MCP
    .mcp.json                   Claude Code + Codex MCP
    .cursor-plugin/plugin.json  Cursor manifest + API key variable
    .claude-plugin/plugin.json  Claude Code manifest + userConfig
    .codex-plugin/plugin.json   ChatGPT / Codex manifest
    skills/manage-voice-agents              Router: confirm MCP, list agents, pick a skill
    skills/edit-voice-agents                Editor tabs, prompts, knowledge files
    skills/fix-agent-behavior               Call behavior fixes
    skills/troubleshoot-voice-agents        Diagnose with Call IDs and tool logs
    skills/manage-phones                    VoIP phones, DIDs, assign numbers to agents
    skills/manage-integrations              3-layer connect → enable tool → test Call ID
    skills/manage-crm                       HubSpot, Zoho, Salesforce
    skills/manage-ecommerce                 Woo, Shopify, and eshop_integration
    skills/manage-erp                       Soft1, Galaxy, Pylon, Entersoft, tunnels
    skills/manage-calendar-google-microsoft Calendar, contacts, Chat, appointments
    skills/manage-sms                       SMS / Viber senders
    skills/manage-helpdesk-ops              Freshdesk, Jira, couriers, payments
    skills/manage-workflows                 Triggers, nodes, versions, apiCall shape
    skills/manage-calls-campaigns           Call logs, outbound, test scenarios
    skills/manage-company-billing           Companies, plans, operating hours
    skills/create-support-ticket            File a ticket in the Voice Logica app
    skills/use-voice-logica-mcp             Full public MCP tool catalog
    assets/favicon.png            Marketplace icon (official)
    assets/logo.png               Wordmark
    assets/logo.svg               Fallback mark

## Brand

Official icon: https://www.voicelogica.ai/images/setup/voice-logica-logo-site-favicon.png
Wordmark: https://app.voicelogica.ai/assets/logo.png
