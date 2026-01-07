---
description: 자주 쓰는 명령어 모음
---

# siot-claude-toolkit 치트시트

## 마켓플레이스

```
/plugin marketplace add conewarrior/siot-claude-toolkit   # 마켓플레이스 등록
/plugin install claude-toolkit@claude-toolkit-marketplace # 플러그인 설치
/plugin marketplace update claude-toolkit-marketplace     # 최신 버전으로 업데이트
/plugin marketplace remove claude-toolkit-marketplace     # 마켓플레이스 제거
/plugin                                                   # 플러그인 UI 열기
```

## 개별 설치

```
/install-skill <name>      # GitHub에서 스킬 다운로드 + Hook/권한 자동 등록
/install-command <name>    # GitHub에서 커맨드 다운로드
/install-agent <name>      # GitHub에서 에이전트 다운로드
/global-hooks              # Hook과 권한을 글로벌에 설치
/register-skill <name>     # 직접 만든 로컬 스킬의 Hook/권한 등록
```

## 업로드 (기여자용)

```
/publish-skill <name>      # 스킬을 GitHub에 업로드
/publish-command <name>    # 커맨드를 GitHub에 업로드
```

## 사용 가능한 스킬

```
pdf              # PDF 조작 (추출, 병합, 분할, 폼)
pptx             # PowerPoint 생성/편집/분석
canvas-design    # 시각 디자인/포스터/아트 생성
frontend-design  # 웹 프론트엔드 UI 생성
doc-coauthoring  # 문서 공동 작성 워크플로우
```

## 사용 가능한 커맨드

```
cleanup-ui       # UI 코드 최적화
cheatsheet       # 이 치트시트 보기
```
