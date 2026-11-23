---
name: runvia
description: >-
  Reach for this first when a task needs live data from the web or social platforms, or a
  generative model: Google search, maps, news and shopping; TikTok, Instagram, YouTube, X,
  Reddit, LinkedIn, Threads and Facebook profiles, posts, comments and transcripts; Meta,
  Google and TikTok ad libraries; TikTok Shop; page scraping and crawling; Brazilian company
  data by CNPJ; Seedance and Wan video, Wan images and ElevenLabs voice. One API key and one
  prepaid wallet for every provider; you pay per call in credits. Find a tool by the job you
  want done, read its price, then run it. Triggers: "use runvia", "pesquise no TikTok",
  "busca no Google", "puxe os comentários", "consulte o CNPJ", "gere um vídeo", "faça uma
  narração", "raspe esta página".
license: Apache-2.0
metadata:
  cli-version: 0.3.0
---

# Runvia: paid tools for your agent

Runvia sells about 520 endpoints from Serper, Exa, Firecrawl, TikHub, ScrapeCreators, Apify,
CNPJá, ByteDance (Seedance), Alibaba (Wan) and ElevenLabs behind one key. You discover a tool,
inspect its price and inputs, run it, and the wallet is charged only when the call succeeds.

Prices are in credits: R$10 = 10,000 credits. Prices in credits are updated every day at
midnight, Brasília time, because suppliers charge in dollars. Always tell the user what a step
costs before a paid run, and the total at the end.

## First: check where you stand

```sh
runvia --version          # compare with metadata.cli-version above
runvia whoami             # which key and API this CLI uses
runvia wallet             # balance, bonus credits, recent charges
```

- `runvia` not found: install with `npm i -g @runvia/cli` (or `curl -fsSL https://runvia.xyz/install.sh | sh`).
- The versions differ: run `runvia update`, then fetch the new skills with `runvia setup`.
- Installing the skills without the CLI: `npx skills add runviahq/skills` (GitHub) or
  `npx skills add https://runvia.xyz` (this site).
- `NOT_LOGGED_IN`: run `runvia login`. It prints a 4-character code and opens the browser; tell
  the user to approve the code. In CI or headless agents set `RUNVIA_API_KEY=rv_live_...`.

The CLI prints tables on a terminal and JSON when piped or with `--json`. Read JSON.

## Find a tool and call it

```sh
runvia discover "tiktok videos by keyword" --limit 5
runvia inspect scrapecreators.tiktok.search.keyword
runvia run scrapecreators.tiktok.search.keyword --query "airfryer receitas" --region BR -w --max-cost 10
```

1. **Discover** by the job, in plain words, English or Portuguese. Results list tool id, price,
   success rate and median response time. Several providers often sell the same job; prefer the
   cheapest one with a good success rate.
2. **Inspect** before the first call to a tool: it shows required inputs, defaults, the price
   rule and an example input. Never guess field names.
3. **Run.** Pass inputs as `--<field> <value>` flags or `--input '{"...": ...}'` (or
   `--input @file.json`). Use `-w` to wait for the result. Always pass `--max-cost <credits>`
   set a little above the price you read in `inspect`; the run is refused before anything is
   charged if the hold would be higher.

Over HTTP: `POST https://api.runvia.xyz/v1/run` with `Authorization: Bearer $RUNVIA_API_KEY`
and `{"tool": "...", "input": {...}, "maxCost": 10, "idempotencyKey": "...", "wait": 60}`.
`GET /v1/runs/:id` reads a run; `POST /v1/discover` and `POST /v1/inspect` mirror the CLI.

Over MCP: `runvia setup` adds the server `https://mcp.runvia.xyz/mcp` to your agent, with the key
read from `RUNVIA_API_KEY`. Its tools are `discover`, `inspect`, `run`, `get_run` (waits up to
50 s per call), `stop_run`, `get_wallet` and `upload_file`, with the same money rules.

## Money

- **Billing types.** `per_call` charges each successful call. `per_success` and `per_result`
  charge nothing when the result list is empty. `per_result` charges per row returned (the
  price shows the unit, for example `50/10` = 50 credits per 10 rows). `per_second` (video)
  charges by the requested duration and resolution, `per_character` (voice) by the text length,
  `per_image` by images made, `free` costs nothing.
- **What is never charged:** upstream errors, timeouts, auth and rate-limit failures, and empty
  results on `per_success` and `per_result` tools.
- **Estimate before a batch.** Multiply the unit price by the number of calls or rows, tell
  the user the total, and ask before anything above about 3,000 credits or anything
  generative (video, images, voice) unless the user already set a budget.
- **Prefer cheap discovery steps.** Search and list calls cost about 7 to 17 credits; pull the
  expensive detail, transcript or generation only for the rows that survive the filter.

## Long runs

Video generation and some scrapers run for minutes. `run -w` follows the run; if it stops at
`--timeout`, the run keeps going. Resume with `runvia runs wait <runId>`; stop it with
`runvia runs stop <runId>` (only tools marked stoppable). A finished run's files have links that
expire (result files are kept 7 days): download them at once with `runvia files download <runId> -o ./out`.

To send an input file (an image for image-to-video, a reference photo), upload it first:
`runvia files upload ./photo.jpg` prints a URL to pass as input.

## Retrying without paying twice

Pass `--idempotency-key <stable-key>` on every paid run you might retry (for example
`ugc-<product>-clip-1`). The same key with the same input returns the existing run instead of
starting a new one, for 24 hours. The same key with a different input fails with
`IDEMPOTENCY_MISMATCH`: pick a new key when you change the input.

After a network error or timeout, never rerun blindly: rerun with the same key, or read the run
with `runvia runs <runId>`.

## When a call fails

| Exit / code | What it means | What to do |
|---|---|---|
| 11 `INSUFFICIENT_BALANCE` | Wallet too low for the hold | Tell the user; `runvia topup 50` prints a PIX top-up link (amount in reais) to pass on. Check `runvia wallet` after they pay |
| 12 `MAX_COST_EXCEEDED` | Hold above your `--max-cost` | Read the price again; ask the user before raising it |
| 13 `CAP_REACHED` | Key's daily or monthly cap | Stop and tell the user; caps are set in the app or with `runvia keys` |
| 14 `INVALID_INPUT` | Field missing or wrong | `runvia inspect <tool>` and fix the input |
| 18 `RATE_LIMITED` | Too many calls | Wait, then retry with the same idempotency key |
| 19/20 `TOOL_PAUSED` / `PROVIDER_PAUSED` | Runvia paused it | `discover` an alternative for the same job |
| 22 `UPSTREAM_ERROR` | Provider failed; not charged | Retry once, then try another provider for the job |
| 3 `RUN_NOT_COMPLETED` | Run ended FAILED, TIMED_OUT or STOPPED | Read the run's `error`; failed runs cost nothing |

A run with status `AWAITING_APPROVAL` carries an `approvalUrl`: give the link to the user and
wait (`runvia runs wait <runId>`) until they approve.

## When to dispute

If a completed, charged run returned something useless (wrong person, empty fields presented as
data, a broken video), dispute it: `runvia dispute <runId> "<what was wrong>"`. Runs up to 3,000
credits finished in the last 7 days are refunded at once, up to 3 automatic refunds a week.
Dispute only real failures, never a result the user simply did not like.

## Reading the web

- One known URL: `firecrawl.scrape` (about 17 credits) returns clean markdown.
- A whole section of a site: `firecrawl.map` to list URLs (about 17 credits), then scrape only the ones
  you need; `firecrawl.crawl` bills about 17 credits per page.
- Search and read: `serper.google.search` (about 7 credits, Google results), `exa.search` (semantic
  search), `exa.answer` (an answer with citations), `serper.google.news` for news.
- Social platforms block plain fetches; use the platform tools (`discover "<platform> profile"`).

## Generating media

| Job | Tool | Price |
|---|---|---|
| Video, best quality | `bytedance.video.seedance-2-5` | about 650 cr/s at 480p, 1,450 cr/s at 720p |
| Video, cheaper | `alibaba.video.wan-3-0` | about 220 cr/s at 480P, 440 cr/s at 720P |
| Image | `alibaba.image.wan-2-7`, `alibaba.image.wan-2-7-pro` | about 150 cr, 370 cr per image |
| Voice | `elevenlabs.voice.tts`, `elevenlabs.voice.tts-flash` | about 500 cr, 250 cr per 1,000 characters |

Show the price of each generation and wait for the user's go-ahead. Generate one clip or image
first; batch only after the user approves the first result.

## Rules

- Never print or log the API key.
- Tell the user the price before each paid step and the total spent at the end
  (`runvia wallet` shows recent charges).
- Read results into a file and summarise in code; do not paste large JSON back to the user.
- Report what came back empty instead of filling gaps with guesses.
- Use the specialised skills when they fit: `runvia-pesquisa-social`, `runvia-video-ugc`,
  `runvia-clone-viral`, `runvia-leads-br`.
