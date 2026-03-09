---
name: grok-cli
description: grok-cli 커맨드라인 도구를 사용하여 Grok API를 호출하고, X(Twitter) 게시물을 검색·분석하는 방법. 사용자가 X에서 트윗을 검색하거나, 특정 계정의 게시물을 조회하거나, 트렌드·여론·감성을 분석하려 할 때 이 스킬을 사용하세요. grok-cli, Grok API, X 검색, 트윗 검색, 웹 검색, 코드 실행 등을 언급하거나, X 플랫폼의 실시간 데이터가 필요한 경우에도 반드시 이 스킬을 참고하세요.
---

# grok-cli

터미널에서 Grok API를 호출하는 CLI 도구. 웹 검색, X(Twitter) 검색, 코드 실행을 지원합니다.

## 핵심 명령어

```bash
grok-cli ask "질문 내용"                    # 기본 질문
grok-cli ask "질문" -t web_search           # 웹 검색 활성화
grok-cli ask "질문" -t x_search             # X 검색 활성화
grok-cli ask "질문" -t code_interpreter     # Python 코드 실행
grok-cli ask "질문" -t web_search -t x_search  # 복수 도구
echo "질문" | grok-cli ask                  # stdin 입력
```

기본 모델: `grok-4-1-fast-reasoning` / 스트리밍 기본 활성

## X Search 빠른 참조

`-t x_search` 지정 시 사용 가능한 옵션:

| 옵션 | 설명 |
|------|------|
| `--x-handles 핸들1,핸들2` | 검색 대상 핸들 (@ 없이) |
| `--x-exclude 핸들1,핸들2` | 제외할 핸들 |
| `--x-from YYYY-MM-DD` | 시작 날짜 |
| `--x-to YYYY-MM-DD` | 종료 날짜 |
| `--x-images` | 이미지 분석 활성화 |
| `--x-videos` | 비디오 분석 활성화 |

`--x-handles`와 `--x-exclude`는 동시 사용 불가.

질문 텍스트에 X 검색 연산자(`from:`, `min_faves:`, `filter:media`, `since:` 등)를 직접 포함할 수도 있습니다.

## 레퍼런스 안내

상세 정보가 필요할 때만 아래 파일을 읽으세요.

| 파일 | 읽어야 할 때 |
|------|-------------|
| [references/x-search.md](./references/x-search.md) | X 검색 연산자 전체 목록, 연산자 조합, CLI 옵션 vs 연산자 비교, 실전 시나리오가 필요할 때 |
| [references/setup.md](./references/setup.md) | API 키 설정, 모델 변경, 기타 CLI 옵션 등 초기 설정이 필요할 때 |
