---
description: 현재 프로젝트의 커맨드를 GitHub claude-toolkit에 업로드하고 마켓플레이스에 등록
arguments:
  - name: command_name
    description: 업로드할 커맨드 이름
    required: true
allowed-tools: Read, Bash, Glob
---

# /publish-command $ARGUMENTS.command_name

현재 프로젝트의 커맨드를 GitHub claude-toolkit 저장소의 toolkit-commands 플러그인에 업로드한다.

## 실행 단계

### 1. 로컬 파일 확인

`.claude/commands/$ARGUMENTS.command_name.md` 파일이 존재하는지 확인.
없으면 에러 출력하고 종료.

### 2. GitHub에 커맨드 파일 업로드

커맨드는 `toolkit-commands` 플러그인의 `commands/` 폴더에 업로드:

```bash
gh api repos/conewarrior/siot-claude-toolkit/contents/commands/commands/$ARGUMENTS.command_name.md \
  -X PUT \
  -f message="Add/Update command: $ARGUMENTS.command_name" \
  -f content="$(base64 < .claude/commands/$ARGUMENTS.command_name.md)" \
  -f sha="$(gh api repos/conewarrior/siot-claude-toolkit/contents/commands/commands/$ARGUMENTS.command_name.md --jq '.sha' 2>/dev/null || echo '')"
```

### 3. toolkit-commands 플러그인 버전 업데이트

GitHub에서 `commands/.claude-plugin/plugin.json`을 가져와서 version bump:

```bash
# 현재 plugin.json 가져오기
gh api repos/conewarrior/siot-claude-toolkit/contents/commands/.claude-plugin/plugin.json \
  --jq '.content' | base64 -d > /tmp/plugin.json

# 버전 업데이트 (patch bump)
# jq로 version 필드 업데이트

# 다시 업로드
gh api repos/conewarrior/siot-claude-toolkit/contents/commands/.claude-plugin/plugin.json \
  -X PUT \
  -f message="Bump toolkit-commands version for $ARGUMENTS.command_name" \
  -f content="$(base64 < /tmp/plugin.json)" \
  -f sha="$(gh api repos/conewarrior/siot-claude-toolkit/contents/commands/.claude-plugin/plugin.json --jq '.sha')"
```

### 4. marketplace.json의 toolkit-commands 버전 동기화

GitHub에서 `.claude-plugin/marketplace.json`을 가져와서:

1. `plugins` 배열에서 `toolkit-commands` 항목 찾기
2. version을 plugin.json과 동일하게 업데이트
3. 업데이트된 marketplace.json을 GitHub에 업로드

### 5. 결과 출력

```
✅ 커맨드 업로드 완료: $ARGUMENTS.command_name

📁 업로드된 파일:
   commands/commands/$ARGUMENTS.command_name.md

📦 toolkit-commands 버전: 1.0.0 → 1.0.1

🔗 GitHub: https://github.com/conewarrior/siot-claude-toolkit/blob/main/commands/commands/$ARGUMENTS.command_name.md

💡 설치: /plugin install toolkit-commands@claude-toolkit-marketplace
```

## 예시

```bash
/publish-command cleanup-ui
```
