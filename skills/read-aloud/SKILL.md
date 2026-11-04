---
name: read-aloud
description: >
  Read your last message out loud with the macOS `say` command, narrated the
  way a person would read it: same order and mostly the same words, so the user
  can follow along on screen, but without URLs, tokens, ticket numbers, code,
  or anything else that is painful to hear. Takes optional instructions as an
  argument ("only the conclusion", "skip the table", "slower"). Use when the
  user runs /read-aloud, or asks to "read that aloud", "read that out loud",
  "read this to me", or "read me your last message". For a one-sentence spoken
  update, use the say skill instead.
---

# Read Aloud

Narrate your previous message through the speakers while the user reads along.

## What to read

Your most recent message with real content, from before this invocation. Skip
over your own acknowledgements from an earlier read-aloud. If there's no such
message, say so instead of speaking.

## Instructions in the argument

Anything the user passes as an argument overrides the defaults below: what to
read, what to skip, how much to condense, voice, or speed.

Without an argument, apply the defaults as they are. Always narrate in English.

## Narrating

**Stay close to the text.** The user is following along on screen, so keep the
original order and most of the original wording. Cover every section, and
condense where the text is long or repetitive. Small changes to the structure
are welcome when they make it flow better as speech: turn terse fragments into
short sentences, and add a transition where a heading or a list would otherwise
get lost.

**Speak human language only.** Ask what a human narrator would say here. Name
the thing, and leave out the unpronounceable details.

| On screen                                 | Narrate                             |
| ----------------------------------------- | ----------------------------------- |
| `https://linear.app/acme/issue/ENG-1234`  | "the Linear ticket"                 |
| `ENG-1234`, `#4821`                       | "the ticket", "the pull request"    |
| `eyJhbGciOiJIUzI1NiIs...`                 | "the token"                         |
| `src/auth/middleware.ts:42`               | "the auth middleware"               |
| `9ed04ac`                                 | "that commit"                       |
| `v2.14.3-rc.1`                            | "the new release candidate"         |
| "412 passed, 3 failed in 38.2s"           | "Three tests failed, the rest pass" |
| A code block                              | One line on what the code does      |
| A table                                   | The takeaway or key rows, in prose  |

- Never speak secrets, not even partially.
- Say short identifiers as words when that's natural (`useThreads` becomes "use
  threads"). Describe long ones instead.
- Remove anything the speech engine would read literally or mangle: markdown,
  symbols, and arrows. Write "and", not "&".
- Don't add a preamble or commentary of your own ("Here's my message"). Just
  start reading.

## How to speak it

Run this with Bash, with `run_in_background: true`, since long narrations
outlast the foreground timeout and the user can keep working meanwhile:

```sh
pkill -f '^say -r '; while pgrep -qf '^say --rate='; do sleep 1; done; say -r 190 <<'SPOKEN'
I fixed it in the auth middleware, and there's more detail in the Linear ticket.

Three tests failed, the rest pass.
SPOKEN
```

- The `pkill` stops a narration or sentence that's still playing. The `while`
  waits for a notification to finish, which is spoken with `--rate=` so it's
  never cut off. See the say skill.
- The quoted heredoc makes apostrophes and dollar signs safe, so write natural
  contractions.
- `-r 190` is a bit above the usual 175 words per minute. Change it only if the
  argument asks for another speed.
- Don't pass a voice unless the argument asks for one. Pick voices from
  `say -v '?'` rather than guessing a name.
- Run the `say` command directly rather than going through the say skill, whose
  one-sentence limit doesn't apply here.

## After speaking

Print one line and nothing else:

```
Reading aloud. To stop: `! pkill -f '^say -r '`
```

Don't repeat the narration in the transcript. If the user asks you to stop, run
`pkill -f '^say -r '`. When the background task reports that it has finished, don't
comment on it.

## Examples

**Message:**

> Fixed in `src/auth/middleware.ts:42`, see
> https://linear.app/acme/issue/ENG-1234. Root cause: expired tokens weren't
> rejected. Tests: 412 passed, 3 failed (unrelated, also on `main`).

**`/read-aloud`** narrates:

> I fixed it in the auth middleware, see the Linear ticket. The root cause was
> that expired tokens were not being rejected. Three tests failed, but they are
> unrelated and also fail on main.

**`/read-aloud just the root cause`** narrates:

> The root cause was that expired tokens were not being rejected.

## Checklist

- [ ] Same order and mostly the same words as the message
- [ ] Argument instructions followed over the defaults
- [ ] No URLs, tokens, ticket numbers, hashes, paths, or code spoken
- [ ] Run in the background, with the `pkill` and `while` first, and a quoted heredoc
- [ ] Only the one-line stop hint printed
