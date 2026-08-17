---
name: edit-voice-agents
description: Edit Voice Logica agent settings, prompts, knowledge files, and editor tabs. Use when the user wants to change a greeting, system prompt, languages, transfers, or cannot find a setting in the agent editor.
---

# Edit Voice Logica agents

Use Voice Logica MCP tools to read the live agent first, then change only the fields the user asked for.

## Find the setting

Many "missing setting" cases are view, role, or plan, not a missing feature.

- **Business View** shows only AI Assistant + Languages.
- **Power View** shows the full tab set. Switch the agent to Power View before calling something missing.
- Prompt-based agents in Power View typically have: Details, Tools and Workflows, Languages and Voices, Analysis, Notifications, Memory, Transcription, Transfer Connection, End Call, Advanced.
- Hidden unless plan or role allows: Outbound, Widget, Tests, Customer Instruction, Appointments, E-shop.

Where things live:

- System prompt and identity: Details (Power View)
- Call forwarding / attended vs blind: Transfer Connection
- Goodbye and end-call reasons: End Call
- Post-call email: Analysis + Notifications
- Knowledge usage (topN): Memory
- Hard words / STT hints: Transcription
- Pronunciation (TTS): Pronunciation Guide on Details
- Outbound campaign settings: Outbound tab (plan-gated)
- Calendar booking: Appointments (needs calendar tool)

If no phone is connected, the agent shell usually points to VoIP phones or billing.

## Prompts vs knowledge files

Keep the system prompt around 15,000 to 20,000 characters.

Keep in the prompt: role, conversation flow, transfer rules, hard constraints, tone, language, short examples.

Move to knowledge files: prices, catalogs, addresses, hours, staff lists, long FAQs, policies, anything the agent should retrieve.

After adding knowledge: assign it to the agent, enable it, wait until embeddings are ready, then trim the prompt and test.

If the user says "I uploaded docs but the agent does not know", check assigned, enabled, still processing, and whether questions match the knowledge entries.

Do not put API keys or internal tool names in customer-facing prompt text.

## How to edit

1. List agents and match by name. Use the real ID.
2. Show the current value, then the proposed change.
3. Apply the change through MCP.
4. Confirm what changed and suggest a test call.
