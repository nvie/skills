---
name: mikado
description: >
  Mikado method workflow for complex refactorings. Use when the user invokes
  /mikado, mentions "mikado graph", or wants to break a large code change into
  safe, incremental steps with automatic revert-on-failure.
---

# Mikado Method

Structured workflow for tackling complex code changes using the Mikado method:
set a goal, try naively, record prerequisites on failure, revert, and work
leaves-first until the goal becomes trivial.

## Directory layout

All state lives in `.mikado/` at the repository root.

```
.mikado/
  current.md -> my-goal.md      # symlink to the active graph
  my-goal.md                    # a Mikado graph file
  another-goal.md               # another graph (inactive)
```

Ensure `.mikado/` is git-ignored (append to `.git/info/exclude` so the
project's `.gitignore` is not touched).

## Graph file format

Each graph file has two sections: a **Graph** (scannable checklist) and
**Details** (rich context for nodes that need it).

Every node in the graph gets a short `[#id]` tag. Only nodes that need
extra explanation get a corresponding heading in the Details section --
self-explanatory nodes can skip it.

```markdown
# Mikado: Extract shared utils

## Graph

- [ ] Extract shared utils into `@app/utils` package [#goal]
  - [ ] Move date helpers to shared package [#move-date]
    - [x] Create `@app/utils` package scaffolding [#scaffold]
    - [ ] Resolve circular dep between `core` and `helpers` [#circular-dep]
  - [ ] Update all import paths [#update-imports]

## Details

### #goal -- Extract shared utils into `@app/utils` package
The backend and frontend both have duplicate date/string utilities.
We want a single shared package to reduce drift.

### #circular-dep -- Resolve circular dep between `core` and `helpers`
`core/index.ts` imports from `helpers/date`, but `helpers/date` imports
`core/types`. Need to extract the shared types into a leaf package first,
or inline the two types that `helpers/date` actually uses.
```

### Graph rules

- `- [ ]` = not yet done
- `- [x]` = completed and committed
- Deeper nesting = must be done first
- A node is **actionable** when all its children (if any) are `[x]`
- The goal is achieved when the top-level item can be checked off

### Node ID rules

- IDs are short lowercase slugs: `[#move-date]`, `[#circular-dep]`
- Every node must have a unique `[#id]` tag
- When adding new prerequisite nodes, generate a short descriptive slug

### Details section rules

- Heading format: `### #id -- short label`
- Only add a details entry when the node needs context beyond its one-line
  label (rationale, links, code references, gotchas)
- Keep details concise -- a few lines, not paragraphs

## Invocation modes

### 1. `/mikado --goal "<description>"`

Create a new Mikado graph:

1. Slugify the description to produce a filename
   (e.g. "Extract shared utils" -> `extract-shared-utils.md`).
2. Create `.mikado/<slug>.md` with the goal as the single top-level item.
3. Symlink `.mikado/current.md` -> `.mikado/<slug>.md`.
4. Ensure `.mikado/` is in `.git/info/exclude`.
5. Show the user the created graph and confirm the goal.

### 2. `/mikado --switch "<slug-or-partial>"`

Switch the `current.md` symlink to a different existing graph file.

### 3. `/mikado --list`

List all graph files in `.mikado/`, indicate which is current, and show
progress (done/total counts).

### 4. `/mikado` (no arguments -- the main loop)

This is the core cycle. Run it repeatedly to make incremental progress.

**Step A -- Load state**
- Read `.mikado/current.md`. If missing, ask the user for a goal (then behave
  like `--goal`).
- Parse the graph. Find all **actionable leaf nodes** (unchecked items whose
  children, if any, are all checked).

**Step B -- Pick the next leaf**
- If multiple actionable leaves exist, present them and ask the user which one
  to tackle. If only one, confirm it with the user briefly.

**Step C -- Attempt the change**
- Try to implement the chosen leaf node naively.
- Run the project's build / type-check / tests if available (read CLAUDE.md or
  ask the user how to verify). If you are unsure what verification to run,
  ask the user.

**Step D -- Evaluate the result**

_If the change works (builds, tests pass):_
- Tell the user: "Leaf done. Nothing broke. You can review the diff and
  commit when ready."
- Do NOT commit automatically. Wait for the user.
- When the user commits (or asks you to), mark the leaf as `[x]` in the
  graph file.
- If the completed leaf's parent now has all children checked, note that the
  parent is now actionable.

_If something breaks:_
- Identify what went wrong.
- Add newly discovered prerequisites as children of the current leaf in the
  graph file. Explain each one briefly to the user and ask for confirmation
  before saving.
- After updating the graph, **revert all uncommitted changes** with
  `git checkout -- .` (only tracked files) so the working tree is clean.
- Show the updated graph and explain what was learned.

**Step E -- Report**
- Show the current state of the graph (what's done, what's next).
- Stop and wait for the user to invoke `/mikado` again or give other
  instructions.

## Important rules

- **Never commit autonomously.** Only commit when the user explicitly asks.
- **Always revert on failure.** Do not try to fix forward during a Mikado
  cycle; the whole point is to revert and record.
- **Ask before complex decisions.** If a leaf is ambiguous or the
  prerequisites are unclear, ask the user rather than guessing.
- **Keep the graph file updated.** It is the single source of truth.
- **One leaf at a time.** Do not try to tackle multiple leaves in one cycle.
- **Respect existing commits.** Never amend, rebase, or rewrite history.
