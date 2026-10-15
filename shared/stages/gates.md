# gates: 상태 전이 검사

## 파싱 규칙

- 먼저 HTML 주석(`<!-- … -->`)을 모두 제거한 뒤 검사한다.
- 기본적으로 섹션 제목 문구가 아니라 ID와 필드 키로 판단한다: `### REQ-n`, `- AC-n.m`, `- [ ] Qn`, `### <KEY>.n`, `- status|refs|blocked_by|files|verify: 값`, `[regression]` 태그.
- 값 없음으로 보는 것: 빈 값, `—`, `-`, 빈 코드 스팬(``` `` ```).
- Sub-task `status` 값: `todo | in-progress | done | blocked | dropped`. dropped는 모든 검사에서 제외한다.
- 예외적으로 섹션을 기준으로 읽는 것(제목을 바꾸지 않는다):
  - task 타입의 Sub-task: `spec.md`의 `## Sub-tasks` 아래. 그 외 타입은 `subtasks.md`.
  - epic의 하위 Spec: `## 하위 Spec` 표의 행.
  - 계획 검토 결과: design.md `## 계획 검토`의 마지막 회차에 있는 `- blocking_open: n`, `- pending_decisions: n` 필드.
  - 구현 검토 결과: spec.md `## 구현 검토`의 마지막 회차에 있는 같은 두 필드.
- `manual:` verify: `verify: manual: <확인 쿼리 파일 또는 절차>` 형식으로, 에이전트가 실행할 수 없어 사용자가 확인하는 verify다. 확인 결과는 Sub-task 아래 `> verified: <YYYY-MM-DD HH:MM> 사용자 확인 — <결과 요약>`으로 기록한다. 관련 파일(이 Sub-task와 `blocked_by`로 이어진 선행 Sub-task의 `files`, 마이그레이션·스키마 정의 파일, Spec의 `queries/`)이 기록 시각 이후 바뀌지 않았으면 유효하다. 다음 중 하나라도 해당하면 유효하지 않다.
  - 기록 시각 이후의 커밋 중 이 Sub-task 자신의 커밋(메시지 첫 줄이 이 Sub-task 키)을 뺀 커밋이 관련 파일을 바꿨다.
  - 커밋되지 않은 관련 파일의 수정 시각이 기록 시각보다 늦다.

## draft → approved

공통
- [ ] `- [ ]` 미해결 질문이 0개

story / bug / task
- [ ] REQ가 1개 이상, 모든 REQ에 AC가 1개 이상 (예외 불가)
- [ ] AC ID가 소속 REQ 번호와 일치(REQ-2 아래에는 AC-2.m) (예외 불가)
- [ ] bug: `## 재현 절차`에 단계가 1개 이상이고, `[regression]` 태그가 붙은 AC가 1개 이상

epic
- [ ] `## 하위 Spec` 표에 후보가 1개 이상
- [ ] `## 완료 조건`에 항목이 1개 이상

## approved → planned

story / bug / task
- [ ] 모든 AC가 1개 이상의 Sub-task `refs`에 포함 (예외 불가)
- [ ] 모든 Sub-task에 `verify` 값이 있음 (예외 불가)
- [ ] `refs`가 존재하는 REQ/AC만 가리킴 (예외 불가)
- [ ] `blocked_by`가 존재하는 Sub-task(dropped 포함, dropped는 충족으로 간주)만 가리키고 순환이 없음 (예외 불가)
- [ ] story: design.md가 있고 검증 전략 표에 모든 AC가 있음
- [ ] 계획 검토 대상이면(`plan-review.md`의 적용 대상) `## 계획 검토` 마지막 회차가 있고 `blocking_open: 0`, `pending_decisions: 0`

epic
- [ ] `## 하위 Spec` 표의 모든 행에 키가 있고, 해당 `.specs/<KEY>-*/spec.md`가 존재하며 `parent`가 epic 키 (예외 불가)

## in-progress → in-review

- [ ] dropped를 제외한 모든 Sub-task가 `status: done` (예외 불가)
- [ ] 이번 실행에서 모든 Sub-task의 `verify`를 다시 실행해 통과함. `manual:` verify는 유효한 `> verified:` 기록이 있음(위 파싱 규칙) (예외 불가)

## in-review → done

story / bug / task
- [ ] dropped를 제외한 모든 Sub-task가 `status: done` (예외 불가)
- [ ] 모든 AC에 충족 근거가 있음
- [ ] principles의 `impl_review`가 off가 아니면 `## 구현 검토` 마지막 회차가 있고 `blocking_open: 0`, `pending_decisions: 0`

epic
- [ ] `## 하위 Spec` 표의 모든 하위 Spec이 `status: done` (예외 불가)
- [ ] `## 완료 조건`의 모든 항목에 충족 근거가 있음

검사 결과는 항목별 통과/실패로 보여준다. `(예외 불가)` 항목은 추적성과 파싱의 전제이므로 반드시 통과해야 한다. 그 밖의 실패 항목은 `rules.md`의 `### 게이트 예외` 절차로만 건너뛸 수 있다.
