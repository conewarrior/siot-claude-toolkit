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
