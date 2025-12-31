# claude-toolkit

Claude Code용 스킬, 커맨드, 서브에이전트 라이브러리

## 최초 설정

bootstrap 폴더의 파일들을 ~/.claude/commands/에 복사:

```bash
mkdir -p ~/.claude/commands
curl -o ~/.claude/commands/install-skill.md \
  https://raw.githubusercontent.com/conewarrior/claude-toolkit/main/bootstrap/install-skill.md
curl -o ~/.claude/commands/install-command.md \
  https://raw.githubusercontent.com/conewarrior/claude-toolkit/main/bootstrap/install-command.md
curl -o ~/.claude/commands/install-agent.md \
  https://raw.githubusercontent.com/conewarrior/claude-toolkit/main/bootstrap/install-agent.md
```

## 사용법

```
/install-skill pdf
/install-command review
/install-agent code-reviewer
```

## 디렉토리 구조

- `skills/` - Claude Code 스킬들 (SKILL.md + 리소스)
- `commands/` - 슬래시 커맨드들 (.md 파일)
- `agents/` - 서브에이전트들 (.md 파일)
- `bootstrap/` - 글로벌 설치용 installer 커맨드들
