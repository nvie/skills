---
name: self-review
description: Scrape Claude Code session history to find repeated user corrections, preferences, and contextual guidance, then surface candidates for CLAUDE.md or memory. Use when the user wants to review past sessions for self-improvement, find patterns in their feedback, or bake recurring preferences into persistent configuration.
---

# Self-Review

Analyze past Claude Code sessions to find repeated user corrections and preferences that should be persisted as memories or CLAUDE.md entries.

## When to Use This Skill

Use this when the user wants to:

- Review past Claude Code sessions for recurring feedback patterns
- Find preferences or corrections they've had to repeat across sessions
- Identify candidates for CLAUDE.md rules or memory entries
- Perform a "self-improvement" pass on their Claude Code setup

## Instructions

### Step 1: Gather session data

Scan all JSONL session files from:

- `~/.claude/projects/` (project-scoped sessions)
- `~/.claude/sessions/` (global sessions, if present)

Each `.jsonl` file is one session. Parse each line as JSON.

### Step 2: Extract user messages

From each session, extract messages where the type is `"user"`. Focus on the text content of these messages.

### Step 3: Identify correction and preference signals

Filter for messages that match these patterns:

- **Corrections**: "no", "not that", "don't", "stop", "wrong", "I said", "I meant", "actually", "instead"
- **Repeated preferences**: instructions about style, workflow, tools, formatting, or conventions
- **Contextual guidance**: "always", "never", "from now on", "every time", "remember that"
- **Frustration signals**: user restating something they've said before, or clarifying after a misunderstanding

Ignore routine messages like "yes", "thanks", "looks good", simple task requests without preference signals, or one-off debugging instructions unlikely to recur.

### Step 4: Classify destinations

For each candidate, determine where it belongs:

- **Global CLAUDE.md** (`~/.claude/CLAUDE.md`): user-wide preferences that apply across all projects (communication style, git workflow, general coding preferences)
- **Project CLAUDE.md** (`.claude/CLAUDE.md` in a project root): project-specific conventions, architecture decisions, or tooling preferences
- **Memory** (user/feedback type): personal context, role info, or nuanced preferences that benefit from the richer memory format

Check whether the preference is **already captured** in an existing CLAUDE.md or memory file. Skip duplicates.

### Step 5: Present findings

Present a summary table sorted by frequency (most repeated first):

```
| # | Pattern | Freq | Sessions | Destination | Status |
|---|---------|------|----------|-------------|--------|
| 1 | description | N | session-ids | global/project/memory | new/exists |
```

For each entry include:

- **One-line description** of the preference or correction
- **Frequency**: how many times it appeared across sessions
- **Session IDs**: which sessions contained it (use the session directory/file name)
- **Destination**: where it should be persisted
- **Status**: whether it's already captured somewhere or new

### Step 6: Interactive review

Walk through the list with the user one by one. For each item:

1. Show the exact user quotes (with session context)
2. Propose the specific text to add to CLAUDE.md or memory
3. Wait for the user to approve, modify, or skip
4. Apply approved changes immediately

Do NOT batch-apply changes without explicit user approval for each one.

## Important Notes

- Session files can be large. Process them efficiently — stream line by line rather than loading entire files into memory.
- Some sessions may be very old or irrelevant. If there are many sessions, consider asking the user if they want to limit the time range.
- Be honest about ambiguous cases. If a message could be a one-off correction or a lasting preference, flag the ambiguity.
- Respect privacy: session data may contain sensitive information. Do not output raw session content beyond the relevant quotes needed for review.
