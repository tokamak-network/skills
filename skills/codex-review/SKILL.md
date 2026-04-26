---
name: codex-review
description: Use when you want Codex to independently review code, a plan, or a document.
  Triggers: "codex로 리뷰해줘", "코덱스 리뷰", "codex review",
  "이 계획 리뷰해줘", "이 문서 리뷰해줘", "plan 검토해줘", "spec 검토해줘",
  "get codex to review this", "independent review", "second opinion from codex",
  "리뷰 준비해줘", "prep review", "prepare for codex".
  With file argument: /codex-review path/to/file.md reviews that file directly.
  Without argument: reviews git diff (code review mode).
  --topic <name>: specify session slug explicitly.
---

# Codex Review

## Overview

코드(git diff), 계획서(PLAN.md), 또는 문서(spec, design doc)를 Codex CLI로 리뷰하고
결과를 `docs/codex-review/<slug>-codex-review.md`에 저장한 뒤 HIGH 이슈를 요약한다.

context 파일(`<slug>-review-context.md`)이 없으면 Claude가 대화에서 직접 준비한 뒤 리뷰를 이어서 실행한다.

## Step 1: 인수 파싱 및 Slug 결정

`$ARGUMENTS`에서 플래그와 파일 경로를 파싱한다:

```bash
TOPIC_FLAG=""
FILE_ARG=""

# --topic 파싱
if echo "$ARGUMENTS" | grep -q -- "--topic"; then
  TOPIC_FLAG=$(echo "$ARGUMENTS" | sed 's/.*--topic[[:space:]]\+\([^[:space:]]*\).*/\1/')
fi

# 파일 경로 파싱 (--topic ... 제거 후 남은 인수)
FILE_ARG=$(echo "$ARGUMENTS" | sed 's/--topic[[:space:]]\+[^[:space:]]*//' | sed 's/^[[:space:]]*//' | sed 's/[[:space:]]*$//')

REVIEW_DIR="docs/codex-review"
mkdir -p "$REVIEW_DIR"
```

### Slug 결정

**`--topic` 지정 시:**

```bash
SLUG=$(echo "$TOPIC_FLAG" | tr '[:upper:]' '[:lower:]' | sed 's|[^a-z0-9-]|-|g' | sed 's/^-\+//' | sed 's/-\+$//')
```

**`--topic` 미지정 시: Claude가 대화 주제에서 slug 파생**

대화에서 작업한 내용을 파악해 **2–4단어 kebab-case** slug를 직접 결정한다.

규칙:
- 무엇을 만들었는지/고쳤는지를 핵심 단어로 표현
- 소문자, 영어, 숫자, 하이픈만 사용 (`[a-z0-9-]`)
- 너무 일반적인 단어 단독 사용 금지 (`fix`, `update`, `change` 단독 X)
- 예시: `usdc-token-support`, `electron-e2e-spec`, `auth-middleware-fix`, `crosstrade-deploy-flow`

```bash
# 빈 slug 방어
if [ -z "$SLUG" ]; then
  SLUG=$(date +%Y-%m-%d-%H%M)
fi

CONTEXT_FILE="${REVIEW_DIR}/${SLUG}-review-context.md"
OUTPUT_FILE="${REVIEW_DIR}/${SLUG}-codex-review.md"
```

## Step 2: Prerequisites 확인

```bash
# Codex CLI 설치 확인
command -v codex >/dev/null 2>&1 || echo "ERROR: codex not installed. Run: npm install -g @openai/codex"
```

오류가 있으면 해당 메시지를 출력하고 중단한다.

## Step 3: Context 준비

파일 인수 없는 코드 리뷰 모드에서만 실행한다.

`$CONTEXT_FILE`이 이미 존재하면 → 재사용한다.

존재하지 않으면 → **Claude가 대화 컨텍스트에서 직접 준비해 파일을 생성한다:**

1. **Task Background** — 무엇을 왜 만들었는가
2. **Design Decisions** — 왜 이 방식을 택했는가 (고려한 대안 포함)
3. **Changed Files** — `git diff --name-only` 실행 결과
4. **Review Focus** — 불확실하거나 특히 확인받고 싶은 부분

생성되는 `$CONTEXT_FILE` 구조:

```markdown
## Task Background
{작업 목적 — 무엇을 왜 만들었나}

## Design Decisions
{핵심 설계 결정과 근거. 택하지 않은 대안도 포함}

## Changed Files
{git diff --name-only 결과}

## Review Focus
{특히 확인받고 싶은 부분, 확신이 없는 부분}
```

## Step 4: 타입 감지

`$FILE_ARG`가 비어있으면 **코드 리뷰 모드**, 있으면 파일명 패턴으로 타입 자동 감지:

| 패턴 | 타입 | 프롬프트 포커스 |
|------|------|----------------|
| `*-PLAN.md`, `*PLAN*.md` | plan | 완결성, 의존성, 실현가능성 |
| `*-design.md`, `*spec*`, `*SPEC*` | doc | 정확성, 일관성, 누락 섹션 |
| `*ROADMAP*`, `*REQUIREMENTS*` | doc | 커버리지, 우선순위, 범위 |
| 그 외 `.md` | doc | 일반 문서 리뷰 |

디렉토리가 지정된 경우 해당 디렉토리의 모든 `.md` 파일을 수집한다:
```bash
# 디렉토리인 경우
find "$FILE_ARG" -name "*.md" | sort | xargs cat
# 파일인 경우
cat "$FILE_ARG"
```

## Step 5: Codex 호출

**코드 리뷰 모드 (FILE_ARG 없음):**

```bash
DIFF=$(git diff HEAD 2>/dev/null || git diff 2>/dev/null)
CONTEXT=$(cat "$CONTEXT_FILE")

codex exec --skip-git-repo-check "
${CONTEXT}

## Actual Changes (git diff)
\`\`\`diff
${DIFF}
\`\`\`

Review this implementation. Provide structured feedback:

1. **Summary** — One paragraph assessment of the overall implementation
2. **Issues**
   - HIGH: bugs, security issues, data loss risks, broken logic
   - MEDIUM: edge cases, missing error handling, performance concerns
   - LOW: style, naming, minor improvements
3. **Suggestions** — Specific actionable improvements with code examples where helpful
4. **Overall Risk** — LOW / MEDIUM / HIGH with one-sentence justification

Output structured markdown.
" > /tmp/codex-review-output.md 2>&1
```

**계획 리뷰 모드 (plan 타입):**

```bash
CONTENT=$(cat "$FILE_ARG")

codex exec --skip-git-repo-check "
${CONTENT}

Review this implementation plan. Provide structured feedback:

1. **Completeness** — Are all requirements covered? Missing steps?
2. **Feasibility** — Are the steps realistic? Any hidden complexity?
3. **Dependencies** — Are task dependencies correct? Anything missing?
4. **Risks** — What could go wrong? HIGH/MEDIUM/LOW
5. **Suggestions** — Specific improvements with concrete examples

Output structured markdown.
" > /tmp/codex-review-output.md 2>&1
```

**문서 리뷰 모드 (doc 타입):**

```bash
CONTENT=$(cat "$FILE_ARG")

codex exec --skip-git-repo-check "
${CONTENT}

Review this document. Provide structured feedback:

1. **Accuracy** — Any factually incorrect or outdated information?
2. **Clarity** — Ambiguous or confusing sections?
3. **Completeness** — Missing sections or important omissions?
4. **Consistency** — Internal contradictions?
5. **Suggestions** — Specific improvements

Output structured markdown.
" > /tmp/codex-review-output.md 2>&1
```

## Step 6: codex-review 파일 저장

```bash
TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
{
  echo "# Codex Review — ${TIMESTAMP}"
  echo ""
  cat /tmp/codex-review-output.md
} > "$OUTPUT_FILE"
rm /tmp/codex-review-output.md
```

## Step 7: 완료 후 요약

`$OUTPUT_FILE`을 읽고 다음 형식으로 요약:

```
${OUTPUT_FILE} 저장 완료. (slug: ${SLUG})

HIGH 이슈:
- {이슈 1}
- {이슈 2}
(없으면 "HIGH 이슈 없음")

전체 리뷰: ${OUTPUT_FILE}
```

## 오류 처리

| 상황 | 대응 |
|------|------|
| `codex` 미설치 | `npm install -g @openai/codex` 안내 후 중단 |
| 지정 파일 없음 | 파일 경로 확인 요청 |
| `git diff` 비어있음 | 경고 출력 후 계속 (커밋된 변경사항일 수 있음) |
| Codex 호출 실패 | `/tmp/codex-review-output.md` 내용 확인 안내 |
