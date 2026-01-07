---
description: 현재 프로젝트의 에이전트를 GitHub claude-toolkit에 업로드하고 마켓플레이스에 등록
arguments:
  - name: agent_name
    description: 업로드할 에이전트 이름
    required: true
allowed-tools: Read, Bash, Glob, Write
---

# /publish-agent $ARGUMENTS.agent_name

현재 프로젝트의 에이전트를 GitHub claude-toolkit 저장소에 업로드하고 마켓플레이스에 등록한다.

## 실행 단계

### 1. 로컬 에이전트 파일 확인

검색 순서:
1. `.claude/agents/$ARGUMENTS.agent_name.md` (로컬 프로젝트)
2. `~/.claude/agents/$ARGUMENTS.agent_name.md` (전역)

없으면 에러 출력하고 종료.

### 2. 에이전트 메타데이터 추출

에이전트 파일의 frontmatter에서 추출:
- `name`: 에이전트 이름
- `description`: 설명
- `allowed-tools`: 허용된 도구 목록

### 3. plugin.json 생성

업로드 전 `/tmp/agent-plugin.json` 생성:

```json
{
  "name": "<agent_name>",
  "description": "<frontmatter의 description>",
  "version": "1.0.0"
}
```

### 4. GitHub에 에이전트 파일 업로드

에이전트 파일과 plugin.json을 GitHub에 업로드:

```bash
# 에이전트 파일 업로드
gh api repos/conewarrior/siot-claude-toolkit/contents/agents/$ARGUMENTS.agent_name/$ARGUMENTS.agent_name.md \
  -X PUT \
  -f message="Add/Update agent: $ARGUMENTS.agent_name" \
  -f content="$(base64 < <agent_file_path>)" \
  -f sha="$(gh api repos/conewarrior/siot-claude-toolkit/contents/agents/$ARGUMENTS.agent_name/$ARGUMENTS.agent_name.md --jq '.sha' 2>/dev/null || echo '')"

# plugin.json 업로드
gh api repos/conewarrior/siot-claude-toolkit/contents/agents/$ARGUMENTS.agent_name/.claude-plugin/plugin.json \
  -X PUT \
  -f message="Add/Update plugin.json for agent: $ARGUMENTS.agent_name" \
  -f content="$(base64 < /tmp/agent-plugin.json)" \
  -f sha="$(gh api repos/conewarrior/siot-claude-toolkit/contents/agents/$ARGUMENTS.agent_name/.claude-plugin/plugin.json --jq '.sha' 2>/dev/null || echo '')"
```

### 5. marketplace.json에 플러그인 등록

GitHub에서 `.claude-plugin/marketplace.json`을 가져와서:

1. `plugins` 배열에 해당 에이전트가 있는지 확인
2. 없으면 새 항목 추가:
   ```json
   {
     "name": "<agent_name>",
     "source": "./agents/<agent_name>",
     "description": "<description>",
     "version": "1.0.0"
   }
   ```
3. 있으면 version bump (patch)
4. 업데이트된 marketplace.json을 GitHub에 업로드

### 6. 결과 출력

```
✅ 에이전트 업로드 완료: $ARGUMENTS.agent_name

📁 업로드된 파일:
   agents/$ARGUMENTS.agent_name/$ARGUMENTS.agent_name.md
   agents/$ARGUMENTS.agent_name/.claude-plugin/plugin.json

📦 마켓플레이스 등록: $ARGUMENTS.agent_name (v1.0.0)

🔗 GitHub: https://github.com/conewarrior/siot-claude-toolkit/tree/main/agents/$ARGUMENTS.agent_name

💡 설치: /install-agent $ARGUMENTS.agent_name
```

## 예시

```bash
/publish-agent prd-to-dev
/publish-agent reviewer
/publish-agent codify
```
