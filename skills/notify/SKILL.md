---
name: notify
description: >
  Audibly notify the user with one short spoken update when work finishes or
  something interesting becomes known: a background job's results are in, a
  file watcher triggered, a polling loop found what it was waiting for, a
  long build or test run finished, a notification arrived. Speaks via the say
  skill. Use when the user runs /notify, or asks to "notify me", "ping me",
  "let me know when", "tell me when it's done", or "give me a shout when".
---

# Notify

The user isn't watching the terminal. When the thing they care about happens,
tell them out loud in one sentence, so they know whether to come back.

## When to speak

Once the user has asked to be notified, speak when:

- A background job, build, test run, or deploy finishes (success **or**
  failure)
- A watcher or polling loop finds what it was waiting for
- Something you were waiting on comes in (a CI result, a notification, a
  reply)
- You're blocked and need the user's input to continue

Don't speak for intermediate progress, routine steps, or "still waiting".
One notification per event. If several things finish at once, combine them
into one sentence.

If the user runs `/notify` with no pending work in sight, ask what they want
to be notified about.

## What to say

The outcome, not the process. Lead with what the user would act on.

- "The build passed." / "The build failed on a type error."
- "The deploy is live."
- "The watcher picked up a change in the config file."
- "CI is green, your branch is ready to merge."
- "I need your input on the database migration."

## How to speak it

Invoke the `say` skill with the sentence. It owns the spoken-sentence rules
(one sentence, no links, code, or long numbers) and runs `say`.

Also write the full details in the transcript as usual. The spoken sentence
is the alert; the transcript is where the user reads what happened.

## Checklist

- [ ] Spoke only for an event worth coming back for
- [ ] One sentence, outcome first
- [ ] Failures notified just like successes
- [ ] Details written in the transcript, not spoken
