# claude-toolkit 레포지토리 구조 설정 가이드

이 문서는 claude-toolkit 레포지토리의 초기 구조를 설정하기 위한 가이드입니다.

## 목적

Claude Code용 스킬, 커맨드, 서브에이전트를 중앙에서 관리하고, 필요한 프로젝트에 선택적으로 설치할 수 있는 라이브러리입니다.

## 디렉토리 구조

다음 구조로 디렉토리와 파일을 생성해주세요:

```
claude-toolkit/
├── skills/                 # 스킬들
│   └── .gitkeep
├── commands/               # 커맨드들
│   └── .gitkeep
├── agents/                 # 서브에이전트들
│   └── .gitkeep
├── bootstrap/              # 최초 설정용 installer 커맨드들
│   ├── install-skill.md
│   ├── install-command.md
│   └── install-agent.md
└── README.md
```

## 생성할 파일들

### 1. bootstrap/install-skill.md

```markdown
---
description: GitHub에서 스킬을 현재 프로젝트에 설치
arguments:
  - name: skill_name
    description: 설치할 스킬 이름
    required: true
---

GitHub 저장소 https://github.com/conewarrior/claude-toolkit 의
skills/$ARGUMENTS.skill_name 폴더를 현재 프로젝트의 .claude/skills/에 설치해줘.

1. .claude/skills/ 디렉토리 없으면 생성
2. 해당 스킬 폴더 다운로드해서 복사
3. 완료 후 SKILL.md의 name과 description 보여줘
```

### 2. bootstrap/install-command.md

```markdown
---
description: GitHub에서 커맨드를 현재 프로젝트에 설치
arguments:
  - name: command_name
    description: 설치할 커맨드 이름
    required: true
---

GitHub 저장소 https://github.com/conewarrior/claude-toolkit 의
commands/$ARGUMENTS.command_name.md 파일을 현재 프로젝트의 .claude/commands/에 설치해줘.
```

### 3. bootstrap/install-agent.md

```markdown
---
description: GitHub에서 서브에이전트를 현재 프로젝트에 설치
arguments:
  - name: agent_name
    description: 설치할 에이전트 이름
    required: true
---

GitHub 저장소 https://github.com/conewarrior/claude-toolkit 의
agents/$ARGUMENTS.agent_name.md 파일을 현재 프로젝트의 .claude/agents/에 설치해줘.
```

### 4. README.md

```markdown
# claude-toolkit

Claude Code용 스킬, 커맨드, 서브에이전트 라이브러리

## 최초 설정

bootstrap 폴더의 파일들을 ~/.claude/commands/에 복사:

\`\`\`bash
mkdir -p ~/.claude/commands
curl -o ~/.claude/commands/install-skill.md \
  https://raw.githubusercontent.com/conewarrior/claude-toolkit/main/bootstrap/install-skill.md
curl -o ~/.claude/commands/install-command.md \
  https://raw.githubusercontent.com/conewarrior/claude-toolkit/main/bootstrap/install-command.md
curl -o ~/.claude/commands/install-agent.md \
  https://raw.githubusercontent.com/conewarrior/claude-toolkit/main/bootstrap/install-agent.md
\`\`\`

## 사용법

\`\`\`
/install-skill pdf
/install-command review
/install-agent code-reviewer
\`\`\`

## 디렉토리 구조

- `skills/` - Claude Code 스킬들 (SKILL.md + 리소스)
- `commands/` - 슬래시 커맨드들 (.md 파일)
- `agents/` - 서브에이전트들 (.md 파일)
- `bootstrap/` - 글로벌 설치용 installer 커맨드들
```

## 작업 순서

1. `skills/`, `commands/`, `agents/`, `bootstrap/` 디렉토리 생성
2. 각 디렉토리에 `.gitkeep` 파일 생성 (bootstrap 제외)
3. `bootstrap/install-skill.md` 생성
4. `bootstrap/install-command.md` 생성
5. `bootstrap/install-agent.md` 생성
6. `README.md` 생성
7. 완료 후 git commit & push

## 완료 확인

- [ ] skills/ 디렉토리 존재
- [ ] commands/ 디렉토리 존재
- [ ] agents/ 디렉토리 존재
- [ ] bootstrap/install-skill.md 존재
- [ ] bootstrap/install-command.md 존재
- [ ] bootstrap/install-agent.md 존재
- [ ] README.md 존재
