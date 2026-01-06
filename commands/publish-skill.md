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
  gh api repos/conewarrior/siot-claude-toolkit/contents/skills/$ARGUMENTS.skill_name/$filename \
    -X PUT \
    -f message="Add/Update skill file: $ARGUMENTS.skill_name/$filename" \
    -f content="$(base64 < "$file")" \
    -f sha="$(gh api repos/conewarrior/siot-claude-toolkit/contents/skills/$ARGUMENTS.skill_name/$filename --jq '.sha' 2>/dev/null || echo '')"
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

🔗 GitHub: https://github.com/conewarrior/siot-claude-toolkit/tree/main/skills/$ARGUMENTS.skill_name
```

## 예시

```bash
/publish-skill pdf
```
