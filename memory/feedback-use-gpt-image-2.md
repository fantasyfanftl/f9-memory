---
name: feedback-use-gpt-image-2
description: "When generating images, always invoke the gpt-image-2 skill (currently v1.0.1) rather than any other image tool or ad-hoc approach."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 20af5d76-72b7-4eb0-bc5c-a629867b8f8b
---

Whenever a task calls for image generation, invoke the `gpt-image-2` skill (current version: v1.0.1). Do not fall back to other image generators or improvise a workflow.

**Why:** The user standardized on gpt-image-2 for their projects and wants consistent output. They also said "需要做图的时候自己调用 skill" — they want the recognition-and-invocation to be autonomous, not gated on them naming the tool each time.

**How to apply:**
- Any request that materially involves an image — icons, illustrations, mockups, game assets, thumbnails, promo images — invoke `Skill(gpt-image-2)` first, then run its `generate_image.py` script per its instructions.
- Recognize the intent from paraphrases too: "画一下", "优化图案", "换个图", "生成图片", "背景重做" all count. Don't wait for the literal skill name.
- **Batch in parallel** for multi-image tasks (e.g. all game scenes): fire N background bash tasks concurrently rather than serializing. Each generation takes ~90–120s at high quality.
- Save into the project's asset folder with descriptive filenames (e.g. `lycheeGame/assets/scenes/01-doorstep.png`), then wire into the HTML in the same iteration so the user sees the result live.
- Even if another image-related skill is listed in the available-skills reminder, prefer gpt-image-2 unless the user explicitly names a different one.
- Only fall back to hand-authored SVG when the user explicitly asks for stylized-minimal / interactive / vector-required output.
