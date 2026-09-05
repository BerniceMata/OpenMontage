# mitchell-clips

YouTube long-form → high-retention 9:16 clips → approval dashboard → Instagram Reels (3/day) via Composio. Runs locally on a Windows desktop with an NVIDIA GPU. Zero API spend by default.

## Open in Claude Code (this is the whole setup)

1. Unzip to `C:\clips` (no spaces in the path).
2. Copy `.env.example` → `.env`, fill `COMPOSIO_API_KEY` (dashboard.composio.dev → Settings → API keys).
3. Edit `config.yaml` → `paths.videos_local` (folder with the original video files).
4. Open a terminal in `C:\clips`, run `claude`, and paste:

   > Read CLAUDE.md. Continue from the first unchecked task in docs/04-BUILD-PLAN.md. Report DB counts when done.

That's it. Claude Code installs the stack (Phase 0), builds the pipeline on one video (Phase 1), then the dashboard, then publishing. It will stop and ask before the first real post.

## Day-to-day (after Milestone 1)

- `python -m clips.dashboard` → approve clips at http://localhost:8787
- Scheduler runs at logon and posts approved clips at 08:30 / 13:00 / 19:30 CT.
- In Claude Code: `/scoreboard` on Fridays, `/clip-selector` to re-pick from a video, `/caption-writer` to fix a caption, `/publish-reel` to post one now.

## What's in here

See the repo map in `CLAUDE.md`. Knowledge files in `knowledge/` are the channel's operating system and research; the selector and caption prompts are built from them.
