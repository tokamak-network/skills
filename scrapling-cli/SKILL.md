---
name: scrapling-cli
description: "Scrapling CLI(명령줄) 도구로 터미널에서 웹 스크래핑하는 방법. 코드 작성 없이 scrapling extract 명령으로 웹페이지를 HTML/Markdown/텍스트로 추출하고, scrapling shell로 인터랙티브 스크래핑 REPL을 실행하며, scrapling install로 브라우저 의존성을 설치한다. 사용자가 scrapling CLI, scrapling extract, scrapling shell, scrapling install, scrapling mcp 명령을 언급하거나, '터미널에서 웹페이지 가져와줘', '커맨드라인으로 사이트 스크래핑', 'CLI로 Cloudflare 사이트 내용 추출', 'curl 명령을 scrapling으로 변환', '코드 없이 웹 데이터 다운로드' 같은 요청을 할 때 이 스킬을 사용하세요. Python 코드로 스크래핑하는 경우가 아니라 터미널 명령어로 스크래핑하는 경우에 해당한다."
---

# Scrapling CLI 스킬

Scrapling CLI는 터미널에서 코드 없이 웹 스크래핑을 수행하는 도구다. 이 스킬은 **명령줄 사용**에 집중한다. Python API로 스크래핑 코드를 작성하는 경우는 별도의 `scrapling` 스킬을 참조한다.

전체 옵션 상세 → `references/extract-options.md`
쉘 상세 사용법 → `references/shell.md`

## 설치

```bash
pip install "scrapling[shell]"    # CLI + 쉘 기능
scrapling install                 # 브라우저 및 시스템 의존성 (필수)
```

`scrapling install`은 Playwright Chromium과 시스템 의존성을 설치한다. `fetch`와 `stealthy-fetch` 명령에 필요하므로 반드시 실행한다.

## 명령어 구조

```
scrapling
├── install             # 브라우저/의존성 설치
├── shell               # 인터랙티브 웹 스크래핑 REPL
├── mcp                 # AI 연동 MCP 서버
└── extract             # 코드 없이 웹페이지 추출
    ├── get / post / put / delete   # HTTP 요청 (Fetcher 기반)
    ├── fetch                       # 브라우저 렌더링 (DynamicFetcher 기반)
    └── stealthy-fetch              # 스텔스 브라우저 (StealthyFetcher 기반)
```

---

## 어떤 명령을 써야 하는가

가장 가벼운 것부터 시도하고, 실패하면 한 단계 올리는 것이 원칙이다.

```
extract get  →  extract fetch  →  extract stealthy-fetch
(HTTP 요청)    (브라우저 렌더링)    (스텔스+Cloudflare 우회)
```

| 시그널 | 의미 | 올려야 할 단계 |
|--------|------|---------------|
| 정상 HTML 반환 | 성공 | — |
| HTML은 오지만 콘텐츠가 비어 있음 | JS 렌더링 필요 | `get` → `fetch` |
| 403, 429, 503 응답 | 봇 감지/차단 | `fetch` → `stealthy-fetch` |
| Cloudflare challenge 페이지 | 인터스티셜 | `stealthy-fetch --solve-cloudflare` |
| IP 차단 의심 | 프록시 필요 | `--proxy` 추가 |

### 빠른 판단 테이블

| 상황 | 명령 |
|------|------|
| 블로그, 뉴스, 정적 HTML | `extract get` |
| API 엔드포인트 호출 | `extract get` 또는 `extract post` |
| React/Vue/Angular SPA | `extract fetch --network-idle` |
| Cloudflare 보호 사이트 | `extract stealthy-fetch --solve-cloudflare` |
| 셀렉터 탐색, 빠른 프로토타이핑 | `shell` |
| curl 명령을 스크래핑으로 변환 | `shell` → `uncurl()` / `curl2fetcher()` |

---

## extract — 웹페이지 추출

### 출력 포맷

파일 확장자로 결정된다:
- `.md` → HTML을 Markdown으로 변환 (문서/블로그에 적합)
- `.html` → HTML 원본 저장
- `.txt` → 텍스트만 추출 (태그 제거)

### extract get / post / put / delete (HTTP 요청)

기본 형식:
```bash
scrapling extract get URL OUTPUT_FILE [OPTIONS]
scrapling extract post URL OUTPUT_FILE [OPTIONS]
```

핵심 패턴:
```bash
# 마크다운으로 저장
scrapling extract get "https://example.com" content.md

# CSS 셀렉터로 특정 부분만 추출 (-s)
scrapling extract get "https://example.com" article.md -s "article"

# 헤더 + 쿠키 + 프록시
scrapling extract get "https://example.com" page.md \
  -H "Authorization: Bearer token" \
  --cookies "session=abc" \
  --proxy "http://user:pass@proxy:8080"

# 브라우저 TLS 위장 (봇 탐지 회피에 도움)
scrapling extract get "https://example.com" page.md --impersonate chrome

# POST: 폼 데이터 또는 JSON
scrapling extract post "https://api.example.com" results.txt \
  -d "query=python&page=1"
scrapling extract post "https://api.example.com" out.json \
  -j '{"q": "test"}'
```

자주 쓰는 옵션 요약: `-H` (헤더, 반복 가능), `-s` (CSS 셀렉터), `-p` (쿼리 파라미터, 반복 가능), `--cookies`, `--proxy`, `--impersonate`, `--timeout` (기본 30초).
전체 옵션 → `references/extract-options.md#공통-옵션`

### extract fetch (브라우저 렌더링)

JS로 콘텐츠를 로드하는 SPA에 사용한다. Playwright 기반 Chromium이 실행되므로 `scrapling install` 필수.

```bash
# 기본 — 헤드리스 브라우저로 렌더링
scrapling extract fetch "https://spa-site.com" content.md

# 네트워크 유휴 대기 — SPA가 추가 API 콜을 마칠 때까지 기다림
scrapling extract fetch "https://spa-site.com" content.md --network-idle

# 특정 요소가 DOM에 나타날 때까지 대기
scrapling extract fetch "https://spa-site.com" data.txt --wait-selector ".loaded"

# 디버깅: 브라우저 보이게 + 리소스 차단
scrapling extract fetch "https://example.com" page.html --no-headless --disable-resources
```

주요 옵션: `--network-idle`, `--wait-selector`, `--wait` (추가 대기 ms), `--headless/--no-headless`, `--real-chrome`, `--timeout` (기본 30000ms — 밀리초 단위).
전체 옵션 → `references/extract-options.md#fetch`

### extract stealthy-fetch (스텔스 + Cloudflare 우회)

봇 탐지/Cloudflare 보호가 있는 사이트에 사용한다. fetch의 모든 기능에 더해 핑거프린팅 방지와 Cloudflare 자동 해결을 지원한다.

```bash
# Cloudflare 자동 해결
scrapling extract stealthy-fetch "https://cf-protected.com" content.md \
  --solve-cloudflare

# 최대 스텔스: Cloudflare + WebRTC 차단 + 캔버스 노이즈 + 프록시
scrapling extract stealthy-fetch "https://protected.com" content.md \
  --solve-cloudflare --block-webrtc --hide-canvas \
  --proxy "http://user:pass@proxy:8080" --network-idle

# 특정 콘텐츠만 추출
scrapling extract stealthy-fetch "https://cf-site.com" data.html \
  --solve-cloudflare -s "#main-content"
```

추가 옵션: `--solve-cloudflare` (Cloudflare 챌린지 해결), `--block-webrtc` (IP 유출 방지 — 프록시 사용 시 권장), `--hide-canvas` (핑거프린팅 방지), `--allow-webgl/--block-webgl` (기본 허용 — 비활성화 비추천).
전체 옵션 → `references/extract-options.md#stealthy-fetch`

---

## shell — 인터랙티브 웹 스크래핑 REPL

IPython 기반 환경으로, import 없이 바로 스크래핑을 시작할 수 있다. 셀렉터 테스트, 빠른 데이터 탐색, curl 명령 변환에 적합하다.

```bash
scrapling shell
scrapling shell -c "get('https://example.com'); print(page.css('title::text').get())"
```

### 핵심 사용법

```python
# 요청 → 자동으로 page/response에 저장
>>> get('https://example.com')
>>> page.css('h1::text').get()       # 셀렉터로 데이터 추출
>>> page.status                       # HTTP 상태 코드
>>> response.json()                   # JSON 응답 파싱

# 브라우저 렌더링이 필요하면
>>> fetch('https://spa-site.com', network_idle=True)
>>> stealthy_fetch('https://cf-site.com', solve_cloudflare=True)

# curl 명령 변환
>>> curl2fetcher('''curl 'https://api.com' -X POST -H 'Content-Type: application/json' -d '{"k":"v"}' ''')
>>> page.json()
```

내장 단축 명령: `get`, `post`, `put`, `delete`, `fetch`, `stealthy_fetch`, `view`, `uncurl`, `curl2fetcher`. `page`/`response`는 마지막 응답, `pages`는 최근 5개 이력.

상세 (curl 변환, 세션 사용, IPython 팁, 워크플로우 예제) → `references/shell.md`

---

## install / mcp

```bash
scrapling install          # Chromium + 시스템 의존성 설치 (최초 1회)
scrapling install --force  # 강제 재설치

scrapling mcp              # MCP 서버 stdio 모드
scrapling mcp --http       # HTTP 트랜스포트 (--host, --port 지정 가능)
```
