---
name: blog-writer
description: siot 블로그 글 작성 가이드. 튜토리얼/경험 공유 스타일의 기술 블로그 작성.
---

# Blog Writer Skill

siot 포트폴리오 블로그에 올릴 글을 작성하는 스킬입니다.

## 블로그 폴더 구조

각 블로그 글은 **폴더** 단위로 관리합니다:

```
docs/content/blog/
├── github-api-claude-commands/
│   ├── index.mdx          # 본문
│   └── images/            # 이미지 폴더
│       ├── before.png
│       └── after.png
├── jsdelivr-cdn-image-upload/
│   ├── index.mdx
│   └── images/
│       └── diagram.png
└── ...
```

**폴더명**: 영어 케밥케이스 (예: `my-new-post`)
**본문 파일**: `index.mdx`
**이미지 폴더**: `images/` (필요시 생성)

## Frontmatter

```yaml
---
title: "제목"
description: "한 줄 설명 (리스트에 표시됨)"
date: "YYYY-MM-DD"
category: "개발"  # "디자인" | "개발"
published: true
---
```

## 이미지 사용법

이미지는 해당 글의 `images/` 폴더에 저장하고, **상대 경로**로 참조:

```markdown
![설명](./images/screenshot.png)
```

커밋 시 pre-commit hook이 자동으로:
1. 로컬 이미지를 GitHub에 업로드
2. 경로를 jsDelivr CDN URL로 변환

## 글 구조 (튜토리얼 스타일)

### 1. 도입부 (Hook)
- 문제 상황 또는 불편함을 간결하게 서술
- 독자가 공감할 수 있는 상황 제시
- "~하다 보니 ~가 불편했다" 형식

### 2. 기존 방식 (Before)
- 처음에 어떻게 해결했는지
- 왜 그 방식을 선택했는지
- 어떤 한계가 있었는지

```
![기존 방식](./images/before.png)
```

### 3. 개선 과정 (Journey)
- 어떤 계기로 개선하게 되었는지
- 어떤 대안들을 고려했는지
- 최종 선택의 이유

### 4. 새로운 방식 (After)
- 개선된 방식 설명
- 핵심 코드나 설정 (필요시)
- 달라진 점

```
![새로운 방식](./images/after.png)
```

### 5. 마무리
- 배운 점 또는 인사이트
- (선택) 다음 개선 방향

## 작성 톤

- **평서체** 사용 ("~했습니다" X, "~했어요" X, "~했다" O)
- 존대말 쓰지 않음
- 편하게 경험을 공유하는 느낌
- 기술적 깊이보다 **과정과 인사이트** 중심
- 코드는 핵심만 간결하게

## 이미지 캡쳐 포인트

글 작성 시 아래 시점의 스크린샷을 `images/` 폴더에 저장:

1. **before.png** - 기존 방식/문제 상황
2. **after.png** - 개선된 방식/결과
3. **diagram.png** - 구조 설명 (필요시)
4. **screenshot-N.png** - 추가 스크린샷

## 예시

### 폴더 구조
```
docs/content/blog/github-api-claude-commands/
├── index.mdx
└── images/
    ├── clone-workflow.png
    └── api-workflow.png
```

### index.mdx
```markdown
---
title: "GitHub API로 Claude Code 커맨드 관리하기"
description: "로컬 클론 없이 커맨드를 설치하고 배포하는 방법"
date: "2026-01-06"
category: "개발"
published: true
---

## 불편했던 점

Claude Code에서 커맨드를 여러 컴퓨터에서 공유하고 싶었다.
처음에는 GitHub 저장소를 클론해서 썼다...

![기존: 저장소 클론 후 수동 복사](./images/clone-workflow.png)

## 개선 아이디어

`gh` CLI가 GitHub API를 지원한다는 걸 알게 됐다.
파일을 직접 GET/PUT 할 수 있다면...

## 새로운 방식

이제 `/install-command` 하나로 바로 설치할 수 있다.

![개선: API로 직접 설치](./images/api-workflow.png)

## 배운 점

로컬 상태에 의존하지 않는 워크플로우가...
```

## 새 글 작성 체크리스트

1. [ ] `docs/content/blog/[slug]/` 폴더 생성
2. [ ] `index.mdx` 작성
3. [ ] 이미지가 있으면 `images/` 폴더에 저장
4. [ ] 이미지 경로는 `./images/파일명.png` 형식
5. [ ] frontmatter의 date를 오늘 날짜로 (YYYY-MM-DD)
