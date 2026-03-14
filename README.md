# MemoriesWeave Skill

AI agent skill for [MemoriesWeave](https://memoriesweave.com) — create beautiful photo memory collections with AI-powered layouts, captions, and print products.

## Installation

```bash
npx skills add Hero988/memoriesweave-skill
```

This works with Claude Code, Cursor, Codex, Gemini CLI, GitHub Copilot, and [40+ other agents](https://skills.sh).

### Install globally (available in all projects)

```bash
npx skills add -g Hero988/memoriesweave-skill
```

## What This Skill Enables

Once installed, your AI agent can:

- **List and browse** your photo library and workspaces
- **Create memories** — new photo collections with titles and descriptions
- **Design layouts** — chat with AI to generate custom HTML layouts
- **Fill content** — AI selects photos and writes captions automatically
- **Manage versions** — create and restore snapshots
- **Order prints** — browse products, design POD items, generate print files
- **Search conversations** — query imported WhatsApp chat history

## Setup

1. Create an account at [memoriesweave.com](https://memoriesweave.com)
2. Go to **Settings > API Keys** and create a new key
3. Set the environment variable:
   ```bash
   export MEMORIESWEAVE_API_KEY=mw_sk_your_key_here
   ```

## API Reference

- **Interactive docs:** https://memoriesweave.com/docs/api
- **OpenAPI spec:** Bundled in [memoriesweave/assets/openapi.json](memoriesweave/assets/openapi.json)
- **Base URL:** `https://grandiose-loris-729.convex.site/api/v1`

## Pricing

- CRUD operations (list, get, create, update, delete) are **free**
- AI operations (design, content, captioning) consume **credits** (1 credit = $1 USD)
- Same credit pool as the website

## License

MIT
