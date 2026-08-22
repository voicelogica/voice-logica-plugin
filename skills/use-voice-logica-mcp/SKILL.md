---
name: use-voice-logica-mcp
description: Master catalog of every Voice Logica MCP tool, grouped by job, with the specialist skill to open next. Use when the user asks what they can do with the Voice Logica MCP, which tool to call, or where a capability lives.
---

# What a user can do with the Voice Logica MCP

This is the full public catalog. Confirm the MCP is connected (Connect / OAuth to https://app.voicelogica.ai; API key only if the client has no Connect flow), list agents if the job is about an agent, then open the specialist skill. Do not invent IDs, fields, or MCP tool names.

Never commit or paste live API keys, OAuth tokens, or sender tokens into chat or git.

Write Greek literally in JSON-string parameters. `\uXXXX` escapes can drop letters. Re-read after write.

## Start here

| Job | Skill |
| --- | --- |
| Not sure / list agents first | `manage-voice-agents` |
| Create / build a new agent | `edit-voice-agents` |
| Edit prompt, languages, transfers, knowledge | `edit-voice-agents` |
| Call behavior went wrong | `fix-agent-behavior` |
| Diagnose with a Call ID | `troubleshoot-voice-agents` |
| DIDs, SIP, inbound numbers, edge devices | `manage-phones` |
| Connect any integration | `manage-integrations` |
| HubSpot / Zoho / Salesforce | `manage-crm` |
| Woo, Shopify, other shops | `manage-ecommerce` |
| Soft1, Galaxy, Pylon, Entersoft | `manage-erp` |
| Google / Microsoft calendar, contacts, Chat | `manage-calendar-google-microsoft` |
| SMS / Viber | `manage-sms` |
| Freshdesk, Jira, couriers, payments | `manage-helpdesk-ops` |
| Workflows and automations | `manage-workflows` |
| Call logs, outbound, test scenarios | `manage-calls-campaigns` |
| Companies, plans, hours | `manage-company-billing` |
| Voice Logica app tickets | `create-support-ticket` |

## Agents

Skill: `edit-voice-agents` (behavior: `fix-agent-behavior`).

- `get_agents` / `get_agent` / `create_agent`
- `get_agent_prompt` / `update_agent_prompt`
- `update_agent_welcome_message`
- `get_agent_end_call_config` / `update_agent_end_call_config`
- `get_agent_transfer_settings` / `update_agent_transfer_settings`
- `get_agent_analysis_config` / `update_agent_analysis_config`
- `get_agent_notification_config` / `update_agent_notification_config`
- `list_agent_tools` / `update_agent_tool`
- `list_agent_languages` / `add_agent_language` / `remove_agent_language`
- `list_voices`
- `list_agent_versions` / `save_agent_version` / `delete_agent_version`
- `get_agent_memory_settings` / `update_agent_memory_settings`
- `get_agent_appointment_settings` / `update_agent_appointment_settings`
- `get_agent_workflows` / `update_agent_workflows`
- `create_customer_instructions` / `get_customer_instructions` / `update_customer_instructions` (admin)

`update_agent_transfer_settings` **replaces** `transferSettings` — `get_` first and resend every field. `update_agent_analysis_config` **replaces** `analysisProperties` the same way. `update_agent_scenario` **replaces** too — omitting `agentId` can detach the scenario.

Prompt/analysis variables are **flat** (`{{phone_number}}`, `{{booked}}`). Workflow variables are **`call.`-prefixed**. `analysisPrompt` is interpolated. System prompt ideal 15–20k characters; above that use knowledge files, not a bigger prompt. See `edit-voice-agents`.

### Agent tools (`list_agent_tools` / `update_agent_tool`)

These are tools the **voice agent** may run on a call, not MCP tools. Always read the live list. Do not invent names.

Documented names you may see:

`skip_turn` (`timeoutSeconds` 1–10), `send_sms` (needs `senderId`), `navigate_ivr`, `schedule_callback`, `detect_live_sentiment`, `detect_live_frustration`, `schedule_appointment` (Google Calendar), `handle_silence`, `lock_on_speaker` (echo), `eshop_integration` / shop-specific (`woocommerce`, `shopify`, `opencart`, `magento`, `megasoft`, `custom_api`), `hubspot` / `zoho` / `salesforce`, ERP (`soft1_erp`, `galaxy_drugstore`, `galaxy_erp`, `pylon_erp`, `entersoft_one`), `helpdesk`, `jira`, `clickup`, `airtable`, `guesty`, `hosthub`, `webhotelier`, `courier`, `outlook`

`schedule_appointment` uses Google Calendar. Shop tools are `manage-ecommerce`. CRM tools are `manage-crm`. `send_sms` needs a `senderId` (`manage-sms`). PrestaShop and Trello often have no voice-agent tool.

## Phones and edge devices

Skill: `manage-phones`. Private PBX: Edge Devices, WireGuard UDP 51820. Health = Online **and** Tunnel up **and** PBX reachable. Direct SIP ≠ Edge ≠ ERP tunnel. Yuboto PBX integration ≠ SIP phone.

- `get_voip_phones` / `create_voip_phone` / `update_voip_phone` / `delete_voip_phone`
- `activate_voip_phone` / `deactivate_voip_phone`
- `get_edge_devices` / `set_edge_device_forward` / `remove_edge_device_forward`

## Calls and campaigns

Skill: `manage-calls-campaigns`.

- `get_calls` / `get_calls_with_prompt` / `get_call_statistics`
- `get_call_logs` / `get_sip_logs` / `get_full_call`
- `get_managed_agent_sessions`
- `initiate_call` / `initiate_bulk_calls`
- `get_scheduled_calls` / `cancel_scheduled_calls`

## Scenarios (test suites)

Skill: `manage-calls-campaigns`. `toolCalls[].succeeded` is a transcript judge, not runtime truth.

- `create_agent_scenario` / `list_agent_scenarios` / `get_agent_scenario`
- `update_agent_scenario` / `delete_agent_scenario`
- `generate_agent_scenarios`
- `run_agent_scenario` / `run_agent_scenario_matrix` / `run_agent_scenario_suite`
- `stop_agent_scenario_suite` / `get_agent_scenario_results`
- `apply_scenario_improvements` / `dismiss_scenario_improvements`

## Workflows

Skill: `manage-workflows`. `apiCall` `headers` / `params` are lists of `{key, value}`. Query params go in `params`, not `queryParams`. Body is a raw JSON string plus `bodyType`.

- `list_workflows` / `get_workflow` / `get_workflows`
- `create_workflow` / `update_workflow` / `delete_workflow`
- `run_workflow` / `test_workflow`
- `add_workflow_trigger` / `update_workflow_trigger` / `remove_workflow_trigger`
- `add_workflow_node` / `update_workflow_node` / `remove_workflow_node`
- `add_switch_case`
- `connect_workflow_nodes` / `disconnect_workflow_nodes`
- `list_workflow_versions` / `save_workflow_version` / `delete_workflow_version`

Triggers: `onAPICall`, `onCallEnd`, `onCallStart`, `onKeywordDetected`, `duringCall`, `onEmailReceived`.

Nodes: `emailConfig`, `apiCall`, `llmCompletion`, `sendSms`, `codeExecution`, `if`, `switch`, `loop`.

## SMS and Viber

Skill: `manage-sms`.

- `send_sms` / `get_sms`
- `list_sms_senders` / `add_sms_sender` / `verify_sms_sender`
- `send_viber`

## Email

Account mailboxes (not Voice Logica support tickets, not Freshdesk).

- `list_mailboxes` / `list_emails` / `get_email` / `send_email`

Workflow sends use `emailConfig`. Helpdesk tickets use `manage-helpdesk-ops`.

## Google and Microsoft

Skill: `manage-calendar-google-microsoft`. Calendar ≠ Meet ≠ Chat. Outlook exists.

- `list_contacts` / `create_or_update_contact`
- `list_calendars` / `list_calendar_events`
- `create_calendar_event` / `update_calendar_event`
- `connect_google_chat` / `list_google_chat_spaces` / `send_google_chat_message`
- `list_google_chat_members` / `get_google_chat_messages`

## Support tickets (Voice Logica app)

Skill: `create-support-ticket`. These are **not** email and **not** Freshdesk/Zendesk. File in the app (Contact Support) with Call IDs. Portal fallback: https://support.voicelogica.ai

- `create_ticket` / `list_tickets` / `get_ticket`
- `list_ticket_mentions` / `reply_to_ticket` / `change_ticket_status`
- `close_all_tickets` (admin)

## Company and billing

Skill: `manage-company-billing`. `companyId` override is admin/reseller only. Normal users omit it.

- `list_companies` / `list_company_users` / `get_company_details`
- `get_company_operating_hours` / `update_company_operating_hours`
- `switch_company` / `request_company_access` / `create_company`
- `upgrade_plan` / `list_plans` / `list_add_ons`
- `get_payment_method` / `get_subscription`
- `get_user_companies_by_email`
- `get_associate` / `update_associate`

Operating hours: `closedAllWeek` closes the week; an empty array errors. `syncWithGoogle` needs a linked Google Place. `seconds: 0` rejects calls as busy.

## MCP settings

- `get_mcp_settings` / `update_mcp_settings`

## Other

- `courier_create_return` — skill: `manage-helpdesk-ops`

## Integrations reminder

Connect the integration → enable the agent tool → test with a Call ID. See `manage-integrations`.

Not self-serve: Odoo, Monday.com, WhatsApp. Splynx is admin-only.

Private HTTP ERP → Tunnels (SSH, admin menu). Private PBX SIP/RTP → Phones → Edge Devices (WireGuard UDP 51820).
