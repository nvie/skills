# nvie/skills

A collection of [Claude Code skills](https://docs.anthropic.com/en/docs/claude-code/skills) for personal productivity.

## Available Skills

| Skill                              | Description                                                                                                                                                                  |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [mikado](skills/mikado/)           | Mikado method workflow for complex refactorings -- break large changes into safe, incremental steps with automatic revert-on-failure.                                        |
| [pr](skills/pr/)                   | Open a pre-populated GitHub PR creation page with title and body drafted in nvie's personal style (no boilerplate "Test plan").                                              |
| [proper](skills/proper/)           | Solve a request the Proper™ way: diagnose architectural friction, propose a refactoring that makes the feature fit naturally, hand off to /mikado for non-trivial execution. |
| [self-review](skills/self-review/) | Analyze past Claude Code sessions to surface repeated corrections and preferences, then persist them as CLAUDE.md rules or memories.                                         |

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
