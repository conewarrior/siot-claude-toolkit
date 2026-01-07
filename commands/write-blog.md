---
description: 블로그 글 초안 작성. 경험/주제를 설명하면 siot 블로그 형식에 맞는 MDX 초안을 생성합니다.
arguments:
  - name: topic
    description: 블로그 글 주제 또는 경험 요약
    required: true
allowed-tools: Read, Write, Skill
---

# /write-blog $ARGUMENTS.topic

## 실행 단계

### 1. blog-writer 스킬 참조

`.claude/skills/blog-writer/SKILL.md`를 읽어 블로그 형식과 톤을 파악한다.

### 2. 주제 분석

사용자가 입력한 `$ARGUMENTS.topic`을 분석:
- 핵심 경험/문제는 무엇인가?
- Before/After가 있는가?
- 어떤 인사이트가 있는가?

### 3. 글 구조 설계

blog-writer 스킬의 구조를 따라:
1. 도입부 (Hook) - 문제/불편함
2. 기존 방식 (Before)
3. 개선 과정 (Journey)
4. 새로운 방식 (After)
5. 마무리 - 인사이트

### 4. 초안 작성

`docs/content/blog/[slug].mdx` 형식으로 초안 작성:
- Frontmatter 포함 (title, description, date, category, published)
- 이미지 캡쳐 포인트 `[📸 설명]` 마커 삽입
- 해요체 사용

### 5. 사용자 확인

작성된 초안을 보여주고:
- 수정할 부분 확인
- 슬러그(파일명) 확인
- 저장 여부 확인

### 6. 파일 저장

확인 후 `docs/content/blog/[slug].mdx`에 저장

## 출력 형식

```
📝 블로그 초안: [제목]

---
[Frontmatter]
---

[본문]

---

📸 캡쳐 포인트:
1. [설명] - [어느 섹션에서]
2. ...

💾 저장하시겠어요? (Y/n)
   → docs/content/blog/[slug].mdx
```

## 예시

```bash
/write-blog GitHub API로 Claude Code 커맨드를 관리하게 된 경험. 처음엔 저장소 클론해서 썼는데 불편해서 API로 바꿈
```
