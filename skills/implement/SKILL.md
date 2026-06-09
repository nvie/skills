---
name: implement
description: >
  Implement a spec while keeping a running implementation-notes.md log of every
  decision, deviation, and tradeoff that wasn't spelled out in the spec. Use when
  the user invokes /implement, hands over a spec / ticket / design doc to build,
  or asks to "implement X and keep notes on the decisions you made / what you had
  to change / anything I should know". The notes file is a throwaway review
  artifact: it captures the *delta* between spec and reality so the user can
  review judgment calls without re-reading the whole diff.
---

# Implement (with running notes)

Build what the spec describes, and as you go, keep a running
`implementation-notes.md` that records only the things the spec **didn't** decide
for you: judgment calls, deviations, tradeoffs, assumptions, and anything else
the user should know before they review.

The notes file is the point. It lets the user review the handful of places where
you exercised judgment instead of re-reading the entire diff to find them.

## When to use this skill

- The user typed `/implement` (with a spec inline, as a file path, or as a ticket
  reference).
- The user hands over a spec, design doc, ticket, or issue and asks you to build
  it.
- The user says things like "implement this and keep notes on your decisions",
  "tell me what you had to change", "flag anything I should know", "log the
  tradeoffs as you go".

## What the notes file is — and is NOT

The notes file captures the **delta between the spec and reality**. That is its
entire job.

- **It is NOT a diary.** Faithfully implementing a clear spec item produces _no
  note_. If the spec said "add a `POST /login` endpoint" and you added exactly
  that, there is nothing to record. The diff already shows it.
- **It IS a record of every place you had to decide something the spec left
  open**, diverged from what the spec literally said, traded one thing off
  against another, or assumed something the user should verify.
- **It is throwaway.** It is git-ignored, never committed, and exists to be read
  once and then cleared. It is a handoff to the user, not project history. (If a
  decision deserves to live on, it belongs in a commit message or an ADR — say
  so, don't bury it here.)

## Log, or stop and ask?

This is the core judgment of the skill. The notes file exists so you can keep
**momentum** on small decisions without interrupting the user for each one — but
it is not a place to hide big ones.

- **Log it and keep going** when the decision is small-to-medium, reversible, and
  any reasonable person would accept your choice (or easily flip it later).
  Examples: a default value the spec omitted, a helper's internal name, which of
  two equivalent libraries already in the project to use.
- **Stop and ask the user** when the decision is high-stakes, hard to reverse, or
  genuinely ambiguous in a way that could waste real work or violate likely
  intent. Examples: a schema/migration shape, a public API contract, deleting
  user data, a choice that forks the rest of the implementation. Do **not** bury
  a decision like this in the notes file and march on.

When unsure which bucket a decision falls in, lean toward asking. The notes file
is for the calls that aren't worth an interruption — not for dodging hard ones.

## Process

### Step 1 — Pin down the spec

Resolve the argument to an actual spec:

- **File path** → read it.
- **Inline text** → that is the spec.
- **Ticket / issue reference** → fetch it if you can, otherwise ask the user to
  paste it.
- **Nothing** → ask the user what they want implemented.

Restate the spec in one or two sentences and confirm scope before writing code.
If the spec is large or architecturally awkward, consider `/proper` (to reshape
first) or `/mikado` (to sequence a complex build) — see _Relationship to other
skills_ below.

### Step 2 — Create the notes file lazily

Do **not** create the file upfront. Create `implementation-notes.md` only when
you have a first real note to write (Step 3). If the whole implementation turns
out faithful to the spec with nothing worth recording, **never create the file** —
and say so when you wrap up.

When the first note arrives, create the file at the repository root and git-ignore
it via `.git/info/exclude` (do **not** touch the project's `.gitignore`):

```markdown
# Implementation notes: <spec name>

Spec: <path / ticket / "inline">
Started: <date>

Decisions, deviations, and tradeoffs that weren't spelled out in the spec.
Faithful spec items are not listed here. Read the ⚠ flagged items first.

---
```

If a notes file already exists from a previous run, ask whether to append to it or
start fresh. If you expect multiple concurrent specs, suffix the name
(`implementation-notes-<slug>.md`).

### Step 3 — Implement, logging as you go

Build the spec. The instant you make a call that belongs in the notes (see
categories below), **append it immediately** — do not wait until the end and try
to reconstruct from memory. The small decisions are exactly the ones you'll
forget, and they're often the ones the user most wants to see.

For high-stakes or ambiguous forks, stop and ask instead of logging (Step _Log,
or stop and ask?_).

### Step 4 — Entry format

One entry per decision. Each entry answers **what / why / where**, and flags
whether the user needs to act:

- Prefix with a **category**: `Decision`, `Deviation`, `Tradeoff`, `Assumption`,
  or `Flag`.
- Prefix attention-needing entries with **⚠** (deviations from the spec and
  assumptions to verify almost always warrant it).
- Include a `file:line` reference where it applies.
- Keep it to a few lines. Add "easy to change later" / "verify before deploy" /
  "you may prefer X" guidance when useful.

```markdown
- **Decision** — `src/auth/session.ts:42` — Spec didn't specify session
  lifetime. Set TTL to 24h to match the existing refresh-token window. Trivial
  to flip.

- **⚠ Deviation** — `src/api/users.ts:88` — Spec asks for `DELETE /users/:id`,
  but `orders` reference users. Implemented a soft-delete (`deleted_at`) to
  avoid breaking the FK. You may want a hard delete + cascade — flagging.

- **Tradeoff** — Validated on the server only, not the client, to ship the
  endpoint faster. Costs a round-trip on bad input; easy to add client-side
  validation later.

- **⚠ Assumption** — Assumed `EMAIL_FROM` is set in prod (it's in the staging
  `.env`). Verify before deploy.

- **Flag** — The spec's pagination section contradicts itself (cursor in §3,
  offset in §5). I went with cursor; please confirm.
```

### Step 5 — Wrap up

When the implementation is done:

1. Tell the user it's built. **Always report the notes file's status:**
   - If notes were written → state that `implementation-notes.md` was created and
     point them at it, with a quick count (e.g. "6 notes, 2 flagged ⚠").
   - If nothing was worth recording → say so plainly ("everything matched the
     spec; no notes file created"). Don't create an empty file just to have one.
2. **Surface the ⚠ items inline in chat** — the user shouldn't have to open the
   file to learn what needs their attention.
3. Do **not** commit the implementation — the user reviews and commits.
4. Once the user has reviewed, offer to clear or delete the notes file (it's
   throwaway). Do **not** delete it autonomously.

## Relationship to other skills

- `/proper` — if the spec doesn't fit the current code shape, reshape first, then
  implement. A "this didn't fit cleanly" finding is itself a strong notes entry.
- `/mikado` — if the build is large or risky, sequence it as a Mikado graph; keep
  the notes file running alongside for the decisions each step forces.

## Rules

- **The file records the delta, not a diary.** No entry for a clear spec item
  implemented faithfully.
- **Create it lazily, never empty.** No notes worth writing → no file. Always
  report at the end whether it was created.
- **Append in real time.** Log each decision when you make it, never reconstruct
  at the end.
- **Log small/reversible calls; stop and ask for big/irreversible/ambiguous
  ones.** Never bury a major decision in the file.
- **Every entry: what + why + where (`file:line`).** Mark attention-needing
  entries with ⚠.
- **Git-ignore the file via `.git/info/exclude`.** Never commit it; never touch
  the project's `.gitignore`.
- **Never commit the implementation autonomously.** The user reviews and commits.
- **Never delete the notes file autonomously.** Offer at the end; the user
  decides.
