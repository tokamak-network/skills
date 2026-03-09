# extract 명령 전체 옵션 레퍼런스

이 파일은 `scrapling extract` 하위 6개 명령의 전체 옵션을 정리한다. SKILL.md의 핵심 패턴만으로 부족할 때 참조한다.

## 목차
1. [공통 옵션 (get/post/put/delete)](#공통-옵션)
2. [get 전체 옵션](#get)
3. [post/put 추가 옵션](#postput-추가-옵션)
4. [fetch 전체 옵션](#fetch)
5. [stealthy-fetch 전체 옵션](#stealthy-fetch)
6. [출력 포맷 규칙](#출력-포맷-규칙)

---

## 공통 옵션

get/post/put/delete가 공유하는 옵션:

| 옵션 | 약어 | 기본값 | 설명 |
|------|------|--------|------|
| `--headers` | `-H` | — | HTTP 헤더 `"Key: Value"` (반복 가능) |
| `--cookies` | — | — | 쿠키 `"name1=value1; name2=value2"` |
| `--timeout` | — | 30 | 타임아웃(초) |
| `--proxy` | — | — | 프록시 URL `"http://user:pass@host:port"` |
| `--css-selector` | `-s` | — | CSS 셀렉터 (매칭되는 모든 요소 추출) |
| `--params` | `-p` | — | 쿼리 파라미터 `"key=value"` (반복 가능) |
| `--follow-redirects` / `--no-follow-redirects` | — | True | 리다이렉트 따라가기 |
| `--verify` / `--no-verify` | — | True | SSL 인증서 검증 |
| `--impersonate` | — | — | 브라우저 TLS 위장 (chrome, firefox 등. 콤마 구분 시 랜덤 선택) |
| `--stealthy-headers` / `--no-stealthy-headers` | — | True | 실제 브라우저 헤더 자동 생성 |

## get

위 공통 옵션 전체를 지원한다. body 데이터 옵션 없음.

```bash
scrapling extract get [OPTIONS] URL OUTPUT_FILE
```

예제:
```bash
# 기본
scrapling extract get "https://example.com" content.md

# CSS 셀렉터로 부분 추출
scrapling extract get "https://example.com" article.md -s "article"

# 복합 옵션
scrapling extract get "https://example.com/search" results.md \
  -p "q=python" -p "page=1" \
  -H "Authorization: Bearer token" \
  --cookies "session=abc" \
  --impersonate chrome \
  --proxy "http://user:pass@proxy:8080" \
  --timeout 60

# Docker
docker run -v $(pwd)/output:/output scrapling \
  extract get "https://example.com" /output/article.md
```

## post/put 추가 옵션

get의 모든 옵션 + 아래 추가:

| 옵션 | 약어 | 설명 |
|------|------|------|
| `--data` | `-d` | 폼 데이터 `"param1=value1&param2=value2"` |
| `--json` | `-j` | JSON 문자열 `'{"key": "value"}'` |

```bash
scrapling extract post "https://api.example.com" results.html -d "query=test"
scrapling extract post "https://api.example.com" out.txt -j '{"q": "test"}'
scrapling extract put "https://api.example.com/1" out.txt -j '{"name": "new"}'
```

delete는 body 옵션(`-d`, `-j`) 없음. 공통 옵션만 사용.

## fetch

DynamicFetcher (Playwright 기반 브라우저)를 사용한다. HTTP 요청 옵션과 완전히 다른 옵션 체계.

```bash
scrapling extract fetch [OPTIONS] URL OUTPUT_FILE
```

| 옵션 | 약어 | 기본값 | 설명 |
|------|------|--------|------|
| `--headless` / `--no-headless` | — | True | 헤드리스 모드 |
| `--disable-resources` / `--enable-resources` | — | False | 불필요 리소스 차단 (이미지, CSS 등 → 속도↑) |
| `--network-idle` / `--no-network-idle` | — | False | 네트워크 유휴 대기 (500ms 무활동까지) |
| `--timeout` | — | 30000 | 타임아웃(**밀리초** — HTTP 명령의 '초'와 다름) |
| `--wait` | — | 0 | 페이지 로드 후 추가 대기(**밀리초**) |
| `--css-selector` | `-s` | — | CSS 셀렉터 |
| `--wait-selector` | — | — | 이 셀렉터가 DOM에 나타날 때까지 대기 |
| `--locale` | — | 시스템 | 브라우저 로케일 (예: `ko-KR`) |
| `--real-chrome` / `--no-real-chrome` | — | False | 시스템에 설치된 실제 Chrome 사용 |
| `--proxy` | — | — | 프록시 URL |
| `--extra-headers` | `-H` | — | 추가 헤더 (반복 가능) |

```bash
scrapling extract fetch "https://spa-site.com" content.md --network-idle
scrapling extract fetch "https://spa-site.com" data.txt --wait-selector ".loaded" --wait 2000
scrapling extract fetch "https://example.com" page.html --no-headless --disable-resources
```

## stealthy-fetch

StealthyFetcher (patchright 기반 스텔스 브라우저)를 사용한다. fetch의 모든 옵션 + 아래 추가:

| 옵션 | 기본값 | 설명 |
|------|--------|------|
| `--solve-cloudflare` / `--no-solve-cloudflare` | False | Cloudflare Turnstile/Interstitial 자동 해결 |
| `--block-webrtc` / `--allow-webrtc` | False | WebRTC 완전 차단 (프록시 사용 시 IP 유출 방지) |
| `--hide-canvas` / `--show-canvas` | False | 캔버스 연산에 노이즈 추가 (핑거프린팅 방지) |
| `--allow-webgl` / `--block-webgl` | True | WebGL 허용 (비활성화하면 감지될 수 있어 비추천) |

```bash
scrapling extract stealthy-fetch "https://cf-site.com" out.md --solve-cloudflare
scrapling extract stealthy-fetch "https://protected.com" out.md \
  --solve-cloudflare --block-webrtc --hide-canvas \
  --proxy "http://user:pass@proxy:8080" --network-idle
```

## 출력 포맷 규칙

출력 파일 확장자가 포맷을 결정한다:

| 확장자 | 동작 |
|--------|------|
| `.md` | HTML → Markdown 변환 후 저장 (문서/블로그에 적합) |
| `.html` | HTML 원본 그대로 저장 |
| `.txt` | 텍스트만 추출하여 저장 (태그 제거) |

`--css-selector` 사용 시 매칭된 요소들의 콘텐츠만 변환/저장된다.
