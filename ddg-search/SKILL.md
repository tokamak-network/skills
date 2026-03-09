---
name: ddg-search
description: DuckDuckGo Lite를 이용한 웹 검색 방법. 일반 DuckDuckGo HTML 버전은 JavaScript 의존으로 빈 결과를 반환하므로 반드시 Lite 버전(lite.duckduckgo.com/lite/)을 사용해야 합니다. 웹 검색이 필요할 때, DuckDuckGo로 정보를 조회할 때, 검색 결과를 파싱해야 할 때, 외부 정보를 수집하여 요약해야 할 때 이 스킬을 사용하세요.
---

# DuckDuckGo Lite 웹 검색

일반 `duckduckgo.com`(HTML 버전)은 JavaScript로 렌더링되어 `fetch`/`curl`로는 빈 페이지를 반환합니다.
**반드시 Lite 버전**(`https://lite.duckduckgo.com/lite/`)을 사용해야 파싱 가능한 HTML을 얻을 수 있습니다.

## 요청 방법

**POST** 요청으로 보냅니다. GET도 가능하지만 POST가 더 안정적입니다.

```
POST https://lite.duckduckgo.com/lite/
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)

q=<URL인코딩된 검색어>
```

### TypeScript/JavaScript 예시

```typescript
const res = await fetch("https://lite.duckduckgo.com/lite/", {
  method: "POST",
  headers: {
    "Content-Type": "application/x-www-form-urlencoded",
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)",
  },
  body: `q=${encodeURIComponent("검색어")}`,
});
const html = await res.text();
```

### curl 예시

```bash
curl -s -X POST "https://lite.duckduckgo.com/lite/" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "User-Agent: Mozilla/5.0" \
  -d "q=search+query"
```

## 응답 HTML 파싱

DDG Lite의 검색 결과는 `<table>` 기반 HTML로 반환됩니다.
세 가지 정보를 추출할 수 있습니다: **제목**, **URL**, **스니펫**.

### 1. 제목 추출

```
<a class='result-link'>제목 텍스트</a>
```

정규식: `/class='result-link'>([^<]+)<\/a>/g`

### 2. URL 추출

`result-link` 클래스 바로 앞의 `href` 속성에서 URL을 추출합니다.
DDG Lite는 두 가지 형태로 URL을 제공할 수 있습니다:

- **직접 URL**: `href="https://example.com/page"` — 그대로 사용
- **리다이렉트**: `href="//duckduckgo.com/l/?uddg=<인코딩된URL>&..."` — `uddg=` 값을 디코딩

정규식: `/href="([^"]+)"[^>]*class='result-link/g`

추출 후 `uddg=` 파라미터가 있으면 디코딩, 없으면 href 값 그대로 사용합니다.

### 3. 스니펫 추출

스니펫은 `<td>` 태그 안에 들어 있습니다. 다만 URL, 번호, 빈 셀 등도 `<td>` 안에 있으므로 필터링이 필요합니다.

정규식: `/<td>([\s\S]*?)<\/td>/g`

필터 조건:
- HTML 태그 제거 후 텍스트 길이 > 40자
- `https://`로 시작하지 않음 (URL 셀 제외)
- `&nbsp;` 포함하지 않음 (빈 셀 제외)
- 숫자+마침표만 있지 않음 (번호 셀 제외)

### HTML 엔티티 디코딩

파싱 후 아래 엔티티들을 반드시 디코딩하세요:

| 엔티티 | 문자 |
|--------|------|
| `&amp;` | `&` |
| `&lt;` | `<` |
| `&gt;` | `>` |
| `&quot;` | `"` |
| `&#39;` | `'` |

## 전체 파싱 함수 (TypeScript)

```typescript
interface SearchResult {
  title: string;
  url: string;
  snippet: string;
}

async function webSearch(query: string, maxResults = 8): Promise<SearchResult[]> {
  const res = await fetch("https://lite.duckduckgo.com/lite/", {
    method: "POST",
    headers: {
      "Content-Type": "application/x-www-form-urlencoded",
      "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)",
    },
    body: `q=${encodeURIComponent(query)}`,
  });
  const html = await res.text();

  const titleRe = /class='result-link'>([^<]+)<\/a>/g;
  const linkRe = /href="([^"]+)"[^>]*class='result-link/g;

  const titles: string[] = [];
  const urls: string[] = [];
  let m: RegExpExecArray | null;
  while ((m = titleRe.exec(html))) titles.push(decode(m[1].trim()));
  while ((m = linkRe.exec(html))) {
    const raw = m[1];
    const uddgMatch = raw.match(/uddg=([^&]+)/);
    urls.push(uddgMatch ? decodeURIComponent(uddgMatch[1]) : raw);
  }

  const tdRe = /<td>([\s\S]*?)<\/td>/g;
  const snippets: string[] = [];
  while ((m = tdRe.exec(html))) {
    const clean = decode(m[1].replace(/<[^>]+>/g, "")).trim();
    if (
      clean.length > 40 &&
      !/^https?:\/\//.test(clean) &&
      !/&nbsp;/.test(clean) &&
      !/^\d+\.\s*$/.test(clean)
    ) {
      snippets.push(clean);
    }
  }

  const results: SearchResult[] = [];
  for (let i = 0; i < Math.min(maxResults, titles.length); i++) {
    results.push({
      title: titles[i],
      url: urls[i] ?? "",
      snippet: snippets[i] ?? "",
    });
  }
  return results;
}

function decode(s: string): string {
  return s
    .replace(/&amp;/g, "&")
    .replace(/&lt;/g, "<")
    .replace(/&gt;/g, ">")
    .replace(/&quot;/g, '"')
    .replace(/&#39;/g, "'");
}
```

## 주의사항

- **일반 DuckDuckGo 사용 금지**: `https://duckduckgo.com/?q=`는 JavaScript 전용 → fetch로 빈 결과
- **User-Agent 필수**: UA 헤더 없으면 403이나 빈 응답 반환 가능
- **POST 권장**: GET(`https://lite.duckduckgo.com/lite/?q=`)도 동작하지만 POST가 더 안정적
- **결과 수**: 보통 페이지당 20~30개 결과 반환
- **속도 제한**: 짧은 시간에 많은 요청 시 CAPTCHA 페이지 반환 가능 — 병렬 요청은 2~3개로 제한 권장
