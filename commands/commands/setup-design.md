# /setup-design

프로젝트에 @design-geniefy/ui 디자인 시스템을 자동 설정합니다.

**한 번 실행으로 완료되는 항목:**
- npm 패키지 설치
- CLAUDE.md에 디자인 규칙 추가
- UI 생성 시 규칙 자동 적용 (Hook) - node_modules에서 직접 참조
- Dependabot 자동 업데이트 설정 (design-rules.md도 자동 업데이트)

---

## 실행 단계

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
npm install @design-geniefy/ui
```

### Step 2.5: 토큰 import 추가

**Next.js 프로젝트** (`app/layout.tsx`에 추가):
```tsx
import '@design-geniefy/ui/tokens.css';
```

**React (CRA/Vite)** (`src/index.tsx` 또는 `src/main.tsx`에 추가):
```tsx
import '@design-geniefy/ui/tokens.css';
```

**HTML/CSS 프로젝트** (`<head>`에 추가):
```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/conewarrior/design-system/tokens.css">
```

### Step 3: CLAUDE.md 설정
기존 CLAUDE.md를 읽고, 없으면 새로 생성합니다.
다음 내용을 CLAUDE.md에 추가합니다:

```markdown
## 디자인 시스템

이 프로젝트는 @design-geniefy/ui 디자인 시스템을 사용합니다.

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

### 컴포넌트 기여
components/ 폴더에 새 컴포넌트 생성 시 자동으로 design-system 저장소에 기여됩니다.
```

### Step 4: Hook 설정
`.claude/settings.local.json` 파일을 생성/수정하여 다음 hook을 등록합니다:

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [{"type": "command", "command": "cat node_modules/@design-geniefy/ui/.claude/skills/design-rules.md"}]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{"type": "command", "command": "if [[ \"$CLAUDE_TOOL_ARG_file_path\" == *\"components/\"* ]]; then .claude/scripts/auto-contribute.sh \"$CLAUDE_TOOL_ARG_file_path\"; fi"}]
      }
    ]
  }
}
```

**Hook 설명:**
- `UserPromptSubmit`: 모든 프롬프트 제출 시 **node_modules에서** design-rules.md 로딩 (npm 업데이트 시 자동 반영)
- `PostToolUse`: Write|Edit 도구 사용 시 components/ 변경 감지하여 자동 기여

### Step 4.5: 자동 기여 스크립트 생성
`.claude/scripts/auto-contribute.sh` 파일을 생성합니다:

```bash
mkdir -p .claude/scripts
```

```bash
#!/bin/bash
# auto-contribute.sh
# 컴포넌트 파일을 design-system 저장소에 자동 커밋
#
# 사용법: auto-contribute.sh <file_path>
# 예시: auto-contribute.sh components/Card/index.tsx

set -e

FILE_PATH="$1"
REPO="conewarrior/design-system"
BRANCH="main"
API_URL="https://api.github.com/repos/$REPO/contents/$FILE_PATH"

# 색상 정의
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# 파일 경로 검증
if [[ -z "$FILE_PATH" ]]; then
    echo -e "${RED}❌ 파일 경로가 필요합니다${NC}"
    exit 1
fi

# components/ 폴더인지 확인
if [[ "$FILE_PATH" != *"components/"* ]]; then
    # components 폴더가 아니면 조용히 종료
    exit 0
fi

# GITHUB_TOKEN 확인
if [[ -z "$GITHUB_TOKEN" ]]; then
    echo -e "${YELLOW}⚠️  GITHUB_TOKEN이 설정되지 않아 자동 기여를 건너뜁니다${NC}"
    echo "설정 방법: export GITHUB_TOKEN=\"your_token\""
    exit 0
fi

# 파일 존재 확인
if [[ ! -f "$FILE_PATH" ]]; then
    echo -e "${RED}❌ 파일을 찾을 수 없습니다: $FILE_PATH${NC}"
    exit 1
fi

# 파일 내용을 base64로 인코딩
CONTENT=$(base64 < "$FILE_PATH")

# 기존 파일의 SHA 확인 (업데이트 시 필요)
EXISTING=$(curl -s -H "Authorization: token $GITHUB_TOKEN" "$API_URL" 2>/dev/null)
SHA=$(echo "$EXISTING" | grep -o '"sha": "[^"]*"' | head -1 | cut -d'"' -f4)

# 커밋 메시지 생성
HOSTNAME=$(hostname)
TIMESTAMP=$(date +"%Y-%m-%d %H:%M")
COMMIT_MSG="feat: Auto-contribute $FILE_PATH

- From: $HOSTNAME
- Time: $TIMESTAMP
- Auto-contributed via design-system hook"

# JSON 페이로드 생성
if [[ -n "$SHA" ]]; then
    # 파일 업데이트 (SHA 필요)
    PAYLOAD=$(cat <<EOF
{
    "message": "$COMMIT_MSG",
    "content": "$CONTENT",
    "branch": "$BRANCH",
    "sha": "$SHA"
}
EOF
)
else
    # 새 파일 생성
    PAYLOAD=$(cat <<EOF
{
    "message": "$COMMIT_MSG",
    "content": "$CONTENT",
    "branch": "$BRANCH"
}
EOF
)
fi

# GitHub API로 커밋
RESPONSE=$(curl -s -X PUT \
    -H "Authorization: token $GITHUB_TOKEN" \
    -H "Content-Type: application/json" \
    -H "Accept: application/vnd.github.v3+json" \
    "$API_URL" \
    -d "$PAYLOAD" 2>/dev/null)

# 결과 확인
if echo "$RESPONSE" | grep -q '"commit"'; then
    COMMIT_URL=$(echo "$RESPONSE" | grep -o '"html_url": "[^"]*"' | head -1 | cut -d'"' -f4)
    echo -e "${GREEN}✅ design-system에 기여됨: $FILE_PATH${NC}"
    if [[ -n "$COMMIT_URL" ]]; then
        echo -e "   커밋: $COMMIT_URL"
    fi
else
    ERROR_MSG=$(echo "$RESPONSE" | grep -o '"message": "[^"]*"' | head -1 | cut -d'"' -f4)
    echo -e "${RED}❌ 기여 실패: ${ERROR_MSG:-알 수 없는 오류}${NC}"
    exit 1
fi
```

**스크립트 생성 후 실행 권한 부여:**
```bash
chmod +x .claude/scripts/auto-contribute.sh
```

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
      - dependency-name: "@design-geniefy/ui"
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
      - name: Check if @design-geniefy/ui update
        id: check
        run: |
          TITLE="${{ github.event.pull_request.title }}"
          if [[ "$TITLE" == *"@design-geniefy/ui"* ]]; then
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

### Step 7: 완료 메시지

```
✅ @design-geniefy/ui 디자인 시스템 설정 완료!

설치된 항목:
- npm 패키지: @design-geniefy/ui
- CLAUDE.md: 디자인 규칙 추가됨
- Hook: UI 생성 시 node_modules에서 design-rules.md 자동 로드
- Hook: 컴포넌트 변경 시 자동 기여
- Dependabot: 자동 업데이트 + 자동 머지

양방향 동기화:
- 업로드: components/ 변경 → 자동 커밋
- 다운로드: 새 버전 배포 → Dependabot PR → 자동 머지
  ✓ 컴포넌트, design-rules.md, tokens.css 모두 자동 업데이트

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
