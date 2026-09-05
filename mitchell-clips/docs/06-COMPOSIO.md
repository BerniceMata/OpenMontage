# 06: Composio integration (verified 2026-09-04 against Mitchell's account)

## Connected accounts (already active)

| Toolkit | Account | IDs |
|---|---|---|
| instagram | `@mitchloveswords` | ig_user_id `28108917608702310` (Business/Creator, linked to FB Page) |
| youtube | "Mitchell Nicholas" | channel_id `UCHCo7ZlH2hCEPlBUqp-Zu-A`, handle `@mitchellnicholas2099` |

Other connected toolkits on his Composio (available if useful): airtable, github, gmail, googlecalendar, googledocs, googledrive, googlesheets, linkedin, slack.

## Two ways to call Composio from this repo

1. **Python SDK (for the scheduler / pipeline, unattended):**
   ```python
   from composio import Composio
   c = Composio(api_key=os.environ["COMPOSIO_API_KEY"])
   res = c.tools.execute("INSTAGRAM_POST_IG_USER_MEDIA", user_id="default", arguments={...})
   ```
   Check the current SDK docs for the exact `execute` signature and how connected accounts are addressed (`user_id` / `connected_account_id`); the API changed across major versions. Wrap it once in `src/clips/composio_client.py` so nothing else touches it.
2. **Composio MCP in Claude Code (for interactive operating):** `claude mcp add composio -- npx -y @composio/mcp@latest ...` (follow current Composio MCP install docs). Then the `.claude/skills/publish-reel` skill can call tools directly.

## Publishing a Reel (exact tool slugs)

Step 0 (daily): `INSTAGRAM_GET_IG_USER_CONTENT_PUBLISHING_LIMIT` → log quota. Nested list structure; parse defensively.

Step 1: `INSTAGRAM_POST_IG_USER_MEDIA`
```json
{
  "ig_user_id": "28108917608702310",
  "media_type": "REELS",
  "video_file": {"name": "<clip_id>.mp4", "mimetype": "video/mp4", "s3key": "<from Composio file upload>"},
  "caption": "<caption_text>",
  "share_to_feed": true,
  "thumb_offset": 0
}
```
`video_file` is marked `file_uploadable`: the SDK uploads the local file to a temporary public URL that Meta fetches. **No R2/S3 needed.** If the SDK path for file upload is awkward, fallback is `video_url` pointing at a Cloudflare R2 presigned URL (free tier, zero egress). Avoid `video_url` with query strings on some accounts (pitfall: error code 100 / subcode 2207076).

Response `data.id` = `creation_id` (container). Containers are single-use.

Step 2: `INSTAGRAM_POST_IG_USER_MEDIA_PUBLISH`
```json
{"ig_user_id": "28108917608702310", "creation_id": "<creation_id>", "max_wait_seconds": 180, "poll_interval_seconds": 5}
```
It polls until FINISHED then publishes. Response `data.id` = `ig_media_id` (NOT the creation_id; don't mix them, error 9007).

Step 3: `INSTAGRAM_GET_IG_MEDIA` with `ig_media_id` → `permalink`. Store both IDs + permalink in `posts`.

Fallback: on 400 with container ERROR, or 409 duplicate publish → create a NEW container (step 1) and retry step 2 once. Then mark failed and surface in dashboard.

Auth: 401 / OAuthException code 190 → token expired. Dashboard banner: "Reconnect Instagram in Composio." Do not retry in a loop.

## Video spec Meta accepts (pre-validate with ffprobe before upload)

MP4/MOV · H.264 (or HEVC) · AAC ≤48kHz · 23-60 fps (use 30) · 3s-15min · ≤1080 wide · 9:16 · ≤25 Mbps video / 128 kbps audio · ≤300 MB. Encode with `-pix_fmt yuv420p -movflags +faststart`.

## Ranking videos (YouTube)

- `YOUTUBE_LIST_CHANNEL_VIDEOS` `{channelId: "UCHCo7ZlH2hCEPlBUqp-Zu-A", maxResults: 50, pageToken}` → paginate. videoId is at `items[].snippet.resourceId.videoId`.
- `YOUTUBE_GET_VIDEO_DETAILS_BATCH` `{id: [...], parts: ["snippet","statistics","contentDetails"]}` → chunks of 50 merged automatically. `viewCount` is a string; `contentDetails.duration` is ISO-8601 (`PT12M34S`).

## Insights (scoreboard)

- `INSTAGRAM_GET_IG_USER_MEDIA` `{ig_user_id: "me", limit: 100, after}` → rows under `data.data`, paging under `data.paging.cursors.after`. Reels: `media_type=VIDEO` + `media_product_type=REELS`.
- `INSTAGRAM_GET_IG_MEDIA_INSIGHTS` `{ig_media_id, metric: ["views","reach","saved","shares","comments","likes","total_interactions","ig_reels_avg_watch_time"]}` → list; map by `name`, value at `values[0].value` or `total_value.value`. **Requires ≥1,000 followers.** Below that, use the counters on `INSTAGRAM_GET_IG_MEDIA` (`view_count, like_count, comments_count, saved_count, shares_count`).
- `INSTAGRAM_GET_IG_MEDIA_COMMENTS` for comment mining on winners.
- Deprecated metric names (`plays`, `impressions`, `video_views`) are rejected; use `views`.
