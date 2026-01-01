# Claude Toolkit 글로벌 설치 가이드

다른 프로젝트나 컴퓨터에서 Claude Code를 사용할 때, bootstrap 커맨드들을 글로벌로 설치하는 방법입니다.

## 설치 방법

### 1. 저장소 클론

```bash
git clone https://github.com/conewarrior/claude-toolkit.git
cd claude-toolkit
```

### 2. 글로벌 커맨드 폴더 생성

```bash
mkdir -p ~/.claude/commands
```

### 3. Bootstrap 커맨드 복사

```bash
cp bootstrap/install-skill.md ~/.claude/commands/
cp bootstrap/install-command.md ~/.claude/commands/
cp bootstrap/install-agent.md ~/.claude/commands/
cp bootstrap/register-skill.md ~/.claude/commands/
```

또는 한 번에:

```bash
cp bootstrap/*.md ~/.claude/commands/
```

## 설치 확인

Claude Code에서 다음 명령어들을 사용할 수 있습니다:

| 명령어 | 설명 |
|--------|------|
| `/install-skill <name>` | GitHub에서 스킬을 다운로드하고 프로젝트에 설치 |
| `/install-command <name>` | GitHub에서 커맨드를 다운로드하고 설치 |
| `/install-agent <name>` | GitHub에서 서브에이전트를 다운로드하고 설치 |
| `/install-hooks` | 미리 설정된 Hook과 권한을 프로젝트에 설치 |
| `/register-skill <name>` | 이미 설치된 스킬의 Hook과 권한을 등록 |

## 사용 예시

```bash
# PDF 스킬 설치
/install-skill pdf

# PPTX 스킬 설치
/install-skill pptx

# frontend-design 스킬 설치
/install-skill frontend-design

# 모든 스킬의 Hook과 권한 한 번에 설치
/install-hooks
```

## Hook 설정 설치

`/install-hooks` 명령어를 사용하면 미리 설정된 Hook과 권한을 한 번에 설치할 수 있습니다.

**포함된 권한:**
- Git 관련: `git config`, `git add`, `git commit`, `git push`
- 스킬: `canvas-design`, `doc-coauthoring`, `frontend-design`, `pdf`, `pptx`

**포함된 Hook (UserPromptSubmit):**

| 스킬 | Matcher 키워드 |
|------|----------------|
| canvas-design | poster, art, design, 포스터, 아트, 디자인... |
| doc-coauthoring | doc, proposal, spec, 문서, 제안서, 스펙... |
| frontend-design | web, page, React, 웹, 페이지, 프론트엔드... |
| pdf | pdf, PDF |
| pptx | pptx, presentation, 발표, 프레젠테이션... |

Hook이 설치되면 관련 키워드가 포함된 메시지를 입력할 때 해당 스킬 사용을 안내받습니다.

## 디렉토리 구조

```
~/.claude/
├── commands/           # 글로벌 커맨드 (모든 프로젝트에서 사용)
│   ├── install-skill.md
│   ├── install-command.md
│   ├── install-agent.md
│   └── register-skill.md
└── settings.json       # 글로벌 설정

프로젝트/
└── .claude/
    ├── commands/       # 프로젝트 커맨드
    ├── skills/         # 설치된 스킬
    ├── agents/         # 설치된 에이전트
    ├── settings.json   # 프로젝트 설정 (팀 공유)
    └── settings.local.json  # 프로젝트 로컬 설정 (개인)
```

## 업데이트

최신 버전으로 업데이트하려면:

```bash
cd claude-toolkit
git pull
cp bootstrap/*.md ~/.claude/commands/
```

## 문제 해결

### 커맨드가 인식되지 않는 경우

1. `~/.claude/commands/` 폴더가 있는지 확인
2. md 파일들이 올바르게 복사되었는지 확인
3. Claude Code를 재시작

### 스킬 설치 실패

- GitHub 저장소에 해당 스킬이 존재하는지 확인
- 네트워크 연결 확인
