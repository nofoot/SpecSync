# implement: Sub-task 실행 (planned / in-progress)

1. Spec frontmatter `links.blocked_by`의 Spec 중 done이 아닌 것이 있으면 알리고 AskUserQuestion으로 진행 여부를 묻는다. `status`가 planned면 in-progress로 바꾼다.
2. `status: blocked`인 Sub-task가 있으면 먼저 처리한다. `> blocked:` 사유를 보여주고 AskUserQuestion으로 묻는다.
   - **DB 실행 완료**(`> blocked: DB 실행 대기`인 경우): 결과 요약을 받아 `> db:`를 기록하고, `> blocked:` 줄만 지운 뒤 `status: in-progress`로 바꿔 멈춘 지점부터 이어서 진행한다. 이미 작성한 쿼리 파일과 구현은 다시 만들지 않는다.
   - **재시도**: 원인이 해소됨(사용자가 직접 고침, 환경 문제 해결 등) → `status: todo`, `> blocked:` 줄 삭제(`> db:`·`> verified:` 기록은 남긴다)
   - **Spec 수정**: `amend.md`로 요구사항·설계·Sub-task를 조정
   - **보류**: 이번에는 건너뛰고 다른 Sub-task를 진행
3. 실행할 Sub-task를 고른다. 사용자가 특정 Sub-task 키를 지정했으면 그것을 고르되, status가 todo/in-progress가 아니거나 `blocked_by`가 충족되지 않았으면 그 사실을 알리고 AskUserQuestion으로 진행 여부를 묻는다. 지정이 없으면 다음 순서로 첫 번째:
   1. `status: in-progress` (이전 세션에서 중단된 것)
   2. `status: todo`이고 `blocked_by`가 모두 done(또는 dropped)인 것, 키 순서

   고를 수 있는 Sub-task가 없으면 이유(모두 blocked, 의존성 대기 등)를 보고하고 멈춘다.
4. 고른 Sub-task를 수행한다.
   - `status: in-progress`로 바꾼다.
   - `spec.md`의 해당 REQ/AC, `design.md`를 읽고 구현한다. 범위는 refs와 비목표를 넘지 않는다.
   - DB 조회·변경이 필요하면 직접 실행하지 않고 `rules.md`의 `## 데이터베이스` 쿼리 전달 절차를 따른다.
   - `verify` 명령을 실행한다. 실패하면 고쳐서 다시 실행한다. `manual:` verify는 쿼리나 절차를 사용자에게 주고 결과를 받아 `gates.md`의 `manual:` verify 형식으로 기록한다.
   - 통과하면 `status: done`.
   - 아래 중단 조건에 해당하면 즉시 멈춘다.
5. 커밋은 principles의 `commit` 정책을 따른다. 메시지 첫 줄은 `<Sub-task 키> <제목>`으로 쓴다.
   - `ask`: AskUserQuestion으로 커밋 여부를 묻는다.
   - `auto`: 바로 커밋한다.
   - `off`: 커밋하지 않는다.
6. 중단 조건. 해당하면 구현을 멈추고 Sub-task를 `status: blocked`로 바꾼 뒤, Sub-task 아래에 `> blocked: <사유>`를 적고 보고한다. Spec을 바꿔야 하는 경우에도 변경안을 먼저 AskUserQuestion으로 결정받은 뒤에만 `amend.md`를 수행한다.

   | 조건 | 이후 처리 |
   |---|---|
   | `verify` 실패를 2회 수정 시도에도 해결하지 못함 | 실패 출력과 시도 내용을 보고 |
   | design.md에 없는 인터페이스(API, 스키마, 설정, 이벤트) 변경이 필요해짐 | 변경안 결정 → `amend.md` 3절 |
   | 요구사항·AC·비목표를 바꿔야 구현할 수 있음 | 변경안 결정 → `amend.md` 2절 (재승인) |
   | principles의 `protected_paths`에 속한 파일을 수정해야 함(Sub-task 아래 `> allowed: <경로>` 기록이 있는 경로는 제외) | 수정 허용 여부를 결정받는다. 허용되면 Sub-task 아래에 `> allowed: <경로>`를 기록하고 재시도, 아니면 대안을 `amend.md` 3절로 |
   | DB 조회·변경이 필요한데 사용자가 아직 실행하지 않았거나 실행을 거부함(`rules.md`의 `## 데이터베이스`) | 쿼리 전달 절차 3항에 따라 기록 |
   | 다른 Sub-task와 충돌(같은 코드를 상반되게 변경, 선행 Sub-task 결과가 전제와 다름) | 조정안 결정 → `amend.md` 3절 |
7. Sub-task 1개를 끝내면 멈추고 진행 현황(done/전체, dropped 제외)을 보고한다. 남은 Sub-task가 있으면 다음 `/execute <KEY>` 실행 시 이어서 진행된다고 알린다.
8. dropped를 제외한 모든 Sub-task가 done이면 **모든 Sub-task의 `verify`를 다시 실행**한 뒤(`manual:` verify는 `gates.md`의 유효 조건을 만족하는 `> verified:` 기록으로 대신하고, 유효하지 않으면 다시 확인받는다) `gates.md`의 in-progress→in-review 검사를 수행한다. 통과하면 `status: in-review`로 바꾸고 `review.md`를 이어서 수행한다. 실패한 verify가 있으면 해당 Sub-task를 `todo`로 되돌리고 보고한다.
