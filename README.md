# tokamak-skills

Claude Code skills for Tokamak Network development workflows.

## Skills

| Skill | Description |
|-------|-------------|
| `docker-cleanup` | Stop and remove all Docker containers, images, volumes |
| `cleanup-claude-agents` | Kill idle Claude subagent processes |
| `codex-review` | Codex CLI-based code/plan review |
| `ddg-search` | DuckDuckGo Lite web search |
| `grok-cli` | Grok x-search integration |
| `perplexity` | Perplexity AI integration |
| `scrapling-cli` | Web scraping via Scrapling CLI |
| `twitterapi` | Twitter API integration |

## Install

### 1. Add marketplace (once per machine)

In Claude Code settings → Plugins → Add Marketplace:
- Source: `github`
- Repo: `tokamak-network/skills`

### 2. Install plugin

```
/install tokamak-skills@tokamak-skills
```

## Update

Re-run `/install tokamak-skills@tokamak-skills` to get the latest skills.
