---
name: post
description: Take a finished file (a reel, a video, a photo, carousel images) or an idea to a post in the member's Fresh2Wow workspace — caption in the brand's voice, upload, a draft with each network's options, then review or scheduling. Use when the user says "post this to Instagram / TikTok / YouTube / LinkedIn / Facebook / X / Threads with Fresh2Wow", "upload this reel to Fresh2Wow", "leave it as a draft", "send it to review", "schedule it for Thursday at 10", "publica esto", "súbelo a Fresh2Wow", "déjalo en borradores", "prográmalo para el jueves".
argument-hint: "[file or idea] [brand] [networks] [when]"
---

# Post through Fresh2Wow

Fresh2Wow holds the member's brands, connected social accounts, approvals, calendar and metrics. This skill takes one piece of content from the conversation to a post there, using the Fresh2Wow connector's tools (their names below are the bare tool names; the connector prefixes them).

Answer in the member's language. Show every time in the workspace's zone: each time the tools return has a `_local` twin, use that one.

## 0. Know the connection

Call `get_workspace` once. It gives the workspace name, the plan, the time zone and the connection **level**:

- `read` — reading only. Uploads and drafts are refused. Say so and stop: a workspace admin grants *Borradores* (drafts) on Fresh2Wow's permission page when the member connects again.
- `draft` — uploads, drafts, sending to review. Nothing publishes from here; a member approves and schedules in the app.
- `schedule` — everything above, and `schedule_post` after the member's explicit yes.

Never promise a step the level does not allow.

## 1. The brand and its voice

`list_brands`, then `get_brand_context` for the brand the member named (ask when there are several and none was named). Read the voice profile (audience, tone, do and don't lists, banned words, sample posts, language) and the strategy and knowledge notes. They are the brand's rules: a caption that breaks them is wrong even if it reads well.

## 2. The destinations

`list_social_accounts` with the `brand_id`. Use only destinations that are active and whose account is connected; a brand can carry an old disconnected row for the same handle, skip it. For each destination note:

- `caption_max_chars` (and `caption_rule` when the network does not count plain characters),
- the `formats` it takes, and whether it takes `trial_reel`, `collaborators`, a `cover`,
- for TikTok, the account's own facts: the privacy levels it allows and whether comments, duet and stitch are permitted.

If the member named networks, match them to destinations by name and confirm any that are missing instead of guessing.

## 3. The file

Skip this section when the post has no media.

1. Measure the file locally: exact size in bytes, content type, and for a video or an image its width and height (and duration for a video), plus a sha256 when you can compute one. `ffprobe` and `sha256sum` do it.
2. `create_media_upload` with `file_name`, `mime_type`, `file_size`, optional `title` and lowercase `tags`. Accepted: `image/jpeg`, `image/png`, `image/webp` up to 30 MB; `video/mp4`, `video/quicktime`, `video/webm` up to 1 GB.
3. Send the bytes with **one HTTP PUT** to `upload_url`, with the `content-type` header exactly as declared and **no Authorization header**. The answer carries a `curl_example`. The URL lasts two hours.
4. `complete_media_upload` with the `media_id`, and the `sha256`, `width`, `height`, `duration_seconds` you measured. A size different from the one declared removes the upload: start again from step 2.

A cover for an Instagram reel or a YouTube video is a second upload: a JPEG (1080 × 1920 for a reel), passed later as `cover_media_id`.

A JPEG under 8 MB with its short side at most 1080 px publishes to every network as it is.

## 4. The caption

If the member gave the text, keep it. Otherwise write it from their idea in the brand's voice: the hook first, one call to action last, the brand's own rules on emoji, hashtags and banned words.

- `base_copy` is what every destination gets.
- A destination whose `caption_max_chars` is smaller than the base text gets its own `copy_override` that keeps the idea and the call to action within the limit (X and Threads usually need one).
- On YouTube the first line becomes the title (100 characters at most): make the first line work as one.
- A caption over a destination's limit is refused with the count. Shorten that destination's override and call again with the same `idempotency_key`.

## 5. The options, per destination

Only what `list_social_accounts` says the destination takes; anything else is refused by name.

- Instagram and Facebook: `format` — `feed`, `reel` or `story`. A 9:16 video goes as `reel`.
- Instagram reel: `trial_reel` (`everyone` = a member graduates it in the app, `auto_if_performs` = Instagram does), `collaborators` (up to 3 usernames, lowercase, no @, never on a story), `cover_media_id`.
- YouTube video: `cover_media_id`.
- TikTok: `tiktok.privacy` (one of the levels the account allows; **required before the post can be scheduled**), `allow_comments`, `allow_duet`, `allow_stitch`, and `disclosure`: `on` + `your_brand` when the post promotes the member's own brand; `branded_content` only for a paid partnership with a third party. Ask when it is not obvious which one applies.
- Music on a reel is picked in the app.

## 6. The draft

`create_post_draft` with a fresh `idempotency_key` (for example `reel-2026-11-06-morning-stretch`), the `brand_id`, `base_copy`, the `targets` (each with its `social_target_id`, `copy_override` and `options` as needed), the `media_ids` in the order they attach, optional `title` and `tags`, and `scheduled_at` when the member gave a time.

Write `scheduled_at` as ISO 8601 **with the workspace zone's offset for that date** (daylight saving changes it), then check the `_local` twin in the answer says the hour the member asked for.

The same key answers the same draft again, so reuse it only to retry this very draft.

## 7. The last step — the member's choice

Tell the member what exists and where, then do only what they asked:

- **Leave it as a draft.** It is under *Publicaciones → Borradores* in the app, marked «vía agente IA», with the proposed time pre-filled. Done.
- **Send it to review.** `submit_for_review`; the answer names who can approve. A reviewer who sends it back gives a reason that `get_post` shows.
- **Schedule or publish it** (level `schedule` only). First show the member the post's title, each destination by name and the local time (or "now"), and wait for a clear yes in the conversation. Then `schedule_post` with `confirm: true` and a fresh `idempotency_key`; no time uses the draft's proposed one, `publish_now: true` publishes at once. A request that already named the post, the networks and the time counts as that yes; anything vaguer does not.

After scheduling, `get_post` and read back the destinations and the local time. A scheduled post is changed or cancelled in the app (*Publicaciones → Programadas*).

## When a tool refuses

The answer says why and what to do next; follow it rather than retrying blind.

- `VALIDATION` — a caption over a limit, an option the destination does not take, a size mismatch. Fix that one thing; keep the same `idempotency_key` for a draft retry.
- `PERMISSION` naming a level — the connection's level is too low (section 0).
- `PLAN_LIMIT` — the workspace's plan does not include that feature; tell the member, do not work around it.
- `RATE_LIMITED` — wait the seconds it gives.
- `INSUFFICIENT_CREDITS` — the member adds credits in the app.

## If the member asks you to make the piece first

The edit is yours and runs on the member's machine with their own tools; Fresh2Wow receives the result. `get_brand_context` gives the kit: palette, fonts, and the logo, font and template files as download links valid for 10 minutes. `list_video_ideas` and `get_video_script` give a script written in the app when the piece comes from one. Render vertical video at 1080 × 1920 and keep text clear of the top and bottom edges, where each network draws its own interface. Then continue from section 3.

## What this skill does not do

It does not connect or disconnect social accounts, edit a post that already exists, pick music, or generate images or video inside Fresh2Wow: those live in the app.
