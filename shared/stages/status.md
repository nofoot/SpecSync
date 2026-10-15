# status: 현황

## 전체

Spec 목록을 epic 기준으로 묶어 표로 보여준다.

| 키 | 타입 | 제목 | 상태 | 진행 |
|---|---|---|---|---|

- 진행: Sub-task done/전체(dropped 제외). epic은 하위 Spec done/전체.
- `done`은 기본적으로 숨기고, 사용자가 전체를 원하면 포함한다.
- blocked인 Sub-task가 있으면 표 아래에 키와 사유를 적는다.
- frontmatter `links.blocked_by`의 Spec이 done이 아니면 상태 옆에 "대기: <키>"를 표시한다.
- 끝에 다음에 진행할 만한 Spec 하나를 실행할 커맨드와 함께 추천한다(대기 중인 Spec 제외, in-progress 우선, 그다음 priority 순).

## Spec 하나

해당 Spec의 상태, 미해결 질문 수, Sub-task 목록(키, 제목, status, blocked 사유), 마지막 계획 검토 결과, (in-review 이후면) 마지막 구현 검토 결과를 보여주고, 다음에 실행할 커맨드와 진행될 단계를 알려준다(draft·approved는 `/spec <KEY>`, planned 이후는 `/execute <KEY>`). epic이면 하위 Spec 표를 보여준다.
