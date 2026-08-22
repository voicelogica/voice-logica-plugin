---
name: manage-workflows
description: Create, edit, test, version, and run Voice Logica workflows, including triggers, nodes, switch cases, and agent workflow assignment. Use when the user wants onCallEnd, API, SMS, email, or branching automation.
---

# Manage Voice Logica workflows

Use workflow MCP tools. Do not invent workflow IDs, node IDs, or trigger names.

The Workflows menu in the app is **admin role**. MCP workflow tools still follow whatever access the connected user has. Tunnels (for private HTTP ERPs) are also admin-only.

## MCP tools

Workflow CRUD and run:

- `list_workflows` / `get_workflows` / `get_workflow`
- `create_workflow` / `update_workflow` / `delete_workflow`
- `run_workflow` / `test_workflow`

Triggers:

- `add_workflow_trigger` / `update_workflow_trigger` / `remove_workflow_trigger`

Nodes and graph:

- `add_workflow_node` / `update_workflow_node` / `remove_workflow_node`
- `add_switch_case`
- `connect_workflow_nodes` / `disconnect_workflow_nodes`

Versions:

- `list_workflow_versions` / `save_workflow_version` / `delete_workflow_version`

Assign to an agent:

- `get_agent_workflows` / `update_agent_workflows`

## Triggers

- `onAPICall` — HTTP into the workflow
- `onCallEnd` — after the call
- `onCallStart` — when the call starts
- `onKeywordDetected` — keyword during the call
- `duringCall` — mid-call automation
- `onEmailReceived` — inbound mailbox email

## Nodes

- `emailConfig` — send email
- `apiCall` — HTTP request
- `llmCompletion` — model step
- `sendSms` — SMS
- `codeExecution` — script step
- `if` — boolean branch
- `switch` — multi-way branch (`add_switch_case`)
- `loop` — iterate

Limits: table nodes max 500 rows / 50 cols. Call-trigger function name max 64; condition max 500.

## `apiCall` shape (easy to get wrong)

Canonical shape:

- `headers` and `params` are **lists of `{key, value}`**, not JSON maps
- Query string values go in `params`, **not** `queryParams` (there is no `queryParams` field)
- Body is a **raw JSON string** plus `bodyType`

```
headers: [{key: "Authorization", value: "Bearer …"}]
params:  [{key: "orderId", value: "{{orderId}}"}]
```

Do not send `{"Authorization": "Bearer …"}`.

If a request works in Postman but fails or returns a plausible-wrong 200 from the node, check this shape first — not firewall or DNS. Other working integrations on the same company already kill the network theory.

Never paste live API tokens into chat. If a header needs a secret, keep it in the workflow form, not in the transcript.

## Variable names in workflows

Workflow nodes use **`call.`-prefixed** names: `{{call.phone_number}}`, `{{call.company_phone_number}}`, `{{call.summary}}`, `{{call.status}}`, `{{call.duration}}`, `{{call.end_call_reason}}`, `{{call.call_dialogue}}`, `{{call.call_url}}`, `{{call.recording_url}}`, `{{call.id}}`, `{{call.agent_id}}`.

Post-call **analysis properties** interpolate **flat** even inside an `onCallEnd` SMS/email node: `{{booked}}`, `{{messageToUser}}` — not `{{call.postAnalysisResults.booked}}`.

Prompt / analysis scope is the opposite (flat `{{phone_number}}`). See `edit-voice-agents`.

## How to edit

1. `list_workflows` or `get_agent_workflows`. Match by name. Use the real ID.
2. `get_workflow` before changing nodes.
3. Add or update one trigger or node, then connect it.
4. `test_workflow` (or `run_workflow` when they asked to run it).
5. `save_workflow_version` after a working change.

`test_workflow` is worthless if you hand-feed the trigger payload. Test with **nothing hand-fed** so the runtime resolves variables — or the first live call finds it.

Do not delete a workflow to "start over" unless the user asked.

Write Greek literally in JSON-string node params. `\uXXXX` escapes can drop letters. Re-read the node.

## Common jobs

**"Email me after every call."**
`onCallEnd` + `emailConfig`. Also check built-in Analysis + Notifications — those are a different sender. Confirm the mailbox / destination. Test on a real Call ID.

**"Call our API when the call ends."**
`onCallEnd` + `apiCall` with list-shaped `headers` / `params` and a string body + `bodyType`.

**"This agent should use workflow X."**
`get_agent_workflows`, then `update_agent_workflows`. Connecting a workflow at company level does not attach it to the agent.

**"Text the caller only if they booked" (or any other condition).**
Proven route: add analysis properties on the agent (`update_agent_analysis_config` — **replaces the whole array**, so `get_agent_analysis_config` first), then `onCallEnd` → `if` on the boolean → `sendSms` with `{{messageToUser}}`. See `manage-sms`. A `duringCall` node that returns text does **not** block transfers (`edit-voice-agents`).

**SMS node not sending.**
Confirm the workflow reached the node, the phone has a country code, the sender is verified (`manage-sms`), and Greek / non-ASCII text is UTF-8. Test to a real phone.

## After a change

Say which workflow, which trigger, and which nodes changed. Ask them to test and keep a version if it works.
