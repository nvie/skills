---
name: say
description: >
  Speak a short sentence out loud using the macOS `say` command. Use when the
  user runs /say, or asks to "say it out loud", "read this to me", "speak",
  or "tell me out loud". Also used by the notify skill to deliver audible
  updates. One plain English sentence at most -- never paragraphs, links,
  code, long numbers, or technical data.
---

# Say

Turn whatever the user asks for into one short sentence and speak it.

## The one hard rule

**One spoken sentence, the way a person would say it out loud.**

If it can't be said in one breath, it's too long. Pick the single thing that
matters and drop the rest.

## What a sentence may contain

Only plain English words a human would speak to a colleague.

| Don't speak                                  | Speak instead                       |
| -------------------------------------------- | ----------------------------------- |
| `https://github.com/nvie/skills/pull/42`     | "Your pull request is up."          |
| "Tests: 412 passed, 3 failed in 38.2s"       | "Three tests failed."               |
| "`TypeError: cannot read 'id' of undefined`" | "The build broke on a type error."  |
| "Deployed v2.14.3-rc.1 to fly-ams-prod-2"    | "The deploy to production is done." |
| "Commit 9ed04ac pushed to origin/main"       | "Your commit is pushed."            |

- No links, file paths, code, commands, hashes, version strings, or IDs.
- No long numbers. Small counts are fine ("three tests failed"); round or
  drop anything else ("about two minutes", not "127 seconds").
- No symbols or markdown that a speech engine would read literally or
  mangle. Write "and", not "&". Abbreviations people say out loud ("CI",
  "PR") are fine.
- No preamble ("Here's an update:"). Just the sentence.

## How to speak it

Run it with Bash, text in single quotes:

```sh
say 'The deploy to production is done.'
```

Avoid apostrophes in the sentence ("it is", not "it's") so the single quotes
stay intact. Don't pass a voice or rate; the user's system defaults apply.

When the user ran `/say` directly, don't print anything after speaking
besides the sentence itself. When another skill (like notify) speaks through
this one, that skill decides what goes in the transcript.

## Examples

**User:** "/say the tests finished"
→ `say 'The tests are done.'`

**User:** "/say summarize the PR review results" (review found two blocking
issues in the auth middleware and five nits)
→ `say 'The review found two blocking issues, both in the auth code.'`

## Checklist

- [ ] Exactly one sentence
- [ ] Sounds natural spoken aloud
- [ ] No links, paths, code, IDs, versions, or long numbers
- [ ] No apostrophes inside the single quotes
- [ ] On a direct `/say`, nothing printed except the sentence
