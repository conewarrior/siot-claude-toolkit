# claude-toolkit 마켓플레이스 전환 작업

## 목표

claude-toolkit을 Claude Code 플러그인 마켓플레이스로 등록하고, publish 커맨드에서 marketplace.json을 자동 업데이트하도록 만들기.

## 현재 구조

```
claude-toolkit/
├── commands/
│   ├── cleanup-ui.md
│   ├── global-hooks.md
│   ├── install-agent.md
│   ├── install-command.md
│   ├── install-skill.md
│   └── register-skill.md
├── skills/
│   ├── canvas-design/
│   ├── doc-coauthoring/
│   ├── frontend-design/
│   ├── pdf/
│   └── pptx/
├── agents/
├── hooks/
└── bootstrap/
```

## 작업 1: 마켓플레이스 구조 추가

### 1.1 `.claude-plugin/marketplace.json` 생성

```json
{
  "name": "claude-toolkit-marketplace",
  "owner": {
    "name": "conewarrior"
  },
  "plugins": [
    {
      "name": "claude-toolkit",
      "source": ".",
      "description": "Commands and skills collection for Claude Code",
      "version": "1.0.0"
    }
  ]
}
```

### 1.2 `.claude-plugin/plugin.json` 생성

```json
{
  "name": "claude-toolkit",
  "description": "Collection of commands and skills for Claude Code workflow",
  "version": "1.0.0"
}
```

### 1.3 디렉토리 생성

```bash
mkdir -p .claude-plugin
```

## 작업 2: `/publish-command` 커맨드 생성

`commands/publish-command.md` 파일 생성:

```markdown
---
description: 현재 프로젝트의 커맨드를 GitHub claude-toolkit에 업로드하고 marketplace.json 버전 업데이트
arguments:
  - name: command_name
    description: 업로드할 커맨드 이름
    required: true
allowed-tools: Read, Bash, Glob
---

# /publish-command $ARGUMENTS.command_name

현재 프로젝트의 커맨드를 GitHub claude-toolkit 저장소에 업로드한다.

## 실행 단계

### 1. 로컬 파일 확인

`.claude/commands/$ARGUMENTS.command_name.md` 파일이 존재하는지 확인.
없으면 에러 출력하고 종료.

### 2. GitHub에 파일 업로드

```bash
# 파일 내용을 base64로 인코딩해서 업로드
gh api repos/conewarrior/claude-toolkit/contents/commands/$ARGUMENTS.command_name.md \
  -X PUT \
  -f message="Add/Update command: $ARGUMENTS.command_name" \
  -f content="$(base64 < .claude/commands/$ARGUMENTS.command_name.md)" \
  -f sha="$(gh api repos/conewarrior/claude-toolkit/contents/commands/$ARGUMENTS.command_name.md --jq '.sha' 2>/dev/null || echo '')"
```

주의: sha는 파일이 이미 존재할 때만 필요. 새 파일이면 sha 파라미터 생략.

### 3. marketplace.json 버전 업데이트

현재 marketplace.json을 가져와서 version을 patch bump하고 다시 업로드:

```bash
# 현재 버전 가져오기
gh api repos/conewarrior/claude-toolkit/contents/.claude-plugin/marketplace.json \
  --jq '.content' | base64 -d > /tmp/marketplace.json

# 버전 업데이트 (1.0.0 -> 1.0.1)
# jq로 version 필드 업데이트

# 다시 업로드
gh api repos/conewarrior/claude-toolkit/contents/.claude-plugin/marketplace.json \
  -X PUT \
  -f message="Bump version for $ARGUMENTS.command_name" \
  -f content="$(base64 < /tmp/marketplace.json)" \
  -f sha="$(gh api repos/conewarrior/claude-toolkit/contents/.claude-plugin/marketplace.json --jq '.sha')"
```

### 4. 결과 출력

```
✅ 커맨드 업로드 완료: $ARGUMENTS.command_name

📁 업로드된 파일:
   commands/$ARGUMENTS.command_name.md

📦 마켓플레이스 버전: 1.0.0 → 1.0.1

🔗 GitHub: https://github.com/conewarrior/claude-toolkit/blob/main/commands/$ARGUMENTS.command_name.md
```

## 예시

```bash
/publish-command cleanup-ui
```
```

## 작업 3: `/publish-skill` 커맨드 생성

`commands/publish-skill.md` 파일 생성:

```markdown
---
description: 현재 프로젝트의 스킬을 GitHub claude-toolkit에 업로드하고 marketplace.json 버전 업데이트
arguments:
  - name: skill_name
    description: 업로드할 스킬 이름
    required: true
allowed-tools: Read, Bash, Glob
---

# /publish-skill $ARGUMENTS.skill_name

현재 프로젝트의 스킬 폴더를 GitHub claude-toolkit 저장소에 업로드한다.

## 실행 단계

### 1. 로컬 스킬 폴더 확인

`.claude/skills/$ARGUMENTS.skill_name/` 폴더가 존재하는지 확인.
최소한 `SKILL.md` 파일이 있어야 함.

### 2. 스킬 폴더의 모든 파일 업로드

폴더 내 모든 파일을 순회하면서 GitHub에 업로드:

```bash
# 각 파일에 대해
for file in .claude/skills/$ARGUMENTS.skill_name/*; do
  filename=$(basename "$file")
  gh api repos/conewarrior/claude-toolkit/contents/skills/$ARGUMENTS.skill_name/$filename \
    -X PUT \
    -f message="Add/Update skill file: $ARGUMENTS.skill_name/$filename" \
    -f content="$(base64 < "$file")" \
    -f sha="$(gh api repos/conewarrior/claude-toolkit/contents/skills/$ARGUMENTS.skill_name/$filename --jq '.sha' 2>/dev/null || echo '')"
done
```

### 3. marketplace.json 버전 업데이트

publish-command와 동일하게 버전 bump.

### 4. 결과 출력

```
✅ 스킬 업로드 완료: $ARGUMENTS.skill_name

📁 업로드된 파일:
   skills/$ARGUMENTS.skill_name/SKILL.md
   skills/$ARGUMENTS.skill_name/reference.md
   ...

📦 마켓플레이스 버전: 1.0.1 → 1.0.2

🔗 GitHub: https://github.com/conewarrior/claude-toolkit/tree/main/skills/$ARGUMENTS.skill_name
```

## 예시

```bash
/publish-skill pdf
```
```

## 작업 4: Git 커밋 & 푸시

모든 파일 생성 후:

```bash
git add .
git commit -m "Add marketplace structure and publish commands"
git push origin main
```

## 사용법 (완료 후)

### 마켓플레이스 등록 (다른 컴퓨터에서)

```bash
/plugin marketplace add conewarrior/claude-toolkit
```

### 플러그인 목록 확인

```bash
/plugin
# Discover 탭에서 claude-toolkit 보임
```

### 전체 설치

```bash
/plugin install claude-toolkit@claude-toolkit-marketplace
```

### 개별 설치 (기존 방식 유지)

```bash
/install-command cleanup-ui
/install-skill pdf
```

## 두 방식 비교

| | 마켓플레이스 | GitHub API (기존) |
|---|---|---|
| 목록 확인 | `/plugin` UI에서 보임 | 저장소 직접 확인 |
| 설치 | `/plugin install` (전체) | `/install-command` (개별) |
| 업데이트 | 버전 관리 | 항상 최신 |
| 용도 | 새 환경 전체 셋업 | 개별 커맨드 빠른 설치 |
