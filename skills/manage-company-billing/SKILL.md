---
name: manage-company-billing
description: List Voice Logica companies, users, plans, add-ons, payment method, and subscription, and update operating hours. Use when the user wants billing, plan upgrades, company switch, or open/closed hours.
---

# Manage company and billing

Report what the API returns. Do not guess prices or invent plan names. Never mention Voice Logica internal costs or margins.

## MCP tools

Company:

- `list_companies`
- `list_company_users`
- `get_company_details`
- `get_company_operating_hours` / `update_company_operating_hours`
- `switch_company`
- `request_company_access`
- `create_company`
- `get_user_companies_by_email`
- `get_associate` / `update_associate`

Billing:

- `upgrade_plan`
- `list_plans`
- `list_add_ons`
- `get_payment_method`
- `get_subscription`

Refunds, credit notes, invoices → billing@voicelogica.ai. Do not promise amounts or timelines.

## `companyId` override

`companyId` on these tools is **admin / reseller only**.

Normal users must omit it and operate on the current company. If a normal user passes another company's id, the call fails or hits the wrong tenant. If they need another company, use `list_companies` / `switch_company` / `request_company_access`.

## Check the subscription before calling a feature missing

UI gates are API-driven per company. `get_subscription` first.

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

A plan with `seconds: 0` (including a container / inactive plan) rejects calls as busy. Quote `endCallReason` from `get_calls`. Do not debug SIP first.

Shopify marketplace billing can disable in-app plan changes.

## Operating hours

Read `get_company_operating_hours` before writing.

- To close the whole week, use the **`closedAllWeek`** flag (or the API's equivalent closed-week field).
- Do **not** send an empty hours array to mean "closed". An empty array is an error, not "closed all week".
- `syncWithGoogle` only works when a **Google Place** is already linked to the company. If it is not linked, say so and update hours locally. Do not toggle sync and hope.

Hours also feed **Block transfer when company is closed** (`edit-voice-agents`).

## Common jobs

**"What plan are we on / what do we pay?"**
`get_subscription` + `list_plans` + `list_add_ons` + `get_payment_method`. Quote the API. Do not convert currencies or invent discounts.

**"Upgrade the plan."**
`list_plans` first. Show the current plan and the target. `upgrade_plan` only after the user confirms.

**"Switch company."**
`list_companies`, match by name, `switch_company`. Then list agents on the new company so later edits hit the right tenant.

**"Create a company / request access."**
`create_company` or `request_company_access` with the details they gave. Do not create a company because list was empty without asking.

**"Set hours / we are closed this week."**
Use `closedAllWeek` for a full close. For normal hours, send the full week the API expects, not a partial empty list.

## After a change

Say which company, what changed (plan, hours, switch), and that `companyId` was omitted unless they are admin/reseller.
