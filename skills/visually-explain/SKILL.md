---
name: visually-explain
description: >
  Produce a single self-contained interactive HTML document that visually
  explains an approach, architecture, algorithm, or concept -- built for visual
  learners who find static markdown insufficient. Use when the user wants to
  *understand* or *explain* something through an interactive visual rather than
  prose: "explain this visually", "make an interactive diagram / explainer",
  "I'm a visual learner, help me understand X", "turn this architecture into an
  interactive HTML doc", or whenever walking a concrete example through states
  (toggles, steppers, diffs, side-by-side comparisons) would land better than a
  wall of text.
---

# Visually Explain

Turn a concept into a single self-contained interactive HTML document that you
can open in a browser and play with. For a visual learner, a toggle that flips
between "what happens now" and "what should happen", or a stepper that walks a
real example through each state, lands an idea that no paragraph can.

The artifact is the explanation. Prose annotates it; it doesn't carry it. If the
result reads like a Markdown doc that happens to be wrapped in `<div>`s, it
failed.

## Correctness is the whole job (non-negotiable)

This is the one rule that overrides everything else in this skill. An interactive
visual _looks_ authoritative -- polished layout, confident color-coding, smooth
transitions all signal "this is right". That makes a wrong explainer far more
dangerous than wrong prose: the reader trusts it and builds a broken mental model
on top. **A bad or buggy explanation is worse than none at all.**

So:

- **Never guess, never hallucinate, never smooth over a gap.** If you don't fully
  understand a mechanic, you cannot visualize it. Don't invent how something works
  to make a cleaner animation or a tidier diagram.
- **Don't visualize anything you can't verify.** Every state transition, index
  shift, color, and label must be something you've confirmed from the source --
  the code, the spec, the user -- not inferred because it seems plausible.
- **When you're unsure, stop and ask** (or leave it visibly out). It is correct
  and expected to tell the user "I don't understand X well enough to draw it
  truthfully -- can you clarify, or should I leave it out?" That is a success, not
  a failure.
- **Mark genuine uncertainty in the artifact itself.** If something is your
  best-effort interpretation rather than confirmed fact, label it as such in the
  document (e.g. a dimmed "assumption" tag) so the reader never mistakes a guess
  for ground truth. Better still: resolve it first.

There is no partial credit for a beautiful diagram that's subtly false.

## First, actually understand the thing

You cannot visualize what you don't understand. Garbage in, garbage out.

Before writing any HTML: read the code, the design, the thread -- whatever the
concept lives in. Trace the concrete worked example through by hand and confirm
each state is what the real system would actually produce. If something is
ambiguous and the ambiguity changes the visual, ask before guessing.

When you're confident you understand it, find the **one concrete worked example**
that exposes the heart of the idea -- the smallest input that still shows the
interesting behavior. The screenshot that inspired this skill threads `"Hello"`
through two concurrent edits to show exactly where client-side rebasing diverges.
That single example does more than three paragraphs of theory. Pick yours before
you design anything.

## Choose an interaction that fits the concept

The interaction is not decoration -- it's how the reader builds the mental model.
Match it to what the concept actually is:

- **Compare toggle / tabs** -- flip the _same_ state between two readings:
  "current behavior" vs "intended result", naive vs correct, before vs after a
  fix. Best when the whole point is a divergence.
- **Stepper / timeline** -- walk one concrete example through sequential states
  (Initial -> Client A -> Server -> ...), forward and back. Best for protocols,
  algorithms, request lifecycles, anything with ordered causality.
- **Side-by-side** -- two approaches running the same scenario in parallel
  (client-side rebasing vs server-side rebasing). Best for "why this design over
  that one".
- **Annotated state** -- render the domain object _concretely_ (text as boxed
  characters, a queue as cells, a tree as nodes) and color-code roles:
  accepted/green, deleted/red, stale/amber, new/blue. The reader should _see_ the
  state, not read a description of it.
- **Diff / highlight** -- show what changed between two states, with the delta
  emphasized and the unchanged parts dimmed.
- **Progressive reveal** -- expandable detail for layered ideas, so the overview
  stays uncluttered and depth is opt-in.
- **Live playground** -- let the reader change an input and watch the result
  recompute. The most powerful option when the concept is a transformation, and
  worth the extra effort when it is.

Combine them when it serves the idea (a stepper _with_ a compare toggle at each
step is exactly the screenshot's pattern). Don't combine them to show off.

## Aesthetics: defer to the frontend-design skill

This skill owns _what to show and how it behaves_. It does not re-specify how it
should look. Follow the **frontend-design** skill for typography, color, motion,
and composition, and to avoid generic AI aesthetics (no Inter, no purple-on-white
gradients). Two domain-specific notes on top of it:

- Color must **encode meaning**, not just look nice. If green means "accepted" in
  one place, it means "accepted" everywhere. Keep a tiny, legible legend.
- Motion should clarify causality (a character sliding as an index shifts), never
  just animate for delight. A transition that shows _why_ an index moved is worth
  it; a fade-in for its own sake is noise.

## Self-contained, always

One `.html` file. All CSS and JS inline. No build step, no framework, no network
calls -- the user double-clicks it, or opens it on a plane, and it just works.
Vanilla JS and CSS are plenty for these; reach for nothing heavier.

The single tolerable exception is one web font from a CDN -- and only with a real
system fallback in the stack so it degrades gracefully offline. Everything else
(icons, logic, state, styles) ships in the file.

## Output

Write the file to the current working directory with a sensible name derived from
the topic (e.g. `livetext-rebasing.html`), then open it:

```sh
open livetext-rebasing.html
```

Tell the user the filename in one line. Don't paste the HTML into the chat -- it's
long and they'll read it in the browser. On revisions, overwrite the same file
and re-open.

## Smell test before you ship

- **Is every single thing in it true and verified?** Can you point to where each
  state, transition, and label comes from in the source? Any "I think it works
  like this" must be resolved or visibly marked as an assumption -- never shipped
  as fact. This gate comes first; a wrong artifact fails regardless of how good it
  looks.
- Could a reader get the core idea from the visual alone, with the prose hidden?
  If not, the visual isn't doing its job yet.
- Is there a real worked example, or just abstract boxes labeled "State A"?
- Does every color and animation mean something?
- Does it open offline by double-clicking, with nothing fetched?
- Is it one focused concept, not a sprawling tour? One artifact, one idea.
