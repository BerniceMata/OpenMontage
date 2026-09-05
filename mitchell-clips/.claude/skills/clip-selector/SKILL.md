---
name: clip-selector
description: Run or re-run clip selection on one video and review the candidates. Use when Mitchell says "find clips in <video>", "re-score", "the picks are weak", or after changing prompts/clip_selector.md.
---

# clip-selector

1. Identify the video: by ID, or fuzzy-match the title against the `videos` table. If ambiguous, ask one question.
2. Ensure transcript + scenes exist (`python -m clips.run --video <id> --stage transcribe`).
3. Run `python -m clips.run --video <id> --stage select --clips-per-video <n>`.
4. Print the candidates as a table: hook_line (verbatim), hook_structure, topic, duration, composite score, one_idea.
5. Read them like Mitchell would. For each, one line: keep / cut and why, using the content rules in CLAUDE.md. If more than half are weak, the prompt is the problem, not the model: propose one concrete edit to `prompts/clip_selector.md`, apply it only after he says yes, re-run.
6. End with the verb: "Render these N?" or "Approve in the dashboard."

Never paraphrase his hook lines. Never pick a clip that opens with "you."
