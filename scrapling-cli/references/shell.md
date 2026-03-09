# Scrapling Interactive Shell 상세 레퍼런스

`scrapling shell`로 시작하는 IPython 기반 인터랙티브 웹 스크래핑 환경의 상세 사용법이다.

## 목차
1. [시작 옵션](#시작-옵션)
2. [내장 단축 명령](#내장-단축-명령)
3. [자동 페이지 관리](#자동-페이지-관리)
4. [curl 명령 변환](#curl-명령-변환)
5. [페이지 시각화](#페이지-시각화)
6. [IPython 기능 활용](#ipython-기능-활용)
7. [실전 워크플로우 예제](#실전-워크플로우-예제)

---

## 시작 옵션

```bash
scrapling shell                  # 기본 실행
scrapling shell --loglevel info  # 로그 레벨 (debug/info/warning/error/critical/fatal)
scrapling shell -c "CODE"        # 코드 실행 후 종료 (스크립팅용)
```

`-c` 모드 예시:
```bash
# 제목만 출력하고 종료
scrapling shell -c "get('https://example.com'); print(page.css('title::text').get())"

# 파이프라인에 활용
scrapling shell -c "get('https://api.example.com/data'); print(page.json())" | jq '.results'
```

## 내장 단축 명령

import 없이 바로 사용 가능한 함수:

| 단축 명령 | 실제 동작 | 비고 |
|-----------|----------|------|
| `get(url, **kw)` | `Fetcher.get(url, **kw)` | HTTP GET |
| `post(url, **kw)` | `Fetcher.post(url, **kw)` | HTTP POST |
| `put(url, **kw)` | `Fetcher.put(url, **kw)` | HTTP PUT |
| `delete(url, **kw)` | `Fetcher.delete(url, **kw)` | HTTP DELETE |
| `fetch(url, **kw)` | `DynamicFetcher.fetch(url, **kw)` | 브라우저 자동화 |
| `stealthy_fetch(url, **kw)` | `StealthyFetcher.fetch(url, **kw)` | 스텔스 브라우저 |
| `view(page)` | 브라우저에서 HTML 열기 | 디버깅용 |
| `uncurl(cmd)` | curl 명령 → Request 객체 | 파싱만 |
| `curl2fetcher(cmd)` | curl 명령 → 즉시 실행 | 실행+page 갱신 |

자동 import된 클래스: `Fetcher`, `AsyncFetcher`, `DynamicFetcher`, `StealthyFetcher`, `Selector`.

단축 명령에 전달하는 kwargs는 각 Fetcher의 인자와 동일하다:
```python
>>> get('https://example.com', impersonate='chrome', timeout=60)
>>> stealthy_fetch('https://protected.com', solve_cloudflare=True, block_webrtc=True)
>>> fetch('https://spa.com', network_idle=True, wait_selector='.loaded')
```

## 자동 페이지 관리

요청할 때마다 결과가 자동으로 추적된다:

```python
>>> get('https://example.com')
>>> page.url           # 마지막 응답 (= response)
'https://example.com'
>>> response.status    # page의 별칭
200

>>> get('https://site2.com')
>>> get('https://site3.com')
>>> len(pages)         # 최근 5개 이력 (Selectors 객체)
3
>>> pages[0].url       # 첫 번째
'https://example.com'
>>> pages[-1].url      # 가장 최근
'https://site3.com'
```

`page`와 `response`는 항상 마지막 요청의 `Response` 객체를 가리킨다. CSS/XPath/find_all 등 모든 파싱 메서드를 바로 사용 가능.

## curl 명령 변환

브라우저 DevTools → Network 탭 → 요청 우클릭 → "Copy as cURL"로 복사한 명령을 활용.

### uncurl: 파싱만

```python
>>> curl_cmd = '''curl 'https://api.example.com/data' \
...   -X POST \
...   -H 'Content-Type: application/json' \
...   -H 'Authorization: Bearer tok123' \
...   -d '{"key": "value"}' '''

>>> req = uncurl(curl_cmd)
>>> req.method
'post'
>>> req.url
'https://api.example.com/data'
>>> req.headers
{'Content-Type': 'application/json', 'Authorization': 'Bearer tok123'}
>>> req.data
'{"key": "value"}'
```

### curl2fetcher: 파싱 + 즉시 실행

```python
>>> curl2fetcher(curl_cmd)
>>> page.status
200
>>> page.json()
{'key': 'value'}
```

실행 후 `page`가 자동 갱신된다.

## 페이지 시각화

```python
>>> get('https://quotes.toscrape.com')
>>> view(page)   # 시스템 기본 브라우저에서 HTML 파일로 열기
```

셀렉터가 올바른 요소를 잡고 있는지 시각적으로 확인할 때 유용하다.

## IPython 기능 활용

```python
>>> %time get('https://example.com')        # 실행 시간 측정
>>> %timeit -n1 get('https://example.com')  # 반복 벤치마크
>>> %history                                 # 전체 명령 이력
>>> %save scraper.py 1-15                    # 명령 1~15를 파일로 저장

>>> page.c<TAB>         # 탭 자동완성 (css, cookies, ...)
>>> page.css?           # 메서드 도움말
>>> Fetcher.get??       # 소스 코드 보기
```

## 실전 워크플로우 예제

### 셀렉터 탐색 → 스크래핑 코드화

```python
>>> get('https://news.ycombinator.com')
>>> page.css('.titleline>a')[:3]             # 요소 확인
>>> for s in page.css('.titleline>a')[:5]:
...     print(f"{s.text}: {s.attrib['href']}")

# 만족스러우면 %save로 저장하여 스크립트화
>>> %save hn_scraper.py 1-4
```

### API 응답 탐색

```python
>>> get('https://jsonplaceholder.typicode.com/posts/1')
>>> page.json()
>>> page.json()['title']

>>> post('https://jsonplaceholder.typicode.com/posts',
...      json={'title': 'New', 'body': 'Content', 'userId': 1})
>>> page.json()['id']
```

### Cloudflare 보호 사이트

```python
>>> stealthy_fetch('https://nopecha.com/demo/cloudflare', solve_cloudflare=True)
>>> page.status
>>> page.css('#padded_content a').getall()
```

### 세션 사용 (FetcherSession)

쉘 안에서 세션 객체를 직접 사용할 수도 있다:

```python
>>> from scrapling.fetchers import FetcherSession
>>> with FetcherSession(impersonate='chrome') as s:
...     p1 = s.get('https://example.com/login')
...     p2 = s.post('https://example.com/login', data={'user': 'me', 'pass': 'pw'})
...     p3 = s.get('https://example.com/dashboard')
...     print(p3.css('h1::text').get())
```
