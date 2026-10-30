---
name: tell-me
description: >
  Tell the user exactly what to do next, as a short numbered checklist of
  concrete actions: which link to open, which menu path to click, which
  setting to change, what to verify, who to inform. Use when the user runs
  /tell-me, or asks "what do I do next", "what should I do now", "give me the
  steps", "walk me through it", or "just tell me what to click". Output is the
  numbered list and nothing else -- no intro, no background, no explanation.
---

# Tell me

The user wants to act, not to read. Give them the next steps as a numbered
list they can follow with one eye while their hands do the work. They'll ask
for context if they need it.

## The one hard rule

**Output the numbered list and nothing else.**

No "Here's what to do:", no summary of the situation, no "why", no closing
"Let me know if...". The first character of the response is `1.`.

## Steps

Each step is one line, starting with an imperative verb, as short as it can
be while still being unambiguous.

- **Be exact.** Exact URL, exact menu path, exact button label, exact value,
  exact command. The user should never have to search for anything.
- **Menu paths** in quotes separated by `>`, button labels in quotes:
  `click "Settings > General > Webhooks" then "Edit..."`
- **Links** as bare URLs. Commands and values in backticks, copy-pasteable.
- **Group trivially sequential actions in one step** when they happen on the
  same screen: `toggle "Enabled", set "Timeout" to 30s, click "Save"`.
  A new screen, tool, or app is a new step.
- **Include the verify step** when there's something to check:
  `check the deploy log shows "migration 0042 applied"`.
- **Include the inform step** when someone else needs to know:
  `post in #eng-infra that the key was rotated`. Name the channel or person.
- **Branching** only when it's real, as a short inline `if X: Y`, never a
  nested tree.
- No nesting, no headings, no bold, no emoji.

## Never guess

A confidently wrong menu path is worse than an honest gap: the user clicks
around looking for a button that doesn't exist.

- If you know the exact path, write it.
- If you don't, say what to find in plain words and mark it:
  `find the webhook secret setting (exact location unknown)`.
- Unknown values (a name, a channel, an ID) get a `<placeholder>`, never an
  invented one.

An unknown location still gets a link. Give the deepest URL you're sure
of, falling back to the app's root if that's all you know:
`open https://dash.cloudflare.com, find the Logpush job for the worker (exact location unknown)`.
Never invent a deep URL; a correct root beats a broken deep link.

## What goes in the list

Derive "next" from the conversation: what's been done, what's left, what the
user is trying to reach. Only list what the **user** has to do. Don't perform
any of the steps yourself while answering.

If there's genuinely no way to tell what the user is trying to do, ask one
short question instead of producing a list.

## Example

**Context:** a webhook secret leaked in a log; the new secret is generated
and already in the env of the staging app.

```
1. Open https://dashboard.example.com/projects/acme/settings
2. Click "Settings > Webhooks > Production" then "Edit..."
3. Paste the new secret into "Signing secret", click "Save"
4. Run `fly secrets set WEBHOOK_SECRET=<new secret> -a acme-prod`
5. Trigger a test event via "Send test webhook", check it returns 200
6. Click "Revoke" on the old secret
7. Post in #security that the webhook secret was rotated and the old one revoked
```

## Checklist

- [ ] First character of the response is `1.`
- [ ] Nothing before or after the list
- [ ] Every step starts with a verb and fits on one line
- [ ] URLs, menu paths, labels, values, commands are exact
- [ ] Anything uncertain is marked or a `<placeholder>`, never invented
- [ ] Every "open/find X in app Y" step has a link, at least the app's root
- [ ] Verify and inform steps present where they apply
