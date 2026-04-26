---
name: perplexity-cli
description: "Perplexity Search API CLI 도구로 터미널에서 웹 검색하는 방법. perplexity-cli search 명령으로 Perplexity Search API를 호출하여 웹 검색 결과를 가져오고, perplexity-cli auth로 API 키를 관리한다. 사용자가 Perplexity로 검색해달라고 하거나, perplexity-cli를 언급하거나, 'Perplexity로 최신 정보 찾아줘', '웹 검색 결과를 Perplexity로 가져와줘', 'Perplexity API로 조회해줘' 같은 요청을 할 때 이 스킬을 사용하세요."
---

# perplexity-cli

Perplexity Search API를 래핑한 CLI 도구다. 터미널에서 웹 검색을 수행하고, 국가·언어·도메인·날짜 필터를 적용하며, JSON 또는 사람이 읽기 좋은 포맷으로 결과를 출력한다.

**Binary location:** `perplexity-cli`
바이너리가 `$PATH`에 없으면 프로젝트 디렉토리의 `./target/release/perplexity-cli`를 확인한다.

## Credential Security — AI 에이전트 주의사항

API 키는 OS 키체인(`perplexity-cli auth save`)에 미리 저장되어 있어야 한다. **절대 `--api-key <KEY>`로 직접 전달하거나 `export PERPLEXITY_API_KEY=...`를 설정하지 않는다.** 에이전트 컨텍스트, 셸 히스토리, 프로세스 목록에 키가 노출된다.

API 키가 설정되지 않아 오류가 발생하면, 사용자에게 별도 터미널에서 직접 실행하도록 안내한다:

```bash
perplexity-cli auth save
```

에이전트가 `auth save`를 직접 실행해서는 안 된다 — 키 값이 컨텍스트에 노출된다.

## AI 에이전트 사용 시 `--json` 플래그

AI 에이전트로 실행할 때는 항상 `--json`을 추가한다. 구조화된 JSON이 출력되어 프로그래밍적으로 파싱할 수 있다.

```bash
perplexity-cli search --json "query"
```

`--json` 없이 실행하면 사람이 읽기 좋은 포맷(번호 매기기, 스니펫 자르기 등)으로 출력된다.

## API 키 우선순위

키는 아래 순서로 탐색된다:

1. **OS Keystore** — `perplexity-cli auth save`로 저장
2. **환경 변수** — `PERPLEXITY_API_KEY`
3. **CLI 플래그** — `--api-key <KEY>` (비권장)

## Commands Reference

### search — 웹 검색

```bash
perplexity-cli search [OPTIONS] <QUERY>...
```

`<QUERY>`를 여러 개 전달하면 multi-query 검색(배열)으로 동작한다.

#### 옵션

| 플래그 | 짧은 형태 | 설명 |
|--------|----------|------|
| `--max-results <N>` | `-n` | 최대 결과 수 (1–20, 기본 10) |
| `--max-tokens <N>` | | 컨텍스트 최대 토큰 수 (기본 10000) |
| `--max-tokens-per-page <N>` | | 페이지 당 최대 토큰 (기본 4096) |
| `--country <CODE>` | `-c` | 국가 코드 — ISO 3166-1 alpha-2 (예: US, KR, JP) |
| `--lang <CODES>` | `-l` | 언어 필터 — ISO 639-1, 쉼표 구분 (예: en,ko) |
| `--domain <DOMAINS>` | `-d` | 도메인 필터, 쉼표 구분. `-` 접두사로 제외 (예: reddit.com,-twitter.com) |
| `--recency <PERIOD>` | `-r` | 최신성 필터: `hour`, `day`, `week`, `month`, `year` |
| `--after <DATE>` | | 이 날짜 이후 결과만 (MM/DD/YYYY) |
| `--before <DATE>` | | 이 날짜 이전 결과만 (MM/DD/YYYY) |
| `--json` | | 원본 JSON 응답 출력 |
| `--full` | | 스니펫 전체 표시 (자르기 없음) |
| `--api-key <KEY>` | | API 키 직접 지정 (비권장) |

#### 사용 예시

```bash
# 기본 검색
perplexity-cli search "Rust async runtime comparison"

# JSON 출력 (AI 에이전트 권장)
perplexity-cli search --json "latest GPT-5 news"

# 한국어, 한국 결과만, 최근 1주일
perplexity-cli search --json -c KR -l ko -r week "LLM 최신 동향"

# 특정 도메인에서만 검색
perplexity-cli search --json -d arxiv.org,github.com "transformer attention mechanism"

# 특정 도메인 제외
perplexity-cli search --json -d "-reddit.com,-twitter.com" "best programming languages 2025"

# 날짜 범위 필터
perplexity-cli search --json --after 01/01/2025 --before 03/01/2025 "AI regulation"

# 결과 수 제한
perplexity-cli search --json -n 5 "Ethereum merge"

# 멀티 쿼리 검색
perplexity-cli search --json "Python vs Rust" "Go vs Rust"

# 전체 스니펫 + JSON
perplexity-cli search --json --full "quantum computing breakthroughs"
```

### auth — API 키 관리

```bash
perplexity-cli auth save      # API 키를 OS 키체인에 저장 (프롬프트 입력)
perplexity-cli auth show      # 현재 키 소스 및 마스킹된 키 표시
perplexity-cli auth delete    # 키체인에서 API 키 삭제
```

## JSON 응답 구조

`--json` 사용 시 출력 형식:

```json
{
  "id": "search-id-string",
  "server_time": "...",
  "results": [
    {
      "title": "페이지 제목",
      "url": "https://example.com/page",
      "snippet": "검색 결과 스니펫 텍스트...",
      "date": "2025-03-01",
      "last_updated": "2025-03-10"
    }
  ]
}
```

| 필드 | 설명 |
|------|------|
| `id` | 검색 요청 ID |
| `server_time` | 서버 응답 시각 (없을 수 있음) |
| `results[].title` | 페이지 제목 |
| `results[].url` | 페이지 URL |
| `results[].snippet` | 내용 스니펫 |
| `results[].date` | 게시일 (없을 수 있음) |
| `results[].last_updated` | 최종 업데이트일 (없을 수 있음) |

## Error Handling

| 오류 | 의미 | 조치 |
|------|------|------|
| API key not found | 키체인, 환경 변수, 플래그 모두 없음 | 사용자에게 `perplexity-cli auth save` 실행 안내 |
| Invalid recency filter | `--recency` 값이 유효하지 않음 | `hour`, `day`, `week`, `month`, `year` 중 선택 |
| Country code must be 2 characters | 국가 코드 형식 오류 | ISO 3166-1 alpha-2 코드 사용 (예: US, KR) |
| API error (401) | 유효하지 않거나 만료된 API 키 | 사용자에게 키 확인 요청 |
| API error (429) | Rate limit 초과 | 잠시 후 재시도 |
| HTTP request failed | 네트워크 오류 | 인터넷 연결 확인 |
