---
name: manage-company-billing
description: Manage the Voice Logica plan, add-ons, payment method, subscription, team members, seats, and operating hours. Use when the user wants billing, plan changes, low-balance alerts, inviting members, seats, or open/closed hours.
---

# Manage company and billing

Report what the API returns. Do not guess prices or invent plan names. Never mention Voice Logica internal costs or margins.

## MCP tools

Company:

- `get_company_operating_hours` / `update_company_operating_hours`
- `team_query` (actions `list_company_users`, `list_invitations`, `get_seats`)
- `team_execute` (actions `invite_member`, `revoke_invitation`, `set_user_seat`, `request_company_access`, `buy_seats` — charges the card, confirm first)
- `query_user_action` / `execute_user_action` for members
- `get_caller_card_settings` / `update_caller_card_settings`
- `get_mcp_settings` / `update_mcp_settings`
- `get_associate`
- `list_companies` / `switch_company` — admins, resellers, and account managers only

Staff-only tools are not in the public plugin.

Billing:

- `upgrade_plan`
- `list_plans`
- `billing_query` (actions `get_subscription`, `list_add_ons`, `get_payment_method`, `get_low_balance_alert`)
- `billing_execute` (actions `cancel_subscription`, `resume_subscription`, `set_company_vat`, `set_low_balance_alert`)

Any `<area>_query` called with no action lists that area's actions and inputs. Do that instead of guessing action names.

Refunds, credit notes, invoices → billing@voicelogica.ai. Do not promise amounts or timelines.

## `companyId` override

`companyId` on these tools is **admin / reseller only**.

Normal users must omit it and operate on the current company. If a normal user passes another company's id, the call fails or hits the wrong tenant. If they need another company, use `team_execute` (action `request_company_access`), or `list_companies` / `switch_company` if they are admin / reseller / account manager.

## Check the subscription before calling a feature missing

UI gates are API-driven per company. `billing_query` (action `get_subscription`) first.

| Symptom | Feature / field to check |
| --- | --- |
| No Outbound tab | `allow-outbound-calls` |
| No Appointments tab | `google-calendar` + `schedule_appointment` enabled |
| Transfers locked / billing link | `transfer-calls` / `did` — Demo, No Plan, Inactive often locked |
| Can't pick model | `can-select-model` (otherwise model in a call dump is a platform default) |
| Prompt won't save | numeric `system-prompt` |
| Knowledge Q&A cap | numeric `knowledge-questions` → `knowledge_questions_limit_reached` |
| Can't add / copy agent | numeric `agents` |
| Concurrent call limit | `concurrent-calls` |
| Test call = busy | `seconds: 0` or inactive subscription — `endCallReason` will say so |
| Workflows / Tunnels menu missing | **admin role**, not a plan flag |
| Widget tab missing | super-admin / account-manager — expected for a normal client |

Minute pools are separate: AI minutes (agent talk time), analysis minutes (post-call / post-transfer analysis), and **telephony provider credits** (PSTN outbound / failover). Attended transfer adds an extra outbound leg that can use AI minutes.

Inbound minutes on a plan do **not** include unlimited PSTN outbound. Outbound needs `allow-outbound-calls` **and** provider credits (`manage-calls-campaigns`).

A plan with `seconds: 0` (including a container / inactive plan) rejects calls as busy. Quote `endCallReason` from `get_calls`. Do not debug SIP first.

Reseller / new client company on a container plan with `seconds: 0` is normal until the **reseller** assigns a real plan. Do not upgrade or activate a client's plan for them — that starts billing. Give them the path and let them do it.

Shopify marketplace billing can disable in-app plan changes.

## Operating hours

Read `get_company_operating_hours` before writing.

- To close the whole week, use the **`closedAllWeek`** flag (or the API's equivalent closed-week field).
- Do **not** send an empty hours array to mean "closed". An empty array is an error, not "closed all week".
- `syncWithGoogle` only works when a **Google Place** is already linked to the company. If it is not linked, say so and update hours locally. Do not toggle sync and hope.

Hours also feed **Block transfer when company is closed** (`edit-voice-agents`).

## Common jobs

**"What plan are we on / what do we pay?"**
`billing_query` (action `get_subscription`) + `list_plans` + `billing_query` (action `list_add_ons`) + `billing_query` (action `get_payment_method`). Quote the API. Do not convert currencies or invent discounts.

**"Upgrade the plan."**
`list_plans` first. Show the current plan and the target. `upgrade_plan` only after the user confirms.

**"Switch company."** (admin / reseller / account manager)
`list_companies`, match by name, `switch_company`. Then list agents on the new company so later edits hit the right tenant.

**"Request access to a company."**
`team_execute` (action `request_company_access`) with the details they gave. New companies come from signup at https://app.voicelogica.ai.

**"Set hours / we are closed this week."**
Use `closedAllWeek` for a full close. For normal hours, send the full week the API expects, not a partial empty list.

## After a change

Say which company, what changed (plan, hours, switch), and that `companyId` was omitted unless they are admin/reseller.
