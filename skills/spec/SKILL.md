---
name: spec
description: Spec-Driven Development 명세서 작성 커맨드. 사용자가 작업/이슈/버그에 대한 명세서(spec) 작성을 요청하거나, 기존 Spec(예: SS-12)의 요구사항·설계를 수정하거나, Spec 현황을 조회하려 할 때 사용한다. 아이디어 정리부터 요구사항 승인, 설계·Sub-task 확인(planned)까지 담당하며 구현은 하지 않는다.
argument-hint: "[작업 설명 | Spec 키 [수정 내용] | 현황]"
---

# SpecSync: 명세서 작성

사용자 입력: $ARGUMENTS

이 커맨드는 Spec을 **실행 가능한 상태(planned)까지** 만든다. 코드를 구현하지 않는다. 입력과 저장소 상태로 할 일을 판단하고, 해당 단계 문서를 읽어 그대로 수행한다. 단계 이름이나 내부 절차를 사용자에게 고르게 하지 않는다.

## 경로

- 공용 폴더: `${CLAUDE_PLUGIN_ROOT}/shared/`
- 단계 문서: `${CLAUDE_PLUGIN_ROOT}/shared/stages/`
- 템플릿: `${CLAUDE_PLUGIN_ROOT}/shared/templates/`

먼저 `${CLAUDE_PLUGIN_ROOT}/shared/rules.md`를 읽고 따른다.

## 0. 컨텍스트 수집

1. 프로젝트 루트의 `.specs/principles.md`를 읽는다. 없으면 `init.md`를 먼저 수행한 뒤 계속한다.
2. `.specs/*/spec.md`의 frontmatter에서 `key`, `type`, `title`, `status`, `parent`를 모은다. (Spec 목록)

## 1. 의도 판별

위에서부터 처음 맞는 규칙을 적용한다.

| 입력 | 대상 | 행동 |
|---|---|---|
| Spec 키(`<PREFIX>-n` 또는 `<PREFIX>-n.m`)를 포함하거나 기존 Spec 제목을 명확히 가리킴 | 해당 Spec | 2단계로 |
| 키 없이 "현황", "목록", "status" 등 조회 요청 | 전체 | `status.md` |
| principles 설정(검토 켜기·끄기, 검토자, `protected_paths`, 커밋 정책, `db_access` 등 frontmatter 설정) 변경 요청 | `.specs/principles.md` | `rules.md`의 `### principles 변경` |
| 비어 있음 | draft·approved Spec | 1개면 그 Spec으로 2단계. 여러 개면 AskUserQuestion으로 선택(최대 4개). 0개면 `status.md` 후 무엇을 명세할지 묻는다 |
| 그 외 새 작업·이슈·버그 설명 | 새 Spec | 방향이 정해지지 않았으면 `ideate.md` → `create.md`, 정해졌으면 `create.md` |

**방향 미정 판단:** 해결할 문제는 있으나 접근 방식이 정해지지 않았거나(예: "~를 개선하고 싶은데"), 사용자가 아이디어·방안을 묻는 경우. 버그 신고와 구체적인 변경 요청은 바로 `create.md`로 간다.

판단이 갈리면(예: 기존 Spec의 변경인지 새 작업인지) 추측하지 말고 AskUserQuestion으로 해석 후보를 제시한다.

## 2. 기존 Spec 처리

1. 입력이 해당 Spec의 현황 조회면 그 Spec 하나에 대해 `status.md`를 수행한다.
2. 입력이 Spec 내용을 바꾸려는 요청(요구사항 추가·변경, 질문 답변, 설계·Sub-task 수정)이면:
   - status가 `done`이 아니면 `amend.md`. 실행 중인(planned 이후) Spec도 여기서 수정한다.
   - `done`이면 수정하지 않고, 해당 Spec을 `links.relates_to`로 참조하는 새 Spec 생성을 제안한다.
3. 그 외에는 상태에 따라 다음 단계를 수행한다.

| status | story / bug / task | epic |
|---|---|---|
| draft | `clarify.md` (질문 해소 → 승인 게이트 → 이어서 plan) | `clarify.md` |
| approved | `plan.md` (설계·Sub-task 작성 → 확인 게이트) | `plan.md` (하위 Spec 생성) |
| planned, in-progress, in-review | `/execute <KEY>`로 실행하라고 안내 | 하위 Spec 현황을 `status.md`로 보여주고, 아직 draft·approved인 하위 Spec이 있으면 `/spec <하위 키>`를 추천 |
| done | 완료 요약을 보여준다 | 완료 요약을 보여준다 |

planned에 도달하면 멈추고 `/execute <KEY>`로 실행하라고 안내한다.
