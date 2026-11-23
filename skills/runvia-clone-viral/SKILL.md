---
name: runvia-clone-viral
description: >-
  Clones the structure of a viral short video for the user's own product with Runvia: finds the
  top TikToks and Reels about the product or niche, reads their transcripts and comments, maps
  the beat-by-beat structure (hook, turn, proof, call to action), writes a new script on the same
  structure, and renders it with Seedance or Wan plus an ElevenLabs voice. Triggers: "clonar
  vídeo viral", "copiar a estrutura deste vídeo", "refazer este TikTok para meu produto",
  "vídeo igual a este", "modelar o viral".
license: Apache-2.0
metadata:
  cli-version: 0.3.0
---

# Clone a viral video's structure with Runvia

Read the `runvia` skill first. This skill copies a video's structure and pacing, never its
footage, voice, people or words: the new video uses the user's product, a new script, a
generated presenter and a stock or designed voice.

## 1. Pick the source

If the user gave a link, use it. Otherwise:

1. `scrapecreators.tiktok.search.keyword` (`query` = product or niche, `region`,
   `date_posted` this week or month, `sort_by` most liked) and
   `scrapecreators.instagram.reels.search`. About 14 credits.
2. Rank by views over the author's follower count, so a small account's breakout beats a big
   account's average post. Show the top 5 with links and let the user pick.

## 2. Take it apart (about 14 credits)

1. Transcript: `scrapecreators.tiktok.video.transcript` or
   `scrapecreators.instagram.media.transcript` (`url`).
2. Comments: `scrapecreators.tiktok.video.comments` or `scrapecreators.instagram.post.comments`;
   note what viewers repeat, ask or object to.
3. Write the beat sheet: for each beat its time range, what is said, what is shown, and its job
   (hook, problem, turn, proof, offer, call to action). Mark the hook's pattern.

## 3. Rewrite

Write the user's version beat by beat on the same timing, with their product, their claims and
their call to action. Show the source beat sheet and the new script side by side, with the
render price, and wait for approval.

## 4. Render

- One clip per beat or one continuous clip, on `bytedance.video.seedance-2-5` (about 650 cr/s at
  480p, 1,450 cr/s at 720p) or `alibaba.video.wan-3-0` (about 220 cr/s at 480P). Match the source's
  aspect ratio (usually `9:16`).
- Narration on `elevenlabs.voice.tts`, one call per script.
- Render the hook beat first at 480p and show it; render the rest after approval, each with
  its own `--idempotency-key`.
- Download every file with `runvia files download <runId> -o ./clone`.

## 5. Deliver

The source link and beat sheet, the new script, the files, the settings, and a cost table
(research, each render, voice, total). If the user wants variants (another presenter, another
hook), reuse the research and rerender only the changed beats.
