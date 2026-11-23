---
name: runvia-video-ugc
description: >-
  Makes short UGC-style video ads with Runvia: studies the hooks that work in the product's
  niche on TikTok and Instagram, writes a script, generates talking-head or product clips with
  Seedance or Wan, narrates with an ElevenLabs voice, and delivers the files with the cost of
  each step. Triggers: "vídeo UGC", "faça um criativo", "anúncio em vídeo", "gere ganchos",
  "vídeo com Seedance", "narração com ElevenLabs", "UGC para meu produto".
license: Apache-2.0
metadata:
  cli-version: 0.3.0
---

# UGC video with Runvia

Read the `runvia` skill first. Video is the most expensive thing Runvia sells: a 5-second 720p
Seedance clip costs about 7,300 credits. Never generate without the user's go-ahead
on the script and the price.

## 1. Brief

Ask in one message: the product (name, what it does, price, a photo if they have one), the
audience, the language and country, the vertical format (9:16 by default), how many hooks to
test (3 by default), and the budget. Ask for a product photo when the product must look exact.

## 2. Research the hooks (about 85 credits)

1. `scrapecreators.tiktok.search.keyword` with the niche keyword and `region`, and
   `scrapecreators.instagram.reels.search` with the same idea. One page each.
2. Keep the 10 videos with the most views relative to their author's following.
3. Pull transcripts for those (`scrapecreators.tiktok.video.transcript`,
   `scrapecreators.instagram.media.transcript`).
4. Extract the first sentence of each transcript and classify the hook pattern (question,
   bold claim, before and after, "POV", problem call-out, demonstration). Show the user the
   patterns with links and view counts.

## 3. Script

Write one script per hook: a 1 to 2 second hook line, 2 to 3 lines of body, a call to action.
Keep each script under 25 seconds of speech (about 60 words). Show the scripts with the
generation price and wait for approval.

## 4. Generate

| Piece | Tool | Notes |
|---|---|---|
| Presenter or product clip | `bytedance.video.seedance-2-5` | `content`: a `{"type":"text","text":"..."}` item, plus an image item for image-to-video; `duration` 4 to 30 s; `resolution` 480p, 720p or 1080p; `ratio` 9:16 (use `adaptive` with a first or last frame) |
| Cheaper clip | `alibaba.video.wan-3-0` | `prompt`; `duration` 2 to 30 s; `resolution` 480P, 720P, 1080P |
| Presenter still / product shot | `alibaba.image.wan-2-7` | `prompt`, optional reference `images` |
| Narration | `elevenlabs.voice.tts` (`voice_id`, `text`), `elevenlabs.voice.tts-flash` for drafts | priced per character |

Steps:

1. Upload any product photo with `runvia files upload` and pass the URL in the clip input.
2. Generate one hook clip at 480p first (about 3,200 credits for 5 s on Seedance). Show it.
3. After approval, generate the rest with one `--idempotency-key` per clip
   (`ugc-<product>-<hook>-<n>`) so a retry never pays twice.
4. Narrate each script; check the audio length matches the clip.
5. Download every file at once: `runvia files download <runId> -o ./ugc`.

Write the prompt for realism: handheld phone framing, natural light, an ordinary room, one
person speaking to the camera, no on-screen text from the model. Keep the product described the
same way in every clip.

## 5. Assemble and deliver

If ffmpeg is available, lay the narration over the clip and burn simple captions from the
script. Deliver the files, the scripts, the settings used and a cost table (research,
each generation, voice, total).
