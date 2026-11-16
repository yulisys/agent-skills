# agent-skills

Claude Code용 Agent Skill을 관리하는 **플러그인 마켓플레이스** 리포지토리입니다.

마켓플레이스 이름은 `yulisys-skills`입니다. 리포 이름 `agent-skills`는 Anthropic 공식 마켓플레이스의 [예약 이름](https://code.claude.com/docs/en/plugins/marketplace-reference.md#reserved-names)이라 마켓플레이스 이름으로 쓸 수 없습니다.
스킬은 플러그인 단위로 묶어 `plugins/` 아래에 두고, `.claude-plugin/marketplace.json`에 등록해서 배포합니다.

## 플러그인 목록

| 플러그인 | 스킬 | 설명 |
| :- | :- | :- |
| `unslop-ko` | `/unslop-ko:unslop-ko` | 번역투와 AI 티를 걷어내 자연스러운 한국어 문장으로 쓰거나 고치거나 검토한다 |

## 디렉터리 구조

```text
agent-skills/
├── .claude-plugin/
│   └── marketplace.json        # 마켓플레이스 카탈로그 (플러그인 목록)
├── plugins/
│   └── <plugin-name>/
│       ├── .claude-plugin/
│       │   └── plugin.json     # 플러그인 매니페스트 (이 폴더에는 plugin.json만 둔다)
│       └── skills/
│           └── <skill-name>/
│               ├── SKILL.md    # 필수
│               ├── reference.md  # 선택: 필요할 때만 읽히는 상세 문서
│               └── scripts/      # 선택: 실행용 스크립트
└── docs/
    └── SKILL_AUTHORING.md      # 스킬 작성 가이드
```

## 설치 (사용하는 쪽)

이 리포는 **private**입니다. Claude Code는 자체 git 토큰이 없고, 사용자 머신의 git 인증을 그대로 씁니다.
따라서 먼저 GitHub 인증을 저장해 둬야 합니다.

```bash
gh auth login
```

```bash
gh auth setup-git
```

SSH 키가 없는 머신이라면 HTTPS를 강제합니다.

```bash
export CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1
```

마켓플레이스를 등록하고 플러그인을 설치합니다.

```bash
claude plugin marketplace add yulisys/agent-skills
```

```bash
claude plugin install <plugin-name>@yulisys-skills
```

예: `claude plugin install unslop-ko@yulisys-skills`

설치된 스킬은 `/<plugin-name>:<skill-name>` 형태로 호출됩니다.

업데이트는 `/plugin` → **Marketplaces** → `yulisys-skills` → **Enable auto-update**로 켜거나, 수동으로 받습니다.

```bash
claude plugin marketplace update yulisys-skills
```

## 새 스킬 추가 (관리하는 쪽)

1. 플러그인이 없으면 `plugins/<plugin-name>/.claude-plugin/plugin.json`을 만든다.
2. `plugins/<plugin-name>/skills/<skill-name>/SKILL.md`를 작성한다. → [작성 가이드](docs/SKILL_AUTHORING.md)
3. 새 플러그인이면 `.claude-plugin/marketplace.json`의 `plugins` 배열에 항목을 추가한다.
4. 검증한다.

   ```bash
   claude plugin validate .
   ```

5. 로컬에서 실제로 로드해 본다.

   ```bash
   claude --plugin-dir ./plugins/<plugin-name>
   ```

6. 커밋 후 push한다.

### 플러그인 등록 예시

`plugins/<plugin-name>/.claude-plugin/plugin.json`

```json
{
  "name": "dev-workflow",
  "description": "개발 워크플로 스킬 모음",
  "author": { "name": "yulisys" }
}
```

`.claude-plugin/marketplace.json`의 `plugins` 배열

```json
{
  "name": "dev-workflow",
  "source": "./plugins/dev-workflow",
  "description": "개발 워크플로 스킬 모음"
}
```

규칙:

- 마켓플레이스 항목의 `name`과 `plugin.json`의 `name`은 **반드시 같게** 한다. 다르면 설치 시 `Plugin "<name>" not found` 오류가 난다.
- `source`는 리포 루트 기준 상대 경로로 쓴다. `..`은 쓸 수 없다.
- 플러그인 `name`은 공백 없이 kebab-case로 쓴다.

## 버전 정책

이 리포는 `version` 필드를 **생략**하는 방식을 기본으로 합니다. 이 경우 사용자는 커밋 SHA 기준으로 변경을 받습니다.
`version`을 쓰기로 했다면 릴리스마다 반드시 올려야 합니다. 올리지 않으면 사용자는 새 커밋을 받지 못합니다.
`version`은 `plugin.json`과 `marketplace.json` 중 한 곳에만 둡니다.

## 참고 문서

- [Create a marketplace](https://code.claude.com/docs/en/plugin-marketplaces)
- [Host and maintain a marketplace](https://code.claude.com/docs/en/plugins/host-marketplace.md)
- [Create a Claude Code plugin](https://code.claude.com/docs/en/plugins/create.md)
- [Skills](https://code.claude.com/docs/en/skills)
