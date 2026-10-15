# SpecSync 공통 규칙

`/spec`과 `/execute`가 함께 따르는 규칙이다.

## 커맨드 역할

| 커맨드 | 담당 status | 하는 일 |
|---|---|---|
| `/spec` | (새 Spec), draft, approved | 아이디어 → 생성 → 질문 해소·요구사항 승인 → 설계·Sub-task 작성·계획 검토 → planned. Spec 수정과 현황 조회 |
| `/execute` | planned, in-progress, in-review | Sub-task 구현 → 리뷰 → done |

상대 커맨드의 담당 status인 Spec을 받으면 처리하지 않고, 그 커맨드로 실행하라고 한 줄로 안내한다(예: "SS-12는 planned 상태입니다. `/execute SS-12`로 실행하세요.").

## 경로

단계 문서 안의 `<이름>.md`는 SKILL.md `## 경로`의 단계 문서 폴더(`shared/stages/`) 기준, `templates/…`는 공용 폴더(`shared/`) 기준 상대 경로다. 사용자 프로젝트 기준으로 해석하지 않는다. 반면 `.specs/…`, `spec.md`, `design.md`, `subtasks.md`는 사용자 프로젝트의 파일이다.

## 진행 규칙

- 자동 전이는 이어서 진행하되, **사람 게이트(→approved, →planned, →done)에서는 반드시 멈추고 AskUserQuestion으로 확인받는다.** 사용자가 명시적으로 승인하지 않으면 상태를 바꾸지 않는다.
- 게이트 검사는 `gates.md`의 규칙을 따른다. 검사에 실패하면 실패 항목을 보여주고 해결을 돕는다.
- **의사결정성 변경은 자동 반영하지 않는다.** 범위 축소·확대, 기능 제거, 방향 전환, 목표·비목표 변경, REQ/AC의 의미 변경은 검토 의견이나 구현 중 발견에서 나왔더라도 AskUserQuestion으로 하나씩 결정받은 뒤 반영한다. 표현 보완, 누락된 리스크·테스트·Sub-task 추가처럼 사용자 의도를 바꾸지 않는 보완만 자동 반영한다.
- 상태를 바꿀 때마다 `spec.md`의 frontmatter `status`를 갱신한다. 승인 시 `approved_at`에 날짜를 기록한다. 요구사항이나 설계가 바뀌면 `spec.md`의 `## 변경 이력`에 한 줄을 추가한다.
- 템플릿에 없는 섹션에 기록해야 하면 섹션을 추가한다.
- 응답 끝에는 현재 상태와 다음에 실행할 커맨드(`/spec <KEY>` 또는 `/execute <KEY>`)를 한 줄로 알려준다.
