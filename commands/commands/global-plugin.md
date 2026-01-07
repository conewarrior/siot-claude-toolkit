---
description: 플러그인을 타입별로 올바른 전역 폴더에 설치 (commands/skills/agents)
arguments:
  - name: plugin_name
    description: 설치할 플러그인 이름 (예: toolkit-commands). 생략시 설치된 플러그인 목록 표시
    required: false
---

# 플러그인 전역 설치 도우미

플러그인을 타입에 맞는 전역 폴더로 복사하여 접두사 없이 사용할 수 있게 해줍니다.

## 타입별 설치 경로

| 타입 | 전역 경로 | 예시 |
|------|-----------|------|
| commands | `~/.claude/commands/` | `/install-command` |
| skills | `~/.claude/skills/[name]/` | blog-writer, frontend-design |
| agents | `~/.claude/agents/` | codify, prd-to-dev, reviewer |

## 실행 방법

### 플러그인 이름이 주어진 경우 (`$ARGUMENTS.plugin_name`)

1. `~/.claude/plugins/installed_plugins.json`에서 플러그인 정보 찾기
   - 플러그인 키: `[name]@[marketplace]` 형식
   - installPath에서 설치 경로 확인

2. 마켓플레이스의 `marketplace.json`에서 **source 경로로 타입 판별**
   - 경로: `~/.claude/plugins/marketplaces/[marketplace]/.claude-plugin/marketplace.json`
   - plugins 배열에서 해당 플러그인의 source 필드 확인
   - `./commands` 또는 `./commands/`로 시작 → commands 타입
   - `./skills/`로 시작 → skills 타입
   - `./agents/`로 시작 → agents 타입

3. 타입별로 올바른 폴더에 복사
   - **commands**: `installPath/commands/*.md` → `~/.claude/commands/`
     - 또는 `installPath/*.md` (commands 폴더가 없는 경우)
   - **skills**: `installPath/` 전체 → `~/.claude/skills/[plugin_name]/`
     - SKILL.md 파일이 있어야 함
   - **agents**: `installPath/*.md` → `~/.claude/agents/`
     - 파일명 그대로 복사 (예: codify.md → ~/.claude/agents/codify.md)

4. 복사 결과 출력
   ```
   ✓ [plugin_name] ([type]) → ~/.claude/[type]/
     - file1.md
     - file2.md
   ```

### 플러그인 이름이 없는 경우

1. `~/.claude/plugins/installed_plugins.json` 읽기
2. 각 플러그인의 마켓플레이스에서 타입 확인
3. 설치된 플러그인 목록을 타입과 함께 표시:
   ```
   설치된 플러그인:

   | 플러그인 | 타입 | 마켓플레이스 |
   |----------|------|--------------|
   | toolkit-commands | commands | claude-toolkit-marketplace |
   | blog-writer | skill | claude-toolkit-marketplace |
   | codify | agent | claude-toolkit-marketplace |
   ```
4. 사용자에게 어떤 플러그인을 전역으로 설치할지 안내

## 주의사항
- 이미 존재하는 파일은 덮어씁니다 (업데이트 용도)
- 플러그인 업데이트 후 이 커맨드를 다시 실행하면 전역 파일도 업데이트됨
- skills는 폴더 전체가 복사됨 (SKILL.md 포함)
- agents는 .md 파일만 복사됨
