---
name: troubleshoot-voice-agents
description: Diagnose Voice Logica call and agent issues using Call IDs, live prompts, and tool call logs. Use when a call failed, a tool did not run, a setting seems broken, or the user thinks the model or STT is the cause.
---

# Troubleshoot Voice Logica agents

Use MCP tools. Do not guess. Do not invent Call IDs, tool results, or plan features.

## Do not start with the model

Call dumps may show model, temperature, or reasoningEffort. Those are usually platform defaults, not the customer's knobs. Only treat the model as relevant if the plan can select a model (`can-select-model`) or the company uses its own API keys.

Look at the prompt, tool prompts, whether the expected tool fired (`toolCalls`), plan features, and view/role gates first.

Expired / zero-second subscriptions look like random product bugs. `get_subscription` and the call `endCallReason` before debugging SIP or the prompt.

## Prompt vs bug

Read the live prompt for that exact situation before agreeing it is a defect.

- Agent broke an explicit rule: fix the mechanism. Check the rule is not already there (a violated rule is not a missing rule).
- Agent followed an explicit rule: this is an SOP change, not a bug. Say so, then edit the rule if the user wants that.
- Prompt is silent: agree the desired behavior, then write it.

A truncated agent line in a transcript does not prove the agent interrupted. Get the recording or call logs if interruption is the complaint.

If the agent speaks JSON / tool names / brackets, collect Call ID + timestamp and file a ticket. Do not prompt-paper over leaked system content.

## Tools and tests

Scenario or judge fields like `toolCalls[].succeeded` are a model reading the transcript. The transcript often does not include tool responses. A tool can return HTTP 200 and still look failed there.

Confirm against real call logs for that Call ID: function about to run, API status, then the agent receiving the result.

Do not conclude a workflow failed from a scenario result alone.

A request that works in Postman but fails (or returns a plausible-wrong 200) from a workflow `apiCall` is almost always node shape (`headers`/`params` as `{key,value}[]`, no `queryParams`, body string + `bodyType`) — not firewall/DNS. See `manage-workflows`.

## Transfers and telephony

- Agent transferred when it should not have: read `promptForTransferReason` first (`get_agent_transfer_settings`), not only the system prompt. See `edit-voice-agents`.
- "I'll transfer" but nothing happens: Follow Prompt Instructions, destination mapping, extension SIP creds, hours block, `"transferring"` not in `endCallReasons`.
- Zero Call IDs in the window: traffic never reached Voice Logica → carrier / portability / PBX (`manage-phones`).
- One-way audio: codec / RTP / Edge UDP 51820, not the prompt.
- Test call busy + working phone: `seconds: 0` on the plan.

## Settings that do nothing

On Soniox agents, `waitForFullTranscription` and `silenceDurationMs` are inert. Turn-taking is `skip_turn` plus the system prompt, not those two boxes.

If a setting is missing in the UI, check Power View, then role, then plan feature, before calling it a bug.

Variables that render as `[name]` in analysis/email: check the VARIABLES block and the name scope (flat in prompts, `call.` in workflows). See `edit-voice-agents`.

## Quick route

1. Get a Call ID. If none exist for the window, stop and treat it as telephony routing.
2. Fetch the live agent and the call. Check `get_subscription` if the call never connected.
3. Check `toolCalls` for the tool you expected.
4. Compare prompt / transfer-prompt instructions to what happened.
5. Change one thing. Ask for a retest Call ID.
