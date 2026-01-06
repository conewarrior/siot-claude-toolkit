# claude-toolkit

Claude Code용 스킬, 커맨드, 서브에이전트 라이브러리

## 최초 설정

commands 폴더의 파일들을 ~/.claude/commands/에 복사:

```bash
mkdir -p ~/.claude/commands
curl -o ~/.claude/commands/install-skill.md \
  https://raw.githubusercontent.com/conewarrior/claude-toolkit/main/commands/install-skill.md
curl -o ~/.claude/commands/install-command.md \
  https://raw.githubusercontent.com/conewarrior/claude-toolkit/main/commands/install-command.md
curl -o ~/.claude/commands/install-agent.md \
  https://raw.githubusercontent.com/conewarrior/claude-toolkit/main/commands/install-agent.md
curl -o ~/.claude/commands/global-hooks.md \
  https://raw.githubusercontent.com/conewarrior/claude-toolkit/main/commands/global-hooks.md
```

## 사용법

```
/install-skill pdf
/install-command review
/install-agent code-reviewer
/global-hooks
```

## 디렉토리 구조

- `skills/` - Claude Code 스킬들 (SKILL.md + 리소스)
- `commands/` - 슬래시 커맨드들 (.md 파일)
- `agents/` - 서브에이전트들 (.md 파일)
- `hooks/` - Hook 설정 데이터 (hooks.json)
