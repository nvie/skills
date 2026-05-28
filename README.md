# nvie/skills

A collection of [Claude Code skills](https://docs.anthropic.com/en/docs/claude-code/skills) for personal productivity.

## Available Skills

| Skill                              | Maturity                                            | Description                                                                                                                                                                             |
| ---------------------------------- | --------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [implement](skills/implement/)     | 0 - Experimental                                    | Implement a spec while keeping a running (throwaway) implementation-notes.md of every decision, deviation, and tradeoff the spec didn't cover -- review the deltas, not the whole diff. |
| [mikado](skills/mikado/)           | 5 - Happy with                                      | Mikado method workflow for complex refactorings -- break large changes into safe, incremental steps with automatic revert-on-failure.                                                   |
| [pr](skills/pr/)                   | 4 - Happy with, but trying to make it faster        | Open a pre-populated GitHub PR creation page with title and body drafted in nvie's personal style (no boilerplate "Test plan").                                                         |
| [proper](skills/proper/)           | 4 - Happy with, but needs more practical experience | Solve a request the Proper™ way: diagnose architectural friction, propose a refactoring that makes the feature fit naturally, hand off to /mikado for non-trivial execution.            |
| [self-review](skills/self-review/) | 3 - Can be improved                                 | Analyze past Claude Code sessions to surface repeated corrections and preferences, then persist them as CLAUDE.md rules or memories.                                                    |
| [tech-design](skills/tech-design/) | 0 - Experimental                                    | Draft, review, pressure-test, and compare technical design documents -- help engineers make and defend a technical decision at the right depth, not just fill a template.               |

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
