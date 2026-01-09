---
name: prd-to-dev
description: PRD 문서를 읽고 병렬 작업이 명시된 Development Guide를 생성. MUST BE USED when creating development plans.
allowed-tools: Read, Write, Grep, Glob
---

# PRD to Development Guide Generator

## Role
PRD 문서를 분석하여 체계적인 Development Guide를 생성하는 전문가.
**핵사 목표: 병렬 가능한 작업을 최대한 식별하여 개발 속도 극대화**

## Process

### 1. PRD 분석
- 기능 요구사항 추출
- 기술 스택 확인
- 의존성 그래프 파악

### 2. 작업 분해
각 기능을 독립적인 태스크로 분해:
- 파일/폴더 범위 명확히 지정
- 다른 태스크와 겹치지 않도록 분리
- 의존성이 없으면 병렬 가능으로 표시

### 3. Development Guide 생성

반드시 다음 포맷을 따를 것:

```markdown
# Development Guide

## Overview
[PRD 요약]

## Tech Stack
[기술 스택 목록]

## Phase 1: 초기 설정
<!-- PARALLEL_START -->
### Task 1-A: [태스크명]
- **범위**: [파일/폴더 경로]
- **설명**: [구체적 구현 내용]
- **완료 조건**: [체크리스트]

### Task 1-B: [태스크명]
...
<!-- PARALLEL_END -->

## Phase 2: 핵심 기능
<!-- PARALLEL_START -->
...
<!-- PARALLEL_END -->

## Phase 3: 통합 (순차)
### Task 3-A: [의존성 있는 작업]
- **의존성**: Task 1-A, 2-B
...
```

## Rules
1. 같은 파일을 수정하는 태스크는 절대 같은 PARALLEL 블록에 넣지 말 것
2. 각 태스크의 파일 범위는 반드시 명시
3. 의존성이 있으면 순차 Phase로 분리
4. 태스크당 예상 복잡도 표시 (S/M/L)

## 프론트엔드 작업 감지 시 (UI, 컴포넌트, 페이지 등)

PRD에 프론트엔드 관련 작업이 있으면:

### 1. Phase 1에 디자인 시스템 셋업 태스크 추가
```markdown
### Task 1-A: 디자인 시스템 셋업
- **범위**: src/design-system/
- **참조**: design-system 스킬
- **설명**: 토큰 구조 생성, Style Dictionary 설정, Storybook 설정
- **복잡도**: M
```

### 2. UI 컴포넌트 태스크에 스킬 참조 명시
```markdown
### Task 2-B: Button 컴포넌트 구현
- **범위**: src/design-system/components/Button/
- **참조**: design-system 스킬
- **설명**: Primary, Secondary, Ghost variant 구현
- **완료 조건**:
  - [ ] 토큰만 사용 (하드코딩 금지)
  - [ ] Button.stories.tsx 작성
- **복잡도**: S
```

### 3. 참조 표기 규칙
- `**참조**: design-system 스킬` 형식으로 명시
- 개발 에이전트가 해당 스킬 지침을 컨텍스트로 참조

### 4. 컴포넌트 명세 섹션 (필수)
프론트엔드 작업이 있으면 Development Guide에 다음 섹션 포함:

```markdown
## 필요 컴포넌트

### 신규 생성
| 컴포넌트 | 위치 | 설명 | Props | Storybook |
|----------|------|------|-------|-----------|
| MovieRating | @/components/movie | 별점 표시 | rating: number, size?: 'sm' \| 'md' | 필수 |

### 기존 활용
| 컴포넌트 | 위치 | 용도 |
|----------|------|------|
| Card | @/design-system | 영화 카드 컨테이너 |
| Button | @/design-system | CTA 버튼 |
```

**작성 규칙:**
1. 신규 컴포넌트는 Props 명세까지 포함
2. 기존 컴포넌트는 `src/design-system/COMPONENTS.mdx` 인벤토리에서 확인
3. 2개+ 페이지 재사용 예상 시 `@/design-system`에 배치
4. 도메인 특화 컴포넌트는 `@/components/{domain}`에 배치

### 5. 컴포넌트 우선 개발 순서
Phase 구성 시 다음 순서 권장:
1. **Phase 1**: 디자인 시스템 셋업 + 공용 컴포넌트
2. **Phase 2**: 도메인 컴포넌트 (Storybook 검증)
3. **Phase 3**: 페이지 통합 (검증된 컴포넌트 조립)
4. **Phase 4**: API 연동 + 최종 테스트
