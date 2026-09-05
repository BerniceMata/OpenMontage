---
name: caption-writer
description: Draft or fix the Instagram caption for a clip in the channel's caption style (lowercase, first person, one carried sentence, no hashtags). Use when Mitchell says "caption this", "the caption is off", or "rewrite the caption for clip X".
---

# caption-writer

1. Load the clip's transcript_text, hook_line, peak line, one_idea, and cta_type from the DB.
2. Apply `prompts/caption_writer.md` rules exactly. The caption must come from his words in the clip.
3. Offer ONE caption, not five. If he asks for options, give three max.
4. Check before returning: lowercase; no hashtags; no emojis; no em dash; doesn't open with "you"; ≤ 300 chars; CTA line matches cta_type.
5. Write it to the DB (`clips.caption_text`) only after he says yes, or if he asked you to "just fix it."
6. End with a verb: "Approve it?"
