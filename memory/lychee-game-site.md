---
name: lychee-game-site
description: "fantasyfanftl's public game hub site (GitHub Pages), local + remote layout"
metadata: 
  node_type: memory
  type: reference
  originSessionId: e9fffeef-0e94-453d-b255-7266207ded35
---

Public game hub site.

- **Local**: `C:\Users\tianlan\lycheeGame\`
- **GitHub repo**: https://github.com/fantasyfanftl/lycheeGame
- **Live URL**: https://fantasyfanftl.github.io/lycheeGame/
- **Deploy**: custom `.github/workflows/deploy.yml` (uses `actions/deploy-pages@v4`) — DO NOT delete or replace with built-in Jekyll deployer, see [[lychee-game-deploy-history]]

**URL rule**: file path in repo = URL path.
- `index.html` → site root (currently the game hub menu page)
- `sudoku.html` → `/sudoku.html`
- `foo/bar.html` → `/foo/bar.html`

**Filename convention**: lowercase ASCII + hyphens only (no Chinese, no spaces, case-sensitive).

**Existing games** (as of 2026-07-03):
- `sudoku.html` — 9×9 sudoku with 4 difficulties, timer, mistakes counter
- `convenience-store-game.html` / `convenience-store-game-v1.html` — untracked, being organized by another AI
