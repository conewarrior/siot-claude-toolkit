---
name: design-system
description: 디자인 토큰 계층 구조와 컴포넌트 라이브러리를 체계적으로 구축. 프론트엔드 개발 시 일관된 구조와 스타일링을 보장한다.
---

# Design System Skill

프론트엔드 개발 시 디자인 시스템을 체계적으로 구축하고 관리하는 스킬.

## 핵심 원칙

1. **토큰 기반 스타일링**: 하드코딩 금지, 모든 값은 토큰 참조
2. **컴포넌트 재사용**: 공통 UI는 반드시 컴포넌트 라이브러리에 등록
3. **Storybook 필수**: 컴포넌트 생성 시 story 파일 동반

---

## 1. 디자인 토큰 계층 구조

### 3단계 토큰 시스템

```
Primitive (기본값)
    ↓ 참조
Semantic (의미)
    ↓ 참조
Component (컴포넌트)
```

### Primitive Tokens (변경 시 전체 영향)
가장 말단의 원시 값. 브랜드 컬러, 기본 스케일 등.

```json
{
  "brand": {
    "color": {
      "blue": {
        "50": { "value": "#EFF6FF" },
        "500": { "value": "#3B82F6" },
        "900": { "value": "#1E3A8A" }
      }
    }
  },
  "base": {
    "spacing": {
      "1": { "value": "4px" },
      "2": { "value": "8px" },
      "4": { "value": "16px" }
    },
    "radius": {
      "sm": { "value": "4px" },
      "md": { "value": "8px" },
      "lg": { "value": "16px" }
    }
  }
}
```

### Semantic Tokens (의미 부여)
Primitive를 참조하여 용도별 의미 부여.

```json
{
  "color": {
    "primary": { "value": "{brand.color.blue.500}" },
    "primary-hover": { "value": "{brand.color.blue.900}" },
    "background": { "value": "{brand.color.blue.50}" }
  },
  "spacing": {
    "component-gap": { "value": "{base.spacing.4}" }
  }
}
```

### Component Tokens (컴포넌트별)
Semantic을 참조하여 컴포넌트 전용 토큰 정의.

```json
{
  "button": {
    "background": { "value": "{color.primary}" },
    "background-hover": { "value": "{color.primary-hover}" },
    "padding": { "value": "{base.spacing.2} {base.spacing.4}" },
    "radius": { "value": "{base.radius.md}" }
  }
}
```

---

## 2. 폴더 구조

```
src/
├── design-system/
│   ├── tokens/
│   │   ├── primitive/
│   │   │   ├── colors.json
│   │   │   ├── typography.json
│   │   │   ├── spacing.json
│   │   │   └── radius.json
│   │   ├── semantic/
│   │   │   ├── colors.json
│   │   │   └── typography.json
│   │   └── component/
│   │       ├── button.json
│   │       ├── card.json
│   │       └── input.json
│   │
│   ├── components/
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.stories.tsx  ← 필수
│   │   │   ├── Button.test.tsx     ← 권장
│   │   │   └── index.ts
│   │   ├── Card/
│   │   ├── Input/
│   │   └── ...
│   │
│   ├── generated/               ← 빌드 결과물 (자동 생성)
│   │   ├── tokens.css
│   │   └── tokens.ts
│   │
│   └── index.ts                 ← 전체 export
│
└── pages/                       ← 컴포넌트 조합만
```

---

## 3. 컴포넌트 생성 규칙

### 필수 사항

1. **토큰만 사용**: 색상, 간격, 폰트 등 모든 스타일 값은 토큰 참조
   ```tsx
   // ❌ 하드코딩
   background: '#3B82F6'

   // ✅ 토큰 참조
   background: var(--button-background)
   ```

2. **Story 파일 필수**: 컴포넌트 생성 시 `.stories.tsx` 동반
   ```tsx
   // Button.stories.tsx
   export default {
     title: 'Components/Button',
     component: Button,
   }

   export const Primary = { args: { variant: 'primary' } }
   export const Secondary = { args: { variant: 'secondary' } }
   export const Disabled = { args: { disabled: true } }
   ```

3. **Variant 패턴**: 상태별 스타일은 variant로 관리
   ```tsx
   type ButtonVariant = 'primary' | 'secondary' | 'ghost'
   type ButtonSize = 'sm' | 'md' | 'lg'
   ```

### 컴포넌트 등록 기준

다음 중 하나라도 해당되면 design-system/components에 등록:
- 2개 이상의 페이지에서 사용
- 브랜드 아이덴티티 관련 (로고, 푸터, 헤더)
- 인터랙션 패턴 포함 (버튼, 인풋, 모달)

---

## 4. Storybook 설정

### 필수 Addon

```js
// .storybook/main.ts
addons: [
  '@storybook/addon-essentials',
  '@storybook/addon-themes',      // 테마 스위칭
  '@storybook/addon-a11y',        // 접근성 체크
]
```

### 글로벌 토큰 주입

```tsx
// .storybook/preview.tsx
import '../src/design-system/generated/tokens.css'

export const parameters = {
  backgrounds: {
    default: 'light',
    values: [
      { name: 'light', value: 'var(--color-background)' },
      { name: 'dark', value: 'var(--color-background-dark)' },
    ],
  },
}
```

---

## 5. 토큰 빌드

### Style Dictionary 사용 (권장)

```bash
npm install style-dictionary
```

```js
// style-dictionary.config.js
module.exports = {
  source: ['src/design-system/tokens/**/*.json'],
  platforms: {
    css: {
      transformGroup: 'css',
      buildPath: 'src/design-system/generated/',
      files: [{
        destination: 'tokens.css',
        format: 'css/variables',
      }],
    },
    ts: {
      transformGroup: 'js',
      buildPath: 'src/design-system/generated/',
      files: [{
        destination: 'tokens.ts',
        format: 'javascript/es6',
      }],
    },
  },
}
```

---

## 6. 디자인 미학 가이드

### Typography
- 제네릭 폰트(Arial, Inter, Roboto) 지양
- 프로젝트 성격에 맞는 특색 있는 폰트 선택
- Display + Body 폰트 페어링 권장

### Color
- 브랜드 컬러 기반 팔레트 구성
- 의미 있는 컬러 네이밍 (primary, error, success)
- 다크모드 고려 시 semantic 토큰 필수

### Spacing & Layout
- 일관된 spacing scale 사용 (4px 배수 권장)
- 컴포넌트 간 간격도 토큰으로 관리

### Motion
- 의미 있는 트랜지션만 추가
- duration, easing도 토큰화 권장
- CSS-only 우선, 복잡한 경우 Framer Motion

---

## 7. 컴포넌트 인벤토리 관리

### 인벤토리 파일
`src/design-system/COMPONENTS.mdx` - Storybook에서 볼 수 있는 프로젝트 전체 컴포넌트 목록

### 인벤토리 구조
```markdown
| 컴포넌트 | 설명 | Storybook | 사용 페이지 |
|----------|------|-----------|------------|
| Button   | 기본 버튼 | ✅ | 전체 |
```

### 필수 업데이트 시점
1. **새 컴포넌트 생성 시**: 해당 섹션에 즉시 추가
2. **컴포넌트 삭제 시**: 인벤토리에서 제거
3. **사용처 변경 시**: 사용 페이지 컬럼 업데이트
4. **Storybook 추가 시**: ❌ → ✅ 변경
5. **기능 개발 완료 시**: Last Updated 날짜 갱신

### 카테고리 분류
| 카테고리 | 경로 | 기준 |
|----------|------|------|
| Design System | `@/design-system` | 2개+ 페이지 재사용 |
| 도메인 | `@/components/{domain}` | 특정 서비스 전용 |
| 공통 | `@/components/common` | 레이아웃, 네비게이션 |
| Provider | `@/components/providers` | 컨텍스트 제공 |

### 주기적 점검 (월 1회 권장)
- Storybook 커버리지 확인 (목표: 80%+)
- 미사용 컴포넌트 정리
- 중복 컴포넌트 통합 검토

---

## 8. 체크리스트

### 프로젝트 시작 시
- [ ] design-system 폴더 구조 생성
- [ ] primitive 토큰 정의 (colors, spacing, radius)
- [ ] Style Dictionary 설정
- [ ] Storybook 설정
- [ ] COMPONENTS.mdx 인벤토리 파일 생성

### 컴포넌트 생성 시
- [ ] 토큰만 사용했는가?
- [ ] Story 파일 작성했는가?
- [ ] 필요한 variant 정의했는가?
- [ ] export를 index.ts에 추가했는가?
- [ ] **COMPONENTS.mdx에 등록했는가?**

### 토큰 변경 시
- [ ] primitive 변경이면 영향 범위 확인
- [ ] Storybook에서 전체 컴포넌트 확인
- [ ] 빌드 스크립트 실행

### 기능 개발 완료 시
- [ ] 인벤토리 업데이트 (새 컴포넌트, 사용처 변경)
- [ ] Last Updated 날짜 갱신
