# 03: Decisions log

Format: decision, who, date, status. Claude Code does not re-open "fixed" items. "Flagged" items have a research conflict Mitchell should resolve once, then they become fixed.

## Fixed (Mitchell, 2026-09-04)

1. **Build target is a Claude Code repo on his desktop.** Cowork packaged; Claude Code builds and operates.
2. **Approve queue first.** No autonomous posting. `publishing.require_approval` is never false.
3. **3 posts/day, one account.** Not 10. Theme accounts are a Milestone 3 conversation after 30 days of data.
4. **Local-first, zero API spend to start.** Ollama for scoring and captions. `llm.provider: anthropic` is the upgrade, one line.
5. **Windows desktop with NVIDIA GPU is the runtime.** Cloud later only if the desktop is off too often to hit slots.
6. **Milestone 1 = 20 clips from top 10 videos in the dashboard**, then one real post.
7. **Style: 9:16 face crop + word captions**, hook line burned in for the first 2s.
8. **Composio is the publishing layer.** Instagram and YouTube are already connected on his Composio account.
9. **Audience: broad perspective-shift crowd, faith as flavor.** The scorer weights toward the channel promise: "I never thought about it like that, but I've felt that before."

## Flagged (today's answer vs. the project's own research)

### F1. Clip length: 45s max vs. research says 45-180s wins
- Mitchell said: max 45s.
- `knowledge/niche-competitor-map.md` Pattern 1 (verified in both sweeps): 45-180s is the winning length in this exact lane; sub-30 loses. `knowledge/content-os.md` set the target at 45-95s.
- **Default shipped:** `max_clip_seconds: 45` (his call). **Recommendation:** raise to 75 after the first 20 clips and let the scoreboard decide. The selector already snaps to the emotional peak, so longer does not mean rambling.

### F2. CTA on every post vs. research says zero asks on the biggest outliers
- Mitchell said: question + drive to YouTube on every caption.
- Niche map Pattern 4 and content-os §3.5: 5 of 7 posts no ask, 1 of 7 a real question, 1 of 7 follow/series breadcrumb. The biggest outliers in this lane have no CTA.
- **Default shipped:** the 7-slot rotation from content-os (`caption_text.cta_rotation`), with the "youtube_push" slot replacing "follow". Mitchell can set all seven to `youtube_push` in config if he wants his version.

### F3. Cadence: 3/day vs. content-os said 1/day max in Phase 1
- Content-os was written for a filming-based workflow at 6 hrs/week. This pipeline removes the filming cost, so 3/day from existing footage is consistent with Brock's volume-first Phase 1 and Caleb's Accordion (expand for data, then contract).
- **Default shipped:** 3/day. Revisit at week 4: if no post exceeds 1.5× median in two consecutive weeks, change hooks (not cadence, not topics).

### F4. Hook overlay text vs. "no text-heavy thumbnails"
- Channel OS: thumbnail text ≤ 4 words. Content-os: cover text ≤ 4 words from the hook.
- **Shipped:** hook overlay is capped at 4 words, pulled verbatim from the hook line, on screen 2s.

## Open (need Mitchell, not research)

- O1. Path to the local originals folder (`paths.videos_local`). Set in `config.yaml` on first run.
- O2. Which GPU / VRAM. Determines `llm.ollama_model` (8GB → qwen3:8b; 12GB+ → qwen3:14b or gemma4:12b).
- O3. Font choice for captions. Default Inter SemiBold. He may prefer whatever CapCut "classic" is.
- O4. Whether reposting to YouTube Shorts/TikTok in Milestone 2 (Lake Method: native re-export, not waterfall).
