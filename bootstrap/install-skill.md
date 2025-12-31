---
description: GitHub에서 스킬을 현재 프로젝트에 설치
arguments:
  - name: skill_name
    description: 설치할 스킬 이름
    required: true
---

GitHub 저장소 https://github.com/conewarrior/claude-toolkit 의
skills/$ARGUMENTS.skill_name 폴더를 현재 프로젝트의 .claude/skills/에 설치해줘.

1. .claude/skills/ 디렉토리 없으면 생성
2. 해당 스킬 폴더 다운로드해서 복사
3. 완료 후 SKILL.md의 name과 description 보여줘
