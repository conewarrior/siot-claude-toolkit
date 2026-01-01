# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 저장소 개요

Claude Code용 스킬, 커맨드, 서브에이전트 라이브러리. 다른 프로젝트에서 `/install-skill`, `/install-command`, `/install-agent` 명령어로 설치하여 사용.

## 디렉토리 구조

```
skills/           # 스킬 (SKILL.md + 리소스 파일들)
commands/         # 슬래시 커맨드 (.md)
agents/           # 서브에이전트 (.md)
bootstrap/        # 글로벌 설치용 installer 커맨드들
```

## 스킬 구조

각 스킬은 `skills/<name>/` 폴더에 위치하며 반드시 `SKILL.md` 파일 포함:

```yaml
---
name: skill-name
description: 스킬 설명 (Hook matcher 키워드 추출용)
license: 라이센스 정보
---
# 스킬 내용
```

### 사용 가능한 스킬

- `canvas-design` - 시각 디자인/포스터/아트 생성 (.png, .pdf)
- `doc-coauthoring` - 문서 공동 작성 워크플로우
- `frontend-design` - 웹 프론트엔드 UI 생성
- `pdf` - PDF 조작 (추출, 병합, 분할, 폼)
- `pptx` - PowerPoint 생성/편집/분석

## Bootstrap 커맨드

`bootstrap/` 폴더의 .md 파일들은 `~/.claude/commands/`에 복사하여 글로벌로 사용:

| 커맨드 | 설명 |
|--------|------|
| `/install-skill <name>` | GitHub에서 스킬 다운로드 및 설치 |
| `/install-command <name>` | GitHub에서 커맨드 다운로드 및 설치 |
| `/install-agent <name>` | GitHub에서 서브에이전트 다운로드 및 설치 |
| `/install-hooks` | 미리 설정된 Hook과 권한 설치 |
| `/register-skill <name>` | 설치된 스킬의 Hook과 권한 등록 |

## Hook 설정

`bootstrap/hooks.json`에 스킬별 Hook과 권한 정의:
- `permissions.allow`: 자동 승인할 권한 목록
- `hooks.UserPromptSubmit`: 키워드 기반 스킬 안내 Hook

## 새 스킬 추가 시

1. `skills/<name>/SKILL.md` 생성 (frontmatter 필수)
2. 필요한 리소스 파일 추가
3. `bootstrap/hooks.json`에 권한과 Hook 추가
4. `/register-skill <name>`으로 테스트
