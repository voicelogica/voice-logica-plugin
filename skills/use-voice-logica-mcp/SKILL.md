---
name: use-voice-logica-mcp
description: Master catalog of every Voice Logica MCP tool, grouped by job, with the specialist skill to open next. Use when the user asks what they can do with the Voice Logica MCP, which tool to call, or where a capability lives.
---

# What a user can do with the Voice Logica MCP

This is the full public catalog. Confirm the MCP is connected (Connect / OAuth to https://app.voicelogica.ai; API key only if the client has no Connect flow), list agents if the job is about an agent, then open the specialist skill. Do not invent IDs, fields, or MCP tool names.

Never commit or paste live API keys, OAuth tokens, or sender tokens into chat or git.

Write Greek literally in JSON-string parameters. `\uXXXX` escapes can drop letters. Re-read after write.

## How the tools are organised

Everyday work has its own tools. The rest of each area sits behind a pair: `<area>_query` reads and `<area>_execute` changes, with `{action, input}`. Call `<area>_query` **without an action** to list every action with its input. Never guess an action name.

Areas: `agents`, `scenarios`, `versions`, `knowledge`, `leads`, `campaigns`, `messages`, `team`, `billing`, `calendar`, `tickets`, `tasks`, `pbx`, `eshop`, `crm`, `telephony`, `channels`, `integration`. Members: `query_user_action` / `execute_user_action`.

How-to questions: `get_guide` with the user's words (no arguments lists the guides), then `search_docs`.

## My things (tasks, calls, phone)

When the person asks about **their own** tasks, callbacks, calls, extension or apps, use `list_mine` (`what`: tasks by default, callbacks, calls, phone, settings) and `execute_mine` (complete, reopen or add my task; my task emails; my call settings). Tasks always mean the Voice Logica Tasks page: never support tickets, and never a connected ClickUp, Trello or other task app unless the user names that app. Answer exactly what was asked.

## Start here

| Job | Skill |
| --- | --- |
| Not sure / list agents first | `manage-voice-agents` |
| Create / build a new agent | `edit-voice-agents` |
| Edit prompt, languages, transfers, knowledge | `edit-voice-agents` |
| Call behavior went wrong | `fix-agent-behavior` |
| Diagnose with a Call ID | `troubleshoot-voice-agents` |
| SIP trunks, numbers, routes, PBX, edge devices | `manage-phones` |
| Connect any integration | `manage-integrations` |
| HubSpot / Zoho / Salesforce | `manage-crm` |
| Woo, Shopify, other shops | `manage-ecommerce` |
| Soft1, Galaxy, Pylon, Entersoft | `manage-erp` |
| Google / Microsoft calendar, contacts | `manage-calendar-google-microsoft` |
| SMS / Viber | `manage-sms` |
| Freshdesk, Jira, couriers, payments | `manage-helpdesk-ops` |
| Workflows and automations | `manage-workflows` |
| Calls, outbound, campaigns, test scenarios | `manage-calls-campaigns` |
| Plan, team, hours | `manage-company-billing` |
| Voice Logica app tickets | `create-support-ticket` |

## Agents

Skill: `edit-voice-agents` (behavior: `fix-agent-behavior`).

- `get_agents` / `get_agent` / `create_agent`
- `get_agent_prompt` / `edit_agent_prompt` (find/replace or append; send `prompt` for a full rewrite)
- `get_agent_settings` (`sections`: one or more; `format: true` lists the fields) / `update_agent_settings` (one `section` plus `changes`). Sections: welcome, voice, languages, transfer, end_call, interruption, memory, analysis, notifications, appointments, ai_disclosure, standby, tools, workflows, pronunciation, transcription, widget
- `list_voices` / `rename_voice`; `agents_query` (`list_cloned_voices`) / `agents_execute` (`clone_voice`, `delete_cloned_voice`, `delete_agent`)
- `versions_query` (`list_agent_versions`, `get_agent_publish_history`) / `versions_execute` (`save_agent_version`, `publish_agent_version`, `delete_agent_version`)
- Hear it: `get_my_agent`, `test_in_browser` (browser voice/chat link), `call_me` then `get_call_result`

Sections that hold lists (transfer, analysis, tools) are **replaced** on update: read the section first and resend every field.

Prompt/analysis variables are **flat** (`{{phone_number}}`, `{{booked}}`). Workflow variables are **`call.`-prefixed**. `analysisPrompt` is interpolated. System prompt ideal 15–20k characters; above that use knowledge files, not a bigger prompt. See `edit-voice-agents`.

### Agent tools (section `tools`)

These are tools the **voice agent** may run on a call, not MCP tools. Read them with `get_agent_settings` section `tools` and change them with `update_agent_settings` section `tools`. Always read the live list. Do not invent names.

Documented names you may see:

`skip_turn` (`timeoutSeconds` 1–10), `send_sms` (needs `senderId`), `navigate_ivr`, `schedule_callback`, `detect_live_sentiment`, `detect_live_frustration`, `schedule_appointment` (Google Calendar), `handle_silence`, `lock_on_speaker` (echo), `eshop_integration` / shop-specific (`woocommerce`, `shopify`, `opencart`, `magento`, `megasoft`, `custom_api`), `hubspot` / `zoho` / `salesforce`, ERP (`soft1_erp`, `galaxy_drugstore`, `galaxy_erp`, `pylon_erp`, `entersoft_one`), `helpdesk`, `jira`, `clickup`, `airtable`, `guesty`, `hosthub`, `webhotelier`, `courier`, `outlook`

`schedule_appointment` uses Google Calendar. Shop tools are `manage-ecommerce`. CRM tools are `manage-crm`. The agent's `send_sms` needs a `senderId` (`manage-sms`). PrestaShop and Trello often have no voice-agent tool.

## Knowledge

Skill: `edit-voice-agents`.

- `list_knowledge_files` / `get_knowledge_file` / `create_knowledge_file`
- `add_knowledge_entries` (Q&A) / `add_knowledge_text` (document text) / `add_knowledge_link` (website; preview with `crawl_website_pages`)
- `search_knowledge` (test retrieval)
- `list_learning_items` / `review_learning_item` / `get_learning_settings` / `update_learning_settings`
- `knowledge_query` / `knowledge_execute`: edit or delete files and entries, assign files to agents, links, Drive folders, unanswered questions
- `extract_document`: text from an uploaded file

## Phone lines

Skill: `manage-phones`. Private PBX: Edge Devices, WireGuard UDP 51820. Health = Online **and** Tunnel up **and** PBX reachable. Direct SIP ≠ Edge ≠ ERP tunnel.

- `get_sip_trunks` / `create_sip_trunk` / `update_sip_trunk`
- `connect_pbx` (office PBX), `get_free_number` (free Greek number)
- `get_user_phone_setup` (devices, extension, SIP credentials, app links)
- `telephony_query` / `telephony_execute`: numbers, inbound and outbound routes, call settings, `send_device_setup`
- `pbx_query` (`check_pbx_connection`, `get_sip_logs`, `get_trunk_sip_logs`, `get_edge_devices`, `get_edge_installer`) / `pbx_execute` (`claim_edge_device`, `set_edge_media_ports`, `update_edge_device`)

## Calls

Skill: `manage-calls-campaigns`.

- `get_calls` (add include `aiDialogue` for the turn-by-turn dialogue) / `get_call_statistics`
- `get_managed_agent_sessions` (tool and orchestration traces)
- `initiate_call` / `initiate_bulk_calls` (1–50) / `assistant_call`
- `set_call_recording` (pause or resume on the user's own live call)
- `get_compliance_report` (monthly AI disclosure report)
- `transcribe_audio` (compare speech-to-text on a recording)

## Campaigns and leads

Skill: `manage-calls-campaigns`.

- `get_campaigns` / `create_campaign` / `update_campaign`
- `get_campaign_results` / `export_campaign_report`
- `campaigns_query` (`get_campaign`) / `campaigns_execute` (`pause_campaign`, `resume_campaign`, `test_campaign`)
- `leads_query` (leads, statistics, `get_scheduled_calls`) / `leads_execute` (create, import, update, delete leads; `cancel_scheduled_calls`)

## Scenarios (test suites)

Skill: `manage-calls-campaigns`. `toolCalls[].succeeded` is a transcript judge, not runtime truth.

- `run_agent_scenario`
- `scenarios_query` (`list_agent_scenarios`, `get_agent_scenario`, `get_agent_scenario_results`, `estimate_agent_test_run`, `generate_agent_scenarios`)
- `scenarios_execute` (create, update, delete scenarios; `run_agent_scenario_matrix`, `run_agent_scenario_suite`, `stop_agent_scenario_suite`, `apply_scenario_improvements`, `dismiss_scenario_improvements`)

## Workflows

Skill: `manage-workflows`. `apiCall` `headers` / `params` are lists of `{key, value}`. Query params go in `params`, not `queryParams`. Body is a raw JSON string plus `bodyType`.

- `get_workflows` / `get_workflow` / `get_workflow_format` (node and trigger settings)
- `create_workflow` / `create_workflow_from_template` (SMS link, email summary) / `create_api_lookup_workflow` / `create_webhook_workflow` / `update_workflow` (name, visibility) / `delete_workflow`
- `edit_workflow`: one batch of trigger and node operations (add, update, remove, connect, disconnect, switch cases)
- `test_workflow` / `run_workflow`
- `versions_query` (`list_workflow_versions`) / `versions_execute` (`save_workflow_version`, `publish_workflow_version`, `delete_workflow_version`)
- Assign to an agent: `update_agent_settings` section `workflows`

Triggers: `onAPICall`, `onCallEnd`, `onCallStart`, `onKeywordDetected`, `duringCall`, `onEmailReceived`.

Nodes: `emailConfig`, `apiCall`, `llmCompletion`, `sendSms`, `codeExecution`, `if`, `switch`, `loop`.

## Messages and channels

Skill: `manage-sms`.

- `messages_query` (`get_sms`, `list_sms_senders`, `list_mailboxes`, `list_emails`, `get_email`)
- `messages_execute` (`send_sms`, `send_viber`, `send_email`, `add_sms_sender`). A new SMS sender is verified by Voice Logica.
- `channels_query` / `channels_execute`: Viber, WhatsApp and Messenger channels

Account mailboxes are not Voice Logica support tickets and not Freshdesk. Workflow sends use `emailConfig`.

## Contacts and CRM records

Skills: `manage-crm`, `manage-calendar-google-microsoft`.

- `list_contacts` / `create_or_update_contact` / `search_contact_activity` / `get_associate`
- `list_contact_companies` / `get_contact_company` / `set_contact_company_stage`
- `list_resources`
- `crm_query` / `crm_execute`: company records, pipeline stages, custom fields, notes, Google Contacts sync, email copy address
- `export_personal_data` / `erase_personal_data` (GDPR; erasure cannot be undone)

## Calendar

Skill: `manage-calendar-google-microsoft`.

- `calendar_query` (`list_calendars`, `list_calendar_events`) / `calendar_execute` (`create_calendar_event`, `update_calendar_event`)

## Integrations

Skill: `manage-integrations`. Connect the integration → enable the agent tool → test with a Call ID.

- `list_integrations` (providers and what is connected)
- `connect_integration` (`provider`; optional `step`; step `disconnect` removes it)
- `describe_integration` (`integration`) lists its actions
- `integration_query` / `integration_execute` (`integration`, `action`, `data`): every per-integration action, e.g. HubSpot, Zoho, Salesforce, shops, ERPs, couriers, Jira, Outlook, Google Sheets, Docs and Drive
- `eshop_query` (`get_eshop_sync_status`) / `eshop_execute` (`sync_eshop_products`, `clear_eshop_products`)

Not self-serve: Odoo, Monday.com. Splynx is admin-only.

Private HTTP ERP → Tunnels (SSH, admin menu). Private PBX SIP/RTP → Phones → Edge Devices (WireGuard UDP 51820).

## Company, team and billing

Skill: `manage-company-billing`.

- `get_company_operating_hours` / `update_company_operating_hours`
- `get_caller_card_settings` / `update_caller_card_settings` / `manage_caller_card_custom_link`
- `get_mcp_settings` / `update_mcp_settings` (default trunk, agent, SMS sender, calendar)
- `list_plans` / `upgrade_plan` (returns `checkoutUrl`: the user confirms and pays on the billing page, nothing is charged from chat)
- `billing_query` (`get_subscription`, `list_add_ons`, `get_payment_method`, `get_low_balance_alert`) / `billing_execute` (`cancel_subscription`, `resume_subscription`, `set_company_vat`, `set_low_balance_alert`)
- `team_query` (`list_company_users`, `list_invitations`, `get_seats`) / `team_execute` (`invite_member`, `revoke_invitation`, `set_user_seat`, `buy_seats`: returns `checkoutUrl` for the billing page, like `upgrade_plan`)
- `query_user_action` / `execute_user_action` (members)
- `list_tasks` / `create_task` / `complete_task`; `tasks_query` / `tasks_execute` (task settings)

Operating hours: `closedAllWeek` closes the week; an empty array errors. `syncWithGoogle` needs a linked Google Place. `seconds: 0` rejects calls as busy.

`switch_company` and `list_companies` work only for resellers, account managers and admins. Normal users stay in their own company and omit `companyId`.

## Support tickets (Voice Logica app)

Skill: `create-support-ticket`. These are **not** email and **not** Freshdesk/Zendesk. Include Call IDs. Help: https://docs.voicelogica.ai/support/ or support@voicelogica.ai.

- `create_ticket` / `tickets_query` (`list_tickets`, `get_ticket`) / `tickets_execute` (`reply_to_ticket`)
- `report_unanswered_question` when the docs have no answer
