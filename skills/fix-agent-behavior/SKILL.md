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

## Common fixes

**Off-topic callers.** Add an early rule: detect wrong-industry intent, say you cannot help, end the call without collecting contacts.

**Too many questions.** Collect only fields required for that branch. A message-only path often needs name + callback number. "Anything else?" delays end-call; if they want it gone, use End Call config for that branch.

**Re-asks.** Add "never re-ask if already provided in this call". Also check language mismatch and interruption / skip_turn.

**Wrong language.** Confirm languages enabled on the agent, put a follow-the-caller-language rule at the top of the prompt, then retest.

**Greeting restarts.** Deduplicate welcome message vs prompt greeting. Add a resume-after-failed-transfer rule.

**Ends call too early or says goodbye then continues.** End-call reasons must match real completion states. Never put "transferring" in endCallReasons. Finish message-taking before end-call.

**Interrupts the caller.** Do not hunt a silence-threshold knob on Soniox agents. Check whether `skip_turn` fired in toolCalls. If it never fired, a "do this immediately" system-prompt rule is usually winning. Subordinate that rule: immediately means after the caller finishes, and skip_turn is evaluated first. Only then change `timeoutSeconds` (1 to 10).

**Mishears vs says a word wrong.** Opposite fixes.

- Agent mishears the caller: Transcription tab, keywords / adaptation phrases (STT).
- Agent says the word wrong: Pronunciation Guide (TTS). Text to replace = mangled written form. Replacement = how it should sound. Do not "fix" pronunciation by rewriting the word inside the system prompt.

**Fabricates bookings or payments.** Add a hard constraint: never invent success. Verify the tool actually ran.

**Loops.** Conflicting prompt rules, echo, or tool failure. Fix the root cause with a Call ID, not a vague "stop looping" line.

**Out-of-hours transfers.** Enable business hours, block transfer when closed, use an off-hours greeting or message flow.

## After a change

Say what you changed, ask them to retest with a new Call ID, and keep one issue per pass.
