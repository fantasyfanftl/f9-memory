---
name: feedback-auto-menu-cards
description: "When working on lycheeGame, always scan for .html files not in index.html menu and add cards for them"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: e9fffeef-0e94-453d-b255-7266207ded35
---

Whenever I touch anything in [[lychee-game-site]], first check for `.html` files under `C:\Users\tianlan\lycheeGame\` that are NOT linked from `index.html`'s card grid, and add cards for them.

**Why:** Another AI works in parallel adding new games and doesn't update the menu. User wants me to notice + link them automatically, not wait to be told. Confirmed 2026-07-04: "以后可以自动补吗".

**How to apply:**
- At the START of any lycheeGame session (or before finishing a lycheeGame task), run: `ls /c/Users/tianlan/lycheeGame/*.html` and compare to `href=` values in `index.html`.
- For each unlinked file, `grep -m 1 "<title>" <file>` to get the title, then infer a description from the first `<meta>` or first paragraph.
- Add a card to `index.html` matching the existing pattern (emoji, h2, description, tags, play link).
- Pick emoji + tags based on the game concept (read the file briefly if unclear).
- Skip `index.html` and `midnight-store.html` (both intentional entries).
- If unsure whether the file is finished/playable, add with a `开发中` status tag.
- Mention in the reply: "另加了 X 个新游戏的卡片: ..." so user knows what changed.
