---
name: execute
description: Spec-Driven Development 실행 커맨드. 승인·계획이 끝난(planned 이후) Spec(예: SS-12)의 Sub-task를 구현하고 검증·리뷰해 완료까지 진행할 때 사용한다. 명세서 작성·수정은 하지 않는다.
argument-hint: "[Spec 키 | Sub-task 키]"
---

# SpecSync: Spec 실행

사용자 입력: $ARGUMENTS

이 커맨드는 `/spec`으로 planned까지 만들어진 Spec을 **구현하고 완료**한다. 요구사항을 새로 쓰거나 승인하지 않는다.

## 경로

- 공용 폴더: `${CLAUDE_PLUGIN_ROOT}/shared/`
- 단계 문서: `${CLAUDE_PLUGIN_ROOT}/shared/stages/`
- 템플릿: `${CLAUDE_PLUGIN_ROOT}/shared/templates/`

먼저 `${CLAUDE_PLUGIN_ROOT}/shared/rules.md`를 읽고 따른다.

## 0. 컨텍스트 수집

1. `.specs/principles.md`를 읽는다. 없으면 "`/spec`으로 먼저 명세서를 작성하세요"라고 안내하고 멈춘다.
2. `.specs/*/spec.md`의 frontmatter에서 `key`, `type`, `title`, `status`, `parent`를 모은다. (Spec 목록)

## 1. 대상 결정

| 입력 | 대상 |
|---|---|
| Sub-task 키(`<PREFIX>-n.m`) | 해당 Spec, 그 Sub-task를 지정 |
| Spec 키(`<PREFIX>-n`) 또는 기존 Spec 제목 | 해당 Spec |
| 비어 있음 | planned·in-progress·in-review인 Spec. 1개면 그것. 여러 개면 AskUserQuestion으로 선택(in-progress 우선, 최대 4개). 0개면 `status.md` 후 `/spec`으로 실행할 Spec을 먼저 만들라고 안내 |

입력에 키 외의 내용이 있고 그것이 Spec을 바꾸려는 요청이면 처리하지 않고 `/spec <KEY> <수정 내용>`으로 안내한다. principles 설정 변경 요청이면 처리하지 않고 `/spec`으로 안내한다.

## 2. 상태별 처리

| status | story / bug / task | epic |
|---|---|---|
| draft, approved | `/spec <KEY>`로 명세서를 먼저 완성하라고 안내 | 같음 |
| planned, in-progress | `implement.md` | 하위 Spec 진행 현황을 `status.md`로 보여준다. 모두 done이면 `status: in-review`로 바꾸고 `review.md`. 아니면 다음에 실행할 하위 Spec(`/execute <하위 키>`) 또는 작성할 하위 Spec(`/spec <하위 키>`)을 추천 |
| in-review | `review.md` | `review.md` |
| done | 완료 요약을 보여준다 | 완료 요약을 보여준다 |

epic은 in-progress를 거치지 않는다(planned → 하위 Spec 모두 done → in-review).

## 3. 실행 중 Spec 변경

구현이나 리뷰 중 Spec을 바꿔야 하면(`implement.md` 중단 조건, `review.md` 불일치):
- 설계·Sub-task 조정(`amend.md` 3절)은 사용자 결정을 받은 뒤 여기서 바로 수행하고 실행을 이어간다.
- 요구사항 변경(`amend.md` 2절)은 결정된 변경을 `spec.md`에 반영하고 draft로 되돌린 뒤 멈춘다. 재승인과 재계획은 `/spec <KEY>`로 진행하라고 안내한다.
