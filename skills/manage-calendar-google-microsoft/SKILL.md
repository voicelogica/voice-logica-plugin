---
name: manage-calendar-google-microsoft
description: Connect Google or Microsoft calendar, contacts, and Google Chat, and configure Voice Logica appointment settings. Use when the user wants bookings, availability, contacts, Outlook, Meet, or Chat on an agent.
---

# Manage Google and Microsoft calendar

Calendar, video meetings, and chat are three different products. Do not treat a working Calendar OAuth as Meet or Chat.

- **Calendar** = events and free/busy. Google Calendar is configured on the agent **Appointments** tab (company Google login) — not an Integrations card. Needs plan feature `google-calendar` plus the `schedule_appointment` tool.
- **Meet** = video meeting links / recording ingest (Integrations → Google Meet). Not created just because Calendar is connected, and it does not fix booking.
- **Chat** = Google Chat spaces and messages (`connect_google_chat` and the Chat tools). Calendar OAuth alone is not Chat.
- **Outlook / Microsoft 365** calendar and contacts exist as their own connection (`outlook` agent tool). Do not tell the user they must use Google.

## MCP tools

Contacts:

- `list_contacts`
- `create_or_update_contact`

Calendar:

- `list_calendars`
- `list_calendar_events`
- `create_calendar_event`
- `update_calendar_event`

Google Chat (only after `connect_google_chat`):

- `connect_google_chat`
- `list_google_chat_spaces`
- `send_google_chat_message`
- `list_google_chat_members`
- `get_google_chat_messages`

Agent appointment config:

- `get_agent_appointment_settings`
- `update_agent_appointment_settings`

The agent-side booking tool is `schedule_appointment`. It uses **Google Calendar**. Enable it on the agent after Calendar is connected.

Limits: appointment future days 1–365; caller verification attempts 1–10; verification TTL 60–900s.

## 3-layer rule

1. Connect Google or Microsoft on the company (OAuth). Some Microsoft tenants need admin consent.
2. Enable `schedule_appointment` (and Chat tools if they asked for Chat).
3. Test with a Call ID. Confirm the event exists on the calendar, not only that the agent said "booked".

If OAuth shows `needsReconnect`, disconnect and reconnect. Never edit the prompt to fix a dead token. Never paste live OAuth tokens into chat.

## Holiday Busy Event is not Out-of-office

To stop bookings for a closure, create an **all-day Event** (not Out-of-office) on the **same calendar the agent books into**, date range start → last closed day, **Availability = Busy**. All-day events default to **Free**, which blocks nothing — that is the usual "I blocked it and it still books". Out-of-office is a Workspace auto-decline feature and may be permission-blocked; it does not drive our slot search.

## Appointment confirmation SMS

Appointment settings can text the **caller** after a booking:

- `sendConfirmation` — on
- `confirmationPrompt` — message text
- `confirmationSmsSenderId` — sender shown on the SMS

Sender fallback: appointment setting → the agent's `send_sms` sender → `VoiceLogica`. Recipient is the caller's number from the call; if no phone was captured, nothing is sent.

Treat this as **unproven until a real test call** — set it up, place a call, confirm the SMS arrived. If nothing arrives, use the analysis-property + `onCallEnd` workflow route in `manage-sms`.

## Common jobs

**"The agent should book appointments."**
Connect Google Calendar, enable `schedule_appointment`, read `get_agent_appointment_settings`, update hours / calendar / duration if needed, then test with a Call ID and `list_calendar_events`.

**"Show my calendars / upcoming events."**
Use `list_calendars` then `list_calendar_events`. Do not invent event IDs.

**"Send a Google Chat message."**
`connect_google_chat` first if Chat is not connected. Then list spaces, then send. Calendar OAuth alone is not Chat. Chat API must be enabled and the account must be Google Workspace for many orgs.

**"We use Outlook, not Google."**
Connect Microsoft / Outlook. Do not force a Google reconnect. `schedule_appointment` is the Google Calendar agent tool — say so if they need agent-side booking on Outlook.

## After a change

Say which provider, which calendar, and whether `schedule_appointment` is on. Ask for a test booking Call ID.
