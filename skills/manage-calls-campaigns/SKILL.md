---
name: manage-calls-campaigns
description: Review Voice Logica calls, start outbound or bulk campaigns, and run agent test scenarios. Use when the user wants call logs, SIP logs, scheduled calls, or scenario suites — and when a scenario judge disagrees with a real Call ID.
---

# Manage calls, campaigns, and scenarios

Use Call IDs. Do not invent them. Do not chase model or temperature first — those are usually platform defaults. Read the prompt, `toolCalls`, and call / SIP logs.

## Call MCP tools

- `get_calls` — list calls
- `get_calls_with_prompt` — calls plus the prompt used
- `get_call_statistics` — aggregates
- `get_call_logs` — application / tool logs for a call
- `get_sip_logs` — signaling
- `get_full_call` — full call record
- `get_managed_agent_sessions` — live / managed sessions
- `initiate_call` — one outbound call
- `initiate_bulk_calls` — campaign / bulk outbound
- `get_scheduled_calls` / `cancel_scheduled_calls`

Default list window: last 7 days unless the user asked for another range. Summarize outcome, duration, and agent.

## Recordings and history

- Attended-transfer outbound legs may have a transcript but **no full recording**.
- Very short / never-established calls may lack usable audio.
- Calls that stay "open" may not finalize recording/transcript until hangup.
- There is no archive feature. Use **seen / unseen** (auto when opened; Unseen only filter; shows who viewed and when).
- Filters customers care about: unseen, date range, agent, direction, phone / Call ID.

If the call completed, a transcript exists, and audio should exist (not a transfer check-leg / not a 2-second hangup), file a ticket (`create-support-ticket`) with that Call ID. Do not invent an archive button.

Incomplete / cut-off audio: collect Call ID + timestamp in the recording (TTS cut, caller hangup, transfer mid-utterance, or network). Then diagnose from `get_full_call` / `get_call_logs`.

## Scenario MCP tools

- `create_agent_scenario` / `list_agent_scenarios` / `get_agent_scenario`
- `update_agent_scenario` / `delete_agent_scenario`
- `generate_agent_scenarios`
- `run_agent_scenario` / `run_agent_scenario_matrix` / `run_agent_scenario_suite`
- `stop_agent_scenario_suite`
- `get_agent_scenario_results`
- `apply_scenario_improvements` / `dismiss_scenario_improvements`

Tests / scenarios tab is admin, super-admin, or account-manager.

`update_agent_scenario` **replaces** — it does not merge. Always `get_agent_scenario` first and resend **every** field, including `agentId`. Omitting `agentId` can silently detach the scenario from the agent.

## Scenario judge is not runtime truth

`toolCalls[].succeeded` on a scenario result is a **judge reading the transcript**, not the live tool runner.

The transcript often omits tool responses. A tool can return HTTP 200 and still look failed to the judge.

When a scenario says a tool failed:

1. Get a real **Call ID** (or the scenario's underlying call if one exists).
2. Confirm with `get_call_logs` / `get_full_call`: function about to run, API status, agent received the result.
3. Only then change the prompt, tool, or workflow.

Do not conclude a workflow or CRM write failed from a scenario field alone. Do not apply scenario improvements without showing the user what they change.

Write judge criteria on **turn order and what was said**, not "after the tool answers" — the judge cannot see tool responses.

## Outbound campaigns

Needs **all** of: configured agent, phone list, campaign settings (schedule, max duration, min duration, retry), a SIP / DID (`manage-phones`), AI minutes **and** provider telephony credits. Outbound tab is plan-gated (`allow-outbound-calls`). Inbound minutes on a plan do not include unlimited PSTN outbound.

Each cloned outbound agent needs its **own SIP extension** (or port). They share the company minute pool.

Typical settings: call schedule, max duration, min duration (filter short rejects from reports), retry. Outbound prompts should identify the purpose immediately and ask if it is a good time.

`initiate_call` / `initiate_bulk_calls` after confirming the agent and caller ID / phone.

**Calls are not being made.** Check in order: campaign enabled/scheduled, `get_subscription` has seconds remaining, SIP / DID active, phone list has valid numbers, provider credits exist.

**Calls connect but the agent is silent.** Greeting in the prompt / welcome, agent waiting for the caller to speak first, or codec mismatch — not a missing campaign toggle.

**AI minutes vs provider credits.** No provider credits = no PSTN outbound, regardless of plan. For Yuboto wallet top-up, send only https://services.yuboto.com/mynumber/ and support@yuboto-telephony.gr.

**Sheet dates (if the campaign reads a Google Sheet).** A US-locale sheet parses `M/D/Y`. Greek clients type `D/M/Y`, so `7/8/2026` becomes 8 July. Confirm by looking at a platform-written column such as `called_at`. Fix both halves: set the sheet locale **and** tell the user to stop entering dates reversed. Static welcome dates (`phoneFirstMessageAiGenerated: false`) are spoken verbatim — feed a spoken-form column, do not expect the pronunciation guide to fix a changing date.

Do not set a caller ID the user does not own. Do not invent a staff email or a Yuboto-side assignment procedure. If they want their own number shown and MCP cannot do it, say Voice Logica / their telephony provider must permit that caller ID on the trunk, **and** the campaign must select it — an unpermitted value **silently falls back** to the default number.

Respect quiet hours, consent, and local outbound laws. Start with a small list.

## Common jobs

**"What happened on this call?"**
Take the Call ID. `get_full_call` + `get_call_logs`. Use `get_sip_logs` for one-way audio, no-answer, or registration issues. Open `troubleshoot-voice-agents` if it is a diagnosis. Zero Call IDs in the window → traffic never reached Voice Logica (`manage-phones`).

**"Call this number now / start a campaign."**
Confirm the agent and caller ID / phone first (`manage-phones`). Confirm `get_subscription` has seconds remaining (0 seconds = busy, not a SIP fault). Then `initiate_call` or `initiate_bulk_calls`. For later, use scheduled calls and `get_scheduled_calls`.

**"Cancel scheduled outbound."**
`get_scheduled_calls`, match the batch, then `cancel_scheduled_calls`. Confirm how many were cancelled.

**"Generate and run test scenarios."**
`generate_agent_scenarios` or `create_agent_scenario`, then `run_agent_scenario` / matrix / suite. Read `get_agent_scenario_results` as a hint, then verify failures on Call IDs.

## After a change

Say which Call IDs or scenario IDs you used. If you started or cancelled outbound, say how many. Point behavior fixes to `fix-agent-behavior`.
