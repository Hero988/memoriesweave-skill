# MemoriesWeave Skill

AI agent skill for [MemoriesWeave](https://memoriesweave.com) — create beautiful photo memory collections with AI-powered layouts, captions, and print products.

## Installation

### Claude Code

**Git Bash / macOS / Linux:**
```bash
mkdir -p ~/.claude/skills/memoriesweave && curl -sL https://raw.githubusercontent.com/Hero988/memoriesweave-skill/main/memoriesweave/SKILL.md -o ~/.claude/skills/memoriesweave/SKILL.md
```

**PowerShell (Windows):**
```powershell
New-Item -ItemType Directory -Force -Path "$HOME/.claude/skills/memoriesweave" | Out-Null; Invoke-WebRequest -Uri "https://raw.githubusercontent.com/Hero988/memoriesweave-skill/main/memoriesweave/SKILL.md" -OutFile "$HOME/.claude/skills/memoriesweave/SKILL.md"
```

The skill auto-loads in every Claude Code session. Verify with: "What skills are available?"

### Other Agents (Cursor, Codex, Gemini CLI, Copilot, etc.)

```bash
npx skills add Hero988/memoriesweave-skill
```

Works with 40+ agents via [skills.sh](https://skills.sh).

## Usage

Provide your API key when asking the agent to use MemoriesWeave:

> Please use the memoriesweave skill with API key: mw_sk_your_key_here to create a phone wallpaper memory

## Setup

1. Create an account at [memoriesweave.com](https://memoriesweave.com)
2. Go to **Settings > API Keys** and create a new key
3. Provide the key to the agent when requesting this skill

## API Reference

- **Interactive docs:** https://memoriesweave.com/docs/api
- **OpenAPI spec:** Bundled in `memoriesweave/assets/openapi.json`
- **Base URL:** `https://grandiose-loris-729.eu-west-1.convex.site/api/v1`

## License

MIT
