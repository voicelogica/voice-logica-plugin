---
name: manage-calendar-google-microsoft
description: Connect Google or Microsoft calendar, contacts, and Google Chat, and configure Voice Logica appointment settings. Use when the user wants bookings, availability, contacts, Outlook, Meet, or Chat on an agent.
---

# Manage Google and Microsoft calendar

Calendar, video meetings, and chat are three different products. Do not treat a working Calendar OAuth as Meet or Chat.

- **Calendar** = events and free/busy
- **Meet** = video meeting links (not created just because Calendar is connected)
- **Chat** = Google Chat spaces and messages (`connect_google_chat` and the Chat tools)
- **Outlook / Microsoft** calendar and contacts exist as their own connection. Do not tell the user they must use Google.

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

## 3-layer rule

1. Connect Google or Microsoft on the company (OAuth).
2. Enable `schedule_appointment` (and Chat tools if they asked for Chat).
3. Test with a Call ID. Confirm the event exists on the calendar, not only that the agent said "booked".

If OAuth shows `needsReconnect`, disconnect and reconnect. Never edit the prompt to fix a dead token. Never paste live OAuth tokens into chat.

## Holiday Busy Event is not Out-of-office

A **Holiday Busy Event** blocks time on the calendar. It is not an Out-of-office / auto-reply state. Do not configure one when the user asked for OOO, and do not tell them a busy holiday event will send an away message.

## Common jobs

**"The agent should book appointments."**
Connect Google Calendar, enable `schedule_appointment`, read `get_agent_appointment_settings`, update hours / calendar / duration if needed, then test with a Call ID and `list_calendar_events`.

**"Show my calendars / upcoming events."**
Use `list_calendars` then `list_calendar_events`. Do not invent event IDs.

**"Send a Google Chat message."**
`connect_google_chat` first if Chat is not connected. Then list spaces, then send. Calendar OAuth alone is not Chat.

**"We use Outlook, not Google."**
Connect Microsoft / Outlook. Do not force a Google reconnect. `schedule_appointment` is the Google Calendar agent tool — say so if they need agent-side booking on Outlook.

## After a change

Say which provider, which calendar, and whether `schedule_appointment` is on. Ask for a test booking Call ID.
