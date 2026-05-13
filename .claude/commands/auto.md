---
description: Show the model-routing cheat-sheet — when to use Sonnet vs Opus on this project.
---

Print the following cheat-sheet exactly, then stop. Do not switch models. Do not start any task. The user is asking which command to type next.

```
╔═══════════════════════════════════════════════════════════════════╗
║  MODEL ROUTING CHEAT-SHEET  (Jo Guest House · Pro plan)           ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  /sonnet   →  Claude Sonnet 4.6     CHEAP · DEFAULT               ║
║  /opus     →  Claude Opus 4.7       EXPENSIVE · USE SPARINGLY     ║
║                                                                   ║
╠═══════════════════════════════════════════════════════════════════╣
║  USE SONNET FOR:                                                  ║
║    • File reads, summaries, listing TODOs                         ║
║    • Mechanical edits (renames, formatting, comments)             ║
║    • CSS / Tailwind tweaks                                        ║
║    • Translating copy EN ↔ ID                                     ║
║    • Building new components (best coding model)                  ║
║    • Running lint / fixing simple errors                          ║
║                                                                   ║
║  USE OPUS FOR:                                                    ║
║    • Multi-phase feature planning                                 ║
║    • Refactoring across many files                                ║
║    • Subtle bug debugging (multiple causes)                       ║
║    • Pre-commit security review                                   ║
║    • Initial architectural decisions                              ║
║                                                                   ║
╠═══════════════════════════════════════════════════════════════════╣
║  RULE OF THUMB:                                                   ║
║    Start every session on /sonnet.                                ║
║    Switch /opus only when Sonnet misses context or oversimplifies.║
║    Switch back to /sonnet immediately after the hard step.        ║
║    Never leave a session on Opus all day.                         ║
╚═══════════════════════════════════════════════════════════════════╝
```

After printing, ask: *"Which one should I switch to?"*
