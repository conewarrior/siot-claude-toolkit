---
description: 새 스킬의 Hook과 권한을 settings.local.json에 자동 등록
allowed-tools: Read, Edit, Write, Glob, Bash
---

# /register-skill $ARGUMENTS

새로 만든 스킬을 프로젝트의 `.claude/settings.local.json`에 등록한다.

## 입력

- `$ARGUMENTS`: 스킬 이름 (예: `my-skill`)

## 실행 단계

### 1. 스킬 존재 확인

`.claude/skills/$ARGUMENTS/SKILL.md` 파일이 존재하는지 확인한다.
- 없으면: 에러 메시지 출력하고 종료
- 있으면: 다음 단계 진행

### 2. SKILL.md 읽기

SKILL.md 파일을 읽어서:
- `name` 필드 확인 (폴더명과 일치해야 함)
- `description` 필드에서 키워드 추출 (Hook matcher용)

### 3. settings.local.json 확인/생성

`.claude/settings.local.json` 파일이 있는지 확인한다.
- 없으면: 기본 구조로 새로 생성
- 있으면: 기존 내용 읽기

### 4. 권한 등록

`permissions.allow` 배열에 `Skill($ARGUMENTS)` 추가:
- 이미 있으면 스킵
- 없으면 추가

### 5. Hook 등록

`hooks.UserPromptSubmit` 배열에 새 Hook 추가:

```json
{
  "matcher": "",
  "hooks": [
    {
      "type": "command",
      "command": "echo '🔔 Use Skill($ARGUMENTS)'"
    }
  ]
}
```

### 6. matcher 키워드 제안

SKILL.md의 description을 분석해서 적절한 matcher 키워드를 제안한다.
사용자에게 물어보기:
- 빈 문자열 (모든 메시지에 반응)
- 제안된 키워드 정규식
- 사용자가 직접 입력

### 7. 결과 출력

완료 후 출력:
- 등록된 스킬 이름
- 추가된 권한
- 설정된 Hook matcher
- settings.local.json 변경 사항 요약

## 예시

```bash
/register-skill blog-writer
```

출력:
```
✅ 스킬 등록 완료: blog-writer

📁 확인된 파일:
   .claude/skills/blog-writer/SKILL.md

🔑 추가된 권한:
   Skill(blog-writer)

🪝 등록된 Hook:
   matcher: "블로그|포스트|글쓰기|blog|post"

📄 settings.local.json 업데이트됨
```

## 주의사항

- 스킬 폴더명과 SKILL.md의 `name` 필드가 일치해야 함
- 이미 등록된 스킬이면 중복 추가하지 않음
- Hook matcher는 사용자 확인 후 설정
