---
description: GitHub에서 스킬 Hook 설정을 글로벌에 설치
allowed-tools: Read, Edit, Write, Bash, WebFetch
---

# /install-hooks

GitHub 저장소에서 미리 설정된 Hook과 권한을 다운로드하고 글로벌 설정 `~/.claude/settings.json`에 병합한다.

## 실행 단계

### 1. hooks.json 다운로드

GitHub 저장소 `https://github.com/conewarrior/claude-toolkit`의 `bootstrap/hooks.json` 파일을 다운로드한다.

```
https://raw.githubusercontent.com/conewarrior/claude-toolkit/main/bootstrap/hooks.json
```

### 2. 기존 settings.json 확인

`~/.claude/settings.json` 파일 확인:
- 없으면 hooks.json 내용을 그대로 사용
- 있으면 기존 내용 읽기

### 3. 설정 병합

기존 settings.json과 hooks.json을 병합:

**permissions.allow 병합:**
- hooks.json의 권한들을 기존 배열에 추가
- 중복은 제거

**hooks.UserPromptSubmit 병합:**
- hooks.json의 Hook들을 기존 배열에 추가
- 같은 matcher를 가진 Hook이 있으면 덮어쓰기

### 4. 저장

병합된 설정을 `~/.claude/settings.json`에 저장한다.

### 5. 결과 출력

완료 후 출력:
- 추가된 권한 목록
- 추가된 Hook 목록
- 변경 사항 요약

## 포함된 설정

### 권한 (permissions.allow)
- `Bash(git config:*)` - Git 설정
- `Bash(git add:*)` - Git 스테이징
- `Bash(git commit:*)` - Git 커밋
- `Bash(git push:*)` - Git 푸시
- `Skill(canvas-design)` - 캔버스 디자인 스킬
- `Skill(doc-coauthoring)` - 문서 공동 작성 스킬
- `Skill(frontend-design)` - 프론트엔드 디자인 스킬
- `Skill(pdf)` - PDF 처리 스킬
- `Skill(pptx)` - PowerPoint 처리 스킬

### Hook (UserPromptSubmit)

| 스킬 | Matcher 키워드 |
|------|----------------|
| canvas-design | poster, art, design, canvas, visual, 포스터, 아트, 디자인, 캔버스 |
| doc-coauthoring | doc, documentation, proposal, spec, RFC, PRD, 문서, 제안서, 기획, draft, 스펙, 드래프트 |
| frontend-design | web, page, component, React, HTML, CSS, UI, landing, dashboard, 웹, 페이지, 프론트엔드, 컴포넌트, 랜딩, 대시보드 |
| pdf | pdf, PDF |
| pptx | pptx, presentation, slide, PowerPoint, 발표, 프레젠테이션, 슬라이드, 파워포인트 |

## 예시

```bash
/install-hooks
```

출력:
```
✅ Hook 설정 설치 완료

🔑 추가된 권한:
   - Bash(git config:*)
   - Bash(git add:*)
   - Bash(git commit:*)
   - Bash(git push:*)
   - Skill(canvas-design)
   - Skill(doc-coauthoring)
   - Skill(frontend-design)
   - Skill(pdf)
   - Skill(pptx)

🪝 추가된 Hook:
   - canvas-design: poster|art|design|...
   - doc-coauthoring: doc|documentation|...
   - frontend-design: web|page|component|...
   - pdf: pdf|PDF
   - pptx: pptx|presentation|...

📄 ~/.claude/settings.json 업데이트됨
```

## 주의사항

- 기존 ~/.claude/settings.json의 다른 설정(MCP 서버 등)은 유지됨
- 같은 matcher를 가진 기존 Hook은 덮어씌워짐
- 스킬 파일이 없어도 Hook은 설치됨 (스킬은 별도로 `/install-skill`로 설치)
- 글로벌 설정이므로 모든 프로젝트에 적용됨
