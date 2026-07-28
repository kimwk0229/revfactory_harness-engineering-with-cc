# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# my-first-harness
2인 팀으로 커밋 메시지를 생성하는 최소 하네스 프로젝트

이 저장소는 실행 가능한 애플리케이션 코드가 아니라, Claude Code의 에이전트·스킬 기능만으로 구성된
최소 하네스 예제다. 빌드/린트/테스트 명령은 존재하지 않는다.

## 규칙
- 커밋 메시지는 Conventional Commits 형식 (type(scope): subject).
- 중간 산출물은 `_workspace/` 에 둔다.
- 에이전트는 `.claude/agents/`, 스킬은 `.claude/skills/` 에 정의한다.

## 사용 방법
자연어로 "커밋 메시지 만들어줘" / "commit message" 라고 요청하면 `commit-message` 스킬
(`.claude/skills/commit-message/SKILL.md`)이 트리거된다. 이미 메시지가 정해진 `git commit -m` 요청의
대체 용도는 아니다.

## 아키텍처: author → reviewer 파이프라인
`commit-message` 스킬이 두 서브에이전트를 순차 호출해 커밋 메시지를 생성·검증한다.

1. **Precondition 체크** — `git diff --cached --quiet` 로 스테이지된 변경 유무를 확인. 변경이 없으면
   `git add` 안내 후 종료.
2. **`commit-msg-author` 에이전트** (`.claude/agents/commit-msg-author.md`) — `git diff --cached` +
   `git log -10 --oneline` 를 읽고 Conventional Commits 초안을 `_workspace/commit-draft.md` 에 작성.
   diff에 없는 내용은 추측해서 넣지 않는다.
3. **`commit-msg-reviewer` 에이전트** (`.claude/agents/commit-msg-reviewer.md`) — 초안의 형식·범위·
   diff 사실 일치를 객관적 기준으로만 검증해 `_workspace/review-report.md` 에 PASS/REDO 판정과
   사유를 기록.
4. **판정 분기** — PASS면 초안을 사용자에게 제시하고 종료. REDO면 reviewer의 수정 지시를 프롬프트에
   포함해 author를 재호출 (최대 2회).
5. **루프 종료** — 2회 재호출 후에도 REDO면 "자동 승인 한계 도달" 경고와 함께 마지막 draft를 그대로
   반환한다 (무한 루프 방지, 오검보다 누락을 우선).

두 에이전트는 서로를 직접 호출하지 않고 `_workspace/` 의 파일(`commit-draft.md`,
`review-report.md`)을 통해서만 상태를 주고받는다 — 새 산출물을 추가할 때도 이 파일 기반 핸드오프
패턴을 유지할 것.
