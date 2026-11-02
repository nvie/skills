---
name: pr
description: >
  Open a pre-populated GitHub PR creation page in the browser, with a title and
  body drafted in nvie's personal writing style. Use when the user runs /pr, or
  asks to "open a PR", "create a PR", or "draft a PR" for the current branch.
  This skill never creates the PR itself -- the user reviews and submits it
  manually.
---

# Open a PR (pre-populated, reviewed manually)

Open a pre-populated GitHub PR creation page in the browser for the current
branch. The user reviews and submits the PR themselves in the browser.

This skill runs on Sonnet (`model: sonnet` in the frontmatter) -- drafting the
body is the bulk of the work and Sonnet does it noticeably faster. The override
lasts only for this turn; the session model resumes afterward.

> [!IMPORTANT]
> You open the page with **`gh pr create --web`**. The `--web` flag is
> **mandatory**: it opens the pre-filled page in the browser without creating
> anything. Running `gh pr create` **without `--web`** creates the PR instantly,
> with no chance to edit -- never do that.

## Process

1. **Determine the current branch.** Run `git rev-parse --abbrev-ref HEAD`.
   (gh detects the repo itself, so `owner/repo` isn't needed.)

2. **Determine the base branch.**

   - If the user specifies one, use it.
   - Otherwise, find the nearest local branch ancestor. This correctly handles
     stacked branches (if `B` is stacked on `A`, base is `A`, not `main`).
     Use `git log --oneline --graph --all` or `git merge-base` as needed.

3. **Check for uncommitted changes.** If any, warn the user and ask whether
   to proceed with only the committed changes.

4. **Check if the branch is pushed.** If not, ask before pushing with
   `git push -u origin <branch>`.

5. **Analyze the diff and draft the title and body** in nvie's style (see
   below):

   - `git log --oneline <base>..HEAD` for commit subjects
   - `git diff <base>...HEAD --stat` for an overview
   - `git diff <base>...HEAD` on the key files for the actual changes. nvie's
     commits are atomic with good messages -- lean on them; deep-read a file's
     diff only when the subjects don't explain it.

6. **Write the body to a temp file and open the pre-filled page.** The body
   contains backticks, `$`, quotes, and newlines that would mangle if passed
   inline, so write it verbatim to `$(mktemp)` and pass it with `--body-file`:

   ```bash
   gh pr create --web \
     --base '<base>' \
     --head '<head>' \
     --title '<title>' \
     --body-file '<tmpfile>'
   ```

   - **`--web` is mandatory** -- it opens the pre-filled "Open a pull request"
     page in the browser and creates nothing. Without it, the PR is created
     instantly. Never omit it.
   - Pass `--head '<branch>'` explicitly so gh skips its fork/push prompt and
     just builds the URL for that branch.
   - gh hands the URL straight to the browser, so there is no clickable
     terminal link and Ghostty's 2048-byte OSC limit never applies.
   - gh refuses to open if the full URL reaches **8192 bytes** (it errors with
     `cannot open in browser: maximum URL length exceeded`). That's a ~8 KB
     body, far beyond normal. If you hit it, tell the user the body is too
     long for the URL rather than silently dropping content.

   This opens the page for review; the user submits it. It does **not** create
   the PR.

## nvie's PR writing style

This is the style learned from hundreds of nvie's own PRs in `liveblocks`,
`liveblocks-backend`, `decoders`, `zenrouter`. **Match it.**

### Structural rules

- **No "Summary" / "Test plan" headings. Ever.** Don't add templated sections.
  Structure emerges from the content.
- **Never include file counts, line counts, or a list of touched files.** The
  diff already shows that.
- **No AI trailers.** No "Generated with Claude Code", "Co-Authored-By:
  Claude", or similar.
- **No generic "how to test" checklist.** If testing guidance is genuinely
  non-obvious, add it inline as a note -- not as a boilerplate section.
- Keep the title under 70 characters. Details go in the body.
- **No ticket numbers in the title.** "Webhook delivery worker, minus the
  actual HTTP request", not "… (LB-4044)". If the branch or commits reference
  a ticket, close the body with a `Fixes LB-4044` line instead.

### Opening and closing lines

Almost every PR opens with `This PR …` -- refactors / adds / fixes /
implements / introduces / removes / changes / reverts / updates:

- `This PR refactors how storage corruption is handled during load…`
- `This PR adds a new admin-only endpoint…`
- `This PR introduces protocol V8, with a change in response to…`

When the PR resolves a ticket or issue, the body's **last line** is
`Fixes LB-2048` (or `Fixes #1127`). For a trivial PR, that line can be the
entire body.

### Tone and voice

- First person: "I've added…", "My solution is…", "I've verified…"
- Direct and factual about what the PR *does*. Hedging about whether it will
  *work* is fine and authentic: "Hopefully we can get some better insights",
  "I have no idea if this is a good starting point, but we can always adjust",
  "Maybe this way we'll figure out if...". Uncertainty about outcomes, yes;
  hedging fluff about the change itself, no.
- Dry humor is welcome when it fits ("The Big Inline™", "Baby steps.",
  "after weeks of climbing 🙌"). Don't force it.
- Em-dashes sparingly
- Use emoji sparingly for emphasis (🙏 😇 🙌 ✅ ❌ ⚠️ 🧹 ✨ 🏷️ 📦)
- When citing a source (docs, a spec, an upstream issue), quote it and link
  it rather than paraphrasing it as your own fact. Keeps clear which part is
  established and which part is your read.

### What the body is for

The body primes the reviewer's brain so they can context-switch into the
diff. It answers "what is the point of this PR?", ideally at the
interface/seam level: what changes for callers, users, or neighbouring
systems. The "Files changed" tab holds the details; the body doesn't have to.

- **Talk in concepts, not code.** Describe ideas ("delivery attempts are now
  claimed before they're sent"), not the names that implement them
  ("`claimAttempt()` sets `claimed_until`").
- **Implementation details only when they matter**: when the reviewer would
  otherwise misread the diff, when a choice is surprising, or when the PR
  *only* changes implementation (a refactor, a perf fix). Then go there.
- **Shorter beats longer**, as long as nothing important is left out. When in
  doubt, cut.

### Content patterns

Use whichever of these fit the change; omit the rest.

- **Before / After**: code blocks or `diff` blocks showing the API shape
  change, when the shape change *is* the point. Much more useful than prose.
- **Tables** with ✅ / ❌ / ⚠️ for protocol or behavior matrices.
- **`## Why`** section when the motivation isn't obvious.
- **`## Rollout`** whenever shipping takes more than merging and deploying
  this code: a migration to run, an env var or secret to add, a flag to flip,
  another PR that must land first, an ordering constraint. Write it as a
  GitHub task list so the user can tick steps off as they go:

  ```markdown
  ## Rollout

  - [ ] Merge #1234 first
  - [ ] Add `WEBHOOK_SIGNING_KEY` to the production env
  - [ ] Run the migration
  - [ ] Merge and deploy this PR
  - [ ] Enable the flag
  ```

  Omit the section when there's nothing beyond a normal deploy.

- **`## Next steps`**: only when the user asks for it. If you think it would
  help, suggest it in your terminal reply instead of adding it.
- **`## Chapters`** (or similar) for long refactorings split into phases.
- **Reviewer guidance**: "Best reviewed commit-by-commit", "Ignore the
  mechanical noise in X", "Don't be scared by the diff size -- most of it is
  tests".
- **Explicit behavioral-change callouts**: "**Technically, this is a
  behavioral change, not a pure refactoring!**". Always flag when a PR that
  looks like a refactoring actually changes runtime behavior.
- **Links**: related PRs (`#1437`), Linear tickets (`LB-2048`), Slack threads
  (`liveblocks.slack.com/archives/...`), tech design docs (Notion). PR bodies
  are GitHub-flavored markdown, so a labelled link is `[label](url)` and a
  bare URL auto-links. Never Slack's `<url|label>` -- that's Slack API mrkdwn
  and GitHub renders it as literal text.
- **GitHub callouts** (`> [!WARNING]` etc.): almost never. At most one, and
  only for something that would cause real damage if missed (data loss, a
  breaking change, a deploy ordering trap). Lint noise, an intentional `XXX`,
  or "I haven't decided yet" are not callout-worthy; mention them in a
  sentence or leave them out.
- **Screenshots**: for UI / rendered output, with "Before" / "After" labels
  when comparing. (The user will attach these manually; don't fabricate image
  URLs.)

### Diagrams

Only when the user asks for one. Use a Mermaid block, and keep it simple:

- Boxes are concepts or components ("Queue", "Worker", "Customer endpoint"),
  not function or variable names, unless a name is genuinely needed.
- When the PR changes how a part of the system works, show it as **Before /
  After** diagrams, or a single diagram with the parts this PR adds or
  changes **highlighted in color** (`classDef changed fill:#fde68a` etc.).
- A handful of nodes. If it needs a legend, it's too big.

### Identifiers and renames

- Use code identifiers sparingly. Each `backticked` term is a speed bump;
  include one only when it carries a thought the prose can't (a public API
  name the reviewer will grep for, a log attribute, a config key). Prefer
  describing the concept in words.
- When you do name one, backtick it.
- Unicode arrow `→` for renames in prose: `` Rename `toolName` → `name`  ``
- Use `diff` fenced code blocks to show type/API changes, e.g.:

  ````markdown
  ```diff
  export type RoomStateServerMsg<U extends BaseUserMeta> = {
  - readonly meta?: Record<string, Json>;
  + readonly meta: JsonObject;
  };
  ```
  ````

### What NOT to write

- No "## Summary" / "## Test plan" / "## Checklist"
- No "Files changed: 42", no "Added 900 lines"
- No "This change is important because…" -- just state what it does
- No AI-trailer lines or co-author tags for AI
- No exhaustive bullet list of every tiny change when a sentence would do
- No ticket numbers in the title
- No `## Callouts` section, no `## Next steps` unless asked
- No walkthrough of functions and variables when the concept would do
- No passive voice boilerplate like "Tests have been added". Say "I've added
  tests for X."
- No Slack markup: `<url|label>` links, `*single-asterisk bold*`, or literal
  `•` bullets. GitHub wants `[label](url)`, `**bold**`, and `-` bullets.

### Short vs long PRs

Length tracks **conceptual complexity**, not diff size and not file count.
Ask: how much does the reviewer need in their head before the diff makes
sense? A 600-line mechanical rename needs one sentence. A 60-line change to
a consensus protocol might need three paragraphs and a diagram.

The two examples below are the calibration point for *simple* work: "add a
log attribute", "add a counter and warn past a threshold". Changes at that
complexity get ~150 words, two or three paragraphs, no headings. If a draft
for something that simple is longer, cut it.

- **Trivial PRs** (typo fix, version bump, flaky test skip): one short sentence
  or just `Fixes LB-1234.` is enough. Sometimes the body is empty -- that's
  fine.
- **Medium PRs** (one self-contained idea, however many files): a paragraph of
  `This PR …`, plus a small bullet list if there are distinct things it does.
  **No headings at all** at this size. Reach for `## Why` only when there's
  real background that doesn't fit the opening paragraph, and it's fine to put
  it at the *bottom*, as context the reviewer can drop into after they know
  what the PR does.
- Default to the shorter shape. Most PRs are medium, and a medium PR written
  long reads like a report. Motivation usually belongs woven into the opening
  sentence, not in its own section.
- **Large/complex PRs** (several interacting ideas, a new subsystem, a
  migration, anything where the reviewer needs orientation before the diff):
  open with `This PR …`, then use headings (`## Why`, `## Before` / `## After`,
  `## Rollout`, chapters). Call out behavioral vs pure-refactoring. Flag
  reviewer guidance up front. Big diff alone doesn't qualify a PR for this
  shape -- interacting moving parts do.

### Example bodies

Two real ones, both small PRs (~60-80 lines changed). Note the register:
concrete, first person, motivation in the opening sentence, illustrative
examples over abstract description, and no headings unless earned.

`Track which isolate each Durable Object runs in` (+64 -1, 5 files):

```markdown
This PR adds an `isolateId` to every log line the Cloudflare worker emits, plus a `doSeq` on Durable Object log lines saying which DO instance it is within that isolate. Hopefully we can get some better insights into when certain rooms that are showing symptoms might actually be victims of other badly-behaving rooms in the same isolate.

New attributes that would show up in Datadog:
- `isolateId` (a random value)
- `doSeq` (a sequential number assigned to each Durable Object within the same isolate)

Therefore `(isolateId, doSeq)` should uniquely describe a durable object instance in our logs.

For context, see this [Slack thread](https://liveblocks.slack.com/archives/...).
```

`Warn when in-flight fetch count exceeds a high-water mark` (+76 -1, 2 files),
where a `## Why` at the bottom earns its place:

```markdown
This PR instruments adds a high-watermark warning for our AsyncRefreshCaches, so we can answer "are Hyperdrive fetches hanging?"

What gets kept in check now is each AsyncRefreshCache's number of _simultaneous_ in-flight promises. This count grows and shrinks naturally as fetches are happening, but if fetches never settle, then the count might just grow unboundedly, and I'd want to know about it. So this PR sets an upper limit of 25 to start with (I have no idea if this is a good starting point, but we can always adjust to reduce log noise).

Once the 25 high water mark is exceeded for a particular cache, it logs a warning and raises the new high water mark. Over time this means we could see logs appear in Datadog like this:

- `Cache projectById has 25 in-flight fetches`
- `Cache externalIdToRoomCache has 25 in-flight fetches`
- ...

Maybe this way we'll figure out if there is a memory leak happening somewhere.

## Why

Sibling to #1895.

There exist 10 `AsynchronousRefreshCache` singletons at module scope in `kv-cache.ts`. Each holds an `inFlightFetches` map that self-cleans in `.finally()`, so it's bounded by concurrent fetches... unless... a fetch never settles.
```

Things to copy from these: the "so we can answer <question>" framing, sample
output the reviewer can picture, `Sibling to #NNNN.` for related PRs, and
trailing-off ellipses where the thought genuinely trails off.

## Output format to show the user

After `gh pr create --web` opens the page, keep the output minimal. Do **not**
echo the title, body, or URL back into the terminal -- the user reviews
everything in the browser. Just confirm:

> I've opened the PR draft for you in the browser.

If you think a `## Next steps` section or a diagram would help, add one short
line suggesting it. Nothing else.

## Iterating on the draft

If the user replies "include more", "include less", "more detail", "shorter",
or similar, redraft the body at the requested level of detail and run
`gh pr create --web` again to open a fresh pre-filled page. Keep the same
title, base, and head unless told otherwise.

- **Less**: drop implementation detail and identifiers first, then secondary
  points, until only the core idea and anything reviewer-critical (rollout,
  behavioral changes) remain.
- **More**: add the next layer down: the key mechanism, a notable design
  choice, or a Before / After. Still concepts before code.

## Critical rules

- **ALWAYS** include `--web` on `gh pr create`. Without it, the PR is created
  instantly with no chance to edit. With it, gh only opens the pre-filled page
  in the browser; the user reviews and submits there.
- **NEVER** add a "Test plan" section.
- Only open the pre-filled page. The user reviews, edits if needed, and submits.
