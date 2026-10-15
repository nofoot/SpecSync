# review: 검토와 완료 (in-review → done)

## story / bug / task

1. 모든 Sub-task의 `verify`를 다시 실행한다.
2. 구현 검토를 수행한다. principles의 `impl_review: off`이면 생략하고 3~5에서 메인 에이전트가 직접 AC 근거와 불일치를 점검한다(이때 4의 기록은 하지 않는다).
   1. 검토자를 정한다. principles의 `impl_reviewers`에 에이전트 이름 목록이 있으면 그것을, 없으면 `specsync:implementation-reviewer`를 쓴다. 사용할 검토자를 한 줄로 알린다. 지정된 에이전트의 frontmatter `tools`에 Write, Edit, NotebookEdit가 있거나 `tools`가 없으면 파일을 수정할 수 있다고 함께 경고한다.
   2. 변경 범위를 정한다. 커밋 메시지 첫 줄이 이 Spec의 Sub-task 키(`<KEY>.`)로 시작하는 커밋 목록을 기본으로 하고, 커밋하지 않은 변경이 있으면 작업 트리 변경도 포함한다.
   3. 각 검토자를 **Agent 도구로 실제 호출**한다. 메인 에이전트가 역할극으로 검토를 흉내 내지 않는다. 여러 명이면 한 메시지에서 병렬로 호출한다. 전달할 것: Spec 폴더 경로, `.specs/principles.md`, 변경 범위(커밋 해시 또는 "작업 트리"), **파일을 수정하지 말 것**, 반환 형식(AC 판정 `AC-n.m / 충족 | 미충족 | 불확실 / 근거`, 지적 `심각도(차단 | 주요 | 경미) / 대상 / 내용 / 제안`).
3. AC 충족 표를 만든다. 각 AC가 어떤 테스트나 동작으로 충족되는지 근거(파일:줄, 테스트 이름)를 적는다. 구현 검토를 했으면 검토자의 AC 판정을 근거로 쓰고, `불확실` 판정은 메인 에이전트가 직접 확인해 충족·미충족으로 정한다.
4. 구현 검토를 했으면 5의 처리 전에 `spec.md` 끝의 `## 구현 검토` 섹션에 회차를 기록한다. 재검토 시에는 기존 기록 아래에 새 회차를 덧붙인다. `blocking_open`은 해소되지 않은 차단 지적 수, `pending_decisions`는 아직 사용자가 결정하지 않은 의사결정성 항목 수다. 두 필드는 게이트가 읽으므로 형식을 지킨다. 이후 지적이 해소되거나 결정이 내려질 때마다 이 회차의 값과 `반영:`/`결정:` 줄을 갱신한다.

   ```markdown
   ## 구현 검토

   ### 1차 (YYYY-MM-DD) — implementation-reviewer, 범위: a1b2c3d..e4f5a6b
   - blocking_open: 1
   - pending_decisions: 0
   - 반영: [implementation-reviewer] AC-2.1 미충족 → SS-12.5 추가 — 잠금 해제 경계값 처리
   - 결정: [implementation-reviewer] 로그 포맷 변경(범위 초과) → 사용자 결정: Spec에 반영
   ```
5. 검토 결과와 불일치를 분류해 처리한다. 의사결정성 항목은 AskUserQuestion으로 하나씩 결정받는다. design.md가 없는 Spec은 design.md 관련 행을 건너뛴다.

   | 분류 | 처리 |
   |---|---|
   | AC 미충족, 명백한 결함, 누락된 테스트 | 수정 Sub-task 추가(아래) |
   | 비목표나 refs 범위를 넘어선 변경 | 되돌릴지 Spec에 반영할지 결정받는다. 되돌리기면 수정 Sub-task 추가, 반영이면 `amend.md` 2절 |
   | design.md와 구현의 차이(표현·세부 수준) | design.md를 실제에 맞게 고치고 `spec.md`의 `## 변경 이력`에 기록한다 |
   | design.md와 인터페이스·동작이 다름 | 구현을 받아들일지 결정받는다. 받아들이면 `amend.md` 3절 1·3항으로 design.md를 고치고(4항의 계획 재검토는 하지 않는다), 아니면 수정 Sub-task 추가 |
   | 승인되지 않은 `protected_paths` 변경 | 허용 여부를 결정받는다. 허용되면 해당 Sub-task 아래 `> allowed: <경로>`를 기록하고, 아니면 수정 Sub-task 추가 |
   | 경미한 지적 | 기록만 하고 반영 여부는 6의 확인 때 함께 보여준다 |

   **수정 Sub-task 추가:** `amend.md` 3절 1·3항을 따른다(3절 4항의 계획 재검토는 하지 않는다). 새 Sub-task는 최대 번호 + 1, `status: todo`, `refs`는 관련 AC, `verify`는 필수, `files`는 아는 만큼 채운다. 하나라도 추가했으면 Spec을 `status: in-progress`로 되돌리고, 추가한 Sub-task와 남은 결정 사항을 보고한 뒤 멈춘다. `/execute <KEY>`로 구현하면 `implement.md` 8단계를 거쳐 이 문서를 다시 수행한다(새 회차).
6. `gates.md`의 in-review → done 검사를 수행한 뒤, AC 충족 표, 불일치 점검 결과, 남은 경미한 지적을 보여주고 AskUserQuestion으로 완료를 확인받는다.
7. 확인되면 `status: done`.
8. 부모 epic이 있으면 epic `spec.md`의 `## 하위 Spec` 표 상태를 갱신한다. 모든 하위 Spec이 done이면 epic을 `status: in-review`로 바꾸고 아래 epic 절을 이어서 수행한다.

## epic

1. `gates.md`의 in-review → done epic 검사를 수행한다(하위 Spec이 모두 done인지, `## 완료 조건`이 충족되는지).
2. 하위 Spec 목록과 완료 조건별 충족 근거를 보여주고 AskUserQuestion으로 완료를 확인받는다.
3. 확인되면 `status: done`. 미충족 조건이 있으면 그것을 다룰 하위 Spec 추가를 제안한다. 추가가 결정되면 `plan.md` epic 2~3단계와 같은 방식(`create.md` 1~2절로 생성, `## 하위 Spec` 표 갱신)으로 만들고 epic을 `planned`로 되돌린다.
