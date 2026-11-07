---
name: act-on-call-instructions
description: "Carry out the instructions team members leave on Voice Logica calls for an AI assistant (dictated on the phone right after the call, or typed on the call): send the email or SMS they asked for, create the task, book the meeting, update the contact, then mark each one done with what was done. Use when the user asks to check or handle call instructions or after-call notes, asks \"what did my team ask for after their calls\", or when this runs on a schedule as a routine."
---

# Act on call instructions

After a call, a team member can leave instructions for an AI assistant on it. They dictate them on the phone once the client hangs up (or after dialling `##` to end only the client's line), or type them on the call under **Instructions for AI**. Each one stays `pending` until an assistant acts on it. You are that assistant: the team member decides, you carry it out.

Confirm the MCP is connected (Connect / OAuth to https://app.voicelogica.ai). Use an API key only if the client has no Connect flow. If `update_call_llm_instructions` or the `llmInstructionsStatus` filter of `get_calls` is missing, the connector was connected before they existed: disconnect and reconnect it.

## 1. Find the open ones

`get_calls` with `llmInstructionsStatus: "pending"` and `include: ["summary"]`, `limit` 10. The filter returns only instructions nobody acted on yet, so this one call is the whole check.

Each call returns `llmInstructions`:
- `text`: what the team member asked, in their own words;
- `source`: `voice` (dictated) or `manual` (typed);
- `createdByUserId`, `createdAt`.

It also returns `summary`, `contactId`, `phoneNumber`, `userId` / `userName` and `startAt`.

Nothing pending: say so in one line and stop. In a scheduled run, do nothing else: no other tool call, no task list. Most runs end here, and every extra step costs a full turn.

Something pending: load every tool the jobs may need **in one go** (`update_call_llm_instructions`, `get_associate`, `list_contacts`, `messages_execute`, `send_call_follow_up`, `create_task`, `list_tasks`, `create_or_update_contact`), not one at a time. Work from the summary and the contact. Fetch a transcript only when they do not settle a detail, and then for all such calls at once (`get_calls` with `callIds` and `include: ["transcript"]`). Never fetch the same call twice.

## 2. For each call: do what the instructions say

Do exactly what the team member asked, with the tools you have. Do not wait for anyone's approval: that includes sending to the client.

1. **Read** the instructions with the summary. Dictated text is a speech transcript: check names, numbers, amounts and addresses against the summary, the contact (`get_associate` / `list_contacts`) or, if still unsure, the transcript (`get_calls` with `callId` and `include: ["transcript"]`).
2. **Do** it. Typical jobs:
   - Email the client: find the address (the call's contact, a connected CRM, the transcript). Write it in the language of the call, short and polite, with exactly the content asked for. Send it with `messages_execute` (action `send_email`, to the `contactId` so it is logged on the contact), or `send_call_follow_up` when it is the call summary or page.
   - SMS or Viber: `messages_execute` (`send_sms` / `send_viber`) to the call's `phoneNumber` or the contact's mobile.
   - Use these inputs exactly; you do not need to look them up first:
     - SMS: `messages_execute { action: "send_sms", input: { messages: [{ to: ["+3069…"], body: "…" }] } }`
     - Viber: `messages_execute { action: "send_viber", input: { to: "+3069…", text: "…", smsFallback: true } }`
     - Email: `messages_execute { action: "send_email", input: { messages: [{ to: ["name@example.com"], subject: "…", body: "…", contactId: "<the call's contactId>" }] } }`
     - Phone numbers in E.164 (a Greek mobile `69…` becomes `+3069…`).
   - Put independent steps of the same job (e.g. the SMS and the callback task) in the same turn.
   - A callback or a reminder: `create_task` with `assignedUserId` = `createdByUserId`, `contactId`, `sourceCallId` = the call's `id`, `kind: "callback"` for a call back, and `dueAt` when they named a time. Look at `list_tasks` first so you don't add a second one.
   - A meeting: the calendar tools, at the time they said (or the first free slot that matches it).
   - Remember something about the client: `create_or_update_contact` (pass `contactId`). For a connected CRM (HubSpot, Zoho, Salesforce…), use its integration tools (`list_integrations`, then `describe_integration`).
   - Anything else they ask that a Voice Logica tool can do.
3. **Mark it.** `update_call_llm_instructions` with `callId`, `action: "done"` and `result`. The result is one or two lines in the language of the instructions, saying exactly what you did, e.g. «Στάλθηκε email στο giorgos@example.com με την προσφορά. Εργασία επανάκλησης για τη Μαρία, Τρίτη 10:00.».
   - Nothing to do (a note only for the record): `action: "dismiss"` with a short `result`.

## When you must not guess

Only these stop you, and then you send nothing for that item:
- you cannot tell who the recipient is, or there is no email or phone for them;
- a name, number, amount, date or address you would send is unclear in the dictation, and the summary, contact or transcript does not settle it;
- no tool can do what they asked.

Then `create_task` (assigned as above) that says exactly what is missing, with a ready draft when there is one. Do the rest of the instructions you safely can, and mark them `done` with a result that says what was done and what is waiting. Never leave an instruction pending: the next run would do it again.

## Rules

- Mark every call you handled, so no run handles it twice. Do not change instructions that are `done` or `dismissed` unless the user asks (`action: "reopen"`).
- Never invent IDs, numbers, prices or dates.
- Do not paste transcripts or phone numbers into tasks or results beyond what the job needs.
- One short report at the end: how many instructions you handled, and one line for each (call, client, what you did, what you sent and to whom).

## Run it on its own

Every 10 to 30 minutes is enough. The prompt for the scheduled run:

> Use the act-on-call-instructions skill: handle the pending Voice Logica call instructions.

- **Claude**: create a scheduled task with that prompt (Voice Logica connector on). Run it once by hand first and allow the Voice Logica tools it uses, so later runs do not stop to ask.
- **ChatGPT**: create a task with that prompt and the schedule.
- **Claude Code**: `/schedule` with that prompt.

Each run sees only the calls of the company the account belongs to, and the member's permissions in the app still apply.

Help: https://docs.voicelogica.ai/integrations/claude-calls-and-team/#instructions-for-ai-after-a-call
