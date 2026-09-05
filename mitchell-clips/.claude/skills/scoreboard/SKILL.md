---
name: scoreboard
description: Friday review and status. Use when Mitchell says "status", "how are the clips doing", "friday review", "what's working", or "what should I clone".
---

# scoreboard

Status (any day): counts from the DB: videos processed / clips rendered / awaiting approval / approved (runway in days) / posted this week. One line each. Then the next verb.

Friday review (`python -m clips.scoreboard --weekly`), report in this order and nothing else:
1. Posts shipped this week vs. target (3/day). This is the only metric he fully controls.
2. Outliers: any post ≥ 3× the rolling 30-post median. Name the hook_structure and topic so the win is attributable to a hook, not a vibe.
3. View-to-follower conversion on any post ≥ 10K views (target 1%).
4. Best comment sentence (from INSTAGRAM_GET_IG_MEDIA_COMMENTS on winners). If a comment has a better sentence than the caption, it's next week's clip idea.
5. One decision, by the rules in knowledge/content-os.md §3.6: clone the winner (same hook different topic / same topic different clip), retire a hook used 3× with all < 1× median, or change hooks after 2 flat weeks. Never change topics unless comments say the idea missed.

Do not report raw views as the headline. Do not add metrics he didn't ask for. End with a verb and a deadline.
