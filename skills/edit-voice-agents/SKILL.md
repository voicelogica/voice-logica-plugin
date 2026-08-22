---
name: edit-voice-agents
description: Create and edit Voice Logica agent settings, prompts, knowledge files, transfers, and editor tabs. Use when the user wants to build a new agent, change a greeting, system prompt, languages, transfers, or cannot find a setting in the agent editor.
---

# Edit Voice Logica agents

Use Voice Logica MCP tools to read the live agent first, then change only the fields the user asked for. Do not invent agent IDs or field names.

## Build a new agent

Order of operations:

1. Confirm purpose, languages, inbound vs outbound, whether they already have a number, and which integrations the call must use.
2. `get_agents` — do not create a duplicate name.
3. `create_agent` with the name and details they gave. Then `get_agent` and configure the rest with the tools below. Do not invent a create payload schema.
4. Switch the agent to **Power View** if they need transfers, tools, analysis, or end-call (`viewMode` is per agent: `business` | `power`). Business View shows only AI Assistant + Languages.
5. Prompt (`update_agent_prompt`) and welcome (`update_agent_welcome_message`). Do not put the same greeting in both.
6. Languages / voices: `list_agent_languages` / `add_agent_language` / `list_voices`.
7. End-call: `get_agent_end_call_config` / `update_agent_end_call_config`. Never put `"transferring"` in `endCallReasons`.
8. Transfers if needed — see below. Always `get_agent_transfer_settings` first.
9. Tools: `list_agent_tools` / `update_agent_tool`. Enable only what the flow needs. Company integrations are a separate step (`manage-integrations`).
10. Knowledge: assign + enable; wait until embeddings are ready; then trim facts out of the prompt.
11. Phone: `manage-phones`. An agent with no DID will not take inbound PSTN calls.
12. `save_agent_version` after a working config.
13. Test with a real call and a Call ID.

Limits (do not invent others): agent name 100 chars; description 500 (internal); tone 200. System prompt ideal 15,000–20,000 characters (plan feature `system-prompt` may block save). Knowledge Q&A: question ~200 chars, answer ~750; document mode subject 100 / content 2500. Split facts across many Q&As. Summary prompt 5000; transcription prompt 160; analysis prompt is plan `analysis-prompt`. Hold-sound upload 50 MB / trim ~60 s.

If the prompt stays above ~20k after moving facts out, use a **router agent + specialist** (transfer / hand-off), not a bigger prompt.

## Find the setting

Many "missing setting" cases are view, role, or plan, not a missing feature.

- **Business View** shows only AI Assistant + Languages.
- **Power View** shows the full tab set. Switch the agent to Power View before calling something missing.
- Prompt-based agents in Power View typically have: Details, Tools and Workflows, Languages and Voices, Analysis, Notifications, Memory, Transcription, Transfer Connection, End Call, Advanced.
- Hidden unless plan or role allows: Outbound (`allow-outbound-calls`), Widget (super-admin / account-manager), Tests (admin / super-admin / account-manager), Customer Instruction (admin / super-admin), Appointments (`google-calendar` + `schedule_appointment`), E-shop (shop tool enabled).
- Workflows and Tunnels menus are **admin role**, not a plan flag.

Where things live:

- System prompt and identity: Details (Power View)
- Call forwarding / attended vs blind: Transfer Connection
- Goodbye and end-call reasons: End Call
- Post-call email: Analysis + Notifications
- Knowledge usage (topN, default 10): Memory
- Hard words / STT hints: Transcription
- Pronunciation (TTS): Pronunciation Guide on Details
- Outbound campaign settings: Outbound tab (plan-gated)
- Calendar booking: Appointments (needs calendar tool)
- Public share page `/w/:agentId`: Widget tab is staff/account-manager; the public page still works once enabled

If no phone is connected, the agent shell usually points to VoIP phones or billing.

## Transfers

Preferred setup: Transfer Connection → enable **Follow Prompt Instructions When Transferring** (`transferCallBasedOnPromptInstructions: true`) → write destination rules in the **transfer settings prompt** (`promptForTransferReason`) and the system prompt.

| Mode | What happens |
| --- | --- |
| **Attended** (blind OFF) | Agent calls the recipient first. If they answer: ask to connect. If no / no-answer → agent continues with the caller. |
| **Blind** (blind ON) | Caller is sent straight through. Agent cannot take the call back. |

### When to pick attended vs blind

Attended is the warm/consult path. Blind is the send-through path.

- **Blind first** on mobiles, unknown PBXs, and any destination that has not passed a real test. Caller goes straight to ringing or voicemail. The agent cannot take the call back.
- **Attended (warm)** only when they need a consult: announce, ask the human, return to the caller if no/no-answer. Many PBXs and carriers do not support this (SIP REFER declined, destination never rings, broken check-leg). If attended fails on that PBX, switch to **blind**. Do not keep retrying warm.
- Internal extensions still need SIP credentials in Phones either way.

If unsure, set blind ON and test a Call ID. Turn attended on only after that specific PBX succeeds.

**SIP REFER declined / attended never rings / broken check-leg** is the PBX or carrier, not the prompt. Switch to blind. If they still need attended, file a ticket (`create-support-ticket`) with Call ID, destination type (mobile / landline / extension), and whether the destination rang. Do not keep retrying warm on that PBX.

Attended-only options:

- Announce summary to recipient
- Analyze transferred call (needs Analysis Minutes)
- Seconds to wait for pickup (`secondsToWaitForCallerToAnswer`; often ~30–45s)
- Show Caller ID: ON = recipient sees the caller number; OFF = sees the agent number
- Block transfer when the company is closed (needs company operating hours)

A **mobile / landline** can be a prompt number. An **internal extension** also needs SIP credentials registered under Phones (`manage-phones`). Without that registration the transfer fails even if the UI looks correct.

Attended transfer creates **two call records** (inbound + outbound check-leg). Both can use AI minutes. The outbound leg may lack a full recording.

Hold after the human answers is a **device / softphone** feature, not an AI setting.

### `update_agent_transfer_settings` replaces `transferSettings`

It does not merge. Always `get_agent_transfer_settings`, then resend **every** field. Sending only `promptForTransferReason` silently resets omitted fields — notably `transferCallBasedOnPromptInstructions` → `false`, which kills prompt-driven transfers. Re-read after write.

### Three places forwarding logic lives — check all

1. Transfer settings prompt — `get_agent_transfer_settings` → `promptForTransferReason` (destination table + whether transfer is allowed)
2. System prompt — `get_agent_prompt`
3. Workflow nodes — `get_agent_workflows` → `get_workflow`

When the agent transferred when it should not have, start at (1). A person→number table answers "which number", never "am I allowed yet". Prepend a precondition; keep the client's numbers and labels byte-identical.

A `duringCall` availability workflow only returns **text**. It does not block the transfer tool. Enforcement is prompt-only.

"I'll transfer" but nothing happens — checklist:

1. `transferCallBasedOnPromptInstructions` is true
2. Prompt has a destination number or name
3. Extension credentials exist if internal
4. Not outside hours with Block Transfer on
5. `"transferring"` is not in `endCallReasons`

Never put `"transferring"` in `endCallReasons` — the agent hangs up instead of transferring.

## Prompts vs knowledge files

Keep the system prompt around 15,000 to 20,000 characters. Client-facing reason is stability and accuracy — never mention internal cost.

### Prompt over ~20k (exact support rule)

1. Measure with `get_agent_prompt` (character count). Check `get_subscription` — plans cap `system-prompt`.
2. Keep in the prompt: role, conversation flow, transfer rules, hard constraints, tone, language, short examples.
3. Move to knowledge files: prices, catalogs, addresses, hours, staff lists, long FAQs, policies, anything the agent should retrieve.
4. Create Q&As (question as a caller would ask + answer + alt questions). One fact family per entry. Greek callers → Greek questions and alt questions. Prefer many small Q&As over one blob.
5. Assign the knowledge file to the agent **and enable it**. Upload alone is not enough. Wait until embeddings are ready.
6. Trim the facts out of the prompt. Do **not** restate a rule that is already there — that eats `system-prompt` budget and does not fix a violated rule (`fix-agent-behavior`).
7. If it is still ~20k+ after extraction (or 30k+), split **router agent + specialist** (transfer / hand-off). Do not grow one prompt.
8. Test with a real Call ID.

If save fails: hit plan `system-prompt`. If add-knowledge fails with `knowledge_questions_limit_reached`: plan `knowledge-questions` quota.

If the user says "I uploaded docs but the agent does not know", check assigned, enabled, still processing, and whether questions match the knowledge entries.

Do not put API keys or internal tool names in customer-facing prompt text.

## Prompt / analysis variables

Two scopes with different names:

| Scope | Naming | Examples |
| --- | --- | --- |
| Prompt, analysis, welcome, email subject | **flat** — no `call.` prefix | `{{phone_number}}`, `{{date}}`, `{{duration}}`, `{{callDirection}}`, `{{transfer_attempted}}`, `{{booked}}` |
| Workflow nodes | **`call.`-prefixed** | `{{call.phone_number}}`, `{{call.summary}}`, `{{call.id}}`, `{{call.recording_url}}` |

`analysisPrompt` **is** interpolated. Post-call analysis properties interpolate **flat**: `{{booked}}`, not `{{call.postAnalysisResults.booked}}`.

An unresolved `{{name}}` still appears as `[name]` in the prompt body. Only the VARIABLES block on the call dump tells you it failed. Do not call that a regression until you diff `postAnalysisResults` keys against an older Call ID — a changed analysis config is the usual cause. The number can still be present as `phone number: "…"` under a different key; the model had it and copied the token.

Write Greek literally in JSON-string parameters. `\uXXXX` escapes can drop letters. Re-read the result.

## Analysis config

`update_agent_analysis_config` replaces the whole `analysisProperties` array. Always `get_agent_analysis_config`, append, send the complete list.

Types: `string | number | boolean | array | date | time | enum | object`. The property **description** is the model instruction. A vague description yields empty values.

Notification SMS (`analysisSmsConfig` / `phoneNumbers`) goes to the **business**, not the caller. For "text the caller", see `manage-sms`.

Built-in notification emails and workflow `emailConfig` nodes are two different systems. Check both when emails are missing, duplicated, or go to the wrong person. Do not invent a "stop all emails" toggle. Do not ask for mailbox passwords.

"Open Call" in the email goes to the dashboard instead of the player: try a logged-in browser; if it still happens, file a ticket with that Call ID.

Emails for a failover / human-answered leg may not be suppressible. Do not invent a toggle.

## How to edit

1. List agents and match by name. Use the real ID.
2. `get_agent` (and the matching get_* settings tool) before any replace-style update.
3. Show the current value, then the proposed change.
4. Apply the change through MCP.
5. Confirm what changed and suggest a test call. `save_agent_version` after a working change.
