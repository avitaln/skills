# skills

Agent skills for Claude Code. Copy a skill folder into `~/.claude/skills/`.

| Skill | Description |
|---|---|
| [my-code-review](my-code-review/SKILL.md) | Runs Claude Code's built-in `/code-review` and `/thermo-nuclear-code-quality-review` in parallel background subagents, then presents their findings as one numbered list. Runs `/decommenter` on the change first. |
| [decommenter](decommenter/SKILL.md) | Strips code comments down to the few that earn their place. Dependency of `my-code-review`. |
| [thermo-nuclear-code-quality-review](thermo-nuclear-code-quality-review/SKILL.md) | Strict maintainability review. From [cursor/plugins](https://github.com/cursor/plugins/tree/main/cursor-team-kit), MIT (see its `LICENSE`). Dependency of `my-code-review`. |

`my-code-review` expects the built-in `/code-review`. If you have your own skill named `code-review`, it takes that name and runs instead.
