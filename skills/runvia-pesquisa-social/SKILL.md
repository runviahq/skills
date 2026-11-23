---
name: runvia-pesquisa-social
description: >-
  Social research into a table with Runvia: find creators in a niche, pull profiles and recent
  posts, rank posts against each creator's own baseline, mine comments for objections and
  questions, read transcripts, and track brands and competitors on TikTok, Instagram, YouTube,
  X, Reddit, Threads, LinkedIn and Facebook. Output is a CSV with the cost per step. Triggers:
  "pesquisa social", "ache criadores de", "o que estão falando de", "puxe os comentários",
  "concorrentes no Instagram", "influenciadores do nicho", "social listening".
license: Apache-2.0
metadata:
  cli-version: 0.3.1
---

# Social research with Runvia

Read the `runvia` skill first for login, prices, `--max-cost` and idempotency keys. This skill
turns a research question into a short series of cheap list calls, a filter, and a few detail
calls on the rows that survive.

## Before the first call

Ask the user, in one message, only what you cannot infer: the niche or brand, the platforms,
the country (`region` BR by default), how many rows they want, and their budget in credits or
reais. Then show the plan with the price of each step and the total, and wait for a yes.

## Tools that do the work

ScrapeCreators rows cost about 7 credits per call; most TikHub rows cost about 7 to 50 credits per call, a few up to 310 credits.
Run `runvia discover "<platform> <job>"` when you need something not listed here.

| Job | Tool |
|---|---|
| TikTok videos by keyword | `scrapecreators.tiktok.search.keyword` (`query`, `region`, `date_posted`, `sort_by`) |
| TikTok creators by keyword | `scrapecreators.tiktok.search.users` |
| TikTok trending feed | `scrapecreators.tiktok.get-trending-feed` (`region`) |
| TikTok profile / recent videos | `scrapecreators.tiktok.profile` (`handle`), `scrapecreators.tiktok.profile.videos` (`handle`, `sort_by`) |
| TikTok comments / transcript | `scrapecreators.tiktok.video.comments`, `scrapecreators.tiktok.video.transcript` (`url`) |
| Instagram profile | `scrapecreators.instagram.profile` (`handle`) |
| Instagram reels by keyword / trending | `scrapecreators.instagram.reels.search` (`query`), `scrapecreators.instagram.reels.trending` |
| Instagram comments / transcript | `scrapecreators.instagram.post.comments`, `scrapecreators.instagram.media.transcript` |
| YouTube search / channel / transcript / comments | `scrapecreators.youtube.search`, `scrapecreators.youtube.channel`, `scrapecreators.youtube.video.transcript`, `scrapecreators.youtube.video.comments` |
| X, Reddit, Threads search | `scrapecreators.x.search.posts`, `scrapecreators.reddit.search`, `scrapecreators.threads.search` |
| LinkedIn profile / posts | `scrapecreators.linkedin.profile` (`url`), `scrapecreators.linkedin.search.posts` |
| Bulk scrapes billed per row | `apify.instagram.scraper`, `apify.tiktok.scraper`, `apify.youtube.scraper` (set `limit`) |

## Recipes

**Creators in a niche.** Search videos by keyword (one call per page) and collect the authors.
Pull each author's profile and recent videos. For each creator compute the median views of
their recent videos and the multiplier of their best video over that median. Return handle,
followers, median views, best video, multiplier and link, sorted by multiplier. Small creators
with a high multiplier are usually what the user wants; say so when follower count and reach
disagree.

**What people say about a brand.** Search TikTok, Instagram reels, YouTube and Reddit for the
brand name (one call each). Keep the top posts by engagement, pull comments on the top 3 to 5
only, and group the comments into questions, complaints and praise, with counts and a link per
group. Do not name ordinary commenters in the summary.

**Competitor feed.** Resolve each competitor's handle with a profile call, pull recent posts,
and compare posting cadence, average views and engagement per post. Flag posts far above the
account's own median.

**Read the video, not just the caption.** For the posts that matter, pull the transcript. Note
when a transcript comes back empty instead of describing a video you did not read.

## Output

- Save raw JSON per step under `./runvia-out/`, build the table in code, and write a CSV.
- Show the CSV's top rows, the row count, what came back empty, and the credits spent per step
  and in total.
- Engagement rate = (likes + comments) / views for video, / followers for image posts; say which
  one you used.

## Keep costs down

- Page through lists only as far as the user's row count needs.
- Filter on fields you already paid for before calling a detail endpoint.
- Reuse results within the session; never fetch the same profile twice.
