# 07: Session state — cloud session 2026-09-05

Read this before doing anything. It carries over the work already done in the Claude Code
cloud session so nothing gets redone. The cloud session moved the project as far as it
could without local file access; this doc is the handoff to the local machine.

## Done (do not redo)

### Video ranking — COMPLETE
All 298 channel uploads ranked by views via Composio YouTube tools (build-plan task 1.2
logic, executed manually). Top-viewed uploads are mostly existing Shorts (<60s) which
cannot be clipped, so the clip queue is the **top 10 by views among videos >= 120s**:
see `data/top10.json`. Test video (1.4): `TEST_VIDEO_ID = kSkn5hlM0Xo`
("i still doom scroll...", 833 views, 6:24).

### Transcripts — method proven
`YOUTUBE_LIST_CAPTION_TRACK` + `YOUTUBE_LOAD_CAPTIONS` (Composio) works for every video —
Mitchell owns the channel, so the auto (asr) track downloads fine as SRT. All 10 were
fetched in the cloud sandbox (lost when the sandbox recycled — refetch is ~1 min, or use
local faster-whisper per build plan 1.5, which is better anyway: word timestamps).

### Clip selection for the test video — COMPLETE (approved criteria applied)
Selected by scoring the transcript against `prompts/clip_selector.md`. Boundaries are
SRT-coarse (+/- 2s); snap to word timestamps at render time (build plan 1.7).

**Clip 1 — "still doom scrolling" (~26s, two segments, jump cut):**
- Segment A 0:00.2–0:13: "I still doom scroll. As somebody who probably has talked about
  not wanting to doom scroll, wanting to chase after their dreams, wanting to blah blah
  blah blah blah."
- Segment B 0:41–0:54: "I think it is so much easier for us to run away from the things
  that we actually want than to actually address them. It is uncomfortable to actually
  chase your dreams. Who would have thought?"
- hook_structure: admission | topic: BECOME
- one_idea: about doom scrolling, really saying avoidance is comfort
- peak: "It is uncomfortable to actually chase your dreams." (ends one beat after)
- caption (CTA slot none): `i still doom scroll. it's so much easier to run away from
  what i actually want than to address it.`

**Clip 2 — "easier to be a hypocrite" (~20s, contiguous 1:41.5–2:01.5):**
- "And then I realized in all the things I'm saying, I'm like, 'Wow, it's so hard to take
  your own advice. It's so much easier to be a hypocrite. I want to chase this. I want to
  have that.' Boom, boom, boom. Nothing's being done. No, nothing is being done."
- hook_structure: admission | topic: BECOME
- one_idea: about giving advice, really saying talk without motion is hypocrisy
- peak: "It's so much easier to be a hypocrite." (clip ends on the "nothing is being
  done" repeat — intentional, it lands)
- caption (CTA slot none): `it's so hard to take your own advice. it's so much easier to
  be a hypocrite.`

### Publishing path — verified live
Composio connections ACTIVE and tested: YouTube (channel UCHCo7ZlH2hCEPlBUqp-Zu-A) and
Instagram (@mitchloveswords, ig_user_id 28108917608702310). `INSTAGRAM_POST_IG_USER_MEDIA`
+ `INSTAGRAM_POST_IG_USER_MEDIA_PUBLISH` slugs confirmed against the live account.

## Learned the hard way (cloud constraints — mostly moot locally)

- yt-dlp from datacenter IPs gets YouTube's "confirm you're not a bot" wall on every
  player client, with or without TLS impersonation. **From a residential/home IP this
  wall normally does not appear** — local yt-dlp fallback (build plan 1.3) should just
  work. If it ever appears locally, `--cookies-from-browser chrome` fixes it (his own
  account, his own machine — fine there, never in a cloud sandbox).
- Mitchell could not find his local original exports; if `paths.videos_local` matches
  nothing, don't stall — yt-dlp the file. YouTube Studio download (Content → ⋮ →
  Download) is the manual fallback, 720p.
- Transcript SRT from the captions API has rolling/overlapping cues — dedup before use
  (each cue repeats the tail of the previous one).

## Next verbs (local session, in order)

1. Phase 0 environment checks (build plan 0.1–0.6). GPU/CUDA per config.yaml; if no
   NVIDIA GPU is found, set `transcription.device: cpu` + `reframe.encoder: libx264`
   and continue — do not stall on hardware.
2. yt-dlp `kSkn5hlM0Xo` (or local file if found), transcribe with word timestamps,
   snap the two clips above to word boundaries, render per build plan 1.8–1.10.
3. Show Mitchell both clips. He says "postable" or says what's wrong. Iterate.
4. Then the batch: remaining 9 videos of `data/top10.json`, 2 clips each -> 20 in the
   dashboard (Phase 2), approve queue, then first real post (Phase 3.4 — ask first).
