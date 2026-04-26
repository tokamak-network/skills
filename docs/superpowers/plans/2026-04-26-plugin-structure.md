# tokamak-skills Plugin Structure Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Claude Code 플러그인 규격에 맞게 `.claude-plugin/plugin.json` 추가 + 스킬을 `skills/` 서브디렉토리로 이동하여 `/install tokamak-skills@tokamak-skills` 명령으로 설치 가능하게 만든다.

**Architecture:** 레포 루트의 스킬 디렉토리를 `skills/`로 이동하고, `.claude-plugin/plugin.json`을 추가한다. 마켓플레이스 디렉토리(`~/.claude/plugins/marketplaces/tokamak-skills/`)를 `git pull`로 동기화한 뒤 캐시를 재빌드한다.

**Tech Stack:** bash, git, Claude Code plugin system

---

## File Map

| Action | Path |
|--------|------|
| Create | `tokamak-skills/.claude-plugin/plugin.json` |
| Create | `tokamak-skills/skills/` (move existing skill dirs here) |
| Delete | `tokamak-skills/cleanup-claude-agents/` (moved to `skills/`) |
| Delete | `tokamak-skills/codex-review/` (moved to `skills/`) |
| Delete | `tokamak-skills/ddg-search/` (moved to `skills/`) |
| Delete | `tokamak-skills/docker-cleanup/` (moved to `skills/`) |
| Delete | `tokamak-skills/grok-cli/` (moved to `skills/`) |
| Delete | `tokamak-skills/perplexity/` (moved to `skills/`) |
| Delete | `tokamak-skills/scrapling-cli/` (moved to `skills/`) |
| Delete | `tokamak-skills/twitterapi/` (moved to `skills/`) |

---

### Task 1: `.claude-plugin/plugin.json` 추가

**Files:**
- Create: `tokamak-skills/.claude-plugin/plugin.json`

- [ ] **Step 1: plugin.json 생성**

```bash
mkdir -p /Users/theo/workspace_tokamak/tokamak-skills/.claude-plugin
```

내용:
```json
{
  "name": "tokamak-skills",
  "version": "1.0.0",
  "description": "Tokamak Network team skills: Docker cleanup, Codex review, agent cleanup, web search, and more",
  "author": { "name": "tokamak-network" },
  "repository": "https://github.com/tokamak-network/skills",
  "license": "MIT"
}
```

- [ ] **Step 2: 파일 확인**

```bash
cat /Users/theo/workspace_tokamak/tokamak-skills/.claude-plugin/plugin.json
```

Expected: JSON 내용 출력

---

### Task 2: 스킬을 `skills/` 서브디렉토리로 이동

**Files:**
- Create: `tokamak-skills/skills/`
- Move: 루트의 8개 스킬 디렉토리 → `skills/`

- [ ] **Step 1: `skills/` 디렉토리 생성 및 이동**

```bash
cd /Users/theo/workspace_tokamak/tokamak-skills
mkdir -p skills
git mv cleanup-claude-agents skills/
git mv codex-review skills/
git mv ddg-search skills/
git mv docker-cleanup skills/
git mv grok-cli skills/
git mv perplexity skills/
git mv scrapling-cli skills/
git mv twitterapi skills/
```

- [ ] **Step 2: 구조 확인**

```bash
ls /Users/theo/workspace_tokamak/tokamak-skills/skills/
```

Expected: `cleanup-claude-agents  codex-review  ddg-search  docker-cleanup  grok-cli  perplexity  scrapling-cli  twitterapi`

- [ ] **Step 3: SKILL.md 경로 확인**

```bash
find /Users/theo/workspace_tokamak/tokamak-skills/skills -name "SKILL.md"
```

Expected: 8개 파일 출력

---

### Task 3: 커밋 및 푸시

- [ ] **Step 1: git status 확인**

```bash
git -C /Users/theo/workspace_tokamak/tokamak-skills status
```

Expected: renamed files + new `.claude-plugin/plugin.json`

- [ ] **Step 2: 커밋**

```bash
cd /Users/theo/workspace_tokamak/tokamak-skills && git add -A && git commit -m "feat: restructure as Claude Code plugin (.claude-plugin/plugin.json + skills/ dir)"
```

- [ ] **Step 3: 푸시**

```bash
git -C /Users/theo/workspace_tokamak/tokamak-skills push
```

---

### Task 4: 로컬 마켓플레이스 동기화

- [ ] **Step 1: 마켓플레이스 디렉토리 git pull**

```bash
git -C ~/.claude/plugins/marketplaces/tokamak-skills pull
```

Expected: docker-cleanup 포함 최신 구조 반영

- [ ] **Step 2: 마켓플레이스 구조 확인**

```bash
ls ~/.claude/plugins/marketplaces/tokamak-skills/skills/
```

Expected: 8개 스킬 디렉토리 (docker-cleanup 포함)

- [ ] **Step 3: 캐시 업데이트 — skills 복사**

```bash
CACHE_DIR=~/.claude/plugins/cache/tokamak-skills/tokamak-skills/1.0.0
cp -r ~/.claude/plugins/marketplaces/tokamak-skills/skills/* $CACHE_DIR/skills/
cp ~/.claude/plugins/marketplaces/tokamak-skills/.claude-plugin/plugin.json $CACHE_DIR/.claude-plugin/plugin.json
```

- [ ] **Step 4: 캐시 검증**

```bash
ls ~/.claude/plugins/cache/tokamak-skills/tokamak-skills/1.0.0/skills/
```

Expected: `docker-cleanup` 포함 8개 디렉토리

---

## 검증

```bash
# 스킬 목록 확인
ls ~/.claude/plugins/cache/tokamak-skills/tokamak-skills/1.0.0/skills/

# plugin.json 확인
cat ~/.claude/plugins/cache/tokamak-skills/tokamak-skills/1.0.0/.claude-plugin/plugin.json
```

신규 머신에서 설치 방법 (README에 추가):
```
Claude Code Settings → Plugins → Add Marketplace
  Name: tokamak-skills
  Source: github
  Repo: tokamak-network/skills

/install tokamak-skills@tokamak-skills
```
