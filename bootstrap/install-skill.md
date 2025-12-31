---
description: GitHub에서 스킬을 현재 프로젝트에 설치하고 hook까지 자동 등록
arguments:
  - name: skill_name
    description: 설치할 스킬 이름
    required: true
allowed-tools: Read, Edit, Write, Glob, Bash, WebFetch
---

# /install-skill $ARGUMENTS.skill_name

GitHub 저장소에서 스킬을 다운로드하고, 프로젝트에 hook과 권한을 자동 등록한다.

## 실행 단계

### 1. 스킬 다운로드

GitHub 저장소 `https://github.com/conewarrior/claude-toolkit`의 `skills/$ARGUMENTS.skill_name` 폴더를 현재 프로젝트의 `.claude/skills/`에 설치한다.

- `.claude/skills/` 디렉토리 없으면 생성
- GitHub API 또는 raw content로 해당 스킬 폴더의 모든 파일 다운로드
- `.claude/skills/$ARGUMENTS.skill_name/`에 저장

### 2. SKILL.md 읽기

설치된 `.claude/skills/$ARGUMENTS.skill_name/SKILL.md` 파일을 읽어서:
- `name` 필드 확인
- `description` 필드에서 키워드 추출

### 3. settings.local.json 확인/생성

`.claude/settings.local.json` 파일 확인:
- 없으면 기본 구조로 새로 생성:
  ```json
  {
    "permissions": { "allow": [] },
    "hooks": { "UserPromptSubmit": [] }
  }
  ```
- 있으면 기존 내용 읽기

### 4. 권한 등록

`permissions.allow` 배열에 `Skill($ARGUMENTS.skill_name)` 추가:
- 이미 있으면 스킵
- 없으면 추가

### 5. Hook 등록

SKILL.md의 description을 분석해서 적절한 matcher 키워드를 제안한다.

사용자에게 matcher 선택 요청:
- 제안된 키워드 정규식
- 빈 문자열 (모든 메시지에 반응)
- 사용자가 직접 입력

선택 후 `hooks.UserPromptSubmit` 배열에 추가:
```json
{
  "matcher": "(선택된 matcher)",
  "hooks": [
    {
      "type": "command",
      "command": "echo '🔔 Use Skill($ARGUMENTS.skill_name) - $ARGUMENTS.skill_name 스킬을 사용하세요'"
    }
  ]
}
```

### 6. 결과 출력

완료 후 출력:
- 설치된 스킬 이름과 description
- 추가된 권한
- 설정된 Hook matcher
- settings.local.json 변경 사항 요약

## 예시

```bash
/install-skill pdf
```

출력:
```
✅ 스킬 설치 및 등록 완료: pdf

📁 설치된 파일:
   .claude/skills/pdf/SKILL.md
   .claude/skills/pdf/forms.md
   .claude/skills/pdf/reference.md
   ...

📝 스킬 정보:
   name: pdf
   description: Comprehensive PDF manipulation toolkit...

🔑 추가된 권한:
   Skill(pdf)

🪝 등록된 Hook:
   matcher: "pdf|PDF|문서|추출|병합"

📄 settings.local.json 업데이트됨
```

## 주의사항

- GitHub에 해당 스킬이 존재해야 함
- 이미 설치된 스킬이면 덮어쓸지 사용자에게 확인
- Hook matcher는 사용자 확인 후 설정
