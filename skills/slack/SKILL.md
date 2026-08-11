---
name: slack
description: >
  Write a message as a ready-to-paste Slack message, in nvie's own casual voice,
  using Slack-flavored markdown that formats correctly when pasted into the
  Slack composer and toggled with cmd+shift+f. Use when the user runs /slack, or
  asks to "write this as a Slack message", "slackify this", "draft a Slack
  message", or "post this in Slack". Output is the message text and nothing
  else -- no preamble, no explanation, no follow-up questions.
---

# Slack message

Turn whatever the user gives you into a single Slack message they can copy,
paste into the composer, and hit cmd+shift+f on to get formatted.

## The one hard rule

**Output the message and nothing else.**

No "Here's your message:", no "Let me know if you want it shorter", no
trailing commentary, no options to pick from. The entire response is the
message body. The user copy-pastes the whole thing, so any extra word is
something they have to delete by hand.

Put the message in a fenced code block. That keeps the terminal from
rendering the markdown away (`*bold*` must stay literal asterisks, not turn
bold), and the fences themselves aren't part of what gets copied.

## Voice

It's nvie typing to a coworker, not a company announcement.

- Casual and direct, but **properly capitalized**. Sentences start with a
  capital letter, proper nouns are capitalized. Casual is about word choice
  and length, not about typing in all-lowercase.
- Contractions always.
- First person: "I've", "I'd", "I think", "Heads up"
- Short. If it can be one sentence, it's one sentence.
- No corporate padding: no "I hope this finds you well", "just wanted to
  reach out", "please don't hesitate to", "circling back on this"
- No sign-off, no greeting unless the message genuinely needs one
- Dry humor is fine when it fits. Don't force it.
- Emoji sparingly and only where nvie would actually use one (🙏 😅 🙌 ✅ ❌
  ⚠️ 🎉). Never one per bullet.
- Don't over-hedge. "This is broken" beats "It seems like this might
  potentially be broken".
- Don't add information the user didn't give you. If something's genuinely
  missing, leave a short `<...>` placeholder inline rather than inventing it
  or stopping to ask.

## No line wrapping

Never hard-wrap. Each paragraph or bullet is **one single line**, however
long. Slack wraps text itself; manual newlines mid-sentence paste as real
line breaks and look broken.

Blank line between paragraphs, one line per bullet. That's it.

## Slack markdown

Use Slack's own markup -- it survives both a plain paste and cmd+shift+f:

| Want          | Write                      |
| ------------- | -------------------------- | ------- |
| bold          | `*bold*` (single asterisk) |
| italic        | `_italic_`                 |
| strikethrough | `~strike~`                 |
| inline code   | `` `code` ``               |
| code block    | triple backticks           |
| quote         | `> quoted`                 |
| bullet        | `- item`                   |
| numbered      | `1. item`                  |
| link          | bare URL, or `<url         | label>` |

Avoid:

- **Markdown headings** (`#`, `##`) -- use a `*bold line*` as a pseudo-heading
  instead
- **`**double asterisks**`** -- Slack's bold is a single asterisk
- **Tables** -- Slack has no table syntax; they paste as garbage. Use bullets.
- **Nested bullets more than one level deep** -- Slack flattens them anyway
- **Horizontal rules** (`---`)

Keep formatting light. A Slack message with five bold phrases and three
nested lists reads like a memo, not a message. Most messages need zero
formatting.

## Length

Match the input. A quick heads-up stays two lines. Only reach for bullets
when there are genuinely several distinct items -- otherwise prose.

If the user's input is long and rambly, the message is the _point_ of it,
not a summary of every clause.

## Examples

**Input:** "tell the team the deploy is delayed until tomorrow because the
migration is still running"

```
Heads up: pushing the deploy to tomorrow, the migration is still running and I'd rather not cut it short. Nothing's broken, just slower than expected.
```

**Input:** "ask sarah if she can review my PR about the websocket reconnect
logic, it's blocking the release"

```
Hey, any chance you could take a look at my PR on the websocket reconnect logic? It's blocking the release, so no rush but also... some rush 😅
```

**Input:** "write a message explaining that we found the root cause of the
flaky tests, it was shared state between test files, and we fixed it by
isolating the fixtures, and CI should be green now"

```
Found the root cause of the flaky tests: shared state leaking between test files. Fixed by isolating the fixtures per file.

CI should be green from here on -- if you still see a random red, ping me, that'd mean there's a second one hiding.
```

## Checklist

- [ ] Response contains the message and nothing else
- [ ] No hard-wrapped lines
- [ ] Single asterisks for bold, no `##` headings, no tables
- [ ] Sounds like a person, not a changelog -- but properly capitalized
- [ ] Sentences start with a capital letter
- [ ] No invented facts
