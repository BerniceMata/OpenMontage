# 02: Architecture

## Data flow

```
[rank]  Composio YOUTUBE_LIST_CHANNEL_VIDEOS + GET_VIDEO_DETAILS_BATCH
   │      → data/videos.json (id, title, views, duration, published)
   ▼
[ingest]  match local file by title/ID in paths.videos_local; else yt-dlp
   │      → data/videos/<video_id>.mp4
   ▼
[transcribe]  faster-whisper CUDA, word timestamps, VAD
   │      → data/transcripts/<video_id>.json  (segments[], words[])
   ▼
[scenes]  PySceneDetect AdaptiveDetector
   │      → data/transcripts/<video_id>.scenes.json
   ▼
[select]  windows → Ollama (JSON schema) → hard-rule filter → rank → snap to word + scene bounds → end-at-peak
   │      → clips table rows (status=selected)
   ▼
[render]  cut → silence trim → face-track crop 1080x1920 → ASS captions + hook overlay → NVENC
   │      → data/clips/<clip_id>.mp4  (+ .ass, .json sidecar)   status=rendered
   ▼
[caption]  Ollama drafts caption_text from clip transcript in channel style; CTA per rotation
   │      status=awaiting_approval
   ▼
[dashboard]  Mitchell: approve / edit / reject / reschedule    status=approved | rejected
   ▼
[schedule]  next free slot → status=scheduled
   ▼
[publish]  Composio POST_IG_USER_MEDIA (video_file) → POST_IG_USER_MEDIA_PUBLISH → GET_IG_MEDIA permalink
   │      status=posted, ig_media_id, permalink
   ▼
[scoreboard]  insights at +1h/+24h/+7d → outlier multiple vs rolling median → clone candidates
```

## Package layout (`src/clips/`)

| Module | Responsibility | Contract |
|---|---|---|
| `config.py` | Load `config.yaml` into a pydantic model | `cfg = load()` |
| `db.py` | SQLite schema + helpers (`data/clips.db`) | `videos`, `clips`, `posts`, `insights` tables |
| `rank.py` | Pull channel video list + stats via Composio; write `videos` | `rank(top_n) -> list[Video]` |
| `ingest.py` | Locate local original or yt-dlp | `ensure_local(video) -> Path` |
| `transcribe.py` | faster-whisper → words JSON | `transcribe(path) -> Transcript` |
| `scenes.py` | PySceneDetect → cut timestamps | `detect(path) -> list[float]` |
| `select.py` | Windowing, LLM scoring, hard filters, boundary snapping, end-at-peak | `select(video, transcript, scenes) -> list[ClipCandidate]` |
| `llm.py` | Provider abstraction: Ollama (`format=` JSON schema) or Anthropic | `score_window(prompt, schema) -> dict` |
| `render/cut.py` | ffmpeg trim to candidate bounds | |
| `render/silence.py` | trim dead air using word gaps (whisper VAD), keep emotional pauses > threshold only if flagged | |
| `render/reframe.py` | MediaPipe face detect per frame (sample every N frames), One-Euro smoothing, scene resets, crop → 1080x1920 | |
| `render/captions.py` | words → ASS karaoke (pysubs2), hook overlay, burn with `subtitles=` filter, NVENC | |
| `caption_text.py` | Draft caption in channel style; apply CTA rotation | `draft(clip) -> str` |
| `publish.py` | Composio calls, encode validation via ffprobe, retries per pitfalls | `publish(clip) -> Post` |
| `scheduler.py` | Slot assignment, missed-slot tolerance, daemon loop (`python -m clips.scheduler`) | |
| `scoreboard.py` | Insights pulls, outlier math, clone suggestions | |
| `run.py` | CLI: `python -m clips.run --top 10 --clips-per-video 2 [--video ID] [--stage select]` | |

`dashboard/` is a FastAPI app + one HTML page (htmx or vanilla JS). Serves `data/clips/` statically. No build step.

## SQLite schema (minimum)

```sql
videos(id TEXT PK, title, views INT, duration_s REAL, published_at, local_path, status)
clips(id TEXT PK, video_id FK, start_s REAL, end_s REAL, hook_line TEXT, hook_structure TEXT,
      topic TEXT, score REAL, score_json TEXT, transcript_text TEXT,
      file_path TEXT, ass_path TEXT, caption_text TEXT, cta_type TEXT,
      status TEXT,  -- selected|rendered|awaiting_approval|approved|rejected|scheduled|posted|failed
      scheduled_at, created_at, updated_at, reject_reason TEXT)
posts(id INTEGER PK, clip_id FK, creation_id TEXT, ig_media_id TEXT, permalink TEXT, posted_at, error TEXT)
insights(id INTEGER PK, clip_id FK, pulled_at, hours_after INT, views INT, reach INT, saved INT,
         shares INT, comments INT, likes INT, avg_watch_time REAL, raw_json TEXT)
```

## Selector output schema (what the LLM must return per window)

```json
{
  "candidates": [
    {
      "start_word_idx": 0, "end_word_idx": 0,
      "hook_line": "verbatim first sentence",
      "hook_structure": "admission|contrarian|declarative|mirror|story",
      "topic": "WORDS|SEEN|PERSP|BECOME|FAITH|HUMAN",
      "one_idea": "This clip is about X, but it's really saying Y",
      "peak_word_idx": 0,
      "scores": {"hook": 0, "tension": 0, "concrete": 0, "standalone": 0, "payoff": 0},
      "reject_reasons": []
    }
  ]
}
```

Composite = 0.35 hook + 0.2 tension + 0.15 concrete + 0.15 standalone + 0.15 payoff. Hard rejects override score: opens with "you"; abstract-noun opener; no fillable one_idea; length outside bounds after snapping.

## Windows specifics

- CUDA DLLs: `import torch` before `faster_whisper`, or `os.add_dll_directory(site-packages/nvidia/cudnn/bin)` and `.../cublas/bin`.
- ffmpeg subtitles filter: `subtitles='C\:/clips/data/clips/x.ass':fontsdir='C\:/clips/assets/fonts'` or run with cwd set and pass relative paths.
- Keep project at `C:\clips`. No spaces in path.
- Scheduler runs as a Task Scheduler job at logon: `pythonw -m clips.scheduler`.
