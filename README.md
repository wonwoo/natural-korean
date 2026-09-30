# natural-korean

사람이 쓴 것처럼 읽히는 한국어로 답하게 하는 Claude Code, Codex 플러그인입니다.

AI가 쓴 한국어는 문법이 맞아도 딱딱하게 읽힙니다. 영어 문장의 뼈대에 한국어 단어만 채워 넣기 때문입니다. 이 플러그인은 문체 지침을 넣어서 결론을 첫 문장에 쓰고, 동작은 동사로 풀고, 번역투와 빈말을 덜어 내게 합니다. 지침 전문은 [`output-styles/dev.md`](output-styles/dev.md)와 [`output-styles/general.md`](output-styles/general.md)에 있습니다.

## 예시

빈 폴더에서 "main 브랜치에 실수로 커밋 두 개 했어. 아직 푸시는 안 했고 이거 feature 브랜치로 옮기고 싶은데 어떻게 해?"라고 물었을 때 실제로 나온 답입니다.

### Claude Code

output style을 기본값으로 둔 경우:

> 새 브랜치를 현재 위치에 만들고, main만 두 커밋 전으로 되돌리면 돼요. 커밋은 새 브랜치에 그대로 남아 있습니다.
>
> ```bash
> git status                     # 커밋 안 한 변경이 있으면 먼저 git stash
> git log --oneline -3           # 맨 위 두 개가 옮길 커밋인지 확인
>
> git branch feature/my-work     # 지금 커밋(두 커밋 포함)을 가리키는 브랜치 생성
> git reset --hard HEAD~2        # main을 두 커밋 전으로 되돌림
> git switch feature/my-work     # feature 브랜치로 이동
> ```
>
> - **데이터 손실 주의:** `reset --hard`는 커밋하지 않은 작업 트리 변경을 지워요. 그래서 `git status`로 먼저 확인해야 합니다. 두 커밋은 `feature/my-work`에 남아 있어서 사라지지 않아요.
> - main이 커밋 전에 원격과 같았다면 `HEAD~2` 대신 `git reset --hard origin/main`을 써도 됩니다. 몇 개를 되돌릴지 셀 필요가 없어서 더 안전해요.
> - feature 브랜치가 이미 있다면 `git branch` 대신 `git switch feature/xxx && git cherry-pick <커밋1> <커밋2>`로 커밋을 가져오세요. 그다음 main으로 돌아가서 `reset`을 하면 됩니다.
>
> 지금 폴더는 git 저장소가 아니라서 직접 실행해 보지는 않았어요.

`natural-korean:dev`를 적용한 경우:

> 현재 위치에 feature 브랜치를 만들고 main만 두 커밋 뒤로 되돌리면 됩니다. 커밋은 새 브랜치에 그대로 남습니다.
>
> ```bash
> git status                  # 커밋하지 않은 변경이 없는지 먼저 확인
> git branch feature/xxx      # 지금 HEAD(커밋 두 개 포함)를 가리키는 브랜치 생성
> git reset --hard HEAD~2     # main을 두 커밋 전으로 되돌림
> git switch feature/xxx
> ```
>
> `reset --hard`를 실행하면 커밋하지 않은 변경이 사라집니다. `git status`에 변경이 보이면 `git stash`로 먼저 치워 두세요. 실수한 커밋이 정확히 두 개인지 헷갈리면 `HEAD~2` 대신 `git reset --hard origin/main`을 써서 main을 원격과 똑같이 맞춰도 됩니다.
>
> 옮겨 갈 feature 브랜치가 이미 있다면 그 브랜치로 가서 `git cherry-pick main~2..main`으로 두 커밋을 가져온 뒤 main에서 위의 `reset`을 하면 됩니다.

### Codex

플러그인을 설치하지 않은 경우:

> 두 커밋이 `main`의 **가장 최근 커밋 두 개**이고 아직 푸시하지 않았다면, 저장소에서 아래 순서로 실행하면 됩니다.
>
> ```bash
> git status
> git switch -c feature/my-work
> git switch main
> git reset --hard HEAD~2
> ```
>
> 첫 명령에서 작업 중인 변경 사항이 없는지 확인하세요. `git switch -c`가 현재 두 커밋을 담은 feature 브랜치를 만들고, 마지막 명령이 로컬 `main`만 두 커밋 전으로 돌립니다. `--hard`는 커밋하지 않은 변경 사항도 버리므로 작업 트리가 깨끗할 때 실행해야 합니다.
>
> 이후 `git switch feature/my-work`로 돌아가 두 커밋을 확인하면 됩니다. 현재 작업 공간은 Git 저장소가 아니어서 실제 브랜치 상태는 확인하지 못했습니다.

플러그인을 설치한 경우:

> 새 `feature` 브랜치를 만들 거라면, 현재 `main`에서 두 커밋을 브랜치에 남긴 뒤 `main`만 되돌리면 됩니다.
>
> ```bash
> git status --short          # 출력이 없어야 합니다
> git switch -c feature
> git switch main
> git reset --hard HEAD~2
> ```
>
> `reset --hard`는 커밋하지 않은 변경 사항도 지웁니다. 첫 명령에서 파일이 보이면 먼저 커밋하거나 `git stash`로 보관하세요. 끝나면 `git log -2 --oneline feature`로 두 커밋이 옮겨졌는지 확인할 수 있습니다.
>
> 이미 `feature` 브랜치가 있다면 방법이 달라집니다. 그 경우에는 두 커밋을 `cherry-pick`해야 합니다.

Claude Code에서는 "돼요", "지워요"와 "있습니다"가 섞이던 말투가 합니다체로 맞춰지고 "데이터 손실 주의:" 같은 굵은 머리말 목록이 문단으로 바뀌었습니다. 길이는 767자에서 555자로 줄었습니다. Codex에서는 굵은 글씨가 빠졌고 길이는 430자에서 397자로 줄었습니다.

## 스타일

| 스타일 | 쓰는 곳 | Claude Code 코딩 지침 |
|---|---|---|
| `natural-korean:dev` | 개발 작업 | 유지 |
| `natural-korean:general` | 일반 대화, 글쓰기 | 유지하지 않음 |

`general`은 Claude Code의 코딩 지침을 빼므로 코드를 다루지 않는 대화나 글쓰기에 맞습니다. 예시도 계약서, 문장 교정 같은 일상적인 내용으로 바꿔 두었습니다. Codex에는 `dev`만 적용됩니다.

## 설치

Claude Code 2.1.285와 codex-cli 0.159.2에서 확인했습니다.

### Claude Code

터미널에서 마켓플레이스를 추가하고 플러그인을 설치합니다.

```sh
claude plugin marketplace add wonwoo/natural-korean
claude plugin install natural-korean@natural-korean
```

Claude Code 안에서는 같은 일을 슬래시 명령으로 합니다.

```
/plugin marketplace add wonwoo/natural-korean
/plugin install natural-korean@natural-korean
```

설치한 뒤 Claude Code를 다시 시작합니다. 이미 열려 있는 세션에서는 `/reload-plugins`를 실행해도 됩니다. 이 단계를 건너뛰면 새 스타일을 찾지 못해서 `Unknown output style`이 나옵니다. 그다음 스타일을 고릅니다.

```
/output-style natural-korean:dev
```

`/config`의 Output style 메뉴에서 골라도 됩니다. 인자 없이 `/output-style`을 실행하면 지금 쓰는 스타일 옆에 `(current)`가 붙습니다.

`/output-style`과 `/config` 중 어느 쪽으로 골라도, 스타일은 지금 작업 중인 프로젝트의 `.claude/settings.local.json`에 저장됩니다. 이 파일이 없으면 새로 만듭니다. 그래서 다른 프로젝트에는 적용되지 않고, 프로젝트마다 다시 골라야 합니다.

모든 프로젝트에서 쓰려면 `~/.claude/settings.json`에 직접 적습니다. 파일에 있던 내용은 그대로 두고 `outputStyle` 한 줄만 추가합니다. 값은 대소문자를 구분합니다.

```json
{
  "outputStyle": "natural-korean:dev"
}
```

프로젝트의 `.claude/settings.local.json`에도 `outputStyle`이 있으면 그 프로젝트에서는 이 값이 `~/.claude/settings.json`보다 우선합니다.

### Codex

```sh
codex plugin marketplace add wonwoo/natural-korean
codex plugin add natural-korean@natural-korean
```

설치한 뒤 Codex를 열고 훅을 허용합니다. Codex는 사용자가 허용하기 전에는 플러그인 훅을 실행하지 않으므로, 이 단계를 건너뛰면 지침이 들어가지 않습니다.

```
/hooks
```

목록에서 natural-korean 훅을 확인하고 허용합니다. 그다음에 새로 여는 세션부터 지침이 들어갑니다. 허용하지 않은 훅은 `codex exec`에서도 건너뜁니다.

훅 명령은 [`.codex-plugin/hooks.json`](.codex-plugin/hooks.json)에 있습니다. `output-styles/dev.md`에서 frontmatter를 뺀 본문을 출력하기만 합니다.

Windows에서는 이 훅이 동작하지 않습니다. Codex는 Windows에서 훅 명령을 PowerShell로 실행하는데, 이 훅은 sh 문법과 `awk`를 쓰기 때문입니다. Windows에서는 아래 방법을 씁니다.

#### 플러그인을 쓸 수 없을 때

Codex IDE 확장은 플러그인을 지원하지 않습니다. IDE 확장을 쓰거나 Windows를 쓴다면 지침 본문을 전역 `AGENTS.md`에 붙여 넣습니다.

```sh
mkdir -p ~/.codex
curl -fsSL https://raw.githubusercontent.com/wonwoo/natural-korean/HEAD/output-styles/dev.md \
  | awk '{sub(/\r$/,"")} NR==1&&/^---$/{f=1;next} f&&/^---$/{f=0;next} !f' >> ~/.codex/AGENTS.md
```

Windows에서는 [`output-styles/dev.md`](output-styles/dev.md)를 열어 맨 위 `---` 두 줄과 그 사이를 뺀 나머지를 `%USERPROFILE%\.codex\AGENTS.md`에 붙여 넣습니다.

`CODEX_HOME`을 따로 정해 두었다면 `~/.codex` 대신 그 폴더에 넣습니다. 같은 폴더에 `AGENTS.override.md`가 있으면 Codex는 `AGENTS.md`를 읽지 않으니, 그때는 `AGENTS.override.md`에 붙여 넣습니다.

이렇게 넣은 지침은 자동으로 업데이트되지 않습니다. 플러그인과 같이 쓰면 지침이 두 번 들어가므로 둘 중 하나만 씁니다.

## 업데이트

Claude Code에서는 마켓플레이스를 갱신한 뒤 플러그인을 업데이트합니다. Claude Code를 다시 시작하면 새 지침이 적용됩니다.

```sh
claude plugin marketplace update natural-korean
claude plugin update natural-korean@natural-korean
```

Codex에서는 마켓플레이스를 갱신한 뒤 플러그인을 다시 설치합니다. 훅 정의가 바뀐 버전이면 `/hooks`에서 다시 허용해야 합니다. 지침 본문만 바뀐 버전은 다시 허용하지 않아도 됩니다.

```sh
codex plugin marketplace upgrade natural-korean
codex plugin add natural-korean@natural-korean
```

## 삭제

Claude Code에서는 플러그인과 마켓플레이스를 지웁니다. 플러그인만 지우면 마켓플레이스 등록은 `~/.claude/settings.json`에 남습니다.

```sh
claude plugin uninstall natural-korean@natural-korean
claude plugin marketplace remove natural-korean
```

`outputStyle`에 적어 둔 값도 지웁니다. 전역에 적었다면 `~/.claude/settings.json`에 있습니다. `/output-style`이나 `/config`로 골랐다면 그 프로젝트마다 `.claude/settings.local.json`에 있습니다. 값을 남겨 두면 없는 스타일을 가리키게 되어 기본 스타일로 동작합니다.

Codex에서도 플러그인과 마켓플레이스를 지웁니다. `AGENTS.md`에 붙여 넣었다면 그 부분도 지웁니다.

```sh
codex plugin remove natural-korean@natural-korean
codex plugin marketplace remove natural-korean
```

## 동작 방식

```
natural-korean/
├── .claude-plugin/
│   ├── marketplace.json    두 도구가 함께 읽는 마켓플레이스
│   └── plugin.json         Claude Code용 매니페스트
├── .codex-plugin/
│   ├── plugin.json         Codex용 매니페스트
│   └── hooks.json          Codex SessionStart 훅
└── output-styles/
    ├── dev.md
    └── general.md
```

Claude Code는 `.claude-plugin/plugin.json`을 읽고 `output-styles/`의 두 파일을 스타일로 등록합니다. 이 플러그인에는 Claude Code가 실행할 훅이 없고, Claude Code는 `.codex-plugin/`을 읽지 않습니다. 그래서 Claude Code에서 지침이 두 번 들어가는 일은 없습니다.

Codex에는 output style 기능이 없어서 훅으로 지침을 넣습니다. Codex는 `.codex-plugin/plugin.json`을 먼저 읽고, 거기 적힌 `.codex-plugin/hooks.json`의 SessionStart 훅을 실행합니다. 훅은 세션을 새로 시작할 때와 `/clear`, compact 뒤에 `dev.md` 본문을 developer 메시지로 넣습니다. 재개한 세션에는 이전 기록에 지침이 이미 들어 있어서 다시 넣지 않습니다.

두 도구가 같은 `dev.md`를 읽으므로 지침 본문은 한 곳에서만 관리합니다. 마켓플레이스 파일도 `.claude-plugin/marketplace.json` 하나를 두 도구가 함께 씁니다.

## 알아 둘 점

- 지침은 약 18KB이고, 두 도구 모두 요청마다 컨텍스트에 들어갑니다. Claude Code는 시스템 프롬프트로 넣고, Codex는 세션을 시작할 때 대화 기록에 넣습니다.
- Claude Code의 서브에이전트에는 지침이 적용되지 않습니다. 그래서 서브에이전트가 써 온 보고는 메인 에이전트가 이 지침에 맞게 다시 써서 전하도록 지침에 적어 두었습니다.
- Codex는 CLI와 ChatGPT 데스크톱 앱에서 확인했습니다.
- 모델이 지침을 매번 지키지는 않습니다.

## 지침 고치기

`output-styles/dev.md`와 `output-styles/general.md`를 고치면 됩니다. Codex도 `dev.md`를 그대로 읽으므로 따로 옮길 파일은 없습니다.

Claude Code에서는 이렇게 확인합니다.

```sh
claude plugin validate .claude-plugin/plugin.json --strict
claude plugin validate .claude-plugin/marketplace.json --strict
claude --plugin-dir .    # 설치하지 않고 이 폴더를 플러그인으로 불러옵니다
```

Codex에서는 평소 설정과 섞이지 않도록 임시 `CODEX_HOME`을 만들어 확인합니다. Codex는 설치할 때 플러그인을 캐시로 복사하므로, `dev.md`를 고친 뒤에는 `codex plugin add`를 다시 실행해야 바뀐 내용이 들어갑니다.

```sh
export CODEX_HOME="$(mktemp -d)"
ln -s ~/.codex/auth.json "$CODEX_HOME/auth.json"    # 다시 로그인하지 않도록 기존 로그인 정보에 연결합니다
codex plugin marketplace add .
codex plugin add natural-korean@natural-korean
codex exec --dangerously-bypass-hook-trust --skip-git-repo-check "developer 지침에 한국어 문체 지침이 있는지 알려 줘"
```

`--dangerously-bypass-hook-trust`는 `/hooks`에서 허용하는 절차를 건너뛰는 옵션입니다. 내용을 직접 확인한 훅을 시험할 때만 씁니다.

배포할 때는 두 `plugin.json`의 `version`을 함께 올립니다. Claude Code는 버전이 그대로면 이미 설치한 사람에게 새 내용을 내려보내지 않습니다.

## 라이선스

[MIT](LICENSE)
