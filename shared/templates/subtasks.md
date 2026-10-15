# {{key}} Sub-tasks

<!-- 규칙
- Sub-task 1개 = 에이전트 세션 1회 = 커밋 1개.
- 필드는 `- key: value` 형식을 유지한다 (게이트 검사가 파싱함).
  - status: todo | in-progress | done | blocked | dropped (dropped는 사유를 한 줄 적는다)
  - refs: 이 Sub-task가 충족하는 REQ / AC ID (쉼표 구분)
  - blocked_by: 선행 Sub-task 키 (없으면 —)
  - files: 주로 변경할 파일 (쉼표 구분)
  - verify: 완료를 확인하는 명령. 필수. 에이전트가 실행할 수 없는 확인(DB 상태 등)은 `manual: <확인 쿼리 파일 또는 절차>`로 쓴다.
- 모든 AC는 1개 이상의 Sub-task refs에 포함되어야 planned로 전이할 수 있다.
-->

### {{key}}.1 {{Sub-task 제목}}
- status: todo
- refs: REQ-1, AC-1.1
- blocked_by: —
- files:
- verify: ``

<!-- 구현 메모 1~3줄. -->
