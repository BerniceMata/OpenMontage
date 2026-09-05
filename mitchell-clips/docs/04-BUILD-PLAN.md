# 04: Build plan (the only plan)

Work top-down. Check boxes as you complete them. Each task has a "done when." Do not start Phase N+1 until Phase N's done-whens are true on Mitchell's machine.

Session opener for Claude Code: "Read CLAUDE.md. Continue from the first unchecked task in docs/04-BUILD-PLAN.md. Report counts from the DB when done."

## Phase 0: Environment (Day 1, ~1 hr)

- [ ] 0.1 Confirm machine: `nvidia-smi` (driver + VRAM), `python --version` (3.11.x), project at `C:\clips`.
      Done when: VRAM known, `llm.ollama_model` set accordingly in `config.yaml`.
- [ ] 0.2 Install ffmpeg (gyan.dev full build via `winget install Gyan.FFmpeg`), verify `ffmpeg -filters | findstr subtitles` and `ffmpeg -encoders | findstr nvenc`.
- [ ] 0.3 Create venv, install `requirements.txt` in this order: torch cu128 wheel first, then faster-whisper + nvidia cudnn/cublas, then the rest. See docs/05 install block.
- [ ] 0.4 Install Ollama (`OllamaSetup.exe`), `ollama pull <model>`, verify `ollama ps` shows 100% GPU after a test prompt.
- [ ] 0.5 Composio: `pip install composio` and set `COMPOSIO_API_KEY` in `.env`. Verify with a script that calls `INSTAGRAM_GET_USER_INFO` (ig_user_id "me") and prints username `mitchloveswords`. Also add Composio MCP to Claude Code (`claude mcp add`) so the operator skills work interactively. See docs/06.
- [ ] 0.6 Set `paths.videos_local`. Run `python -m clips.ingest --check` (build this first, trivial) to list how many of the channel's videos match a local file by title.
      Done when: a match count prints. Anything unmatched falls back to yt-dlp.

## Phase 1: One video, end to end (Day 1-2)

- [ ] 1.1 `config.py`, `db.py` with schema from docs/02. `python -m clips.db init` creates `data/clips.db`.
- [ ] 1.2 `rank.py`: Composio `YOUTUBE_LIST_CHANNEL_VIDEOS` (channelId from config, paginate) + `YOUTUBE_GET_VIDEO_DETAILS_BATCH` (statistics, contentDetails). Parse ISO-8601 duration. Write `videos` table. Print top 10 by views.
- [ ] 1.3 `ingest.py`: fuzzy title match against local folder (normalize punctuation/case), else yt-dlp best mp4. Store `local_path`.
- [ ] 1.4 Pick the test video: the highest-view video that has a local file. Record its ID here: `TEST_VIDEO_ID = ________`.
- [ ] 1.5 `transcribe.py`: faster-whisper, word timestamps, VAD. Save JSON. Done when: `data/transcripts/<id>.json` has `words[]` with start/end and the text reads correctly.
- [ ] 1.6 `scenes.py`: PySceneDetect AdaptiveDetector → cut list. Cache.
- [ ] 1.7 `llm.py` + `select.py`: window the transcript (90s / 15s overlap), send each window with `prompts/clip_selector.md` and the JSON schema from docs/02 to Ollama with `format=schema`. Apply hard rejects. Snap bounds to word boundaries, never cross a scene cut mid-sentence. Apply end-at-peak (trim after `peak_word_idx` + 1 sentence). Keep top `clips_per_video`. Write `clips` rows (status=selected).
      Done when: 2 candidates for the test video print with hook_line, one_idea, score, and duration within bounds. Read them. If they'd embarrass Mitchell, fix the prompt before moving on.
- [ ] 1.8 `render/cut.py` + `render/silence.py`: trim to bounds, remove gaps > `silence_min_ms` where no word is spoken, except pauses immediately before a `peak` line (keep those).
- [ ] 1.9 `render/reframe.py`: sample every 3rd frame, MediaPipe face center, One-Euro smoothing, reset at scene cuts, crop to 9:16, upscale to 1080x1920, NVENC. Fallback to center crop if no face in >30% of samples.
- [ ] 1.10 `render/captions.py`: pysubs2 ASS, 2-3 words per line, current-word highlight, classic font, subtle shadow, lower-third; hook overlay (≤4 words, first 2s, top third). Burn with `subtitles=` (see Windows path escaping in docs/02). Verify output with ffprobe: 1080x1920, h264, aac, 30fps, ≤45s.
- [ ] 1.11 `caption_text.py`: `prompts/caption_writer.md` → lowercase first-person one-liner from the clip's own words; append CTA per rotation slot. Status → awaiting_approval.
- [ ] 1.12 `run.py --video TEST_VIDEO_ID` runs 1.3→1.11 and prints the two clip paths.
      Done when: Mitchell watches both clips and says they're postable or says what's wrong. Fix, re-run. Do not proceed to Phase 2 until he says "postable."

## Phase 2: Dashboard (Day 2-3)

- [ ] 2.1 FastAPI app in `dashboard/`. Routes: `GET /` (queue), `GET /clips/{id}` (video file), `POST /clips/{id}/approve|reject|caption|schedule`, `GET /status` (counts JSON).
- [ ] 2.2 Page: grid of awaiting_approval clips; each card = video preview (autoplay muted, tap for sound), hook line, one_idea, score, editable caption textarea, Approve / Reject (with reason dropdown: weak hook, rambles, bad crop, caption wrong, not me) / Reschedule. Keyboard: A approve, R reject, → next.
- [ ] 2.3 Tabs: Awaiting, Approved (with next slot), Posted (with permalink + first insights), Rejected. Header shows runway in days = approved / posts_per_day.
- [ ] 2.4 `python -m clips.dashboard` starts it on :8787 and opens the browser.
      Done when: Mitchell approves one clip from the browser and the DB row flips to approved.

## Phase 3: Publish + schedule (Day 3)

- [ ] 3.1 `publish.py`: pre-validate with ffprobe (spec in docs/06). Call `INSTAGRAM_POST_IG_USER_MEDIA` with `ig_user_id`, `video_file` (Composio uploads it), `caption`, `media_type: REELS`, `share_to_feed`. Then `INSTAGRAM_POST_IG_USER_MEDIA_PUBLISH` with `max_wait_seconds` 180, `poll_interval_seconds` 5. On 400/9007/ERROR container: create a NEW container and retry once. Store creation_id, ig_media_id; fetch permalink via `INSTAGRAM_GET_IG_MEDIA`. Status → posted.
- [ ] 3.2 `scheduler.py`: assign approved clips to next free slots (config `slots_local`, tz America/Chicago). Loop every 60s. If a slot was missed (desktop off), post at the next slot; never post two in one slot; never exceed `posts_per_day`. Check `INSTAGRAM_GET_IG_USER_CONTENT_PUBLISHING_LIMIT` once per day and log it.
- [ ] 3.3 Register a Windows Task Scheduler job at logon: `pythonw -m clips.scheduler` with cwd `C:\clips`. Document the exact `schtasks` command in README.
- [ ] 3.4 **Ask Mitchell before the first real publish.** Then publish one approved clip. Store permalink.
      Done when: the Reel is live on @mitchloveswords and the dashboard shows its permalink.

## Phase 4: Milestone 1 batch (Day 3-4)

- [ ] 4.1 `python -m clips.run --top 10 --clips-per-video 2`. Expect 20 clips awaiting approval.
- [ ] 4.2 Report: "20 clips ready. Open the dashboard. Approve by tonight."
- [ ] 4.3 Scheduler picks up approvals and posts 3/day.
      Done when: Milestone 1 checklist in CLAUDE.md is all true.

## Phase 5: Scoreboard (Week 2)

- [ ] 5.1 `scoreboard.py`: for posted clips, pull `INSTAGRAM_GET_IG_MEDIA_INSIGHTS` (metrics: views, reach, saved, shares, comments, likes, ig_reels_avg_watch_time) at +1h, +24h, +7d. Note: insights require ≥1,000 followers; until then fall back to `INSTAGRAM_GET_IG_MEDIA` counters (view_count, like_count, comments_count, saved_count, shares_count).
- [ ] 5.2 Outlier multiple = views(24h) / rolling median of last 30 posts. Flag ≥ 3×. Dashboard "Winners" tab lists them with hook_structure + topic so wins are attributable.
- [ ] 5.3 Clone suggestions: for each winner, list other candidates from the same video / same hook_structure not yet rendered. One click renders them.
- [ ] 5.4 Friday report: `python -m clips.scoreboard --weekly` prints posts shipped, outliers, view-to-follower %, top comment sentences (via `INSTAGRAM_GET_IG_MEDIA_COMMENTS`), one recommendation. Ends with a verb.

## Phase 6: Backlog (Week 2-3)

- [ ] 6.1 `run.py --all` processes remaining videos overnight in batches of 20, resumable (skip videos with status=done).
- [ ] 6.2 Runway alert: if approved count / posts_per_day < 2 days, print a loud line in the dashboard header and in the scheduler log.

## Not in plan (log ideas in Mitchell's Ideas Inbox, not here)

TikTok/Shorts cross-post, theme accounts, comment-keyword funnels, B-roll VO format, Claude API upgrade (config flip when he asks).
