---
name: edit-voice-agents
description: Create and edit Voice Logica agent settings, prompts, procedures, knowledge files, transfers, and editor tabs. Use when the user wants to build a new agent, change a greeting, system prompt, languages, transfers, move topic steps (returns, account questions, complaints, bookings) into procedures, or cannot find a setting in the agent editor.
---

# Edit Voice Logica agents

Use Voice Logica MCP tools to read the live agent first, then change only the fields the user asked for. Do not invent agent IDs or field names.

## Build a new agent

Order of operations:

1. Confirm purpose, languages, inbound vs outbound, whether they already have a number, and which integrations the call must use.
2. `get_agents` — do not create a duplicate name.
3. `create_agent` with the name and details they gave. Then `get_agent` and configure the rest with the tools below. Do not invent a create payload schema.
4. Switch the agent to **Power View** if they need transfers, tools, analysis, or end-call (`viewMode` is per agent: `business` | `power`). Business View shows only AI Assistant + Languages.
5. Prompt (`edit_agent_prompt`; send `prompt` for a full rewrite) and welcome (`update_agent_settings` section `welcome`). Do not put the same greeting in both.
6. Languages / voices: `get_agent_settings` / `update_agent_settings` section `languages`, plus `list_voices`.
7. End-call: `get_agent_settings` / `update_agent_settings` section `end_call`. Never put `"transferring"` in `endCallReasons`.
8. Transfers if needed — see below. Always read section `transfer` first.
9. Tools: `get_agent_settings` / `update_agent_settings` section `tools`. Enable only what the flow needs. Company integrations are a separate step (`manage-integrations`).
10. Knowledge: assign + enable; wait until embeddings are ready; then trim facts out of the prompt.
11. Procedures: steps for one kind of request (returns, account questions, complaints, booking changes) go in procedures, not the prompt — see Procedures below.
12. Phone: `manage-phones`. An agent with no DID will not take inbound PSTN calls.
13. `versions_execute` (action `save_agent_version`) after a working config.
14. Test with a real call and a Call ID.

Limits (do not invent others): agent name 100 chars; description 500 (internal); tone 200. System prompt ideal 15,000–20,000 characters (plan feature `system-prompt` may block save). Knowledge Q&A: question ~200 chars, answer ~750; document mode subject 100 / content 2500. Split facts across many Q&As. Summary prompt 5000; transcription prompt 160; analysis prompt is plan `analysis-prompt`. Hold-sound upload 50 MB / trim ~60 s.

If the prompt stays above ~20k after moving facts to knowledge and topic steps to procedures, use a **router agent + specialist** (transfer / hand-off), not a bigger prompt.

## Find the setting

Many "missing setting" cases are view, role, or plan, not a missing feature.

- **Business View** shows only AI Assistant + Languages.
- **Power View** shows the full tab set. Switch the agent to Power View before calling something missing.
- Prompt-based agents in Power View typically have: Details, Tools and Workflows, Languages and Voices, Analysis, Notifications, Memory, Transcription, Transfer Connection, End Call, Advanced.
- Hidden unless plan or role allows: Outbound (`allow-outbound-calls`), Widget (super-admin / account-manager), Tests (admin / super-admin / account-manager), Customer Instruction (admin / super-admin), Appointments (`google-calendar` + `schedule_appointment`), E-shop (shop tool enabled).
- Workflows and Tunnels menus are **admin role**, not a plan flag.

Where things live:

- System prompt and identity: Details (Power View)
- Steps for one kind of request, loaded only when needed: Procedures
- Call forwarding / attended vs blind: Transfer Connection
- Goodbye and end-call reasons: End Call
- Post-call email: Analysis + Notifications
- Knowledge usage (topN, default 10): Memory
- Hard words / STT hints: Transcription
- Pronunciation (TTS): Pronunciation Guide on Details
- Outbound campaign settings: Outbound tab (plan-gated)
- Calendar booking: Appointments (needs calendar tool)
- Public share page `/w/:agentId`: Widget tab is staff/account-manager; the public page still works once enabled

If no phone is connected, the agent shell usually points to Phones or billing.

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

### `update_agent_settings` section `transfer` replaces `transferSettings`

It does not merge. Always `get_agent_settings` section `transfer`, then resend **every** field. Sending only `promptForTransferReason` silently resets omitted fields — notably `transferCallBasedOnPromptInstructions` → `false`, which kills prompt-driven transfers. Re-read after write.

### Three places forwarding logic lives — check all

1. Transfer settings prompt — `get_agent_settings` section `transfer` → `promptForTransferReason` (destination table + whether transfer is allowed)
2. System prompt — `get_agent_prompt`
3. Workflow nodes — `get_agent_settings` section `workflows` → `get_workflow`

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

Keep the system prompt around 15,000 to 20,000 characters: long prompts make the agent less stable and less accurate.

### Prompt over ~20k (exact support rule)

1. Measure with `get_agent_prompt` (character count). Check `billing_query` (action `get_subscription`) — plans cap `system-prompt`.
2. Keep in the prompt: role, conversation flow, transfer rules, hard constraints, tone, language, short examples.
3. Move to knowledge files: prices, catalogs, addresses, hours, staff lists, long FAQs, policies, anything the agent should retrieve. Move to procedures: sections of steps for one kind of request (see Procedures below).
4. Create Q&As (question as a caller would ask + answer + alt questions). One fact family per entry. Greek callers → Greek questions and alt questions. Prefer many small Q&As over one blob.
5. Assign the knowledge file to the agent **and enable it**. Upload alone is not enough. Wait until embeddings are ready.
6. Trim the facts out of the prompt. Do **not** restate a rule that is already there — that eats `system-prompt` budget and does not fix a violated rule (`fix-agent-behavior`).
7. If it is still ~20k+ after moving facts and topic steps out (or 30k+), split **router agent + specialist** (transfer / hand-off). Do not grow one prompt.
8. Test with a real Call ID.

If save fails: hit plan `system-prompt`. If add-knowledge fails with `knowledge_questions_limit_reached`: plan `knowledge-questions` quota.

If the user says "I uploaded docs but the agent does not know", check assigned, enabled, still processing, and whether questions match the knowledge entries.

Do not put API keys or internal tool names in customer-facing prompt text.

Callback number: the agent asks whether it is the number the caller is calling from, without saying it ("Να σας καλέσουμε στο τηλέφωνο από το οποίο μας καλείτε;"). A yes means `{{phone_number}}` and the agent moves on. Only a different number is taken down and read back. Reading the caller's own number back is slow and adds nothing.

Welcome message: start with `{{greeting}}` (Καλημέρα before 12:00 / Καλησπέρα after, company time zone), then the agent's name as the company's digital assistant (Greek: company in the genitive), then one question. Open: «{{greeting}} σας. Είμαι η Δανάη, η ψηφιακή βοηθός του λογιστικού γραφείου PKP. Πώς θα μπορούσα να σας βοηθήσω;». Closed (`phoneFirstMessageOffHours`): same, plus «Αυτή τη στιγμή είμαστε κλειστά, αλλά πείτε μου πώς θα μπορούσα να σας βοηθήσω.». Set with `update_agent_settings` section `"welcome"`.

## Procedures

A procedure is a named block of steps for one kind of request. It belongs to the **company**, like a workflow: write it once, attach it to every agent that needs it. The prompt carries only one line per attached procedure (its key and trigger); the agent loads the steps mid-call with `load_procedure` when the caller's request matches.

What goes where:

- **Prompt**: who the agent is, tone, general rules, how to tell requests apart, transfer, closing.
- **Procedure**: a section that handles one kind of request over several lines (returns, account questions, complaints, changing a booking, identity check).
- **Knowledge**: facts the agent quotes (prices, hours, policies).

Tools: `procedures_query` reads, `procedures_execute` changes, both with `{action, input}`. Call `procedures_query` without an action to list every action with its input. Steps: `get_guide` with `guide: "agent_procedures"`.

Order of operations:

1. `procedures_query` (action `list_procedures`) — reuse what the company has before writing a new one. With `agentId` it shows what that agent uses.
2. Start from one of three:
   - Presets: `list_procedure_presets`, then `procedures_execute` (action `add_procedure_presets`): `account_questions`, `product_returns`, `complaints`, `booking_change`, `verify_identity`. A key the company already has is reused, never overwritten. Edit the steps to how the business works.
   - The current prompt: `split_prompt_into_procedures` lists each `====` section with a suggested key. It saves nothing; pick the topic sections with the user.
   - Blank: `create_procedure` with `name`, `trigger`, `content`.
3. `attach_procedures` with `destination`: `"version"` saves into a numbered version to test first, `"prod"` changes the live agent at once. Ask which when the user did not say.
4. Delete each moved section from the prompt (`edit_agent_prompt`), with the same destination, so the agent does not get the same steps twice.
5. Test: `run_agent_scenario` with `agentVersionId`. Each load shows in the call as a `load_procedure` tool row, so a scenario can expect it in `expectedToolCalls`. Then `versions_execute` (action `publish_agent_version`).

Writing them:

- **Trigger** is one line (max 200 characters) saying when it applies, worded the way callers ask ("Ο πελάτης θέλει να επιστρέψει προϊόν"). No `{{variables}}` in the trigger; the steps can use them. If the agent answers without loading the procedure, rewrite the trigger closer to the caller's words — do not add a rule to the prompt.
- **Key** is lowercase letters, digits and `_`, unique in the company.
- **Links**: write `@key` in the steps to point to another procedure ("first follow @verify_identity"). It loads even when not attached to that agent. Set `autoTrigger: false` on a procedure that should only be reached through a link. A link to a missing key, or a loop, is refused on save.
- Limits per agent: 60 procedures, 6,000 characters for the trigger list; 12,000 characters of steps per procedure.

Editing:

- `update_procedure` changes it on **every agent that uses it**, from their next call. Read `usedBy` (`get_procedure`) and name those agents to the user before saving. Use `edits` (find/replace) or `append` for a small change; `content` replaces all the steps.
- For one agent only: `duplicate_procedure`, then detach the shared one and attach the copy there.
- Each save keeps the previous text: `list_procedure_revisions` (last 20), `restore_procedure_revision`.
- `detach_procedures` removes it from one agent and keeps it for the company. `delete_procedure` is refused while an agent uses it unless `detachFromAgents: true`; list those agents and confirm with the user first.
- A saved agent version stores which procedures it uses, not their text: an edit to a procedure reaches every version that uses it.

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

`update_agent_settings` section `analysis` replaces the whole `analysisProperties` array. Always `get_agent_settings` section `analysis` first, append, send the complete list.

Types: `string | number | boolean | array | date | time | enum | object`. The property **description** is the model instruction. A vague description yields empty values.

Notification SMS (`analysisSmsConfig` / `phoneNumbers`) goes to the **business**, not the caller. For "text the caller", see `manage-sms`.

Built-in notification emails and workflow `emailConfig` nodes are two different systems. Check both when emails are missing, duplicated, or go to the wrong person. Do not invent a "stop all emails" toggle. Do not ask for mailbox passwords.

"Open Call" in the email goes to the dashboard instead of the player: try a logged-in browser; if it still happens, file a ticket with that Call ID.

Emails for a failover / human-answered leg may not be suppressible. Do not invent a toggle.

## How to edit

1. List agents and match by name. Use the real ID.
2. `get_agent` (and `get_agent_settings` for that section) before any replace-style update.
3. Show the current value, then the proposed change.
4. Apply the change through MCP.
5. Confirm what changed and suggest a test call. `versions_execute` (action `save_agent_version`) after a working change.
