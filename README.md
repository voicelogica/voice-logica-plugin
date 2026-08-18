# Voice Logica plugin

One plugin folder for Cursor, Claude Code, ChatGPT, Codex, Gemini CLI, and Grok. Users install it, click Connect, and sign in at Voice Logica. After that they can manage AI voice agents, phones, integrations, workflows, calls, and billing from chat.

The default MCP is the hosted endpoint at https://api.voicelogica.ai/api/v1/mcp. Clients that support OAuth open a Connect flow to https://app.voicelogica.ai. This is not the Voice Logica backend.

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

Privacy: https://legal.voicelogica.ai/privacy. Terms: https://legal.voicelogica.ai/terms.

## Connect

1. Install the plugin
2. Click Connect on the Voice Logica MCP
3. Sign in at https://app.voicelogica.ai

No API key is required when the client supports OAuth.

### Fallback for clients without Connect

Some local/stdio clients cannot open a browser. For those only:

1. Sign in at https://app.voicelogica.ai
2. Open Settings, then API Keys
3. Run `npx -y @voicelogica/mcp-server` with `VOICE_LOGICA_API_KEY` set

Never commit a real key into this repo.

## Local test

### Cursor

Copy this folder to %USERPROFILE%\.cursor\plugins\local\voice-logica, reload Cursor, click Connect, then ask it to list your agents.

### Claude Code

    claude plugin validate .
    claude --plugin-dir .

Connect Voice Logica MCP if prompted. Then try /voice-logica:manage-voice-agents or ask it to list your agents.

### ChatGPT / Codex

Use the hosted MCP URL `https://api.voicelogica.ai/api/v1/mcp` with OAuth. Codex can also load this folder as a local plugin.

### Gemini CLI

    gemini extensions install voicelogica/voice-logica-plugin

Then authenticate the remote MCP if Gemini asks.

## Publish

Same folder, several submissions. Listing on one store does not list you on the others.

- Cursor official: https://cursor.com/marketplace/publish
- cursor.directory: https://cursor.directory/plugins/new
- Claude community: run claude plugin validate ., then https://platform.claude.com/plugins/submit
- Official Claude marketplace is invite-only
- ChatGPT / Codex: https://developers.openai.com/plugins/deploy/submission
- Gemini CLI: `gemini-extension.json` plus the `gemini-cli-extension` GitHub topic
- Grok: listing PR against xai-org/plugin-marketplace

## Layout

    gemini-extension.json        Gemini CLI gallery manifest
    plugin.json                 Agent Plugins (portable)
    mcp.json                    Cursor + Agent Plugins MCP (hosted OAuth)
    .mcp.json                   Claude Code + Codex MCP (hosted OAuth)
    .cursor-plugin/plugin.json  Cursor manifest
    .claude-plugin/plugin.json  Claude Code manifest
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