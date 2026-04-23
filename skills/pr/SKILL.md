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

Generate a pre-populated GitHub PR creation URL for the current branch. The
user will review and submit the PR themselves. **You MUST NOT run
`gh pr create`.**

## Process

1. **Determine the current branch and repo.** Run:
   - `git rev-parse --abbrev-ref HEAD` for the current branch
   - `gh repo view --json nameWithOwner -q .nameWithOwner` for `owner/repo`

2. **Determine the base branch.**
   - If the user specifies one, use it.
   - Otherwise, find the nearest local branch ancestor. This correctly handles
     stacked branches (if `B` is stacked on `A`, base is `A`, not `main`).
     Use `git log --oneline --graph --all` or `git merge-base` as needed.

3. **Check for uncommitted changes.** If any, warn the user and ask whether
   to proceed with only the committed changes.

4. **Check if the branch is pushed.** If not, ask before pushing with
   `git push -u origin <branch>`.

5. **Analyze the diff to draft the PR content.**
   - `git log --oneline <base>..HEAD` for commit subjects
   - `git diff <base>...HEAD --stat` for an overview
   - `git diff <base>...HEAD` for key files, to understand the actual changes

6. **Draft the title and body** following the style guide below.

7. **Construct the URL:**

   ```
   https://github.com/{owner}/{repo}/compare/{base}...{head}?expand=1&title={url_encoded_title}&body={url_encoded_body}
   ```

   URL-encode using Node or Python:
   ```bash
   node -e "console.log(encodeURIComponent(process.argv[1]))" "your string"
   ```

8. **Present to the user:**
   - Base branch and head branch
   - The drafted title and body (as readable text)
   - The URL as a short markdown link: `[Click here to open the PR](<url>)`.
     **Never print the raw URL** -- long pre-populated URLs wrap across many
     lines in the terminal and become impossible to click reliably. Always
     hide the URL behind the short link text.

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

### Opening line

Almost every PR opens one of two ways:

- `This PR …` -- refactors / adds / fixes / implements / introduces / removes /
  changes / reverts / updates
- A one-liner like `Fixes LB-1234.` or `Fixes #1127.` when the change is
  trivial

Example openings:

- `This PR refactors how storage corruption is handled during load…`
- `This PR adds a new admin-only endpoint…`
- `This PR introduces protocol V8, with a change in response to…`
- `Fixes LB-2048.`
- `Fixes #1127.`

### Tone and voice

- First person: "I've added…", "My solution is…", "I've verified…"
- Direct, factual, no hedging fluff
- Dry humor is welcome when it fits ("The Big Inline™", "Baby steps.",
  "after weeks of climbing 🙌"). Don't force it.
- Em-dashes sparingly
- Use emoji sparingly for emphasis (🙏 😇 🙌 ✅ ❌ ⚠️ 🧹 ✨ 🏷️ 📦)

### Content patterns

Use whichever of these fit the change; omit the rest.

- **Before / After**: code blocks or `diff` blocks showing the API shape
  change. Much more useful than prose.
- **Tables** with ✅ / ❌ / ⚠️ for protocol or behavior matrices.
- **`## Why`** section when the motivation isn't obvious.
- **`## Next steps`** when this is one step in a larger effort.
- **`## Callouts`** for surprising decisions you want the reviewer to notice.
- **`## Chapters`** (or similar) for long refactorings split into phases.
- **Reviewer guidance**: "Best reviewed commit-by-commit", "Ignore the
  mechanical noise in X", "Don't be scared by the diff size -- most of it is
  tests".
- **Explicit behavioral-change callouts**: "**Technically, this is a
  behavioral change, not a pure refactoring!**". Always flag when a PR that
  looks like a refactoring actually changes runtime behavior.
- **Links**: related PRs (`#1437`), Linear tickets (`LB-2048`), Slack threads
  (`liveblocks.slack.com/archives/...`), tech design docs (Notion).
- **GitHub callouts**: `> [!NOTE]`, `> [!IMPORTANT]`, `> [!WARNING]` for
  info the reviewer really should not miss.
- **Screenshots**: for UI / rendered output, with "Before" / "After" labels
  when comparing. (The user will attach these manually; don't fabricate image
  URLs.)

### Identifiers and renames

- Backticks around all code identifiers, types, file paths
- Unicode arrow `→` for renames in prose: ``Rename `toolName` → `name` ``
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
- No passive voice boilerplate like "Tests have been added". Say "I've added
  tests for X."

### Short vs long PRs

- **Trivial PRs** (typo fix, version bump, flaky test skip): one short sentence
  or just `Fixes LB-1234.` is enough. Sometimes the body is empty -- that's
  fine.
- **Medium PRs**: a paragraph of `This PR …`, plus a small bullet list if there
  are distinct things it does.
- **Large/refactoring PRs**: open with `This PR …`, then use headings
  (`## Why`, `## Before` / `## After`, `## Next steps`, chapters). Call out
  behavioral vs pure-refactoring. Flag reviewer guidance up front.

## Output format to show the user

When done, show:

- **Base:** `<base-branch>`
- **Head:** `<current-branch>`
- **Title:** _drafted title_
- **Body:** _drafted body as readable markdown_
- **URL:** `[Click here to open the PR](<url>)` -- render as a markdown link
  so the terminal shows a short, reliably-clickable label. Never paste the
  raw URL.

## Critical rules

- **NEVER** run `gh pr create`. The user reviews and submits manually.
- **NEVER** add a "Test plan" section.
- Only generate the URL. The user opens it, edits if needed, and submits.
