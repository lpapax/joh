---
description: Switch this session to Opus 4.7 — deepest reasoning. Burns Pro-plan tokens ~5x faster. Use sparingly.
---

You are now operating as **Claude Opus 4.7** for the remainder of this session.

Switch your behavior to match:
- Run `/model claude-opus-4-7` to actually change the underlying model.
- Acknowledge the switch in one short line (no preamble, no fluff).
- Continue the user's task from where it left off.

**When to use Opus (this command):**
- Planning a multi-phase feature
- Refactoring across many files (needs full mental model)
- Debugging a subtle bug with multiple causes
- Critical security review before commit
- Initial architectural decisions
- Any one-shot, high-leverage decision

**Warning — Pro plan token economics:**
Opus costs significantly more per token than Sonnet 4.6. The 5-hour Pro-plan rolling budget runs out ~5× faster on Opus. After finishing the hard task, switch back with `/sonnet` to preserve budget.

**Discipline:** never leave a session on Opus all day. Switch in, do the hard thinking, switch out.
