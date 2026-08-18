---
name: manage-sms
description: Send SMS or Viber, manage sender IDs, and configure the send_sms agent tool. Use when the user wants texts from an agent, a new sender, verification, or garbled Greek / UTF-8 messages.
---

# Manage Voice Logica SMS and Viber

## MCP tools

- `send_sms`
- `get_sms`
- `list_sms_senders`
- `add_sms_sender`
- `verify_sms_sender`
- `send_viber`

Agent tool: `send_sms` on the agent (`list_agent_tools` / `update_agent_tool`) with a `senderId`.

Sender management in the app is **admin or account-manager**.

## 3-layer rule

1. Add and verify a sender (`add_sms_sender` → `verify_sms_sender`).
2. Set `senderId` on the agent's `send_sms` tool.
3. Test with a Call ID or a one-off `send_sms`, then `get_sms`. Always test to a **real phone**.

Never paste live sender tokens, provider passwords, or API keys into chat.

Phone numbers need a valid format (country code). Workflow `sendSms` nodes have the same requirement.

## Not sending vs garbled UTF-8

These are different failures. Do not treat them as one.

**Not sending** (no message, provider error, pending verify):

1. `list_sms_senders` — is the sender there and verified?
2. Is `senderId` set on the agent tool?
3. `get_sms` for the attempt — status and error.
4. Check the Call ID `toolCalls` if this was during a call.
5. If this is a workflow SMS: did execution reach the `sendSms` node? Any credit / balance issue?

**Garbled UTF-8** (Greek or other non-ASCII looks like `????` or mojibake):

The sender and route are working. The payload encoding is wrong. Resend with proper UTF-8 text. Do not rotate the sender or rewrite the prompt as the first step. English-fine / Greek-broken is the signature.

## Text the caller after a call

Two routes. Notification SMS (`analysisSmsConfig` / `phoneNumbers` / `sendSmsCondition: "evaluate_condition"`) goes to the **business**, not the caller — wrong feature for this ask.

**Route A — appointment was booked.** Built-in confirmation on appointment settings: `sendConfirmation`, `confirmationPrompt`, `confirmationSmsSenderId` (`manage-calendar-google-microsoft`). Sender fallback: appointment setting → agent `send_sms` sender → `VoiceLogica`. **Test a real call before promising it.**

**Route B — any other condition (proven).**

1. Add analysis properties (`update_agent_analysis_config` **replaces the whole array** — `get_agent_analysis_config` first). Typical pair: `booked` (boolean) + `messageToUser` (string). Types: `string | number | boolean | array | date | time | enum | object`. The property **description** is the model instruction.
2. `onCallEnd` workflow: `if` on `booked` → `sendSms` body `{{messageToUser}}`.
3. Analysis properties interpolate **flat**: `{{booked}}`, `{{messageToUser}}`. `{{call.postAnalysisResults.booked}}` does **not** work.

Analysis tab is Power View only — not a subscription gate.

Letting the model write `messageToUser` beats stitching date/time fields.

## Common jobs

**"The agent should text the caller."**
List senders, pick a verified one, set `senderId` on the agent `send_sms` tool, add a prompt rule for when to text, then test with a Call ID.

**"Add a sender."**
`add_sms_sender`, then `verify_sms_sender`. Do not send from an unverified sender.

**"Send a Viber message."**
Use `send_viber`. Confirm the destination and sender first. This is not `send_sms`.

## After a change

Say which sender ID is on which agent. Do not print tokens. Ask them to confirm the test SMS arrived.
