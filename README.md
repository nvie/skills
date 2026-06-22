# nvie/skills

A collection of [Claude Code skills](https://docs.anthropic.com/en/docs/claude-code/skills) for personal productivity.

## Available Skills

| Skill                                                            | Maturity                                            | Description                                                                                                                                                                                                                                                                                                                                                                  |
| ---------------------------------------------------------------- | --------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [liveblocks-new-npm-package](skills/liveblocks-new-npm-package/) | 5 - Happy with                                      | Publish a new placeholder package to NPM under the `@liveblocks/\*` org.                                                                                                                                                                                                                                                                                                     |
| [mikado](skills/mikado/)                                         | 5 - Happy with                                      | Mikado method workflow for complex refactorings -- break large changes into safe, incremental steps with automatic revert-on-failure.                                                                                                                                                                                                                                        |
| [pr](skills/pr/)                                                 | 4 - Happy with, but trying to make it faster        | Open a pre-populated GitHub PR creation page with title and body drafted in nvie's personal style (no boilerplate "Test plan").                                                                                                                                                                                                                                              |
| [pr-review](skills/pr-review/)                                   | 0 - Experimental                                    | Review one or more GitHub PRs as a progressive, understanding-first code review -- start at the highest level and drill down one abstraction layer at a time, treating the diff as source of truth and surfacing only interesting findings and good questions to ask the author. Built for a senior engineer who wants to understand a PR before spending review time on it. |
| [proper](skills/proper/)                                         | 4 - Happy with, but needs more practical experience | Solve a request the Proper™ way: diagnose architectural friction, propose a refactoring that makes the feature fit naturally, hand off to /mikado for non-trivial execution.                                                                                                                                                                                                 |
| [self-review](skills/self-review/)                               | 3 - Can be improved                                 | Analyze past Claude Code sessions to surface repeated corrections and preferences, then persist them as CLAUDE.md rules or memories.                                                                                                                                                                                                                                         |
| [tech-design](skills/tech-design/)                               | 3 - Getting pretty happy with, but needs refinement | Draft, review, pressure-test, and compare technical design documents -- help engineers make and defend a technical decision at the right depth, not just fill a template.                                                                                                                                                                                                    |
| [visually-explain](skills/visually-explain/)                     | 0 - Experimental                                    | Produce a single self-contained interactive HTML document that visually explains how a specific codebase mechanism works -- a concrete example walked through its states with toggles, steppers, diffs, and color-coded state, for visual learners who find static markdown insufficient. Correctness over polish: a buggy explainer is worse than none.                     |

## Installation

```sh
npx skills add nvie/skills@self-review
```

Or install all skills from this repo:

```sh
npx skills add nvie/skills
```

## License

MIT
