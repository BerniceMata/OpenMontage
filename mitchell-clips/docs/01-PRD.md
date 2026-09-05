# 01: PRD: mitchell-clips

## Problem

289 long-form videos are sitting on YouTube doing nothing for distribution. The channel's stated thesis (validated by Leila Hormozi, Caleb Ralston, and the niche map in `knowledge/`) is: long-form is the source, short-form is distribution, Instagram is the platform that grows a solo commentary creator from zero. Mitchell has the raw material. He does not have the machine.

## Goal

A pipeline that turns existing YouTube footage into approved, high-retention Reels on a fixed cadence with near-zero weekly effort from Mitchell beyond approving a batch.

## Users

One. Mitchell. Operator. He approves, edits a caption occasionally, and reads the Friday scoreboard.

## Success metrics

| Metric | Milestone 1 | 30 days |
|---|---|---|
| Clips in dashboard | 20 from top 10 videos | rolling ≥ 21 (a week of runway) |
| Posts shipped | 1 (proof) | 3/day, 90 total |
| Mitchell's time | one approval session | ≤ 30 min/week approving |
| Outliers (≥3× rolling median views) | n/a | ≥ 2 identified and cloned |
| View-to-follower conversion | n/a | ≥ 1% on any post ≥ 10K views |

Views are not the scoreboard. Posts shipped and outlier multiple are.

## Scope

**In (Milestone 1):**
- Rank channel videos by views (Composio YouTube) and locate local originals
- GPU transcription with word timestamps
- LLM scoring of transcript windows against the channel's content rules
- 9:16 face-tracked reframe, silence trim, end-at-peak trim
- Burned karaoke captions + 2s hook overlay
- Caption text drafted in the channel's caption style
- Local approval dashboard (preview, edit, approve, reject, reschedule)
- Composio publish + permalink stored
- Scheduler for 3 slots/day

**In (Milestone 2):**
- Scoreboard: pull insights at 1h/24h/7d, compute outlier multiple, surface clone candidates
- Comment mining: pull comments on outliers, flag sentences better than the caption
- Process the full 289 backlog

**Out (for now):**
- TikTok / YouTube Shorts cross-posting (Lake Method says native re-export; later)
- Any comment-keyword / ManyChat funnel (nothing to sell yet)
- Multi-account theme pages
- Paid API spend by default

## Constraints

- Runs on Mitchell's Windows desktop only when it's on. Scheduler must tolerate missed slots (post the next slot, never double-post).
- Zero API spend to start. Ollama local. `llm.provider` flips to `anthropic` when he wants sharper picks.
- Nothing posts without an explicit approval in the dashboard. No exceptions, no flags to bypass.
- No hashtags. No "you need to" openers. No em dashes in captions.

## Risks

| Risk | Mitigation |
|---|---|
| Local 8B model picks mediocre clips | Rubric + JSON schema + hard rule filters do most of the work; dashboard rejection is cheap; upgrade path is one config key |
| Face tracking jitters on handheld footage | One-Euro smoothing, scene-cut resets, center-crop fallback per clip |
| Instagram rejects the file | Encode to spec (H.264, AAC, 30fps, ≤1080 wide, 9:16, ≤300MB); pre-validate with ffprobe before upload |
| Composio publish quota / auth expiry | Check `INSTAGRAM_GET_IG_USER_CONTENT_PUBLISHING_LIMIT` daily; alert in dashboard on 190/OAuth errors |
| Mitchell stops approving | Dashboard shows runway in days; scheduler emails/notifies when runway < 2 days |
