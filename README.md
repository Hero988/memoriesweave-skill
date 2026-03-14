# MemoriesWeave Skill

AI agent skill for [MemoriesWeave](https://memoriesweave.com) — create beautiful photo memory collections with AI-powered layouts, captions, and print products.

## Installation

### Claude Code (recommended)

Add the skill to your project with one command:

```bash
curl -sL https://raw.githubusercontent.com/Hero988/memoriesweave-skill/main/SKILL.md -o .claude/SKILL.md
```

Or if you prefer, add a reference to your `CLAUDE.md`:

```markdown
## Skills
See [MemoriesWeave API Skill](.claude/SKILL.md) for photo memory collection management.
```

### Cursor / Copilot / Other Agents

Copy `SKILL.md` into your project root or your agent's context directory:

```bash
# Clone and copy
git clone https://github.com/Hero988/memoriesweave-skill.git
cp memoriesweave-skill/SKILL.md /path/to/your/project/.cursor/SKILL.md
```

### Manual

Download [`SKILL.md`](SKILL.md) and place it wherever your AI agent reads context files from.

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
- **OpenAPI spec:** [assets/openapi.json](assets/openapi.json) (also at https://memoriesweave.com/api/openapi.json)
- **Base URL:** `https://grandiose-loris-729.convex.site/api/v1`

## Pricing

- CRUD operations (list, get, create, update, delete) are **free**
- AI operations (design, content, captioning) consume **credits** (1 credit = $1 USD)
- Same credit pool as the website

## License

MIT
