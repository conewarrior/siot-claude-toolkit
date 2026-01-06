# claude-toolkit

Claude Code용 스킬, 커맨드, 서브에이전트 라이브러리

## 설치 방법

### 방법 1: 마켓플레이스 (권장)

```bash
# 마켓플레이스 등록
/plugin marketplace add conewarrior/siot-claude-toolkit

# 전체 설치
/plugin install claude-toolkit
```

### 방법 2: 수동 설치

```bash
mkdir -p ~/.claude/commands
curl -o ~/.claude/commands/install-skill.md \
  https://raw.githubusercontent.com/conewarrior/siot-claude-toolkit/main/commands/install-skill.md
curl -o ~/.claude/commands/install-command.md \
  https://raw.githubusercontent.com/conewarrior/siot-claude-toolkit/main/commands/install-command.md
curl -o ~/.claude/commands/install-agent.md \
  https://raw.githubusercontent.com/conewarrior/siot-claude-toolkit/main/commands/install-agent.md
curl -o ~/.claude/commands/global-hooks.md \
  https://raw.githubusercontent.com/conewarrior/siot-claude-toolkit/main/commands/global-hooks.md
```

## 사용법

```bash
/install-skill pdf           # 스킬 설치
/install-command cleanup-ui  # 커맨드 설치
/install-agent code-reviewer # 에이전트 설치
/global-hooks                # Hook과 권한 설치
```

## 사용 가능한 스킬

| 스킬 | 설명 |
|------|------|
| `pdf` | PDF 조작 (추출, 병합, 분할, 폼) |
| `pptx` | PowerPoint 생성/편집/분석 |
| `canvas-design` | 시각 디자인/포스터/아트 생성 |
| `frontend-design` | 웹 프론트엔드 UI 생성 |
| `doc-coauthoring` | 문서 공동 작성 워크플로우 |

## 사용 가능한 커맨드

| 커맨드 | 설명 |
|--------|------|
| `/install-skill <name>` | 스킬 다운로드 및 설치 |
| `/install-command <name>` | 커맨드 다운로드 및 설치 |
| `/install-agent <name>` | 서브에이전트 다운로드 및 설치 |
| `/global-hooks` | Hook과 권한을 글로벌에 설치 |
| `/register-skill <name>` | 설치된 스킬의 Hook/권한 등록 |
| `/publish-command <name>` | 커맨드를 GitHub에 업로드 |
| `/publish-skill <name>` | 스킬을 GitHub에 업로드 |
| `/cleanup-ui` | UI 코드 최적화 |

## Hook 설정

`/global-hooks` 실행 시 설치되는 설정:

**자동 승인 권한:**
- Git: `git config`, `git add`, `git commit`, `git push`
- 스킬: `canvas-design`, `doc-coauthoring`, `frontend-design`, `pdf`, `pptx`

**음성 알림 Hook:**

| Hook | 트리거 | 음성 메시지 |
|------|--------|-------------|
| Stop | 작업 완료 시 | "야 일할 시간이야" |
| PreToolUse | 질문할 때 | "야 이거 좀 확인해봐" |
| Notification | bash 실행 시 | "bash다" |
| PermissionRequest | 권한 요청 시 | "어렵다 퍼미션" |
| SessionEnd | 세션 종료 시 | "아 뭐라 쳐야 하냐 세션엔드" |

**스킬 안내 Hook:**

| 스킬 | 키워드 |
|------|--------|
| canvas-design | poster, art, design, 포스터, 아트, 디자인... |
| doc-coauthoring | doc, proposal, spec, 문서, 제안서, 스펙... |
| frontend-design | web, page, React, 웹, 페이지, 프론트엔드... |
| pdf | pdf, PDF |
| pptx | pptx, presentation, 발표, 프레젠테이션... |

## 디렉토리 구조

```
claude-toolkit/
├── skills/           # 스킬 (SKILL.md + 리소스)
├── commands/         # 슬래시 커맨드 (.md)
├── agents/           # 서브에이전트 (.md)
├── hooks/            # Hook 설정 (hooks.json)
└── .claude-plugin/   # 마켓플레이스 메타데이터
```

## 업데이트

```bash
# 마켓플레이스
/plugin update claude-toolkit

# 수동
cd claude-toolkit && git pull
cp commands/*.md ~/.claude/commands/
```

## 문제 해결

### 커맨드가 인식되지 않는 경우
1. `~/.claude/commands/` 폴더 확인
2. md 파일들이 올바르게 복사되었는지 확인
3. Claude Code 재시작

### 스킬 설치 실패
- GitHub 저장소에 해당 스킬이 존재하는지 확인
- 네트워크 연결 확인
