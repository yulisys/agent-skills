# agent-skills

yulisys가 만든 Claude Code 스킬을 모아 배포하는 플러그인 마켓플레이스(플러그인 목록을 담은 저장소)입니다. 리포 루트가 마켓플레이스 루트이고, 스킬 하나를 플러그인 하나로 등록합니다.

마켓플레이스 이름은 `yulisys-skills`입니다. 리포 이름인 `agent-skills`는 Anthropic 공식 마켓플레이스가 쓰는 [예약 이름](https://code.claude.com/docs/en/plugins/marketplace-reference.md#reserved-names)이라 마켓플레이스 이름으로 쓸 수 없습니다.

## 스킬 목록

| 플러그인 | 호출 이름 | 하는 일 |
| :- | :- | :- |
| `unslop-ko` | `/unslop-ko:unslop-ko` | 한국어 문장에서 번역투와 AI 티를 걷어낸다 |

## unslop-ko

한국어 문서를 새로 쓰거나 고치거나 검토할 때 번역투와 AI 티를 찾아 우리말 구조로 고칩니다. 뜻, 숫자, 이름, 링크, 코드 표기는 바꾸지 않고, 원문에 없는 사실을 새로 만들지 않습니다.

고치는 유형은 9가지입니다.

| 구분 | 유형 |
| :- | :- |
| 번역투 | 사물 주어, "A는 B이다" 명사문, 직역 어휘와 "두다" 반복, 풀이 없는 영어 단어 |
| AI 티 | 굵은 제목과 콜론 목록, 기호로 문장 대신하기, 의문형 소제목, "X가 아니라 Y" 대비 구문, 상투 표현과 군더더기 |

고친 예입니다.

| 고치기 전 | 고친 뒤 |
| :- | :- |
| 그림 3은 전체 흐름을 보여 준다. | 전체 흐름은 그림 3에 나타냈다. |
| 빌드 → 테스트 → 배포 순서로 실행한다. | 빌드하고 테스트한 뒤 배포한다. |
| 로그 보관: 접속 로그는 30일 동안 보관한다. | 접속 로그는 30일 동안 보관한다. |

Claude Code 세션에서 스킬을 직접 호출할 수 있습니다. 요청에 "번역투", "AI 티" 같은 말을 넣으면 Claude가 알아서 스킬을 골라 쓰기도 합니다.

```text
/unslop-ko:unslop-ko docs/guide.md를 검토해 줘
```

```text
이 릴리스 노트에서 AI 티 나는 문장을 고쳐 줘
```

## 설치

이 리포는 private이라 git 인증이 먼저 필요합니다. Claude Code는 별도 토큰 없이 내 컴퓨터의 git 인증을 그대로 씁니다.

```bash
gh auth login
```

```bash
gh auth setup-git
```

GitHub SSH 키가 없는 컴퓨터에서는 HTTPS로 받도록 설정합니다.

```bash
export CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1
```

마켓플레이스를 등록하고 플러그인을 설치합니다.

```bash
claude plugin marketplace add yulisys/agent-skills
```

```bash
claude plugin install unslop-ko@yulisys-skills
```

새 버전은 아래 명령으로 받습니다. 자동으로 받으려면 `/plugin` 화면의 Marketplaces 탭에서 `yulisys-skills`를 고르고 Enable auto-update를 켭니다.

```bash
claude plugin marketplace update yulisys-skills
```

## 스킬 추가

1. `skills/<skill-name>/SKILL.md`를 작성합니다. 작성 규칙은 공식 문서 [Skills](https://code.claude.com/docs/en/skills)를 따릅니다.
2. `.claude-plugin/marketplace.json`의 `plugins` 배열에 항목을 추가합니다.

   ```json
   {
     "name": "review-diff",
     "source": "./",
     "description": "커밋하지 않은 변경을 요약하고 위험한 부분을 표시한다",
     "author": { "name": "yulisys" },
     "skills": ["./skills/review-diff"]
   }
   ```

3. 위 스킬 목록 표에 한 줄을 추가합니다.
4. 검증하고, 스킬 폴더를 직접 읽어 들여 동작을 확인합니다.

   ```bash
   claude plugin validate .
   ```

   ```bash
   claude --plugin-dir ./skills/<skill-name>
   ```

5. 커밋하고 push합니다.

항목을 쓸 때 지킬 점입니다.

- `source`는 리포 루트인 `"./"`로 씁니다.
- `skills`에는 이 플러그인에 넣을 스킬 폴더만 적습니다. 빠뜨리면 `skills/` 아래 모든 스킬이 이 플러그인에 들어갑니다([근거](https://code.claude.com/docs/en/plugins/manifest-reference.md#how-entry-fields-combine-with-plugin-json)).
- 리포 루트에 `.claude-plugin/plugin.json`을 만들지 않습니다. 만들면 모든 항목이 이 파일을 플러그인 설정으로 읽어서 항목별 구분이 깨집니다.
- `version`은 쓰지 않습니다. 그러면 사용자는 커밋이 바뀔 때마다 새 버전을 받습니다. `version`을 쓰면 릴리스할 때마다 값을 올려야 하고, 올리지 않으면 사용자가 새 커밋을 받지 못합니다.
