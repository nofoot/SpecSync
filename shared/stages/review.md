# review: 검토와 완료 (in-review → done)

## story / bug / task

1. 모든 Sub-task의 `verify`를 다시 실행한다.
2. AC를 하나씩 확인한다. 각 AC가 어떤 테스트나 동작으로 충족되는지 근거(파일:줄, 테스트 이름)를 표로 만든다. 충족되지 않은 AC가 있으면 Sub-task를 추가(최대 번호 + 1)하고 `status: in-progress`로 되돌린 뒤 보고한다.
3. 불일치를 점검한다.
   - 구현이 비목표나 refs 범위를 넘어선 변경을 포함하는가. 포함하면 되돌릴지 Spec에 반영할지 AskUserQuestion으로 결정받는다(반영이면 `amend.md` 2절).
   - design.md와 실제 구현이 다른가. 다르면 design.md를 실제에 맞게 고치고 `spec.md`의 `## 변경 이력`에 기록한다. 인터페이스나 동작이 달라진 경우는 의사결정성 변경으로 보고 먼저 결정받는다.
4. AC 충족 표와 불일치 점검 결과를 보여주고 AskUserQuestion으로 완료를 확인받는다.
5. 확인되면 `status: done`.
6. 부모 epic이 있으면 epic `spec.md`의 `## 하위 Spec` 표 상태를 갱신한다. 모든 하위 Spec이 done이면 epic을 `status: in-review`로 바꾸고 아래 epic 절을 이어서 수행한다.

## epic

1. 하위 Spec이 모두 done인지, `## 완료 조건`이 충족되는지 확인한다.
2. 하위 Spec 목록과 완료 조건별 충족 근거를 보여주고 AskUserQuestion으로 완료를 확인받는다.
3. 확인되면 `status: done`. 미충족 조건이 있으면 그것을 다룰 하위 Spec 추가를 제안한다. 추가가 결정되면 `plan.md` epic 2~3단계와 같은 방식(`create.md` 1~2절로 생성, `## 하위 Spec` 표 갱신)으로 만들고 epic을 `planned`로 되돌린다.
