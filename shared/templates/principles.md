---
key_prefix: {{prefix}}
commit: ask        # ask | auto | off — Sub-task 완료 시 커밋 정책
plan_review: on    # on | off — 설계·Sub-task 작성 후 에이전트 패널 검토
reviewers: []      # 계획 검토 에이전트 이름(플러그인 에이전트는 specsync:design-reviewer 형식). 비우면 .claude/agents/의 읽기 전용 에이전트 → 플러그인 기본 검토자 순
impl_review: on    # on | off — 구현 완료 후 독립 에이전트의 구현 검토
impl_reviewers: [] # 구현 검토 에이전트 이름. 비우면 specsync:implementation-reviewer
protected_paths: [] # 구현 중 수정 금지 경로(glob). 수정이 필요하면 멈추고 사용자 결정을 받음
---

# Principles

<!-- SDD 전용 규칙만 둔다. 일반 코딩 컨벤션과 기술 스택은 CLAUDE.md에 둔다. -->

## 검증 명령

<!-- Sub-task의 verify 작성 기준이 되는 명령. -->

- 테스트:
- 린트:
- 빌드:

## 품질 게이트

<!-- 모든 Spec에 공통으로 요구하는 조건. 예: 공개 API 변경 시 문서 갱신, 새 로직은 단위 테스트 필수. -->

-

## 컴포넌트

<!-- Spec frontmatter의 components에 쓰는 이름 목록. -->

-

## Spec 작성 규칙

<!-- 프로젝트 고유 규칙. 예: 성능 요구사항은 수치로, UI 변경은 스크린샷 첨부. -->

-
