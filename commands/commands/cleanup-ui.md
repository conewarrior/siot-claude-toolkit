---
description: UI 코드 구조 정리 및 최적화 - 비효율적인 스타일링과 레이아웃 계층 정리
allowed-tools: Read, Edit, Glob, Grep
---

# /cleanup-ui

UI 작업 후 비효율적인 코드 구조를 정리한다.

## 목적

프론트엔드 작업 시 발생하는 다음 문제들을 해결:
- 중복 배경색/스타일 정의
- 불필요한 wrapper div
- 레이아웃 계층 혼란
- 컴포넌트 간 스타일 충돌

## UI 구조 최적화 원칙

### 1. 레이아웃 계층 원칙

**배경색은 가장 상위에서 1회만 정의:**
```tsx
// 좋은 예: layout.tsx에서 배경 정의
<div className="bg-background">
  <Header />  // transparent
  <main>{children}</main>  // transparent
</div>

// 나쁜 예: 각 컴포넌트마다 배경색
<Header className="bg-background" />
<main className="bg-background">{children}</main>
```

**z-index 사용 전 레이아웃 전체 구조 파악:**
- 상위 컨테이너의 배경이 하위를 덮지 않는지 확인
- fixed/absolute 요소의 z-index 충돌 체크

### 2. 컴포넌트 스타일 원칙

**하위 컴포넌트는 transparent가 기본:**
```tsx
// 공용 컴포넌트
export function Header({ className }: { className?: string }) {
  return (
    <header className={cn(
      "sticky top-0 z-10",  // 위치/레이아웃만
      className  // 배경색은 사용처에서 주입
    )}>
      ...
    </header>
  )
}
```

**스타일 주입은 사용처에서:**
```tsx
// 페이지에서 필요에 따라 배경 지정
<Header className="bg-background/80 backdrop-blur" />
// 또는 투명하게
<Header />
```

### 3. 작업 전 확인사항

1. **레이아웃 파일 먼저 확인** (`layout.tsx`)
   - 전체 배경색 정의 위치
   - 컨테이너 구조

2. **상위 컴포넌트 스타일 확인**
   - 부모가 이미 배경색을 가지고 있는지
   - overflow, position 설정

3. **변경 영향 범위 파악**
   - 이 스타일이 다른 페이지에도 영향을 주는지
   - 공용 컴포넌트 수정 시 모든 사용처 확인

## Cleanup 체크리스트

현재 페이지/컴포넌트에서 다음을 확인하고 정리:

### 배경 & 색상
- [ ] 중복 배경색 제거 (상위에서 이미 정의된 경우)
- [ ] 불필요한 `bg-*` 클래스 정리
- [ ] 다크모드 대응 확인 (`text-foreground` vs 하드코딩된 색상)

### 레이아웃 구조
- [ ] 불필요한 wrapper div 제거
- [ ] padding/margin 중복 정리
- [ ] flex/grid 컨테이너 최적화

### 컴포넌트 독립성
- [ ] 공용 컴포넌트가 특정 페이지 스타일에 의존하지 않는지
- [ ] className prop으로 커스터마이징 가능한 구조인지

### 반응형
- [ ] 불필요한 breakpoint 클래스 정리
- [ ] mobile-first 원칙 준수 확인

## 실행 방법

1. 정리할 페이지/컴포넌트 경로 확인
2. 해당 파일과 관련 layout.tsx 읽기
3. 위 체크리스트 기준으로 문제점 식별
4. 수정 제안 또는 직접 수정

## 예시

```bash
/cleanup-ui
# → 현재 작업 중인 파일들 대상으로 정리

/cleanup-ui src/app/(movie)/[id]/page.tsx
# → 특정 페이지 대상으로 정리
```
