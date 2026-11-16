# 스킬 작성 가이드

이 문서는 `plugins/<plugin-name>/skills/<skill-name>/SKILL.md`를 작성하는 규칙입니다.
근거는 Claude Code 공식 문서 [Skills](https://code.claude.com/docs/en/skills)입니다.

## 1. 위치와 이름

- 스킬 1개 = 디렉터리 1개. 디렉터리 안에 `SKILL.md`가 반드시 있어야 한다.
- `skills/` 디렉터리는 플러그인 루트에 둔다. `.claude-plugin/` 안에 두면 로드되지 않는다.
- 디렉터리 이름은 kebab-case로 쓴다. `name`을 생략하면 디렉터리 이름이 스킬 이름이 된다.
- 플러그인 스킬은 `/<plugin-name>:<skill-name>`으로 호출된다.

## 2. Frontmatter

`---`는 파일의 **첫 줄**이어야 합니다. 그렇지 않으면 frontmatter를 읽지 않습니다.
YAML 파싱이 실패해도 스킬은 로드되지만, 모든 필드가 비어 있는 상태가 됩니다.

```markdown
---
name: review-diff
description: 커밋하지 않은 변경을 요약하고 위험한 부분을 표시한다. 사용자가 변경 내용, 커밋 메시지, diff 리뷰를 요청할 때 사용한다.
---

(본문)
```

| 필드 | 필수 | 용도 |
| :- | :- | :- |
| `name` | 아니오 | `/` 메뉴에 보이는 이름. 기본값은 디렉터리 이름 |
| `description` | 권장 | 무엇을 하고 **언제 쓰는지**. Claude가 이 값으로 스킬 로드 여부를 판단한다 |
| `when_to_use` | 아니오 | 트리거 문구, 예시 요청. `description` 뒤에 붙는다 |
| `argument-hint` | 아니오 | 자동완성 힌트. 예: `[issue-number]` |
| `disable-model-invocation` | 아니오 | `true`면 Claude가 스스로 호출하지 않는다. 사용자가 직접 호출할 때만 실행 |
| `user-invocable` | 아니오 | `false`면 `/` 메뉴에서 숨기고 Claude만 호출한다 |
| `allowed-tools` | 아니오 | 스킬 실행 턴에서 권한 확인 없이 쓸 수 있는 도구 |

필드 이름은 하이픈까지 정확히 일치해야 합니다. 모르는 필드는 오류 없이 무시됩니다.

### 금지 사항

- `name`에 `<`, `>`, 제어 문자, zero-width space를 쓰지 않는다.
- `anthropic-skills`, `anthropic-skills:`로 시작하는 이름은 예약어다.

## 3. description 작성법

`description` + `when_to_use`는 스킬 목록에서 **1,536자**에서 잘립니다. 핵심 용도를 맨 앞에 둡니다.

- 첫 문장: 무엇을 하는가.
- 둘째 문장: "~할 때 사용한다." 형태로 트리거 조건을 쓴다.
- 사용자가 실제로 칠 법한 표현을 넣는다.

좋은 예:

> Spring Boot 프로젝트의 Flyway 마이그레이션 파일을 생성한다. 사용자가 DB 스키마 변경, 테이블 추가, 컬럼 변경을 요청할 때 사용한다.

나쁜 예:

> DB 관련 도우미.

## 4. 본문 작성법

- `SKILL.md`는 **500줄 이하**로 유지한다.
- 하는 방법과 이유를 서술하지 말고, **무엇을 할지**를 지시한다.
- 스킬 본문은 로드된 뒤 계속 컨텍스트에 남는다. 모든 줄이 반복 토큰 비용이다.

## 5. 점진적 공개 (Progressive disclosure)

긴 참고 자료는 별도 파일로 분리하고, `SKILL.md`에서 링크합니다. 분리한 파일은 필요할 때만 읽힙니다.

```text
my-skill/
├── SKILL.md        # 필수: 개요와 안내
├── reference.md    # 상세 API 문서 (필요할 때 로드)
├── examples.md     # 사용 예시 (필요할 때 로드)
└── scripts/
    └── helper.py   # 실행용 스크립트 (로드하지 않고 실행)
```

`SKILL.md` 안에서 이렇게 연결합니다.

```markdown
## Additional resources

- 전체 API 상세는 [reference.md](reference.md)를 본다.
- 사용 예시는 [examples.md](examples.md)를 본다.
```

## 6. 호출 방식 선택

| 스킬 성격 | 설정 |
| :- | :- |
| 배포, 데이터 삭제처럼 부작용이 있는 작업 | `disable-model-invocation: true` |
| 배경 지식처럼 명령으로 실행할 의미가 없는 것 | `user-invocable: false` |
| 그 외 | 기본값 |

## 7. 제출 전 체크리스트

- [ ] `SKILL.md` 첫 줄이 `---`이다.
- [ ] `description`이 "무엇을 + 언제"를 담고 있다.
- [ ] `SKILL.md`가 500줄 이하다.
- [ ] 긴 참고 자료는 별도 파일로 분리했다.
- [ ] 부작용이 있는 스킬은 `disable-model-invocation: true`를 설정했다.
- [ ] `claude plugin validate .`가 통과한다.
- [ ] `claude --plugin-dir ./plugins/<plugin-name>`으로 실제 호출해 봤다.
