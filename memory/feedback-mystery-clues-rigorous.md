---
name: feedback-mystery-clues-rigorous
description: "For 案卷九章 (sudoku murder-mystery hub), the 9 clues per case must rigorously support the correct answer. No hand-waving, no \"you couldn't have known that\" reveals."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 20af5d76-72b7-4eb0-bc5c-a629867b8f8b
---

Every case in [[lychee-game-site]]'s 案卷九章 (sudoku.html) must have 9 clues that form a **fair, complete deduction chain** to the correct suspect + method. Clues cannot be atmospheric filler.

**Why:** User called this out explicitly. The game is 数独 + 剧本杀 — the 剧本杀 half falls apart if the clues don't rigorously imply the answer. Bad mysteries feel cheap; users notice when a "reveal" comes from information the player couldn't have deduced.

**How to apply:**
- **Motive**: at least one clue establishes the killer's motive (why THIS suspect, not the other 3).
- **Means**: at least one clue points to the specific method (why THIS 手法, not the other 2 — e.g., signs of the specific weapon/poison/mechanism used).
- **Opportunity**: at least one clue puts the killer at the scene / rules out others' alibis.
- **Identity misdirection resolved**: if the killer's identity has a twist (private child, hidden loyalty, past grudge), one clue must plant the twist and one must confirm it.
- **Alibi coverage**: for each of the OTHER 3 suspects, some clue should weaken them as candidates (alibi, contradiction, or explicit rule-out).
- **No orphan clues**: every clue should serve motive/means/opportunity/misdirection or actively rule out a suspect. Delete or rewrite atmospheric-only clues.
- **The `wrong` line**: the 「你漏看了 X」 pointer must reference an actual clue in the array (not something invented in the reveal).
- When adding a new case OR editing a case's answer, verify against these criteria BEFORE saving.
