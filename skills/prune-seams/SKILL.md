---
name: prune-seams
description: >
  Audit and shrink a feature's module boundaries so that only real seams
  remain. Every module and every export must earn its place: either another,
  unrelated part of the system imports it as a proper seam (including known
  upcoming callers like a test harness or simulator on another branch), or
  tests need direct access to it. Everything else gets folded into its only
  caller as local code. Use when the user invokes /prune-seams (or /seams),
  says a directory has "too many modules" or "too many files", finds its
  structure unclear, asks to "collapse", "inline", "consolidate", "merge" or
  "simplify" modules, asks "what's the role of this seam?" or "why is this
  exported?", or complains about types.ts grab-bags and one-function-per-file
  layouts.
---

# Prune seams: as few module boundaries as possible

A directory should explain itself from its file list plus one header per
module. When a feature is spread over many small modules (`types.ts`,
`decideX.ts`, `callY.ts`, `createZ.ts`, ...), a reader can't tell which ones
are load-bearing seams and which are just one caller's implementation
details. This skill finds the real seams, folds everything else into its
only caller, and labels what's left.

## The principle: small interfaces, deep modules

This skill applies John Ousterhout's "deep modules" principle from _A
Philosophy of Software Design_: **a module should hide a lot of
functionality behind a small, simple interface.** The interface is the cost a
module imposes on everyone who uses it; the functionality behind it is the
benefit. Deep modules give a lot of benefit for little cost.

- **Shallow modules are the smell.** A `types.ts`, a one-function
  `decideX.ts`, a `createZ.ts` that only one caller uses: each adds an
  interface (exports, an import path, a file to find) while hiding almost
  nothing.
- **Expect fewer, bigger files.** That's the point, not a side effect. Judge a
  module by how much its interface hides, never by its line count.
- **Every export widens the interface.** That's why only real seams and test
  hooks get exported, and everything else stays local.

## The rule

An export, and therefore a module boundary, exists only when one of these is
true:

1. **Seam:** another, _unrelated_ part of the system imports it. That includes
   callers that don't exist on this branch yet but are known to be coming
   (a simulator or test framework stacked on top, a sibling PR). Pretend they
   already exist.
2. **Test hook:** a test imports it directly, because testing it through its
   caller would be impractical (e.g. SQL that needs real Postgres).

If only one module imports `Xyz`, and no test does, then `Xyz` is an
implementation detail: it becomes local, unexported code inside that module.

**A module may also keep its own existence, even with a single caller, when
it is:**

- **Library-shaped.** Generic, not project-specific code that forms a
  complete, clearly scoped abstraction of its own: something that could, with
  some effort, be packaged and published as a separate open-source library.
  Example: the FSM in the public `liveblocks` repo. Only the Liveblocks client
  uses it, but nothing about it is Liveblocks-specific.
- **Unrelated to its caller.** Code that has barely anything to do with the
  module that needs it, so folding it in would muddy that module's purpose.
- **Tested in isolation.** Code that is (or naturally could be) tested purely
  on its own, independent of the caller, as a unit with its own contract. The
  FSM is again a good example.
- **On a runtime boundary.** Code that must load in another runtime can't
  share a module with code that can't. Example: Node-run simulator code vs.
  modules importing `cloudflare:workers` or a workerd-only Prisma client.
- **A genuinely separate concept** with its own independent callers.

The test for all of these: would the module still make sense if you deleted
its current caller? If yes, it's its own thing. If it only exists to serve
that one caller, fold it in.

**Not justifications:** "it's a different kind of thing" (types, constants,
helpers), "the file would get long" (length alone is never the problem), or
"it might be reused some day" for project-specific code. That last one only
counts for library-shaped code, as above.

## Process

### 1. Scope

Take the directory, feature or branch the user points at. If unclear, ask.
Stay inside it: never move or "clean up" code in unrelated subsystems to
reuse a helper. If you spot something outside the scope, mention it and stop.

### 2. Inventory every module and export

For each module in scope, and for each of its exports, find who imports it:

```sh
git grep -n "from \"<module-path>\"" -- src test
```

Also ask about, or check, **upcoming callers** on other branches or worktrees
(e.g. a simulator branch that will be rebased on top). They count as seams.

Classify each export:

| Class     | Meaning                                                                                  | Verdict                           |
| --------- | ---------------------------------------------------------------------------------------- | --------------------------------- |
| Seam      | imported by an unrelated part of the system (incl. upcoming callers)                     | keep exported                     |
| Test hook | imported only by tests (and its one caller)                                              | keep exported, comment why        |
| Local     | imported by exactly one module, no tests, and no reason from "The rule" to keep it apart | inline into that module, unexport |
| Dead      | imported by nothing                                                                      | delete                            |

Then per module: if every export is Local to the same caller, the whole
module folds into that caller.

### 3. Look for contracts living in the wrong place

When an implementation module defines the contract (docs, types) that the
seam then `Pick`s from, flip it: the contract belongs on the seam (e.g. the
port type the consumer depends on), and the implementation satisfies it.
That often removes an export: the implementation's own type, if still
useful, stays behind as a local, unexported name.

### 4. Propose, then wait

Present the target layout as a table, and the moves that get there:

| Module                   | Imported by                  | Exports                                                       |
| ------------------------ | ---------------------------- | ------------------------------------------------------------- |
| `feature/coordinator.ts` | the DO, **simulator**        | the class + its ports                                         |
| `feature/worker.ts`      | the consumer, **simulator**  | `runWorker`, its ports and the types needed to implement them |
| `consumers/feature.ts`   | queue router, **store test** | the handler; `createStore` (test hook only)                   |

For each remaining module, say how small its interface is (its exports) compared to what it hides. Say what disappears (files, exported types) and what, if anything, is lost.
Be honest when nothing is. **Do not start moving code until the user says
go.** A question like "could we...?" or "do you need anything before
starting?" is not a go-ahead.

### 5. Execute as a pure move

- Concatenate, don't rewrite. Behaviour must not change.
- **Move verbatim, never improve.** Moved code keeps its exact body, names,
  signatures and comments. Don't fix bugs you discover, rename things,
  tighten types, simplify logic, merge near-duplicates or "modernize" syntax
  along the way, however tempting or obvious. The only edits allowed are the
  ones the move itself forces: dropping `export`, fixing import paths, and
  removing imports that became local.
- **Collect what you'd improve, don't do it.** Keep a list of every bug,
  naming issue, or conceptual wart you notice while moving, and present it at
  the end as suggested next steps, each a separate follow-up the user can pick
  up (or not) after the move lands.
- Order folded code so the module reads top-down: its entry point (or
  exported seam) first, then the folded pieces, smaller ones before big ones
  (e.g. a short stub before a long block of SQL).
- Unexport everything classified Local. Delete `types.ts` grab-bags by moving
  each type next to its seam or its only user.
- **Unexport, don't dissolve.** Folding removes exports and files, not names.
  An unexported type or helper costs no interface, so keep every local name
  that names a concept (`type DeliveryStore = …`, with its docs). Never
  replace a named type with `ReturnType<typeof …>`, `Parameters<…>` or an
  inline anonymous type just because it's no longer exported.
- Delete emptied modules with explicit paths (`git rm path/a.ts path/b.ts`),
  never globs.
- Make sure every remaining module has a short top-level comment that gives a
  quick glance at:

  1. **Its purpose:** what the module is and does, in one or two sentences.
  2. **Its usage, only when that's news:** who uses it, plus any constraint
     that follows (e.g. "must stay free of workerd-only imports").

  ```ts
  /*
   * The webhook delivery worker: runs one worker per wake, and decides what
   * each attempt's result means. Talks to the outside world only through
   * DeliveryWorkerPorts.
   *
   * Used by the delivery queue consumer (the production ports) and by the
   * delivery simulator (ports on a virtual clock). Must stay free of
   * workerd-only imports, as the simulator runs it under Node.
   */
  ```

  - **Succinct.** As short as the content allows. Say what the reader can't
    see from the code at a glance, then stop. No restating, no elaboration of
    mechanics the code already shows.
  - **No filler, ever.** Obvious things don't belong in comments. Skip any
    sentence whose content follows from the module's location or kind: a
    queue consumer is "used by the queue router", a route handler "by the
    router", a DO class "by its namespace". If usage carries no news, leave it
    out; the purpose alone is fine.
  - **No file paths or line numbers.** Refer to other code by its concept or
    symbol name ("the worker", `runDeliveryWorker`), never by path. Paths make
    comments harder to read and rot on the next move.

  - **Follow the repo's conventions** for top-level module comments: their
    style (`/* */` vs `/** */`) and placement (above or below the imports).
    Look at a few existing modules nearby first. If there's no clear
    convention, put it directly below the imports.
  - **Plain words, no labels.** Write "Used by …", not "Seam: used by …".
    Seam/Local/Test hook are this skill's working terms, not something to
    leave in the code.
  - **Build on what's there.** If a module already has a top-level comment,
    add only the purpose or usage it's missing, where it fits naturally.
    Don't repeat what it already says, even if it says it differently, and
    don't force a sentence in. If it already covers both, leave it alone.
  - **Write the purpose for every module,** not just the usage. "Who imports
    this" alone doesn't tell a reader what the module is for. A doc comment
    on the module's main export (e.g. its class) already counts when it's the
    first thing below the imports. If other declarations come first, write a
    purpose line in the header anyway.
  - **A purpose line must add something the module's name doesn't.** "The
    logic of the delivery coordinator" in `DeliveryCoordinator.ts` says
    nothing new. Say what it decides or owns instead ("decides when delivery
    workers are woken, and how much each may send").

- On a test-only export, say so: `/** Exported only for its tests, which run its SQL against real Postgres. */`
- Fix references in docs (design docs, READMEs) that point at moved code. Name
  the concept or symbol in plain text ("the consumer's `createDeliveryStore()`");
  don't add file paths or links, and drop dead ones rather than repointing
  them. Leave each edit as tidy as the text around it: rewrap the edited
  paragraph to the doc's line width, and keep table columns aligned, even
  when the doc as a whole isn't prettier-formatted.

### 6. Verify

Typecheck, lint, build, and run the tests that cover the moved code (scoped,
per the project's rules). A pure move must leave them green with no test
changes beyond import paths.

### 7. Commit (only when asked)

One commit per move. Explain **what it simplifies** in the body: which
seams remain and for whom, what became local, and that it's a pure move. If
the project has a commit-message helper (e.g. `gcm --yolo "<context>"`), pass
that explanation as its context. Mention follow-ups the move causes on other
branches (e.g. "the simulator's imports of `runDeliveryWorker.ts` now point
at `deliveryWorker.ts`").

## Example

Before: six modules, unclear which matter.

```
src/webhook-delivery/
  callEndpoint.ts          only the consumer imports it
  createDeliveryStore.ts   consumer + its test; imports ~/prisma (workerd-only)
  decideOutcome.ts         only runDeliveryWorker imports it
  DeliveryCoordinator.ts   the DO + the simulator
  runDeliveryWorker.ts     the consumer + the simulator
  types.ts                 grab-bag, used across the above
```

After: two seams in the feature directory, production plumbing with its only
caller.

```
src/webhook-delivery/
  DeliveryCoordinator.ts   used by the DO + the simulator
  deliveryWorker.ts        used by the consumer + the simulator (ports, loop,
                           decideOutcome, and the types the simulator needs)
src/queues/consumers/webhook-delivery.ts
                           handler + local store + local callEndpoint;
                           createDeliveryStore exported only for its test
```

What disappeared: four modules, and the `DeliveryStore` export: its contract
moved onto the worker's port, and the name stays as a local type in the
consumer. What was lost: nothing.

## Checklist

- [ ] Every remaining module is deep: a small interface over a lot of hidden functionality
- [ ] Every remaining export is a seam (incl. upcoming callers) or a test hook
- [ ] No module has exactly one importer and no tests, unless it's library-shaped, unrelated to its caller, tested in isolation, or on a runtime boundary
- [ ] No `types.ts` / `constants.ts` / `utils.ts` grab-bags in scope
- [ ] Contracts live on the seam, not on the implementation
- [ ] Every module has a short top-level comment: its purpose, plus its users only when that's news. Repo's style, no "Seam" label, no filler, no file paths, nothing repeated from what was already there
- [ ] Folded code keeps its local names: no `ReturnType<typeof …>` or inline anonymous types where a named type used to be
- [ ] Nothing outside the agreed scope was touched
- [ ] Pure move: typecheck, lint, tests green; docs references updated
- [ ] Moved code is verbatim: no bug fixes, renames or "improvements"; those are listed as suggested next steps instead
