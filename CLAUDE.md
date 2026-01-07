# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 저장소 개요

Claude Code용 스킬, 커맨드, 서브에이전트 라이브러리. 플러그인 마켓플레이스로 배포되어 `/plugin`에서 개별 설치 가능.

## 디렉토리 구조

```
.claude-plugin/
└── marketplace.json     # 마켓플레이스 메타데이터 (플러그인 목록)

skills/                  # 개별 플러그인으로 등록된 스킬들
├── pdf/
│   ├── .claude-plugin/
│   │   └── plugin.json
│   └── SKILL.md
├── canvas-design/
├── pptx/
├── frontend-design/
└── doc-coauthoring/

commands/                # toolkit-commands 플러그인
├── .claude-plugin/
│   └── plugin.json
└── commands/            # 실제 커맨드 파일들
    ├── install-skill.md
    ├── publish-skill.md
    └── ...

agents/                  # 서브에이전트 (.md)
hooks/                   # Hook 설정 데이터 (hooks.json)
```

## 플러그인 마켓플레이스

마켓플레이스 등록 (settings.json):
```json
{
  "extraKnownMarketplaces": {
    "claude-toolkit-marketplace": {
      "source": {
        "source": "github",
        "repo": "conewarrior/siot-claude-toolkit"
      }
    }
  }
}
```

설치: `/plugin` → Discover 탭에서 선택 또는 `/plugin install <name>@claude-toolkit-marketplace`

## 스킬 구조

각 스킬은 `skills/<name>/` 폴더에 위치:

```
skills/<name>/
├── .claude-plugin/
│   └── plugin.json      # 플러그인 메타데이터
├── SKILL.md             # 스킬 본문 (frontmatter 필수)
└── ...                  # 추가 리소스
```

SKILL.md frontmatter:
```yaml
---
name: skill-name
description: 스킬 설명
license: 라이센스 정보
---
```

## 커맨드

| 커맨드 | 설명 |
|--------|------|
| `/install-skill <name>` | GitHub에서 스킬 다운로드 및 설치 |
| `/install-command <name>` | GitHub에서 커맨드 다운로드 및 설치 |
| `/publish-skill <name>` | 스킬을 GitHub에 업로드하고 마켓플레이스에 등록 |
| `/publish-command <name>` | 커맨드를 GitHub에 업로드 |
| `/global-hooks` | Hook과 권한을 글로벌에 설치 |

## 새 스킬 추가 시

1. `skills/<name>/SKILL.md` 생성 (frontmatter 필수)
2. `.claude-plugin/plugin.json` 생성 (또는 publish 시 자동 생성)
3. `/publish-skill <name>` 실행 → marketplace.json에 자동 등록
