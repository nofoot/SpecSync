# clarify: 질문 해소와 승인 (draft → approved)

1. `## 미해결 질문`에 `- [ ]` 항목이 있으면 AskUserQuestion으로 묻는다. 한 번에 최대 4개, 가능하면 선택지를 제시한다.
2. 답변을 반영한다.
   - 질문을 `- [x] Qn … → 답: …`으로 바꾼다.
   - 답에 따라 요구사항·AC·비목표를 수정한다.
3. `gates.md`의 draft→approved 검사를 수행한다. 실패하면:
   - 기계적으로 고칠 수 있는 항목(AC 번호 불일치, 누락된 `[regression]` 태그 등)은 고친다.
   - 사용자 판단이 필요한 항목(AC를 정할 수 없는 REQ, 재현 절차 불명 등)은 `- [ ] Qn`으로 추가하고 1로 돌아간다.
4. 통과하면 Spec 요약을 보여주고 AskUserQuestion으로 승인을 묻는다.
   - story/bug/task: 목표, REQ/AC 목록, 비목표
   - epic: 목표, 범위, 하위 Spec 후보, 완료 조건
5. 승인되면 `status: approved`, `approved_at: <오늘>`로 바꾸고 `plan.md`를 이어서 수행한다. 수정 요청이면 반영 후 3부터 반복한다.
