---
description: Development Guide의 병렬 작업을 서브에이전트로 동시 실행
allowed-tools: Task, Read, Write, Bash, Grep, Glob
---

# Parallel Development Executor

## Input
- Development Guide 경로 (기본: docs/DEVELOPMENT.md)

## Process

### 1. 파싱
DEVELOPMENT.md에서 `<!-- PARALLEL_START/END -->` 블록 추출

### 2. 실행 전략
```python
for phase in phases:
    if phase.is_parallel:
        # 모든 태스크를 동시에 Task 도구로 실행
        spawn_all_subagents_simultaneously(phase.tasks)
        wait_for_all()
    else:
        # 순차 실행
        for task in phase.tasks:
            execute_and_wait(task)
```

### 3. 서브에이전트 프롬프트 템플릿
```
You are implementing: [Task Name]

## Context
- PRD: docs/PRD.md (참고용)
- Development Guide: docs/DEVELOPMENT.md

## Your Scope
Files you CAN modify:
[file paths from task definition]

Files you CANNOT touch:
[everything else]

## Task Details
[task description from DEVELOPMENT.md]

## Completion Criteria
[checklist from task definition]

## Rules
1. 범위 밖 파일 절대 수정 금지
2. 완료 후 구현 내용 요약 필수
3. 막히면 중단하고 이유 보고
```

### 4. 결과 수집
각 서브에이전트 완료 시:
- 성공: 구현 요약 기록
- 실패: 에러 내용 기록, 재시도 또는 수동 처리 플래그
