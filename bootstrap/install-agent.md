---
description: GitHub에서 서브에이전트를 현재 프로젝트에 설치
arguments:
  - name: agent_name
    description: 설치할 에이전트 이름
    required: true
---

GitHub 저장소 https://github.com/conewarrior/claude-toolkit 의
agents/$ARGUMENTS.agent_name.md 파일을 현재 프로젝트의 .claude/agents/에 설치해줘.
