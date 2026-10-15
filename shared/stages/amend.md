# amend: 기존 Spec 수정

`done`인 Spec은 수정하지 않는다(대신 `relates_to`로 연결한 새 Spec을 만든다). 이 문서는 done이 아닌 Spec에만 적용한다.

## 1. 분류

- **질문 답변**: 미해결 질문에 대한 답
- **요구사항 변경**: REQ/AC 추가·수정·삭제, 목표/비목표 변경
- **설계·Sub-task 변경**: design.md 또는 Sub-task 추가·수정·순서 변경, blocked 해소를 위한 조정

## 2. 요구사항 변경

1. `spec.md`를 고치고 `## 변경 이력`에 `<날짜> <변경 요약>`을 추가한다.
2. 상태가 approved 이후라면 `status: draft`로 되돌리고 `## 변경 이력`에 `[되돌림] <이전>→draft`를 기록한다(`rules.md`의 이력 태그).
3. 영향받는 Sub-task(refs가 바뀐 REQ/AC를 가리키는 것)를 목록으로 보여준다. done Sub-task는 그대로 두고, 재작업은 재계획 때 새 Sub-task로 추가된다(`plan.md`의 재계획 규칙).
4. `/spec`에서 호출됐으면 `clarify.md`를 이어서 수행한다(재승인 → 재계획). `/execute`에서 호출됐으면 여기서 멈추고 `/spec <KEY>`로 재승인하라고 안내한다.

## 3. 설계·Sub-task 변경

1. 해당 파일을 고친다. 새 Sub-task는 최대 번호 + 1로 추가하고, 불필요해진 미완료 Sub-task는 `status: dropped`로 바꾼다.
2. 이 변경으로 원인이 해소된 `blocked` Sub-task는 `status: todo`로 되돌리고 blocked 메모를 지운다.
3. 요구사항은 그대로이므로 status는 유지한다. 단 in-review에서 todo Sub-task가 생기면 `status: in-progress`로 바꾼다(구현 재개). 단 design.md의 접근 방식, 변경 영역, 인터페이스가 실질적으로 바뀌었으면 `spec.md`의 `## 변경 이력`에 기록한다.
4. planned 이후라면:
   - 설계가 실질적으로 바뀌었고 plan-review 적용 대상이면 `plan-review.md`를 다시 수행한다(새 회차로 기록).
   - `gates.md`의 approved→planned 검사를 다시 수행하고, 결과를 보여준 뒤 AskUserQuestion으로 확인받는다.

## 4. 질문 답변

1. 질문을 `- [x] Qn … → 답: …`으로 바꾼다.
2. status가 draft면 `clarify.md`를 이어서 수행한다.
3. approved 이후라면, 답 때문에 REQ/AC/목표/비목표가 바뀌는 경우는 2절(요구사항 변경)로 처리한다. 바뀌지 않으면 기록만 하고 status를 유지한다.
