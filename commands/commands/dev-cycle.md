---
description: PRD부터 개발, 리뷰, 학습까지 전체 개발 사이클 자동 실행
allowed-tools: Task, Read, Write, Bash, Grep, Glob
---

# Full Development Cycle Orchestrator

## Usage
```
/dev-cycle <PRD 경로>
/dev-cycle docs/PRD.md
```

## Execution Flow

### Phase 1: PRD → Development Guide
```
1. prd-to-dev 서브에이전트 실행
2. Input: $ARGUMENTS (PRD 경로)
3. Output: docs/DEVELOPMENT.md
4. 완료 확인 후 다음 단계
```

### Phase 2: 병렬 개발
```
1. DEVELOPMENT.md 읽기
2. PARALLEL 블록 내 태스크 추출
3. 각 Phase별로:
   - 병렬 가능 태스크들을 동시에 서브에이전트로 실행
   - 모든 서브에이전트 완료 대기
   - 다음 Phase로 진행
4. Output: 구현된 코드
```

### Phase 3: 리뷰 & 수정
```
1. reviewer 서브에이전트 실행
2. 전체 변경 파일 검토
3. 자동 수정 가능한 항목 수정
4. Output: docs/REVIEW_REPORT.md
```

### Phase 4: 학습 명문화
```
1. codify 서브에이전트 실행
2. Input: DEVELOPMENT.md, REVIEW_REPORT.md
3. Output: CLAUDE.md 업데이트
```

## Progress Output

각 단계에서 진행 상황 출력:

```
🚀 Development Cycle Started
   PRD: docs/PRD.md

📋 Phase 1: Generating Development Guide...
   ✅ Created docs/DEVELOPMENT.md
   Found 3 parallel phases, 8 total tasks

⚡ Phase 2: Parallel Development
   Phase 2-1: Starting 3 parallel tasks...
   [Task 1-A] 🔄 인증 모듈...
   [Task 1-B] 🔄 DB 스키마...
   [Task 1-C] 🔄 API 라우트...
   [Task 1-B] ✅ 완료
   [Task 1-A] ✅ 완료
   [Task 1-C] ✅ 완료

   Phase 2-2: Starting 2 parallel tasks...
   ...

🔍 Phase 3: Code Review
   Reviewing 24 files...
   ✅ Fixed 5 issues automatically
   ⚠️ 2 suggestions for manual review
   Created docs/REVIEW_REPORT.md

📚 Phase 4: Codifying Learnings
   ✅ Added 3 new rules to CLAUDE.md

✨ Development Cycle Complete!
   Total time: 12m 34s
   Files created/modified: 28
   Learnings codified: 3
```

## Error Handling

- 서브에이전트 실패 시: 해당 태스크 재시도 (최대 2회)
- 병렬 작업 충돌 시: 충돌 파일 리스트업 후 순차 처리로 전환
- 전체 실패 시: 진행 상황 저장 후 재개 가능하도록 체크포인트 생성
