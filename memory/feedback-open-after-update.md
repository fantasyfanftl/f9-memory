---
name: feedback-open-after-update
description: "For any lycheeGame iteration, open the deployed URL in the browser at the end of the response — don't wait for the user to ask"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: e9fffeef-0e94-453d-b255-7266207ded35
---

After editing any HTML file under [[lychee-game-site]], open the corresponding **deployed URL** in the browser as the final step of the response.

**Why:** The user iterates by looking at the change live, not by reading a diff. They confirmed this multiple times ("以后更新完直接打开看看"). Skipping this step forces them to type "打开" every turn.

**How to apply:**
- After the last Edit/Write, run one Bash call:
  `start "" "https://fantasyfanftl.github.io/lycheeGame/<file>"`
  where `<file>` is the file just edited (or `` for site root when editing `index.html`).
- Use the **public URL**, not the local file path — the site auto-syncs via [[lychee-game-site]]'s deploy workflow, and the deployed page is what the user actually looks at. Local file preview is stale (module scripts and relative paths behave differently anyway).
- Wait a moment first if the sync hasn't fired yet: since auto-sync polls every ~15s, the change is usually live within 30–60s. If the change is fresh, mention "Ctrl+Shift+R to bypass cache" in the response.
- Skip only when the change is purely internal (renaming a JS variable, moving comments) or when the user explicitly said "don't open."
