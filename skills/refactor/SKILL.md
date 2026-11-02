---
name: refactor
description: >
  Perform a pure mechanical refactoring without changing logic, meaning, or
  behavior: renames, extractions, inlinings, moves. Use when the user invokes
  /refactor, or asks for a "pure refactor", a "mechanical refactoring", or a
  change that must not alter behavior.
---

# Refactor

The refactoring to perform is whatever the user asked for when invoking this
skill.

## Operating Constraints

You are performing a **PURE REFACTORING**. This means:

### MUST NOT change:

- **Logic or behavior** - The code must do exactly the same thing before and after
- **Meaning** - Variable semantics, function contracts, return values must stay identical
- **Comments** - Keep existing comments unless they would become factually incorrect after the refactoring
- **Input/output types** - Unless changing types IS the explicit goal of this refactoring
- **Syntactical markers** - Do not touch `// prettier-ignore`, `// eslint-disable`, `@ts-expect-error`, etc. unless explicitly asked

### MUST do:

- Keep the refactoring as **mechanical** as possible
- Preserve all existing behavior exactly
- If renaming, rename consistently everywhere
- If extracting, ensure the extracted code is called correctly
- **When inlining**, rename variables if needed to avoid shadowing or conflicts (e.g., inlining `bar` into `foo` when both use `y` → rename to `y_2` or similar)

### When in doubt:

- **ASK before proceeding** - If there's ambiguity about whether a change is purely mechanical
- **ASK before proceeding** - If you see a strong reason to deviate from these constraints
- **ASK before proceeding** - If the refactoring request seems to imply logic changes

## Execution

1. Read and understand the code to be refactored
2. Identify the minimal set of changes needed
3. Apply changes mechanically
4. Verify the refactoring is pure (no logic/behavior changes)
5. Run typechecks if applicable to ensure no type errors were introduced
