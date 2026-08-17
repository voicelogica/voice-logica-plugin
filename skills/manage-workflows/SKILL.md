---
name: manage-workflows
description: Create, edit, test, version, and run Voice Logica workflows, including triggers, nodes, switch cases, and agent workflow assignment. Use when the user wants onCallEnd, API, SMS, email, or branching automation.
---

# Manage Voice Logica workflows

Use workflow MCP tools. Do not invent workflow IDs, node IDs, or trigger names.

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

## `apiCall` shape (easy to get wrong)

`headers` and `params` are **lists of `{key, value}`**, not JSON maps.

```
headers: [{key: "Authorization", value: "Bearer …"}]
params:  [{key: "orderId", value: "{{orderId}}"}]
```

Do not send `{"Authorization": "Bearer …"}`.

Query string values go in `params`, **not** `queryParams`. There is no `queryParams` field. Putting search params in the URL path or in a made-up `queryParams` object will fail.

Never paste live API tokens into chat. If a header needs a secret, keep it in the workflow form, not in the transcript.

## How to edit

1. `list_workflows` or `get_agent_workflows`. Match by name. Use the real ID.
2. `get_workflow` before changing nodes.
3. Add or update one trigger or node, then connect it.
4. `test_workflow` (or `run_workflow` when they asked to run it).
5. `save_workflow_version` after a working change.

Do not delete a workflow to "start over" unless the user asked.

## Common jobs

**"Email me after every call."**
`onCallEnd` + `emailConfig`. Confirm the mailbox / destination. Test on a real Call ID.

**"Call our API when the call ends."**
`onCallEnd` + `apiCall` with list-shaped `headers` / `params`.

**"This agent should use workflow X."**
`get_agent_workflows`, then `update_agent_workflows`. Connecting a workflow at company level does not attach it to the agent.

## After a change

Say which workflow, which trigger, and which nodes changed. Ask them to test and keep a version if it works.
