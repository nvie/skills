---
name: mand
description: >
  Shrink the previous message to about half its size, focused on what's still
  left to do. Each repeated call halves it again, trading details (function
  names, file paths, IDs) for the bigger picture, until only a single word is
  left. Takes an optional focus as an argument ("what happened", "the risks",
  "for Slack"). Use when the user runs /mand, or asks to "make it shorter",
  "shorter", "tl;dr", "what's left", or "give me the gist". Named after the
  Dutch "meneer Mandje" interview, where the story got shorter and shorter
  until all that was left was "Mand!".
---

# Mand!

The user juggles many sessions and comes back to walls of text. Give them the
same message, shorter, so they can pick up where they left off. Every call
shrinks it again, until one word is left.

## What to shrink

- **First call:** your most recent message with real content, from before
  this invocation.
- **Repeated call:** your previous /mand output. Each call builds on the last
  one, not on the original.

If there's no such message, say so in one line.

## Focus

**Default: what's left.** Drop everything that's done ("I fixed X", "tests
pass now", "I renamed Y") and keep only what's still open: decisions the user
has to make, actions they have to take, work that's unfinished, open
questions, known risks. A done item survives only when the user needs it to
act on what's left ("the fix is on a branch, not merged yet").

If nothing is left, say so: "Nothing left, it's done." On the next call, that
becomes "Done!".

**With an argument**, the argument sets the focus and replaces the default.
`/mand what happened` keeps the done parts and drops the rest. Any other
focus works the same way ("the risks", "just the decision").

The focus sticks: a bare /mand after `/mand what happened` keeps shrinking the
"what happened" version. A new argument switches the focus, starting from the
previous /mand output.

## How much to cut

**Each call halves the previous text**, measured in words. Rough ladder:

1. The original message
2. About half: short paragraphs or a handful of bullets
3. A few bullets
4. One or two sentences
5. One line
6. One or two words, the core noun of what's left, with an exclamation mark
   ("Migration!", "Auth review!")
7. "Mand!"

Short source messages start further down the ladder. Never output something
longer than its input.

Once at the one-word level, every further call answers just `Mand!`.

## Bigger picture as you go

Each step trades detail for overview. Drop the most specific details first:

- **Step 2:** drop function names, file paths, line numbers, commit hashes,
  ticket numbers, URLs, and exact numbers. Say what they are instead ("the auth
  middleware", "the migration script", "a few flaky tests").
- **Step 3 and beyond:** drop the how and the why. Keep the what.
- **The one-liner:** the single most important thing left, in plain words.

Only keep a precise detail when the user can't act without it, like the exact
command they have to run next. Even then, drop it by step 3.

## Output

The shortened text and nothing else. No preamble ("Shorter:"), no step
counter, no "Run /mand again to...". Plain markdown, matching what fits the
size: bullets for a few items, prose for a sentence.

## Example

**Original message:**

> Fixed the race in `src/sync/reconnect.ts:88` by awaiting `flushPending()`
> before reopening the socket, and added a regression test in
> `test/sync/reconnect.test.ts`. All 412 tests pass. I also renamed
> `retryCount` → `attempts` across the package.
>
> Still open:
>
> - The same pattern exists in `src/presence/heartbeat.ts`, but it's less
>   likely to trigger. Do you want me to fix that too, or file a ticket?
> - `CHANGELOG.md` needs an entry before you release.
> - CI on `ENG-1234` is red because of an unrelated flaky e2e test in
>   `comments.spec.ts`; rerunning it should do.

**`/mand`:**

> - Same race also in the presence heartbeat: fix it now or file a ticket?
> - Add a changelog entry before releasing.
> - CI is red from an unrelated flaky test; rerun it.

**`/mand`:**

> Decide on the heartbeat race, add a changelog entry, rerun CI.

**`/mand`:**

> Heartbeat race decision.

**`/mand`:**

> Heartbeat!

**`/mand`:**

> Mand!

**`/mand what happened`** on the original:

> Fixed the reconnect race and added a regression test, all tests pass.
> Also did a small rename across the package.

## Checklist

- [ ] Shrinks the previous /mand output when there is one, otherwise the last real message
- [ ] Default focus: only what's left; done items dropped
- [ ] Argument focus followed, and kept for later bare calls
- [ ] About half the words of the input, never longer
- [ ] Paths, names, IDs, URLs gone from step 2 on
- [ ] Nothing but the shortened text in the output
- [ ] "Mand!" once it can't get shorter
