# Runvia skills

Agent Skills for [Runvia](https://runvia.xyz), one API key and one prepaid wallet for many
third-party APIs: web search, social platforms, Brazilian company data, and video, image and
voice models. Each skill teaches a coding agent to find a Runvia tool, read its price and run it
with the `runvia` CLI.

## Install

```sh
npx skills add runviahq/skills
```

The [skills CLI](https://skills.sh) installs them into Claude Code, Codex, Cursor, OpenCode,
Gemini CLI and the other agents it supports. The same skills install from the Runvia site with
`npx skills add https://runvia.xyz`.

With the Runvia CLI installed (`npm i -g @runvia/cli`), `runvia setup` installs the skills
and adds the Runvia MCP server to every agent it finds.

### Claude Code plugin

```
/plugin marketplace add runviahq/skills
/plugin install runvia@runvia
```

## Skills

### runvia

Reach for this first when a task needs live data from the web or social platforms, or a generative model: Google search, maps, news and shopping; TikTok, Instagram, YouTube, X, Reddit, LinkedIn, Threads and Facebook profiles, posts, comments and transcripts; Meta, Google and TikTok ad libraries; TikTok Shop; page scraping and crawling; Brazilian company data by CNPJ; Seedance and Wan video, Wan images and ElevenLabs voice. One API key and one prepaid wallet for every provider; you pay per call in credits. Find a tool by the job you want done, read its price, then run it.

### runvia-clone-viral

Clones the structure of a viral short video for the user's own product with Runvia: finds the top TikToks and Reels about the product or niche, reads their transcripts and comments, maps the beat-by-beat structure (hook, turn, proof, call to action), writes a new script on the same structure, and renders it with Seedance or Wan plus an ElevenLabs voice.

### runvia-leads-br

Builds Brazilian B2B lead lists with Runvia: finds businesses by segment and city on Google Maps or in the CNPJ registry, enriches each one with Receita Federal data (CNPJ, partners, activity, size, Simples Nacional status, state registration), reads the company's website for public contacts, and looks for buying signals in news and job posts. Output is a CSV with sources and the cost per row.

### runvia-pesquisa-social

Social research into a table with Runvia: find creators in a niche, pull profiles and recent posts, rank posts against each creator's own baseline, mine comments for objections and questions, read transcripts, and track brands and competitors on TikTok, Instagram, YouTube, X, Reddit, Threads, LinkedIn and Facebook. Output is a CSV with the cost per step.

### runvia-video-ugc

Makes short UGC-style video ads with Runvia: studies the hooks that work in the product's niche on TikTok and Instagram, writes a script, generates talking-head or product clips with Seedance or Wan, narrates with an ElevenLabs voice, and delivers the files with the cost of each step.

## Usage

The skills run tools through the `runvia` CLI. Install it with `npm i -g @runvia/cli`, sign
in with `runvia login`, and ask your agent for the job, for example "find the top TikTok
creators in this niche and put them in a CSV". The agent tells you what each step costs before
it runs.

## Skill structure

Each skill is a folder in `skills/` holding a `SKILL.md`: YAML frontmatter (`name`,
`description`, `license`, `metadata.cli-version`) followed by the instructions.
`metadata.cli-version` is the `@runvia/cli` version the skill was checked against; this
release matches `@runvia/cli` 0.3.2.

## Contributing

This repository is generated from Runvia's main repository and published with each CLI release.
Changes made here directly are replaced on the next release.

## License

[Apache-2.0](LICENSE)
