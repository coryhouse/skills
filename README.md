# skills

Reusable [Claude Code](https://docs.claude.com/en/docs/claude-code) skills.

## Skills

| Skill | What it does |
| --- | --- |
| [`tune-agents-md`](./tune-agents-md) | Audit and improve a project's `AGENTS.md` / `CLAUDE.md` so it's a high-leverage instructions file for AI assistants. |

## Install

Clone this repo once, then symlink the skills you want into your user-level Claude Code skills directory. Symlinking (rather than copying) means `git pull` updates the skills in place.

```sh
git clone https://github.com/coryhouse/skills.git ~/skills
ln -s ~/skills/tune-agents-md ~/.claude/skills/tune-agents-md
```

To install every skill in this repo:

```sh
ln -s ~/skills/*/ ~/.claude/skills/
```

Verify Claude Code sees a skill by running `/skills` inside a session — it should appear in the list.
