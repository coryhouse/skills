# skills

Reusable [Claude Code](https://docs.claude.com/en/docs/claude-code) skills.

## Skills

| Skill | What it does |
| --- | --- |
| [`tune-agents-md`](./skills/tune-agents-md) | Audit and improve a project's `AGENTS.md` / `CLAUDE.md` so it's a high-leverage instructions file for AI assistants. |

## Install

```bash
npx skills@latest add coryhouse/skills
```

The installer prompts for which skills to install and which agent(s) (Claude Code, Codex, Cursor, etc.) to install them into. See [skills.sh](https://skills.sh) for details.

### Manual install

If you'd rather not run the installer, clone the repo and symlink the skill into your user-level Claude Code skills directory. Symlinking (rather than copying) means `git pull` updates the skill in place.

```sh
git clone https://github.com/coryhouse/skills.git ~/.skills-repo
ln -s ~/.skills-repo/skills/tune-agents-md ~/.claude/skills/tune-agents-md
```

To install every skill in this repo:

```sh
ln -s ~/.skills-repo/skills/*/ ~/.claude/skills/
```

Verify Claude Code sees a skill by running `/skills` inside a session — it should appear in the list.
