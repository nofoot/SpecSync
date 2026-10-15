# plan-review: 에이전트 패널 계획 검토

`plan.md`에서 design.md와 Sub-task를 작성한 뒤, approved→planned 게이트 전에 수행한다.

## 적용 대상

- story: 항상
- bug: design.md가 있을 때만
- task, epic: 생략
- principles의 `plan_review: off`이면 생략

## 1. 검토자 결정

1. principles의 `reviewers`에 에이전트 이름 목록이 있으면 그것을 쓴다.
2. 없으면 프로젝트 `.claude/agents/*.md` 중 description이 계획·설계·품질·테스트·리뷰 관점인 에이전트를 고른다(최대 4명). frontmatter `tools`에 Write, Edit, NotebookEdit, Bash가 포함되거나 `tools`가 없어 모든 도구를 쓰는 에이전트는 자동 선택하지 않는다(파일 수정과 DB 접속을 막기 위해서다).
3. 둘 다 없으면 플러그인 기본 검토자를 쓴다: `specsync:design-reviewer`, `specsync:test-reviewer`, `specsync:scope-reviewer`.

사용할 검토자 목록을 한 줄로 알린다.

## 2. 검토 실행

- 각 검토자를 **Agent 도구로 실제 호출**한다. 메인 에이전트가 역할극으로 검토를 흉내 내지 않는다(에이전트별 모델·도구 제한이 적용되어야 한다).
- 독립적인 검토자는 한 메시지에서 병렬로 호출한다.
- 각 검토자에게 전달할 것:
  - Spec 폴더 경로(spec.md, design.md, subtasks.md)와 `.specs/principles.md`
  - 자기 관점에서 실제 코드베이스를 확인하며 검토할 것. **파일을 수정하지 말 것**
  - principles의 `db_access`가 `direct`가 아니면 **DB에 접속하거나 DB CLI·MCP 도구를 쓰지 말 것**. 스키마는 코드와 마이그레이션 파일에서 확인할 것
  - 반환 형식: 지적 사항마다 `심각도(차단 | 주요 | 경미) / 대상(REQ·AC·Sub-task 키 또는 design 섹션) / 내용 / 제안`

## 3. 반영

검토 결과를 메인 에이전트가 분류해 처리한다.

| 분류 | 처리 |
|---|---|
| 표현 보완, 누락된 리스크·테스트·Sub-task, verify 명령 보강 | design.md / subtasks.md에 자동 반영 |
| 의사결정성 변경(범위, 방향, 기능 제거, REQ/AC 의미) | AskUserQuestion으로 하나씩 결정받는다. 요구사항을 바꾸기로 하면 `amend.md` 2절 |
| 차단 이슈(계획의 전제를 뒤집는 결함, 아키텍처 충돌) | 아래 해결 절차 |

**차단 이슈 해결 절차**
1. 이슈와 해결안 후보(2~3개)를 AskUserQuestion으로 제시해 결정받는다.
2. 결정을 반영한다. 요구사항이 바뀌면 `amend.md` 2절로 간다(재승인 후 재계획).
3. 이슈를 낸 검토자를 다시 호출해 해소 여부를 확인받는다.
4. `## 계획 검토`의 `blocking_open`을 갱신한다.

**카운터 갱신:** `blocking_open`은 해소되지 않은 차단 이슈 수, `pending_decisions`는 아직 사용자가 결정하지 않은 의사결정성 항목 수다. 검토 직후 두 값을 기록하고, 이슈가 해소되거나 결정이 내려질 때마다 마지막 회차의 값을 갱신한다.

## 4. 기록

design.md 끝의 `## 계획 검토` 섹션에 기록한다. 재검토 시에는 기존 기록 아래에 새 회차를 덧붙인다. `blocking_open`과 `pending_decisions`는 게이트가 읽는 필드이므로 형식을 지킨다.

```markdown
## 계획 검토

### 1차 (YYYY-MM-DD) — design-reviewer, test-reviewer, scope-reviewer
- blocking_open: 0
- pending_decisions: 0
- 반영: [test-reviewer] SS-12.4 추가 — 잠금 해제 경계값 테스트
- 결정: [scope-reviewer] IP 차단 포함 제안 → 사용자 결정: 제외 유지
- 차단 해소: [design-reviewer] 세션 저장소 전제 오류 → Redis 사용으로 변경
```

판정은 마지막 회차의 `blocking_open`, `pending_decisions` 값을 쓴다.

## 5. 요약 보고

검토자별 핵심 지적 1~2줄, 자동 반영 건수, 사용자 결정 건수, 남은 차단 이슈를 보고한 뒤 `plan.md`의 게이트로 돌아간다.
