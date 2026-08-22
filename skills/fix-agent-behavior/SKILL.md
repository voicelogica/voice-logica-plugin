---
name: fix-agent-behavior
description: Fix Voice Logica agent behavior on live calls: off-topic callers, too many questions, re-asks, wrong language, abrupt hangups, interruptions, misheard names, and wrong pronunciation. Use when a call went badly and the user wants the agent to behave differently.
---

# Fix Voice Logica agent behavior

Most "the platform is down" reports are prompt or config tweaks. Read the live agent and a Call ID before changing anything.

## Start with

1. Call ID
2. Expected vs actual
3. Language of the call
4. Whether this used to work

Then decide: the agent broke a written rule, followed a rule the user now wants changed, or the prompt is silent. Those need different edits.

If the rule is already in the prompt and was violated, restating it again wastes `system-prompt` budget (`get_subscription` before adding text). Fix the mechanism (tool, transfer prompt, end-call, language) instead.

If `get_agent_prompt` is approaching or above ~20,000 characters, do **not** add more rules. Move facts to knowledge and trim first (`edit-voice-agents`). A 30k+ prompt needs router + specialist, not another paragraph.

If the agent speaks JSON, tool names, or brackets out loud, collect Call ID + timestamp and file a ticket (`create-support-ticket`). Do not try to prompt-paper over leaked system content.

## Common fixes

**Off-topic callers.** Add an early rule: detect wrong-industry intent, say you cannot help, end the call without collecting contacts.

**Too many questions.** Collect only fields required for that branch. A message-only path often needs name + callback number. "Anything else?" delays end-call; if they want it gone, use End Call config for that branch.

**Re-asks.** Add "never re-ask if already provided in this call". Also check language mismatch and interruption / `skip_turn`.

**Wrong language.** Confirm languages enabled on the agent, put a follow-the-caller-language rule at the top of the prompt, then retest.

**Greeting restarts.** Deduplicate welcome message vs prompt greeting. Add a resume-after-failed-transfer rule.

**Ends call too early or says goodbye then continues.** End-call reasons must match real completion states. Never put `"transferring"` in `endCallReasons`. Finish message-taking before end-call.

**Interrupts the caller.** Do not hunt a silence-threshold knob on Soniox agents (`waitForFullTranscription` and `silenceDurationMs` are inert). `skip_turn` is the turn-taking lever. Knobs: the tool prompt and `timeoutSeconds` (1 to 10).

Check `toolCalls` first:

- `skip_turn` fired too early/late → `timeoutSeconds` or the tool prompt.
- `skip_turn` never fired → a system-prompt "do this immediately" rule is usually winning. Subordinate that rule: immediately means after the caller finishes, and `skip_turn` is evaluated first.

`skip_turn` must fire at decision points, not only mid-sentence: immediately **before** a clarifying question, immediately **before** a routing tool call, and when the last utterance is a **fragment** (bare name, one-word answer).

If the transcript shows the agent talking to itself, check echo / enable `lock_on_speaker`.

**Mishears vs says a word wrong.** Opposite fixes.

- Agent mishears the caller: Transcription tab, keywords / adaptation phrases (STT). Confirm digits one-by-one for phone numbers.
- Agent says the word wrong: Pronunciation Guide (TTS). Text to replace = mangled written form. Replacement = how it should sound. Do not "fix" pronunciation by rewriting the word inside the system prompt.

**Fabricates bookings or payments.** Add a hard constraint: never invent success. Verify the tool actually ran.

**Loops.** Conflicting prompt rules, echo, or tool failure. Fix the root cause with a Call ID, not a vague "stop looping" line.

**Out-of-hours transfers.** Enable business hours, block transfer when closed, use an off-hours greeting or message flow. Also read the transfer settings prompt first (`edit-voice-agents`) — a bare number table will dial as soon as a person is identified.

**Transferred when it should not have.** Start at `promptForTransferReason` (`get_agent_transfer_settings`), not the system prompt. See `edit-voice-agents`. Attended / warm failures (SIP REFER) are telephony — switch to blind; do not add more transfer prose.

## After a change

Say what you changed, ask them to retest with a new Call ID, and keep one issue per pass. `save_agent_version` after a working change.
