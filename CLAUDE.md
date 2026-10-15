# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

SpecSync는 **Spec-Driven Development(SDD)를 지원하는 Claude Code 플러그인**(`specsync`)입니다. 사용자 커맨드는 두 개뿐입니다. **`/spec`**은 명세서 작성(아이디어 → 요구사항 승인 → 설계·Sub-task 확인, `planned`까지)과 수정·현황 조회를, **`/execute`**는 실행(구현 → 리뷰 → `done`)을 맡습니다. 각 커맨드 안에서는 입력과 Spec 상태를 보고 단계를 자동으로 고르며, 단계별 커맨드를 따로 노출하지 않는 것이 핵심 설계 원칙입니다.

## 개발 명령

- 로컬 플러그인으로 Claude Code 실행: `claude --plugin-dir .` (스킬은 `/specsync:spec`, `/specsync:execute`, 충돌이 없으면 `/spec`, `/execute`로 호출). Git Bash에서는 경로에 백슬래시 대신 `/`를 쓰세요.
- 매니페스트/구조 검증: `claude plugin validate .` (CI에서는 `--strict`로 경고도 실패 처리)

## 아키텍처

모든 로직은 마크다운 지시문이며 실행 코드는 아직 없습니다.

- `.claude-plugin/plugin.json` — 매니페스트
- `skills/spec/SKILL.md` — **작성 라우터**(`/spec`). 의도 판별(기존 Spec 키 / 조회 / 빈 입력 / 새 작업, 방향 미정이면 ideate 경유) 후 draft·approved를 처리하고 planned에서 멈춥니다. Spec 수정(`amend`)은 실행 중인 Spec도 여기서 합니다.
- `skills/execute/SKILL.md` — **실행 라우터**(`/execute`). planned·in-progress·in-review Spec을 `implement` → `review`로 진행합니다. 실행 중 요구사항 변경은 draft로 되돌리고 `/spec`으로 넘기며, 설계·Sub-task 조정만 직접 수행합니다.
- `shared/rules.md` — 두 커맨드 공통 규칙(커맨드 역할 분담, 경로 해석, 사람 게이트, 의사결정성 변경 자동 반영 금지, 상태 갱신).
- `shared/stages/*.md` — 단계 절차. `init`(principles 생성), `ideate`(발산/수렴), `create`, `clarify`(draft→approved), `plan`(approved→planned), `plan-review`(에이전트 패널 검토), `implement`(중단 조건 포함), `review`(in-review→done), `amend`, `status`, `gates`(전이 검사 규칙).
- `shared/templates/` — `spec.{story,bug,epic,task}.md`, `design.md`, `subtasks.md`, `principles.md`. 자리표시자는 `{{key}}`, `{{title}}`, `{{parent}}`, `{{date}}`, `{{prefix}}` 형식입니다.
- `agents/` — 기본 계획 검토자(`design-reviewer`, `test-reviewer`, `scope-reviewer`, 읽기 전용)와 구현 검토자(`implementation-reviewer`, 파일 수정 없음, Bash는 git 조회만). 호출 이름은 `specsync:<name>`. 계획 검토자는 principles `reviewers` → 프로젝트 `.claude/agents/` → 기본 검토자 순으로, 구현 검토자는 principles `impl_reviewers` → `implementation-reviewer` 순으로 결정됩니다. 구현을 맡은 메인 에이전트가 자기 결과를 스스로 승인하지 않도록 review 단계에서 구현 검토자를 Agent 도구로 호출합니다.

`reference/`는 이 설계의 참고 자료(다른 프로젝트의 `/plan:*` 커맨드 세트)이며 플러그인 소스가 아닙니다. 수정하지 마세요. 시간 추정, 이모지 우선순위, HTML 주석 기반 상태 추정, MCDA 평가 게이트는 의도적으로 가져오지 않았습니다.

경로 참조 주의: `${CLAUDE_PLUGIN_ROOT}`/`${CLAUDE_SKILL_DIR}`는 SKILL.md(및 command/agent) 본문에서만 치환되고, Bash 명령 환경변수로도 설정되지 않습니다. 두 스킬이 `shared/`를 함께 쓰므로 SKILL.md는 `${CLAUDE_PLUGIN_ROOT}/shared/`로 참조합니다. `shared/` 안의 문서에는 이 변수를 쓰지 말고 `<이름>.md`(→ `shared/stages/`), `templates/…`(→ `shared/`) 같은 상대 경로만 씁니다(`rules.md`의 `## 경로`).

라우팅·단계 동작을 바꿀 때는 두 SKILL.md의 분기 표, `rules.md`의 역할 분담, 해당 stage 문서, `gates.md`가 서로 일치하는지 함께 확인하세요.

## SDD 도메인 모델 (설계 결정)

Jira의 계층·워크플로 개념을 참고하되, 원본은 사용자 프로젝트의 로컬 `.specs/` 파일입니다. 외부 트래커 동기화는 하지 않으며, 추후 어댑터로 붙일 수 있게만 둡니다.

- **계층:** Principles(`.specs/principles.md`) → Epic Spec → Spec(`story` | `bug` | `task`) → Sub-task
  - Principles에는 SDD 전용 규칙(키 접두어, `commit`, `plan_review`, `reviewers`, `impl_review`, `impl_reviewers`, `protected_paths`, 검증 명령, 품질 게이트, 컴포넌트)만 둡니다. 프로젝트 고유 제약은 스테이지 문서에 하드코딩하지 말고 principles 설정으로 받습니다. 일반 컨벤션은 사용자 프로젝트의 `CLAUDE.md`에 둡니다.
  - Epic은 목표와 하위 Spec 목록만 가지며 직접 구현되지 않습니다.
  - Spec = 사람이 승인하는 단위(독립 배포·검증 가능한 변화 1건). Sub-task = 기계가 검증하는 단위(세션 1회, 커밋 1개, `verify` 필수).
  - "Task"는 Spec 타입을 뜻합니다. 실행 단위는 반드시 "Sub-task"라고 부릅니다.
- **키:** Spec은 `<PREFIX>-<n>`(예: `SS-12`), Sub-task는 계층형 `SS-12.3`. 폴더는 `.specs/<KEY>-<slug>/`.
- **상태:** `draft → approved → planned → in-progress → in-review → done`. →approved, →planned, →done은 사람 게이트로, 기계 검사 통과 후 AskUserQuestion으로 명시적 승인을 받아야만 전이합니다. 승인 이후 요구사항이 바뀌면 draft로 되돌아가고, 재계획은 기존 design/Sub-task를 덮어쓰지 않고 수정합니다(done Sub-task와 키 보존). `done` Spec은 수정하지 않고 `links.relates_to`로 새 Spec을 만듭니다.
  - Epic: draft→approved(후보·완료 조건 검사) → approved→planned(후보 확인 후 하위 Spec을 draft로 생성만 함) → 하위 Spec 전부 done이면 in-review → done.
  - Sub-task status: `todo | in-progress | done | blocked | dropped`. 중단된 in-progress를 먼저 재개하고, blocked는 사용자 결정(재시도/수정/보류)으로만 풀립니다. 불필요해진 Sub-task는 지우지 않고 dropped로 둡니다.
- **메타데이터:** `spec.md` frontmatter에만 둡니다(`key`, `type`, `title`, `status`, `parent`, `priority`, `components`, `links.blocked_by`, `links.relates_to`, `created`, `approved_at`). Sprint, story point, 보드 개념은 의도적으로 제외합니다.
- **거버넌스(`rules.md`의 `## 거버넌스`):** 개인·소규모 사용을 전제로 실행 코드 없이 지시문과 기록으로 운영합니다. 사람 게이트 승인·상태 되돌림·게이트 예외는 `## 변경 이력`에 `[승인]`/`[되돌림]`/`[예외]` 태그로 남기고, 이 기록이 없는 status는 "기록 없는 상태 변경"으로 감지합니다. `gates.md`의 `(예외 불가)` 항목은 예외로 건너뛸 수 없습니다. principles의 검토·보호 설정 변경은 확인을 받고 principles `## 변경 이력`에 기록합니다. 훅·CI로 강제하지 않는 것은 의도적인 선택입니다.
- **추적성:** REQ → Sub-task(`refs`) → 커밋(메시지 첫 줄에 Sub-task 키)으로 이어지는 ID 추적으로 Spec과 코드의 불일치를 감지합니다.

### Spec 파일 형식

- 타입별 파일: epic은 `spec.md`만, story는 `spec.md` + `design.md` + `subtasks.md`, bug는 `design.md` 선택, task는 `spec.md` 안에 `## Sub-tasks` 섹션.
- 요구사항은 `### REQ-n` 아래에 EARS 문장, 수용 기준은 `- AC-n.m Given / When / Then`(bug의 회귀 AC는 `[regression]` 태그), 미해결 질문은 `- [ ] Qn`.
- Sub-task는 `### <KEY>.n 제목` 아래에 `- status|refs|blocked_by|files|verify: 값` 필드.
- 파싱은 HTML 주석을 제거한 뒤, 섹션 제목 문구가 아니라 **ID와 필드 키에 의존**합니다. 예외는 `gates.md`에 명시된 네 곳(`## Sub-tasks`, `## 하위 Spec` 표, `## 계획 검토`와 `## 구현 검토`의 `blocking_open`/`pending_decisions`)뿐이므로 이 제목들은 바꾸지 마세요. 빈 값·`—`·빈 코드 스팬은 값 없음입니다.

## 환경 참고

- 개발 환경은 Windows입니다. 훅 스크립트를 추가할 경우 Windows와 POSIX 셸 모두에서 동작하도록 작성하세요.
- 플러그인 루트의 `CLAUDE.md`는 플러그인 사용자에게 로드되지 않습니다(이 저장소 개발용). 사용자에게 전달할 지시는 스킬에 둡니다.
- `.omc/`는 oh-my-claudecode 런타임 상태 디렉터리로, 플러그인 소스가 아닙니다.
