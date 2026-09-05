# CLAUDE.md: mitchell-clips

You are building and operating an auto-clipper: Mitchell Nicholas's YouTube channel (289 long-form videos) → high-retention 9:16 clips → approval dashboard → Instagram Reels via Composio.

Read this file fully before doing anything. Then read `docs/04-BUILD-PLAN.md` and pick up the first unchecked task.

## Who you're working for

Mitchell, 21, DFW. Operator, not a craftsman. He cares that the goal was accomplished, not how elegant the code is. He will not read long explanations. He voice-to-texts and doesn't re-read; read for intent, ask one clarifying question max.

Rules from his global CLAUDE.md that apply here:
- Claude builds the machine. Mitchell makes the calls (approvals, final captions, what posts).
- Every work response ends with a verb or a deadline.
- No plan inflation. The plan is `docs/04-BUILD-PLAN.md`. Do not write a new one. Execute it.
- Ask before shipping anything to production. "Production" here means: posting to Instagram, changing what's in the approved queue, deleting source files.
- No em dashes in anything user-facing (captions, dashboard copy, docs). Use commas, periods, or colons.
- Never invent facts, stats, or repo details. If unsure, search or say so.

## The one-sentence goal

Ship 20 approved clips from his top 10 videos into the dashboard (Milestone 1), then run 3 posts/day on autopilot with him approving batches.

## Fixed decisions (do not re-open these)

| Decision | Value | Source |
|---|---|---|
| Source | Local originals on his desktop; fallback `yt-dlp` from channel `UCHCo7ZlH2hCEPlBUqp-Zu-A` (@mitchellnicholas2099) | Mitchell, 2026-09-04 |
| Runtime | His Windows 11 desktop, NVIDIA GPU, Python 3.11, project at `C:\clips` | Mitchell |
| Transcription | faster-whisper (CUDA), word timestamps | docs/05 |
| Clip selection | Local LLM via Ollama (zero API spend to start). Model per `config.yaml`. Upgrade path: Claude API, flip one config key | Mitchell |
| Reframe | MediaPipe face detect + smoothed crop to 1080x1920, scene-cut resets via PySceneDetect | docs/05 |
| Captions | ASS karaoke (pysubs2) burned in via ffmpeg libass. 2-3 words per line, classic font, subtle shadow | knowledge/content-os.md §3.3 |
| Approval | Local web dashboard. Nothing posts unapproved. | Mitchell |
| Publishing | Composio `INSTAGRAM_POST_IG_USER_MEDIA` (with `video_file` upload) → `INSTAGRAM_POST_IG_USER_MEDIA_PUBLISH` | docs/06 |
| Cadence | 3 Reels/day on one account (`@mitchloveswords`, ig_user_id `28108917608702310`) | Mitchell |
| Clip length | Default 45s max (Mitchell's call). Config knob `max_clip_seconds`. See docs/03 for the research conflict | Mitchell |
| Caption | Lowercase, first person, the one sentence the viewer carries. CTA rotation per `config.yaml` | knowledge |
| Hashtags | None. Ever. | knowledge/niche-competitor-map.md |
| Audience | Broad "perspective" crowd; faith present, not the lead | Mitchell + channel OS |

## Non-negotiable content rules (bake these into the selector and the dashboard)

1. Hook indicts the speaker, never the viewer. Clips that open with "you need to" get rejected by the scorer.
2. Concrete beats abstract in the first line. A named person, number, or story beat. Never an abstract noun.
3. End at the emotional peak. Cut there, delete everything after (Yasin rule).
4. Cut dead air, even 0.2s. Keep pauses that carry emotion, laughs, self-corrections, "bro."
5. Caption is the takeaway written so it works alone on Explore. Not a description.
6. One idea per clip. Test: "This clip is about ___, but it's really saying ___." Can't fill it in → reject.
7. AI organizes. Mitchell provides the soul. Never rewrite his words inside a clip. Captions are drafted from his words, he approves.
8. Faith stays natural. If a clip reads as doctrine, tag it, don't lead with it.

## Repo map

```
CLAUDE.md                 ← you are here
config.yaml               ← every knob; change behavior here, not in code
docs/01-PRD.md            ← what and why
docs/02-ARCHITECTURE.md   ← modules, data flow, file layout, schemas
docs/03-DECISIONS.md      ← decisions + the research conflicts to surface
docs/04-BUILD-PLAN.md     ← THE task list. Work top-down. Check boxes as you go.
docs/05-STACK-RESEARCH.md ← repo research; install commands; Windows gotchas
docs/06-COMPOSIO.md       ← exact tool slugs, payloads, pitfalls, account IDs
knowledge/                ← channel OS, hooks bank, niche data. The scorer's brain.
prompts/                  ← LLM prompts (selector, caption). Edit here.
.claude/skills/           ← slash-command skills for operating the pipeline
src/clips/                ← the pipeline package (you build this)
dashboard/                ← the approval UI (you build this)
data/                     ← videos, transcripts, clips, queue (gitignored)
```

## How to work

- Work through `docs/04-BUILD-PLAN.md` in order. Each task has a "done when" line. Don't skip to the fun part.
- Before writing a module, read `docs/02-ARCHITECTURE.md` for its contract.
- Test on ONE video end-to-end before batching. The test video is chosen in Build Plan task 1.4.
- Windows. Use `pathlib`, forward slashes in ffmpeg filter args, `os.add_dll_directory` for CUDA DLLs. See docs/05 gotchas before debugging any CUDA/ffmpeg error.
- Log everything to `data/pipeline.log` and to SQLite `data/clips.db`. The dashboard reads the DB.
- When a step fails, fix the step. Don't rebuild the pipeline.
- When Mitchell asks "what's the status," answer with counts from the DB: videos processed, clips generated, awaiting approval, approved, posted. Then the next verb.

## Definition of done for Milestone 1

- [ ] `python -m clips.run --top 10 --clips-per-video 2` completes on his desktop
- [ ] 20 clips in `data/clips/` with burned captions, 1080x1920, ≤45s, H.264/AAC
- [ ] Dashboard at `http://localhost:8787` shows them with preview, caption editor, approve/reject
- [ ] One approved clip published to Instagram via Composio and its permalink stored in the DB
- [ ] Scheduler posts approved clips at the configured 3 slots/day when the desktop is on

Then: the next verb is "approve the batch."
