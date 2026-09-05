---
name: publish-reel
description: Publish one approved clip to Instagram now via Composio, or check what's scheduled. Use when Mitchell says "post it", "post clip X now", "what's going out today", or "why didn't it post".
---

# publish-reel

Guardrails: only clips with status=approved or scheduled can be published. Never publish awaiting_approval or rejected. Always confirm with Mitchell before an immediate publish: show the clip path, caption, and ask "Post this now?" Wait for yes.

Publish now:
1. Pre-validate the file with ffprobe against the spec in docs/06.
2. `INSTAGRAM_GET_IG_USER_CONTENT_PUBLISHING_LIMIT` (ig_user_id from config). If near quota, stop and say so.
3. `INSTAGRAM_POST_IG_USER_MEDIA` with video_file, caption, media_type REELS, share_to_feed. Capture creation_id.
4. `INSTAGRAM_POST_IG_USER_MEDIA_PUBLISH` with max_wait_seconds 180, poll 5. Capture ig_media_id.
5. `INSTAGRAM_GET_IG_MEDIA` for permalink. Write posts row, set clip status=posted.
6. Reply with the permalink and the next scheduled slot.

Diagnose "why didn't it post":
- Check scheduler log `data/pipeline.log` for the slot time.
- Common causes in order: desktop was asleep (missed slot, will post next slot); container ERROR (re-create container); code 190 (reconnect Instagram in Composio); ffprobe validation failed (re-encode).
- Fix the cause, don't rebuild the scheduler.

End with a verb.
