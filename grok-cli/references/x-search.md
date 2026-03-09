# X Search 상세 레퍼런스

## CLI 옵션 요약

```bash
# 특정 사용자의 게시물만 검색
grok-cli ask "최근 AI 관련 발언을 정리해줘" -t x_search --x-handles elonmusk

# 날짜 범위 + 핸들 필터
grok-cli ask "xAI 발표 내용 요약" -t x_search --x-handles xai --x-from 2026-01-01 --x-to 2026-02-28

# 이미지가 포함된 게시물 분석
grok-cli ask "AI 관련 인포그래픽을 찾아줘" -t x_search --x-images

# 봇 계정 제외
grok-cli ask "GPT-5 반응" -t x_search --x-exclude gptbot,aibot
```

**제약 사항:**
- `--x-handles`와 `--x-exclude`는 동시에 사용할 수 없습니다.
- 핸들은 `@` 없이 입력합니다 (`elonmusk` ✓, `@elonmusk` ✗).
- `--x-images`/`--x-videos` 활성화 시 토큰 소비가 증가합니다.

---

## 검색 연산자

질문 텍스트에 직접 포함하면 Grok이 자동으로 인식합니다.

### 키워드

| 연산자 | 동작 | 예시 |
|--------|------|------|
| `A B` | AND (모두 포함) | `watching now` |
| `"구문"` | 정확한 구문 일치 | `"happy hour"` |
| `A OR B` | OR (하나 이상 포함) | `love OR hate` |
| `-키워드` | 해당 키워드 제외 | `beer -root` |
| `#태그` | 해시태그 | `#AI` |

### 계정

| 연산자 | 동작 | 예시 |
|--------|------|------|
| `from:핸들` | 해당 계정 작성 | `from:elonmusk` |
| `to:핸들` | 해당 계정에 답글 | `to:xai` |
| `@핸들` | 해당 계정 멘션 | `@NASA` |
| `list:소유자/이름` | 리스트 멤버의 게시물 | `list:NASA/astronauts-in-space-now` |

### 콘텐츠 필터

| 연산자 | 동작 |
|--------|------|
| `filter:media` | 이미지 또는 비디오 포함 |
| `filter:images` | 이미지 포함 |
| `filter:native_video` | 네이티브 비디오 포함 |
| `filter:links` | URL 포함 |
| `filter:replies` | 답글만 |
| `-filter:replies` | 답글 제외 (원본만) |
| `-filter:retweets` | 리트윗 제외 |
| `-filter:media` | 미디어 없는 게시물만 |
| `-filter:links` | URL 없는 게시물만 |

### 날짜

| 연산자 | 예시 |
|--------|------|
| `since:YYYY-MM-DD` | `since:2026-01-01` |
| `until:YYYY-MM-DD` | `until:2026-03-01` |

### 참여도

| 연산자 | 예시 |
|--------|------|
| `min_retweets:N` | `min_retweets:1000` |
| `min_faves:N` | `min_faves:5000` |

### URL / 감성

| 연산자 | 동작 |
|--------|------|
| `url:키워드` | URL에 키워드 포함 |
| `:)` | 긍정적 감성 |
| `:(` | 부정적 감성 |
| `?` | 질문형 |

---

## CLI 옵션 vs 검색 연산자

| 기능 | CLI 옵션 | 검색 연산자 |
|------|---------|------------|
| 특정 사용자 필터 | `--x-handles` | `from:핸들` |
| 사용자 제외 | `--x-exclude` | — |
| 날짜 범위 | `--x-from`/`--x-to` | `since:`/`until:` |
| 미디어 분석 | `--x-images`/`--x-videos` | _(없음)_ |
| 참여도 필터 | _(없음)_ | `min_faves:`/`min_retweets:` |
| 콘텐츠 필터 | _(없음)_ | `filter:media` 등 |
| 감성 필터 | _(없음)_ | `:)`/`:(` |

CLI 옵션은 API 수준 강제 필터, 검색 연산자는 세밀한 쿼리 조건에 사용합니다.

---

## 실전 시나리오

```bash
# 여론/감성 분석
grok-cli ask "최근 일주일간 ChatGPT에 대한 X 반응을 긍정/부정으로 분류해줘" \
  -t x_search --x-from 2026-02-26 --x-to 2026-03-05

# 특정 인물의 발언 추적
grok-cli ask "일론 머스크의 최근 AI 관련 발언을 시간순으로 정리해줘" \
  -t x_search --x-handles elonmusk

# 웹 + X 동시 검색으로 종합 리서치
grok-cli ask "Grok-4 발표에 대한 뉴스와 X 반응을 종합해서 요약해줘" \
  -t web_search -t x_search

# 인기 게시물 찾기 (검색 연산자 활용)
grok-cli ask "AI agent min_faves:10000 since:2026-01-01 관련 인기 게시물을 분석해줘" \
  -t x_search

# 경쟁사 부정 감성 모니터링
grok-cli ask "\"competitor_name\" stopped using OR \"bad service\" OR :(" -t x_search

# 특정 계정 간 대화 검색
grok-cli ask "from:typefully to:linuz90" -t x_search

# 고객 질문 탐지 (답글 제외)
grok-cli ask "\"how do I\" OR \"need help\" our_product -filter:replies" -t x_search
```
