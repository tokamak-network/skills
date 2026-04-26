# grok-cli 설정 레퍼런스

## API 키 설정

API 키는 [xAI Console](https://console.x.ai/)에서 발급받습니다.

키 탐색 우선순위:

| 우선순위 | 방식 | 설정 방법 |
|---------|------|----------|
| 1 | **OS 키스토어 ⭐ 권장** | `grok-cli auth save` |
| 2 | 환경변수 | `export XAI_API_KEY="xai-..."` |
| 3 | CLI 옵션 | `--api-key <KEY>` |

키스토어 사용을 권장합니다. 셸 히스토리나 환경변수에 키가 남지 않아 가장 안전합니다. macOS에서는 Keychain, Linux에서는 Secret Service(GNOME Keyring 등)를 사용합니다.

### 키 관리 명령어

```bash
# 키스토어에 저장 (입력 내용은 화면에 표시되지 않음)
grok-cli auth save

# 현재 설정 확인
grok-cli auth show
# 소스:  키스토어
# API 키: xai-****...**Dk2f

# 키스토어에서 삭제
grok-cli auth delete
```

---

## 기타 CLI 옵션

| 옵션 | 설명 | 기본값 |
|------|------|--------|
| `--model <MODEL>` | 사용할 모델 변경 | `grok-4-1-fast-reasoning` |
| `--no-stream` | 스트리밍 비활성화 (전체 응답을 한 번에 출력) | 스트리밍 활성 |
| `--verbose` | 디버그: API 원본 응답 출력 | 비활성 |
| `--api-key <KEY>` | API 키 직접 지정 | — |

```bash
# 모델 변경
grok-cli ask "복잡한 추론 문제" --model grok-4-1

# 동기 모드
grok-cli ask "짧은 질문" --no-stream

# API 원본 응답 확인 (디버그)
grok-cli ask "테스트 질문" --verbose
```

---

## 전체 명령어 요약

```
grok-cli ask [OPTIONS] [PROMPT]         Grok에게 질문한다
grok-cli auth save                      API 키를 키스토어에 저장한다
grok-cli auth show                      현재 API 키 소스를 확인한다
grok-cli auth delete                    키스토어에서 API 키를 삭제한다
```

---

## 설치

```bash
cargo install --path .
# 또는
cargo build --release
./target/release/grok-cli --help
```
