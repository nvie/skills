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

5. **Delegate diff-analysis and drafting to a fast subagent.** This is the slow
   part -- reading the whole diff and writing a styled body -- so hand it to a
   faster model and keep the large diff out of this conversation. Spawn one
   Agent (`subagent_type: general-purpose`, `model: sonnet`) whose prompt
   contains:

   - The `<base>` and `<head>` branches from steps 1-2, and that the repo is
     the current working directory.
   - The instruction to analyze the change with `git log --oneline <base>..HEAD`
     (commit subjects), `git diff <base>...HEAD --stat` (overview), and
     `git diff <base>...HEAD` on the key files (the actual changes). nvie's
     commits are atomic with good messages -- lean on them; only deep-read a
     file's diff when the commit subjects don't explain it.
   - The **entire "nvie's PR writing style" section below, verbatim.** This
     conversation already has it in context -- paste it into the subagent
     prompt so the subagent matches the style without guessing.
   - This exact output contract:

     > Write the PR body (GitHub-flavored markdown) verbatim to a fresh temp
     > file created with `mktemp`. Do **not** run `gh`, push, commit, or modify
     > anything else. Then return exactly these two lines and nothing else:
     >
     > ```
     > TITLE: <the one-line PR title>
     > BODY_FILE: <absolute path to the temp file>
     > ```

   Parse `TITLE:` and `BODY_FILE:` from the subagent's result.

6. **Open the pre-filled page with `gh pr create --web`:**

   ```bash
   gh pr create --web \
     --base '<base>' \
     --head '<head>' \
     --title '<title>' \
     --body-file '<BODY_FILE>'
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

After `gh pr create --web` opens the page, keep the output minimal. Do **not**
echo the title, body, or URL back into the terminal -- the user reviews
everything in the browser. Just confirm:

> I've opened the PR draft for you in the browser.

## Critical rules

- **ALWAYS** include `--web` on `gh pr create`. Without it, the PR is created
  instantly with no chance to edit. With it, gh only opens the pre-filled page
  in the browser; the user reviews and submits there.
- **NEVER** add a "Test plan" section.
- Only open the pre-filled page. The user reviews, edits if needed, and submits.
