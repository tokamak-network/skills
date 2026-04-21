---
name: cleanup-claude-agents
description: Kill idle/orphaned Claude subagent processes while preserving active sessions and critical infrastructure
---

# Cleanup Claude Agents

GSD 또는 Agent tool 사용 후 누적되는 유휴 Claude 서브에이전트 프로세스를 정리합니다.

## Overview

Claude Code는 GSD planner/executor/verifier 등의 서브에이전트를 생성하며, 이 프로세스들이 작업 완료 후에도 종료되지 않고 남아있는 경우가 있습니다. 이 스킬은 유휴 서브에이전트만 선별적으로 종료합니다.

## Execution

### 1. 현황 파악

```bash
# 전체 Claude 프로세스 확인
ps aux | grep '[c]laude' | grep -c '--disallowedTools'
```

### 2. 서브에이전트 프로세스 종료

```bash
# --disallowedTools 플래그가 있는 서브에이전트만 종료 (GSD planner/executor/verifier)
ps aux | grep '[c]laude' | grep '--disallowedTools' | awk '{print $2}' | xargs kill -15
```

### 3. 잔여 orphan 프로세스 정리

```bash
# --output-format 플래그가 있는 orphan 프로세스 종료
ps aux | grep '[c]laude' | grep '--output-format' | awk '{print $2}' | xargs kill -15
```

## Preservation Targets (DO NOT KILL)

| Process | Identification | Reason |
|---------|---------------|--------|
| Interactive sessions | `--dangerously-skip-permissions` 또는 TTY attached | 사용자가 직접 사용 중 |
| claude-mem worker | `worker-service.cjs` | MCP 메모리 서비스 |
| MCP servers | `thedotmack plugin`, `claude-mem` | 플러그인 인프라 |
| chroma-mcp | `chroma-mcp` | 벡터 DB MCP 서버 |
| Claude.app helpers | Chrome native host processes | 브라우저 확장 |

## Termination Targets

| Target | Flag | Description |
|--------|------|-------------|
| GSD subagents | `--disallowedTools` | planner, executor, verifier 등 |
| Orphaned agents | `--output-format` | 정상 종료되지 않은 에이전트 |

## Safety

- 종료 전 실행 중인 GSD 작업이 없는지 확인하세요
- `kill -15` (SIGTERM)을 사용하여 graceful shutdown을 허용합니다
- SIGTERM으로 종료되지 않는 경우에만 `kill -9` (SIGKILL) 사용
- 현재 대화 중인 인터랙티브 세션은 절대 종료하지 마세요
