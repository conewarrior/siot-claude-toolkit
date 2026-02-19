# /setup-design

프로젝트에 @gpters-internal/ui 디자인 시스템을 자동 설정합니다.

**한 번 실행으로 완료되는 항목:**
- npm 패키지 설치
- CLAUDE.md에 디자인 규칙 추가
- UI 생성 시 규칙 자동 적용 (Hook) - node_modules에서 직접 참조
- Dependabot 자동 업데이트 설정 (design-rules.md도 자동 업데이트)
- **버전 기반 상태 추적** (setup-design 업데이트 시 자동 감지)

---

## 사전 요구사항 (자동 기여 기능)

컴포넌트 자동 기여 기능을 사용하려면 GitHub 인증이 필요합니다.

### 방법 1: gh CLI (권장)

로컬 개발 환경에서 Hook이 자동 실행될 때 필요합니다.

```bash
# 설치
brew install gh        # macOS
# apt install gh       # Ubuntu
# winget install gh    # Windows

# 인증 (브라우저 로그인)
gh auth login
```

### 방법 2: GITHUB_TOKEN (CI/CD용)

GitHub Actions 등 CI 환경에서 사용합니다.

```bash
export GITHUB_TOKEN="ghp_xxxxxxxxxxxx"
```

> **참고**: Hook은 별도 프로세스로 실행되어 터미널 환경변수를 상속받지 못합니다.
> 로컬에서는 gh CLI를 사용하세요.

---

## 실행 단계

### Step 0: 상태 검증 (필수)

> ⚠️ **"이미 설정됨"으로 스킵하지 말 것. 항상 검증 스크립트 결과를 기준으로 판단.**

**`.claude/scripts/verify-design-setup.sh`가 있으면:**
```bash
.claude/scripts/verify-design-setup.sh
```
검증 결과에서 ❌ 표시된 항목만 설정하고, ✅ 항목도 **최신 스펙과 비교**

**검증 스크립트가 없으면:**
→ Step 1부터 전체 설정 진행

### Step 1: 프로젝트 타입 확인

**package.json이 있으면:**
→ Node.js 프로젝트로 진행 (Step 2로)

**package.json이 없으면:**
AskUserQuestion으로 물어봅니다:

질문: "프로젝트 타입을 선택하세요"
- **Node.js 프로젝트** (권장): npm 패키지 설치 (package.json 자동 생성)
- **HTML/CSS만**: CDN 링크만 사용 (npm 설치 없음)

### Step 2: npm 패키지 설치 (Node.js 프로젝트)

**package.json이 없으면 먼저 생성:**
```bash
npm init -y
```

**패키지 설치:**
```bash
npm install @gpters-internal/ui
```

### Step 2.5: 토큰 로드 추가

> **주의**: Tailwind CSS v4는 CSS `@import`로 외부 URL을 번들링 시도하므로, 반드시 HTML `<link>` 태그로 로드해야 합니다.

**Next.js 프로젝트** (`app/layout.tsx`의 `<head>` 또는 metadata에 추가):
```tsx
<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/conewarrior/design-system/tokens.css" />
```

**React (CRA/Vite)** (`index.html`의 `<head>`에 추가):
```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/conewarrior/design-system/tokens.css" />
```

**HTML/CSS 프로젝트** (`<head>`에 추가):
```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/conewarrior/design-system/tokens.css" />
```

### Step 2.6: Tailwind v4 설치 (shadcn/ui 사용 시)

> shadcn/ui 컴포넌트를 사용하려면 Tailwind CSS v4가 필요합니다.
> tokens.css의 값을 @theme으로 매핑하여 shadcn 컴포넌트가 디자인 토큰을 사용하도록 합니다.

**AskUserQuestion으로 확인:**
```
질문: "Tailwind CSS v4를 설치하시겠습니까?"
- **설치** (권장): shadcn/ui 컴포넌트 사용 가능
- **건너뛰기**: 기존 CSS 방식 유지
```

**"설치" 선택 시:**

1. **패키지 설치:**
```bash
npm install tailwindcss@latest @tailwindcss/postcss@latest
```

2. **postcss.config.mjs 생성:**
```javascript
export default {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
```

3. **globals.css (또는 메인 CSS 파일)에 Tailwind 추가:**

tokens.css `<link>` 로드 후, globals.css에 추가:
```css
/* Tailwind CSS v4 */
@import "tailwindcss";

/* Tailwind Theme - tokens.css 변수를 Tailwind 테마에 매핑 */
@theme {
  /* Colors - shadcn compatible */
  --color-background: var(--color-background);
  --color-foreground: var(--color-foreground);
  --color-primary: var(--color-primary);
  --color-primary-foreground: var(--color-primary-foreground);
  --color-secondary: var(--color-secondary);
  --color-secondary-foreground: var(--color-secondary-foreground);
  --color-muted: var(--color-muted);
  --color-muted-foreground: var(--color-muted-foreground);
  --color-accent: var(--color-accent);
  --color-accent-foreground: var(--color-accent-foreground);
  --color-destructive: var(--color-destructive);
  --color-destructive-foreground: var(--color-destructive-foreground);
  --color-border: var(--color-border);
  --color-input: var(--color-input);
  --color-ring: var(--color-ring);

  /* Radius */
  --radius-sm: var(--radius-sm);
  --radius-md: var(--radius-md);
  --radius-lg: var(--radius-lg);
  --radius-xl: var(--radius-xl);
  --radius-2xl: var(--radius-2xl);
  --radius-full: var(--radius-full);

  /* Font Family */
  --font-sans: var(--font-family-sans);
  --font-mono: var(--font-family-mono);
}
```

> 💡 **참고**: Tailwind v4는 CSS-first 설정 방식을 사용합니다.
> `tailwind.config.js` 대신 `@theme` 블록에서 테마를 정의합니다.

### Step 3: CLAUDE.md 설정
기존 CLAUDE.md를 읽고, 없으면 새로 생성합니다.
다음 내용을 CLAUDE.md에 추가합니다:

```markdown
## 디자인 시스템

이 프로젝트는 @gpters-internal/ui 디자인 시스템을 사용합니다.

### 토큰
- CDN: https://cdn.jsdelivr.net/gh/conewarrior/design-system/tokens.css
- 모든 색상, 간격, radius는 tokens.css의 CSS 변수 사용 필수

### 규칙 (자동 적용)
UI 생성 시 design-rules skill이 node_modules에서 자동 로드됩니다:
- 하드코딩 색상 금지 (#fff, rgb 등) → var(--color-*) 사용
- 8px 단위 간격만 사용 → var(--spacing-*) 사용
- radius는 토큰만 사용 → var(--radius-*) 사용
- 화면당 컴포넌트 최대 7개
- 배경/강조 색상 최대 3개

> 💡 design-rules.md는 npm 업데이트 시 자동으로 최신 버전이 적용됩니다.

### 컴포넌트 생성 규칙 (필수)

**⚠️ 기존 컴포넌트 코드를 참고하지 마라. 기존 코드가 틀렸을 수 있다.**

컴포넌트 생성 시 반드시 다음 순서를 따른다:

1. **design-rules.md를 유일한 소스로 사용**
   - `node_modules/@gpters-internal/ui/.claude/skills/design-rules.md` 규칙 확인
   - 기존 컴포넌트 패턴 복사 금지

2. **생성 전 체크리스트**
   - [ ] 토큰만 사용 (하드코딩 색상/간격 금지)
   - [ ] SVG 아이콘만 사용 (이모지/텍스트 문자 금지)
   - [ ] Shadow 사용 금지 (Modal/Dropdown/Toast 제외)
   - [ ] 적절한 radius 토큰 사용

3. **생성 후 자가 검증**
   - 작성한 코드가 design-rules 위반하는지 점검
   - lint-design-rules.sh 자동 실행됨

4. **기존 컴포넌트 위반 발견 시**
   - 별도로 사용자에게 보고
   - 새 컴포넌트는 규칙대로 작성

### 컴포넌트 기여
components/ 폴더에 새 컴포넌트 생성 시 자동으로 design-system 저장소에 기여됩니다.

### 설정 버전 관리

**/setup-design 실행 시 반드시 다음 순서를 따른다:**

1. **먼저 현재 상태 검증**
   ```bash
   .claude/scripts/verify-design-setup.sh
   ```

2. **누락된 항목만 적용**
   - 검증 결과에서 ❌ 표시된 항목만 설정
   - 이미 ✅인 항목도 **최신 스펙과 비교**

3. **상태 파일 업데이트**
   - `.claude/design-system-state.json` 버전 및 날짜 갱신

> ⚠️ "이미 설정됨"으로 스킵하지 말 것. 항상 verify 스크립트 결과를 기준으로 판단.
```

### Step 4: Hook 설정
`.claude/settings.local.json` 파일을 생성/수정하여 다음 hook을 등록합니다:

> ⚠️ **중요**: PostToolUse hook은 환경변수가 아닌 **stdin으로 JSON**을 받습니다.
> `jq`를 사용하여 `tool_input.file_path`를 파싱해야 합니다.

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [{"type": "command", "command": "cat node_modules/@gpters-internal/ui/.claude/skills/design-rules.md"}]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {"type": "command", "command": "file_path=$(jq -r '.tool_input.file_path // empty') && if [[ \"$file_path\" == *\"components/\"* ]]; then \"$CLAUDE_PROJECT_DIR\"/.claude/scripts/auto-contribute.sh \"$file_path\"; fi"},
          {"type": "command", "command": "file_path=$(jq -r '.tool_input.file_path // empty') && if [[ \"$file_path\" == *\"components/\"* ]]; then \"$CLAUDE_PROJECT_DIR\"/.claude/scripts/lint-design-rules.sh \"$file_path\"; fi", "statusMessage": "Design Rules 검증 중..."}
        ]
      }
    ]
  }
}
```

**Hook 설명:**
- `UserPromptSubmit`: 모든 프롬프트 제출 시 **node_modules에서** design-rules.md 로딩 (npm 업데이트 시 자동 반영)
- `PostToolUse`: Write|Edit 도구 사용 시:
  - **stdin에서 JSON 파싱**: `jq -r '.tool_input.file_path'`로 파일 경로 추출
  - components/ 변경 감지하여 자동 기여
  - **design-rules 위반 자동 검증** (lint-design-rules.sh)

**필수 의존성:**
- `jq`: JSON 파싱 도구 (macOS: `brew install jq`, Ubuntu: `apt install jq`)

### Step 4.5: 자동 기여 스크립트 생성
`.claude/scripts/auto-contribute.sh` 파일을 생성합니다:

```bash
mkdir -p .claude/scripts
```

```bash
#!/bin/bash
# auto-contribute.sh
# 컴포넌트 파일을 design-system 저장소에 PR로 기여
#
# 사용법: auto-contribute.sh <file_path>
# 예시: auto-contribute.sh components/Card/index.tsx
#
# 인증 방법 (우선순위):
# 1. gh CLI (권장) - gh auth login으로 설정
# 2. GITHUB_TOKEN 환경변수 - CI/CD용

set -e

FILE_PATH="$1"
REPO="conewarrior/design-system"
REPO_OWNER="conewarrior"
REPO_NAME="design-system"

# 색상 정의
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# 파일 경로 검증
if [[ -z "$FILE_PATH" ]]; then
    exit 0  # Hook에서 호출 시 인자 없으면 조용히 종료
fi

# components/ 폴더인지 확인
if [[ "$FILE_PATH" != *"components/"* ]]; then
    exit 0  # components 폴더가 아니면 조용히 종료
fi

# 인증 방법 결정
USE_GH=false
USE_TOKEN=false

if command -v gh &> /dev/null && gh auth status &> /dev/null; then
    USE_GH=true
elif [[ -n "$GITHUB_TOKEN" ]]; then
    USE_TOKEN=true
else
    echo -e "${YELLOW}⚠️ GitHub 인증이 필요합니다. 다음 중 하나를 설정하세요:${NC}"
    echo "   1. gh CLI: brew install gh && gh auth login"
    echo "   2. 환경변수: export GITHUB_TOKEN=\"your_token\""
    exit 0
fi

# 절대 경로 변환
if [[ "$FILE_PATH" = /* ]]; then
    ABS_FILE_PATH="$FILE_PATH"
else
    ABS_FILE_PATH="$(pwd)/$FILE_PATH"
fi

# 파일 존재 확인
if [[ ! -f "$ABS_FILE_PATH" ]]; then
    echo -e "${RED}❌ 파일을 찾을 수 없습니다: $ABS_FILE_PATH${NC}"
    exit 1
fi

# components/ 이후 경로 추출
RELATIVE_PATH=$(echo "$FILE_PATH" | sed 's|.*\(components/.*\)|\1|')
COMPONENT_DIR=$(echo "$RELATIVE_PATH" | cut -d'/' -f2)

echo -e "🚀 Auto-contributing: ${COMPONENT_DIR}"

# ===== gh CLI 방식 (PR 생성) =====
if [[ "$USE_GH" == "true" ]]; then
    TEMP_DIR=$(mktemp -d)
    trap "rm -rf $TEMP_DIR" EXIT

    cd "$TEMP_DIR"
    gh repo clone "$REPO" design-system -- --depth 1 -q 2>/dev/null
    cd design-system

    BRANCH_NAME="contrib/${COMPONENT_DIR}-$(date +%Y%m%d%H%M%S)"
    git checkout -b "$BRANCH_NAME" -q

    mkdir -p "$(dirname "$RELATIVE_PATH")"
    cp "$ABS_FILE_PATH" "$RELATIVE_PATH"

    git add "$RELATIVE_PATH"
    git commit -m "feat(components): add ${COMPONENT_DIR}" -q
    git push -u origin "$BRANCH_NAME" -q 2>/dev/null

    PR_URL=$(gh pr create \
        --repo "$REPO" \
        --title "feat(components): add ${COMPONENT_DIR}" \
        --body "Auto-contributed component from project.

- Component: \`${COMPONENT_DIR}\`
- File: \`${RELATIVE_PATH}\`
- From: $(hostname)" \
        --base main \
        --head "$BRANCH_NAME" 2>/dev/null)

    echo -e "${GREEN}✅ PR 생성됨: ${PR_URL}${NC}"
    exit 0
fi

# ===== GITHUB_TOKEN 방식 (직접 API 호출) =====
if [[ "$USE_TOKEN" == "true" ]]; then
    API_URL="https://api.github.com/repos/$REPO/contents/$RELATIVE_PATH"
    CONTENT=$(base64 < "$ABS_FILE_PATH")

    # 기존 파일 SHA 확인
    EXISTING=$(curl -s -H "Authorization: token $GITHUB_TOKEN" "$API_URL" 2>/dev/null)
    SHA=$(echo "$EXISTING" | grep -o '"sha": "[^"]*"' | head -1 | cut -d'"' -f4)

    # 커밋 메시지
    COMMIT_MSG="feat(components): add ${COMPONENT_DIR}

- File: $RELATIVE_PATH
- From: $(hostname)
- Auto-contributed via design-system hook"

    # JSON 페이로드
    if [[ -n "$SHA" ]]; then
        PAYLOAD="{\"message\": \"$COMMIT_MSG\", \"content\": \"$CONTENT\", \"branch\": \"main\", \"sha\": \"$SHA\"}"
    else
        PAYLOAD="{\"message\": \"$COMMIT_MSG\", \"content\": \"$CONTENT\", \"branch\": \"main\"}"
    fi

    RESPONSE=$(curl -s -X PUT \
        -H "Authorization: token $GITHUB_TOKEN" \
        -H "Content-Type: application/json" \
        -H "Accept: application/vnd.github.v3+json" \
        "$API_URL" \
        -d "$PAYLOAD" 2>/dev/null)

    if echo "$RESPONSE" | grep -q '"commit"'; then
        COMMIT_URL=$(echo "$RESPONSE" | grep -o '"html_url": "[^"]*"' | head -1 | cut -d'"' -f4)
        echo -e "${GREEN}✅ design-system에 기여됨: ${COMPONENT_DIR}${NC}"
        [[ -n "$COMMIT_URL" ]] && echo -e "   커밋: $COMMIT_URL"
    else
        ERROR_MSG=$(echo "$RESPONSE" | grep -o '"message": "[^"]*"' | head -1 | cut -d'"' -f4)
        echo -e "${RED}❌ 기여 실패: ${ERROR_MSG:-알 수 없는 오류}${NC}"
        exit 1
    fi
fi
```

**스크립트 생성 후 실행 권한 부여:**
```bash
chmod +x .claude/scripts/auto-contribute.sh
```

### Step 4.6: Design Rules 검증 스크립트 생성
`.claude/scripts/lint-design-rules.sh` 파일을 생성합니다:

```bash
#!/bin/bash
# design-rules.md 위반 탐지 스크립트
# Usage: ./lint-design-rules.sh [file_path]

FILE_PATH="$1"
VIOLATIONS=0

# 색상 출력
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# 파일이 components/ 내부인지 확인
if [[ ! "$FILE_PATH" == *"components/"* ]]; then
  exit 0
fi

echo "🔍 Design Rules 검증: $FILE_PATH"
echo "---"

# 1. 텍스트 아이콘/이모지 탐지 (규칙 2.8)
# 유니코드 심볼 (curly quotes "" 포함)
TEXT_ICONS=$(grep -E '[""]|[✓✔✅⚠️❌✕✖×▼▾⌄›❯←→ℹ🔍📋📌📝💡⬆⬇⬅➡◀▶●○■□★☆♥♦]' "$FILE_PATH" 2>/dev/null || true)
if [ -n "$TEXT_ICONS" ]; then
  echo -e "${RED}❌ [2.8 위반] 텍스트 아이콘/이모지 사용 금지${NC}"
  echo "$TEXT_ICONS"
  echo "   → SVG 아이콘으로 교체 필요 (lucide-react 권장)"
  echo ""
  VIOLATIONS=$((VIOLATIONS + 1))
fi

# 2. 하드코딩 색상 탐지 (규칙 2.3)
HARDCODED_COLORS=$(grep -E "(#[0-9a-fA-F]{3,8}|rgb\(|rgba\(|hsl\()" "$FILE_PATH" 2>/dev/null | grep -v "var(--" || true)
if [ -n "$HARDCODED_COLORS" ]; then
  echo -e "${RED}❌ [2.3 위반] 하드코딩 색상 사용 금지${NC}"
  echo "$HARDCODED_COLORS"
  echo "   → var(--color-*) 토큰으로 교체 필요"
  echo ""
  VIOLATIONS=$((VIOLATIONS + 1))
fi

# 3. 하드코딩 간격 탐지 (규칙 2.2)
HARDCODED_SPACING=$(grep -oE "(padding|margin|gap|top|right|bottom|left):\s*[0-9]+px" "$FILE_PATH" 2>/dev/null || true)
if [ -n "$HARDCODED_SPACING" ]; then
  echo -e "${YELLOW}⚠️ [2.2 주의] 하드코딩 간격 발견${NC}"
  echo "$HARDCODED_SPACING"
  echo "   → var(--spacing-*) 토큰 사용 권장"
  echo ""
fi

# 4. 하드코딩 border-radius 탐지 (규칙 2.1)
HARDCODED_RADIUS=$(grep -E "border-radius:\s*[0-9]+px" "$FILE_PATH" 2>/dev/null | grep -v "var(--radius" || true)
if [ -n "$HARDCODED_RADIUS" ]; then
  echo -e "${RED}❌ [2.1 위반] 하드코딩 border-radius 사용 금지${NC}"
  echo "$HARDCODED_RADIUS"
  echo "   → var(--radius-*) 토큰으로 교체 필요"
  echo ""
  VIOLATIONS=$((VIOLATIONS + 1))
fi

# 5. 불필요한 shadow 탐지 (규칙 2.9)
COMPONENT_NAME=$(basename "$(dirname "$FILE_PATH")")
if [[ ! "$COMPONENT_NAME" =~ ^(Modal|Dropdown|Toast|Popover|Tooltip)$ ]]; then
  SHADOW_USAGE=$(grep -E "box-shadow|boxShadow" "$FILE_PATH" 2>/dev/null || true)
  if [ -n "$SHADOW_USAGE" ]; then
    echo -e "${RED}❌ [2.9 위반] Shadow 사용 금지 (해당 컴포넌트)${NC}"
    echo "$SHADOW_USAGE"
    echo "   → border로 대체하거나 제거 필요"
    echo ""
    VIOLATIONS=$((VIOLATIONS + 1))
  fi
fi

# 결과 출력
echo "---"
if [ $VIOLATIONS -eq 0 ]; then
  echo -e "✅ Design Rules 검증 통과"
else
  echo -e "${RED}❌ $VIOLATIONS개 위반 발견 - 수정 필요${NC}"
fi

exit $VIOLATIONS
```

**스크립트 생성 후 실행 권한 부여:**
```bash
chmod +x .claude/scripts/lint-design-rules.sh
```

### Step 4.7: 설정 상태 검증 스크립트 생성
`.claude/scripts/verify-design-setup.sh` 파일을 생성합니다:

```bash
#!/bin/bash
# design-system 설정 상태 검증 스크립트
# Usage: ./verify-design-setup.sh

set -e

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

MISSING=0

echo "🔍 Design System 설정 검증 중..."
echo "---"

# 1. npm 패키지 확인
if [ -d "node_modules/@gpters-internal/ui" ]; then
  echo -e "${GREEN}✅ npm 패키지: @gpters-internal/ui 설치됨${NC}"
else
  echo -e "${RED}❌ npm 패키지: @gpters-internal/ui 미설치${NC}"
  MISSING=$((MISSING + 1))
fi

# 2. tokens.css <link> 로드 확인
TOKENS_LOADED=false
for f in index.html public/index.html src/index.html; do
  if [ -f "$f" ] && grep -q "tokens.css" "$f" 2>/dev/null; then
    TOKENS_LOADED=true
  fi
done
if [ -f "app/layout.tsx" ] && grep -q "tokens.css" "app/layout.tsx" 2>/dev/null; then
  TOKENS_LOADED=true
fi

if [ "$TOKENS_LOADED" = true ]; then
  echo -e "${GREEN}✅ tokens.css <link> 로드 완료${NC}"
else
  echo -e "${YELLOW}⚠️ tokens.css <link> 태그 확인 필요 (HTML <head>에 추가)${NC}"
fi

# 3. CLAUDE.md 디자인 시스템 섹션 확인
if [ -f "CLAUDE.md" ] && grep -q "## 디자인 시스템" "CLAUDE.md" 2>/dev/null; then
  echo -e "${GREEN}✅ CLAUDE.md: 디자인 시스템 섹션 있음${NC}"

  # 컴포넌트 생성 규칙 섹션 확인
  if grep -q "컴포넌트 생성 규칙" "CLAUDE.md" 2>/dev/null; then
    echo -e "${GREEN}✅ CLAUDE.md: 컴포넌트 생성 규칙 섹션 있음${NC}"
  else
    echo -e "${RED}❌ CLAUDE.md: 컴포넌트 생성 규칙 섹션 없음${NC}"
    MISSING=$((MISSING + 1))
  fi

  # 설정 버전 관리 섹션 확인
  if grep -q "설정 버전 관리" "CLAUDE.md" 2>/dev/null; then
    echo -e "${GREEN}✅ CLAUDE.md: 설정 버전 관리 섹션 있음${NC}"
  else
    echo -e "${RED}❌ CLAUDE.md: 설정 버전 관리 섹션 없음${NC}"
    MISSING=$((MISSING + 1))
  fi
else
  echo -e "${RED}❌ CLAUDE.md: 디자인 시스템 섹션 없음${NC}"
  MISSING=$((MISSING + 1))
fi

# 4. Hook 설정 확인
if [ -f ".claude/settings.local.json" ]; then
  if grep -q "design-rules.md" ".claude/settings.local.json" 2>/dev/null; then
    echo -e "${GREEN}✅ Hook: UserPromptSubmit (design-rules 로드)${NC}"
  else
    echo -e "${RED}❌ Hook: UserPromptSubmit 미설정${NC}"
    MISSING=$((MISSING + 1))
  fi

  if grep -q "auto-contribute.sh" ".claude/settings.local.json" 2>/dev/null; then
    echo -e "${GREEN}✅ Hook: PostToolUse (auto-contribute)${NC}"
  else
    echo -e "${RED}❌ Hook: PostToolUse (auto-contribute) 미설정${NC}"
    MISSING=$((MISSING + 1))
  fi

  if grep -q "lint-design-rules.sh" ".claude/settings.local.json" 2>/dev/null; then
    echo -e "${GREEN}✅ Hook: PostToolUse (lint-design-rules)${NC}"
  else
    echo -e "${RED}❌ Hook: PostToolUse (lint-design-rules) 미설정${NC}"
    MISSING=$((MISSING + 1))
  fi
else
  echo -e "${RED}❌ Hook 설정 파일 없음${NC}"
  MISSING=$((MISSING + 1))
fi

# 5. 스크립트 파일 확인
if [ -x ".claude/scripts/auto-contribute.sh" ]; then
  echo -e "${GREEN}✅ 스크립트: auto-contribute.sh${NC}"
else
  echo -e "${RED}❌ 스크립트: auto-contribute.sh 없거나 실행 권한 없음${NC}"
  MISSING=$((MISSING + 1))
fi

if [ -x ".claude/scripts/lint-design-rules.sh" ]; then
  echo -e "${GREEN}✅ 스크립트: lint-design-rules.sh${NC}"
else
  echo -e "${RED}❌ 스크립트: lint-design-rules.sh 없거나 실행 권한 없음${NC}"
  MISSING=$((MISSING + 1))
fi

if [ -x ".claude/scripts/verify-design-setup.sh" ]; then
  echo -e "${GREEN}✅ 스크립트: verify-design-setup.sh${NC}"
else
  echo -e "${YELLOW}⚠️ 스크립트: verify-design-setup.sh (현재 실행 중)${NC}"
fi

# 6. 상태 파일 확인
if [ -f ".claude/design-system-state.json" ]; then
  VERSION=$(grep -o '"setupVersion": "[^"]*"' ".claude/design-system-state.json" | cut -d'"' -f4)
  UPDATED=$(grep -o '"lastUpdated": "[^"]*"' ".claude/design-system-state.json" | cut -d'"' -f4)
  echo -e "${GREEN}✅ 상태 파일: v${VERSION} (${UPDATED})${NC}"
else
  echo -e "${YELLOW}⚠️ 상태 파일: .claude/design-system-state.json 없음${NC}"
fi

# 7. Dependabot 설정 확인
if [ -f ".github/dependabot.yml" ] && grep -q "@gpters-internal/ui" ".github/dependabot.yml" 2>/dev/null; then
  echo -e "${GREEN}✅ Dependabot: 자동 업데이트 설정됨${NC}"
else
  echo -e "${YELLOW}⚠️ Dependabot: 미설정 (선택사항)${NC}"
fi

# 8. ESLint 설정 확인
if [ -f "eslint.config.mjs" ] && grep -q "no-restricted-syntax" "eslint.config.mjs" 2>/dev/null; then
  echo -e "${GREEN}✅ ESLint: 페이지 컴포넌트 정의 금지 규칙 설정됨${NC}"
else
  echo -e "${RED}❌ ESLint: 페이지 컴포넌트 정의 금지 규칙 미설정${NC}"
  MISSING=$((MISSING + 1))
fi

# 결과 출력
echo "---"
if [ $MISSING -eq 0 ]; then
  echo -e "${GREEN}✅ Design System 설정 완료!${NC}"
else
  echo -e "${RED}❌ $MISSING개 항목 누락 - /setup-design 실행 필요${NC}"
fi

exit $MISSING
```

**스크립트 생성 후 실행 권한 부여:**
```bash
chmod +x .claude/scripts/verify-design-setup.sh
```

### Step 4.8: ESLint 설정 (페이지 내 컴포넌트 정의 금지)

page.tsx 파일에서 로컬 컴포넌트를 직접 정의하는 것을 방지하는 ESLint 규칙을 설정합니다.

**1. ESLint 의존성 설치:**
```bash
npm install -D @eslint/js typescript-eslint eslint
```

**2. `eslint.config.mjs` 생성:**
```javascript
import js from '@eslint/js';
import tseslint from 'typescript-eslint';

export default tseslint.config(
  js.configs.recommended,
  ...tseslint.configs.recommended,
  {
    files: ['app/**/page.tsx', 'src/**/page.tsx'],
    rules: {
      // 페이지 파일에서 로컬 컴포넌트 정의 금지 (Page 컴포넌트 제외)
      'no-restricted-syntax': [
        'error',
        {
          selector: 'FunctionDeclaration[id.name=/^[A-Z]/]:not(ExportDefaultDeclaration > FunctionDeclaration):not([id.name=/Page$/])',
          message: '❌ 페이지 파일에서 컴포넌트를 직접 정의하지 마세요. @components/ 또는 ui/에서 import하세요.',
        },
        {
          selector: 'VariableDeclarator[id.name=/^[A-Z]/][init.type="ArrowFunctionExpression"]:not([id.name=/Page$/])',
          message: '❌ 페이지 파일에서 컴포넌트를 직접 정의하지 마세요. @components/ 또는 ui/에서 import하세요.',
        },
      ],
    },
  },
);
```

**3. package.json scripts 업데이트:**
기존 lint 스크립트를 확인하고 없으면 추가:
```json
{
  "scripts": {
    "lint": "eslint app/**/page.tsx",
    "lint:fix": "eslint app/**/page.tsx --fix"
  }
}
```

**목적:**
- page.tsx 파일에 하드코딩된 컴포넌트가 있으면 ESLint 에러 발생
- `npm run lint`로 검증 가능
- CI/CD에서도 자동 검증

### Step 4.9: 상태 추적 파일 생성
`.claude/design-system-state.json` 파일을 생성합니다:

```json
{
  "setupVersion": "2.1.0",
  "lastUpdated": "YYYY-MM-DD",
  "appliedFeatures": {
    "npm-package": true,
    "tokens-import": true,
    "claude-md-base": true,
    "claude-md-component-rules": true,
    "claude-md-version-management": true,
    "hook-user-prompt-submit": true,
    "hook-post-tool-use-contribute": true,
    "hook-post-tool-use-lint": true,
    "script-auto-contribute": true,
    "script-lint-design-rules": true,
    "script-verify-design-setup": true,
    "eslint-page-component-rule": true,
    "dependabot": false
  },
  "changelog": [
    {
      "version": "2.2.0",
      "date": "2025-01-30",
      "changes": [
        "ESLint 설정 추가 (페이지 파일 내 컴포넌트 정의 금지)",
        "verify-design-setup.sh에 ESLint 설정 검증 추가"
      ]
    },
    {
      "version": "2.1.0",
      "date": "YYYY-MM-DD",
      "changes": [
        "버전 기반 상태 추적 추가",
        "verify-design-setup.sh 스크립트 추가",
        "CLAUDE.md에 설정 버전 관리 섹션 추가"
      ]
    },
    {
      "version": "2.0.0",
      "date": "2025-01-27",
      "changes": [
        "컴포넌트 생성 규칙 추가 (기존 코드 참고 금지)",
        "lint-design-rules.sh 스크립트 추가",
        "PostToolUse Hook에 lint 자동 실행 추가"
      ]
    }
  ]
}
```

> ⚠️ `YYYY-MM-DD`는 설정 시점의 실제 날짜로 교체

### Step 5: GitHub 토큰 확인 (자동 기여 기능)
GITHUB_TOKEN 환경변수가 설정되어 있는지 확인합니다.

**토큰이 있으면:**
```
✅ GITHUB_TOKEN 감지됨 - 자동 기여 기능 활성화
```

**토큰이 없으면 AskUserQuestion으로 물어봅니다:**

질문: "자동 기여 기능을 설정하시겠습니까?"
- **지금 설정** (권장): 토큰 생성 가이드를 따라 바로 설정
- **나중에 설정**: 설정 스킵, 나중에 수동으로 설정 가능
- **사용 안 함**: auto-contribute Hook 제거

**"지금 설정" 선택 시:**
```
🔧 GitHub Token 설정 가이드

1. 토큰 생성 페이지 열기:
   https://github.com/settings/tokens/new

2. 설정값:
   - Note: design-system-auto-contribute
   - Expiration: No expiration (또는 원하는 기간)
   - ✅ repo (전체 체크)

3. "Generate token" 클릭 후 토큰 복사

4. 터미널에서 실행:
   echo 'export GITHUB_TOKEN="복사한_토큰"' >> ~/.zshrc
   source ~/.zshrc

5. 설정 확인:
   echo $GITHUB_TOKEN

완료 후 /setup-design 다시 실행하면 자동 기여가 활성화됩니다.
```

**"사용 안 함" 선택 시:**
PostToolUse Hook에서 auto-contribute 부분을 제거합니다.

### Step 6: 자동 업데이트 설정 (Dependabot)
`.github/` 폴더에 자동 업데이트 설정 파일들을 생성합니다.

```bash
# 디렉토리 생성
mkdir -p .github/workflows
```

#### .github/dependabot.yml
```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
      time: "09:00"
      timezone: "Asia/Seoul"
    allow:
      - dependency-name: "@gpters-internal/ui"
    commit-message:
      prefix: "chore(deps)"
      include: "scope"
    labels:
      - "dependencies"
      - "auto-merge"
    open-pull-requests-limit: 5
```

#### .github/workflows/dependabot-auto-merge.yml
```yaml
name: Auto-merge Dependabot PRs

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: write
  pull-requests: write

jobs:
  auto-merge:
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'

    steps:
      - name: Check if @gpters-internal/ui update
        id: check
        run: |
          TITLE="${{ github.event.pull_request.title }}"
          if [[ "$TITLE" == *"@gpters-internal/ui"* ]]; then
            echo "is_geniefy_ui=true" >> $GITHUB_OUTPUT
          else
            echo "is_geniefy_ui=false" >> $GITHUB_OUTPUT
          fi

      - name: Check for major version bump
        id: major
        if: steps.check.outputs.is_geniefy_ui == 'true'
        run: |
          TITLE="${{ github.event.pull_request.title }}"
          if [[ "$TITLE" =~ from\ ([0-9]+)\.[0-9]+\.[0-9]+\ to\ ([0-9]+)\.[0-9]+\.[0-9]+ ]]; then
            FROM_MAJOR="${BASH_REMATCH[1]}"
            TO_MAJOR="${BASH_REMATCH[2]}"
            if [[ "$FROM_MAJOR" != "$TO_MAJOR" ]]; then
              echo "is_major=true" >> $GITHUB_OUTPUT
            else
              echo "is_major=false" >> $GITHUB_OUTPUT
            fi
          else
            echo "is_major=false" >> $GITHUB_OUTPUT
          fi

      - name: Wait for CI checks
        if: steps.check.outputs.is_geniefy_ui == 'true' && steps.major.outputs.is_major != 'true'
        uses: lewagon/wait-on-check-action@v1.3.4
        with:
          ref: ${{ github.event.pull_request.head.sha }}
          repo-token: ${{ secrets.GITHUB_TOKEN }}
          wait-interval: 10
          running-workflow-name: 'Auto-merge Dependabot PRs'

      - name: Enable auto-merge
        if: steps.check.outputs.is_geniefy_ui == 'true' && steps.major.outputs.is_major != 'true'
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Comment on major update
        if: steps.major.outputs.is_major == 'true'
        run: |
          gh pr comment "$PR_URL" --body "## Major 버전 업데이트

          Breaking change가 포함되어 있을 수 있습니다.
          수동 리뷰 후 머지해 주세요.

          - [CHANGELOG 확인](https://github.com/conewarrior/design-system/blob/main/CHANGELOG.md)"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Step 7: 상태 파일 업데이트
`.claude/design-system-state.json`의 버전과 날짜를 현재 시점으로 갱신합니다.

### Step 8: 완료 메시지

```
✅ @gpters-internal/ui 디자인 시스템 설정 완료!

설치된 항목:
- npm 패키지: @gpters-internal/ui
- CLAUDE.md: 디자인 규칙 + 컴포넌트 생성 규칙 + 설정 버전 관리
- Hook: UI 생성 시 node_modules에서 design-rules.md 자동 로드
- Hook: 컴포넌트 변경 시 자동 기여
- Hook: 컴포넌트 작성 시 design-rules 위반 자동 검증 (lint-design-rules.sh)
- ESLint: 페이지 파일 내 컴포넌트 직접 정의 금지 (eslint.config.mjs)
- 상태 추적: .claude/design-system-state.json
- 검증 스크립트: .claude/scripts/verify-design-setup.sh
- Dependabot: 자동 업데이트 + 자동 머지

양방향 동기화:
- 업로드: components/ 변경 → 자동 커밋
- 다운로드: 새 버전 배포 → Dependabot PR → 자동 머지
  ✓ 컴포넌트, design-rules.md, tokens.css 모두 자동 업데이트

자동 검증:
- 텍스트 아이콘/이모지 사용 → ❌ 위반 탐지
- 하드코딩 색상 (#fff, rgb) → ❌ 위반 탐지
- 하드코딩 border-radius → ❌ 위반 탐지
- 불필요한 shadow 사용 → ❌ 위반 탐지
- 페이지 파일 내 컴포넌트 정의 → ❌ ESLint 에러 (npm run lint)

버전 추적:
- 현재 설정 버전: 2.1.0
- /setup-design 재실행 시 → verify 스크립트로 누락 항목만 적용
- 업데이트 이력: .claude/design-system-state.json

토큰 참조:
- CDN: https://cdn.jsdelivr.net/gh/conewarrior/design-system/tokens.css
- 문서: https://design.geniefy.ai (또는 localhost:3333)
```

## 에러 처리

| 상황 | 처리 |
|------|------|
| package.json 없음 | npm 설치 스킵, CDN만 설정 |
| npm install 실패 | 에러 출력, 나머지 단계 계속 진행 |
| GITHUB_TOKEN 없음 | 경고 출력, 자동 기여 비활성화 안내 |
| .github 폴더 없음 | 폴더 생성 후 파일 생성 |
| GitHub 저장소 아님 | Dependabot 설정 스킵, 안내 메시지 출력 |
| verify 스크립트 없음 | 전체 설정 진행 |
| state.json 없음 | 새로 생성 |
