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

## Scenario MCP tools

- `create_agent_scenario` / `list_agent_scenarios` / `get_agent_scenario`
- `update_agent_scenario` / `delete_agent_scenario`
- `generate_agent_scenarios`
- `run_agent_scenario` / `run_agent_scenario_matrix` / `run_agent_scenario_suite`
- `stop_agent_scenario_suite`
- `get_agent_scenario_results`
- `apply_scenario_improvements` / `dismiss_scenario_improvements`

## Scenario judge is not runtime truth

`toolCalls[].succeeded` on a scenario result is a **judge reading the transcript**, not the live tool runner.

The transcript often omits tool responses. A tool can return HTTP 200 and still look failed to the judge.

When a scenario says a tool failed:

1. Get a real **Call ID** (or the scenario's underlying call if one exists).
2. Confirm with `get_call_logs` / `get_full_call`: function about to run, API status, agent received the result.
3. Only then change the prompt, tool, or workflow.

Do not conclude a workflow or CRM write failed from a scenario field alone. Do not apply scenario improvements without showing the user what they change.

## Common jobs

**"What happened on this call?"**
Take the Call ID. `get_full_call` + `get_call_logs`. Use `get_sip_logs` for one-way audio, no-answer, or registration issues. Open `troubleshoot-voice-agents` if it is a diagnosis.

**"Call this number now / start a campaign."**
Confirm the agent and caller ID / phone first (`manage-phones`). Then `initiate_call` or `initiate_bulk_calls`. For later, use scheduled calls and `get_scheduled_calls`.

**"Cancel scheduled outbound."**
`get_scheduled_calls`, match the batch, then `cancel_scheduled_calls`. Confirm how many were cancelled.

**"Generate and run test scenarios."**
`generate_agent_scenarios` or `create_agent_scenario`, then `run_agent_scenario` / matrix / suite. Read `get_agent_scenario_results` as a hint, then verify failures on Call IDs.

## After a change

Say which Call IDs or scenario IDs you used. If you started or cancelled outbound, say how many. Point behavior fixes to `fix-agent-behavior`.
