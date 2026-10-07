# skills

Agent skills for coding assistants, LLMs, and AI agents. One directory per skill, each with a `SKILL.md`.

| Skill | What it does |
| --- | --- |
| [`drill-me`](./drill-me/SKILL.md) | The reverse of Matt Pocock's `grill-me`. The model reads up on a ticket, PR, epic, or subsystem, then interrogates *you* about it so the concepts stick through retrieval practice instead of a summary you'll forget by lunchtime. Supports open-ended and multi-option questions with Claude Code CLI and Desktop interactive pickers, ending with a score, score reasoning, and a domain debrief. |

## Setup

Clone the repository:

```bash
git clone https://github.com/riccivr/skills ~/code/skills
```

### Agents with native skill folders

Link the skill into your agent's skills directory:

**Claude Code**
```bash
ln -s ~/code/skills/drill-me ~/.claude/skills/drill-me
# Or per project:
ln -s ~/code/skills/drill-me <repo>/.claude/skills/drill-me
```

**Antigravity**
```bash
ln -s ~/code/skills/drill-me ~/.gemini/antigravity/skills/drill-me
# Or per project:
ln -s ~/code/skills/drill-me <repo>/.gemini/skills/drill-me
```

Run with `/drill-me ENROLY-1234`, `/drill-me 7905`, `/drill-me src/billing`, or bare `/drill-me` on the current branch. Add `--options` to use multi-option questions with interactive pickers in Claude Code CLI and Desktop. Add `--quick` or `--deep` to adjust depth.

### Rule-based tools (Cursor, Windsurf, Cline, Copilot)

Add the skill file or reference it in your tool's instructions:

- **Cursor:** copy or link `drill-me/SKILL.md` to `.cursor/rules/drill-me.mdc`
- **Cline / Roo Code:** add to `.clinerules` or link under custom modes
- **Windsurf:** add to `.windsurfrules`
- **GitHub Copilot:** reference in `.github/copilot-instructions.md`

Trigger by asking: "drill me on `<ticket / path / PR>`".

### Any LLM (ChatGPT, Claude web, local models)

Paste the contents of [`drill-me/SKILL.md`](./drill-me/SKILL.md) into your system prompt, custom instructions, or project knowledge.

To use it in a conversation, share the ticket, code, or PR diff and say:

> Drill me on this.

Add `--quick` (5 questions) or `--deep` (until edge cases pass) if needed.

## Credit

`drill-me` exists because of [`grill-me`](https://github.com/mattpocock/skills). Same trick, opposite direction.
