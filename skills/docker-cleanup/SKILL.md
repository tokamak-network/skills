---
name: docker-cleanup
description: Use when you need to stop and remove all Docker containers, images, and volumes to free up disk space or reset Docker state. Triggers: "clean docker", "docker 정리", "remove all containers", "docker reset", "free up docker space", "kill all containers".
---

# Docker Cleanup

## Overview

현재 실행 중인 모든 Docker 리소스(컨테이너, 이미지, 볼륨, 네트워크)를 정리한다.

## Quick Reference

```bash
# 1. 실행 중인 컨테이너 중지
docker stop $(docker ps -q) 2>/dev/null || true

# 2. 모든 컨테이너 제거
docker rm $(docker ps -aq) 2>/dev/null || true

# 3. 모든 이미지 제거
docker rmi $(docker images -q) 2>/dev/null || true

# 4. 모든 볼륨 제거
docker volume rm $(docker volume ls -q) 2>/dev/null || true

# 5. 모든 커스텀 네트워크 제거 + 빌드 캐시
docker network prune -f
docker builder prune -af
```

단일 명령으로 전부 정리 (컨테이너+네트워크+이미지+빌드캐시, 볼륨 제외):
```bash
docker system prune -af
```

볼륨까지 포함:
```bash
docker system prune -af --volumes
```

## 정리 범위 선택

| 목적 | 명령 |
|------|------|
| 실행 중 컨테이너만 중지 | `docker stop $(docker ps -q)` |
| 중지된 컨테이너 제거 | `docker container prune -f` |
| 사용하지 않는 이미지 제거 | `docker image prune -af` |
| 사용하지 않는 볼륨 제거 | `docker volume prune -f` |
| 전체 정리 (볼륨 포함) | `docker system prune -af --volumes` |

## 주의사항

- `docker system prune -af --volumes`는 **데이터 손실** 위험 있음 — PostgreSQL 볼륨 등 영속 데이터 포함
- trh-platform 사용 중이라면 `make down` 또는 `make clean`으로 먼저 graceful shutdown
- `2>/dev/null || true` 패턴: 컨테이너/이미지가 없을 때 오류 무시

## trh-platform 전용

```bash
# trh-platform 디렉토리에서
make clean      # docker compose down + volume 삭제 (graceful)

# 또는 강제 정리
make down && docker system prune -af --volumes
```
