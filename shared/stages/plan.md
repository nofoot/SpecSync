# plan: 설계와 Sub-task 작성 (approved → planned)

## 기존 파일이 있을 때 (재계획)

요구사항 변경으로 재승인된 Spec처럼 design.md, subtasks.md(또는 task의 `## Sub-tasks`)가 이미 있으면 새로 만들지 않고 **수정만 한다.**
- `done` Sub-task와 기존 키는 그대로 둔다. 커밋이 키를 참조한다.
- 변경된 REQ/AC 때문에 재작업이 필요하면 새 Sub-task를 최대 번호 + 1로 추가한다.
- 더 이상 필요 없는 미완료 Sub-task는 지우지 말고 `status: dropped`로 바꾸고 사유를 한 줄 적는다.
- `## 계획 검토` 기록은 유지하고 새 검토 결과를 아래에 덧붙인다.

## story / bug

1. 관련 코드를 조사한다.
2. `templates/design.md`로 `design.md`를 작성한다. bug는 원인과 수정 방법이 자명하면 생략할 수 있다.
3. `templates/subtasks.md`로 `subtasks.md`를 작성한다.
   - Sub-task 1개 = 세션 1회 = 커밋 1개. 서로 무관한 영역은 나눈다.
   - 각 Sub-task에 `refs`, `blocked_by`, `files`, `verify`를 채운다. `verify`에는 principles의 검증 명령을 기준으로 실제 실행 가능한 명령을 쓴다.
   - Sub-task가 15개를 넘으면 epic으로 나누자고 제안한다.
4. `plan-review.md`를 수행한다(적용 대상이 아니면 생략).

## task

`spec.md`의 `## Sub-tasks` 섹션을 채운다. design.md와 subtasks.md는 만들지 않는다. Sub-task가 3개를 넘으면 story로 바꾸자고 제안한다.

## epic

1. `## 하위 Spec` 후보 목록(제목, 타입)을 보여주고 AskUserQuestion으로 확인받는다. 이것이 epic의 →planned 게이트다. 수정 요청이면 목록을 고친 뒤 다시 묻는다.
2. 확인되면 각 후보에 대해 `create.md`의 1~2절만 수행해 하위 Spec 파일을 만든다(`parent`는 epic 키). 하위 Spec의 clarify로 이어가지 않는다. 각자 draft에서 시작한다.
3. 표에 키와 상태를 채운다.
4. `gates.md`의 approved→planned 검사를 수행하고 통과하면 `status: planned`. 먼저 작성할 하위 Spec을 `/spec <하위 키>`로 추천한다.

## 게이트 (story / bug / task)

1. `gates.md`의 approved→planned 검사를 수행한다. 실패하면:
   - 기계적 결함(refs 누락, verify 누락, 존재하지 않는 키 참조, 순환)은 고친다.
   - 계획 검토의 차단 이슈나 미결정 항목은 `plan-review.md` 3절의 해결 절차를 따른다.
2. 설계 요지와 Sub-task 목록(키, 제목, refs, blocked_by)을 보여주고 AskUserQuestion으로 확인받는다.
3. 확인되면 `status: planned`. 수정 요청이면 반영 후 1부터 반복한다.
