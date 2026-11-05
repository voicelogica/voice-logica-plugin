---
name: act-on-call-instructions
description: Act on the instructions team members leave on Voice Logica calls for an AI assistant (dictated on the phone right after the call, or typed on the call), then mark each one done with what was done. Use when the user asks to check or handle call instructions or after-call notes, asks "what did my team ask for after their calls", or when this runs on a schedule as a routine.
---

# Act on call instructions

After a call, a team member can leave instructions for an AI assistant on it. They dictate them on the phone once the client hangs up (or after dialling `##` to end only the client's line), or type them on the call under **Instructions for AI**. Each one stays `pending` until an assistant acts on it. You are that assistant.

Confirm the MCP is connected (Connect / OAuth to https://app.voicelogica.ai). Use an API key only if the client has no Connect flow.

## 1. Find the open ones

`get_calls` with `llmInstructionsStatus: "pending"` and `include: ["summary"]`, `limit` 10.

Each call returns `llmInstructions`:
- `text`: what the team member asked, in their own words;
- `source`: `voice` (dictated) or `manual` (typed);
- `createdByUserId`, `createdAt`.

It also returns `summary`, `contactId`, `phoneNumber`, `userId` / `userName` and `startAt`.

Nothing pending: say so in one line and stop. In a scheduled run, do nothing else.

## 2. For each call

1. **Read** the instructions with the summary. Dictated text is a speech transcript: names and numbers can be misheard, so check them against the summary, the contact (`get_associate` / `list_contacts`) or, if still unsure, the transcript (`get_calls` with `callId` and `include: ["transcript"]`).
2. **Do** what they ask with the tools you have. Typical jobs:
   - A callback or a reminder: `create_task` with `assignedUserId` = `createdByUserId` (the team member who left the instructions), `contactId`, `sourceCallId` = the call's `id`, `kind: "callback"` for a call back, and `dueAt` when they named a time. Look at `list_tasks` first so you don't add a second one.
   - Remember something about the client: `create_or_update_contact` (pass `contactId`). For a connected CRM (HubSpot, Zoho, Salesforce…), use its integration tools (`list_integrations`, then `describe_integration`).
   - An offer, a summary or a link for the client: draft it with `get_call_follow_up`. To send it, use `send_call_follow_up`, or `messages_execute` for SMS, Viber or email.
   - A meeting: find a slot with the calendar tools. A booking that invites the client counts as reaching the client (see the rules below).
3. **Mark it.** `update_call_llm_instructions` with `callId`, `action: "done"` and `result`. The result is one or two lines in the language of the instructions, saying what you did, e.g. «Δημιούργησα εργασία για τη Μαρία: επανάκληση Τρίτη 10:00. Πρόχειρο email με την προσφορά περιμένει έγκριση.».
   - Nothing to do (a note only for the record): `action: "dismiss"` with a short `result`.
   - You could not do it (missing information, no tool for it): `create_task` (assigned as above) that says what is missing, then mark `done` with a result that says so. Never leave an instruction pending: the next run would try it again.

## Rules

- **Nothing reaches the client without a person's OK.** That covers email, SMS, Viber, a call, or a booking that invites them. When someone is in this chat, show the draft and ask. In a scheduled run nobody is there to ask: create a task for the team member with the draft and what to approve, and mark the instructions `done` with "draft ready, waiting for your OK".
- Mark every call you handled, so no run handles it twice. Do not change instructions that are `done` or `dismissed` unless the user asks (`action: "reopen"`).
- Do not invent IDs, numbers, prices or dates. When the instructions are unclear, ask through a task to the person who left them.
- Keep the client's data in Voice Logica: do not paste transcripts or phone numbers into tasks or results beyond what the job needs.
- One short report at the end: how many instructions you handled, and one line for each (call, client, what you did).

## Run it on its own

Every 15 to 30 minutes during office hours is enough. The prompt for the scheduled run:

> Use the act-on-call-instructions skill: handle the pending Voice Logica call instructions.

- **Claude**: create a scheduled task with that prompt (Voice Logica connector on).
- **ChatGPT**: create a task with that prompt and the schedule.
- **Claude Code**: `/schedule` with that prompt.

Each run sees only the calls of the company the account belongs to, and the member's permissions in the app still apply.

Help: https://docs.voicelogica.ai/integrations/claude-calls-and-team/#instructions-for-ai-after-a-call
