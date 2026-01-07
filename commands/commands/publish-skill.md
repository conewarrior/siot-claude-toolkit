---
description: 현재 프로젝트의 스킬을 GitHub claude-toolkit에 업로드하고 마켓플레이스에 등록
arguments:
  - name: skill_name
    description: 업로드할 스킬 이름
    required: true
allowed-tools: Read, Bash, Glob, Write
---

# /publish-skill $ARGUMENTS.skill_name

현재 프로젝트의 스킬 폴더를 GitHub claude-toolkit 저장소에 업로드하고 마켓플레이스에 등록한다.

## 실행 단계

### 1. 로컬 스킬 폴더 확인

`.claude/skills/$ARGUMENTS.skill_name/` 폴더가 존재하는지 확인.
최소한 `SKILL.md` 파일이 있어야 함.

### 2. plugin.json 확인 및 생성

`.claude/skills/$ARGUMENTS.skill_name/.claude-plugin/plugin.json`이 없으면 자동 생성:

1. `SKILL.md`의 frontmatter에서 `name`, `description` 추출
2. `.claude-plugin/plugin.json` 생성:
   ```json
   {
     "name": "<skill_name>",
     "description": "<SKILL.md의 description>",
     "version": "1.0.0"
   }
   ```

### 3. 스킬 폴더의 모든 파일 업로드

폴더 내 모든 파일을 순회하면서 GitHub에 업로드 (`.claude-plugin/plugin.json` 포함):

```bash
# 일반 파일 업로드
for file in .claude/skills/$ARGUMENTS.skill_name/*; do
  [ -f "$file" ] || continue
  filename=$(basename "$file")
  gh api repos/conewarrior/siot-claude-toolkit/contents/skills/$ARGUMENTS.skill_name/$filename \
    -X PUT \
    -f message="Add/Update skill file: $ARGUMENTS.skill_name/$filename" \
    -f content="$(base64 < "$file")" \
    -f sha="$(gh api repos/conewarrior/siot-claude-toolkit/contents/skills/$ARGUMENTS.skill_name/$filename --jq '.sha' 2>/dev/null || echo '')"
done

# .claude-plugin/plugin.json 업로드
gh api repos/conewarrior/siot-claude-toolkit/contents/skills/$ARGUMENTS.skill_name/.claude-plugin/plugin.json \
  -X PUT \
  -f message="Add/Update plugin.json for $ARGUMENTS.skill_name" \
  -f content="$(base64 < .claude/skills/$ARGUMENTS.skill_name/.claude-plugin/plugin.json)" \
  -f sha="$(gh api repos/conewarrior/siot-claude-toolkit/contents/skills/$ARGUMENTS.skill_name/.claude-plugin/plugin.json --jq '.sha' 2>/dev/null || echo '')"
```

### 4. marketplace.json에 플러그인 등록

GitHub에서 `.claude-plugin/marketplace.json`을 가져와서:

1. `plugins` 배열에 해당 스킬이 있는지 확인
2. 없으면 새 항목 추가:
   ```json
   {
     "name": "<skill_name>",
     "source": "./skills/<skill_name>",
     "description": "<description>",
     "version": "1.0.0"
   }
   ```
3. 있으면 version bump (patch)
4. 업데이트된 marketplace.json을 GitHub에 업로드

### 5. 결과 출력

```
✅ 스킬 업로드 완료: $ARGUMENTS.skill_name

📁 업로드된 파일:
   skills/$ARGUMENTS.skill_name/SKILL.md
   skills/$ARGUMENTS.skill_name/.claude-plugin/plugin.json
   ...

📦 마켓플레이스 등록: $ARGUMENTS.skill_name (v1.0.0)

🔗 GitHub: https://github.com/conewarrior/siot-claude-toolkit/tree/main/skills/$ARGUMENTS.skill_name

💡 설치: /plugin install $ARGUMENTS.skill_name@claude-toolkit-marketplace
```

## 예시

```bash
/publish-skill pdf
```
