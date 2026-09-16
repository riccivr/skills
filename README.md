# skills

Personal agent skills for Claude Code and friends. One directory per skill, each with a `SKILL.md`.

| Skill | What it does |
| --- | --- |
| [`drill-me`](./drill-me/SKILL.md) | The reverse of Matt Pocock's `grill-me`. The model reads up on a ticket, PR, epic or subsystem, then interrogates *you* about it so the concepts stick through retrieval practice instead of a summary you'll forget by lunchtime. |

## Install

Claude Code loads skills from `~/.claude/skills/<name>/SKILL.md` (personal) or `<repo>/.claude/skills/<name>/SKILL.md` (per project). Clone and link the ones you want:

```bash
git clone https://github.com/riccivr/skills ~/code/skills
ln -s ~/code/skills/drill-me ~/.claude/skills/drill-me
```

Then `/drill-me ENROLY-1234`, `/drill-me 7905`, `/drill-me src/billing`, or bare `/drill-me` for the current branch. Add `--quick` or `--deep` to change how far it goes.

## Credit

`drill-me` exists because of [`grill-me`](https://github.com/mattpocock/skills). Same trick, opposite direction.
