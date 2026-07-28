# ex-02-02 — my-first-harness

2인 팀(author·reviewer)이 순차 협업해 Conventional Commits 형식의 커밋 메시지를 생성하는 최소 하네스.

## 실행

```bash
git add <변경 파일>
# 이후 대화에서:
"커밋 메시지 만들어줘"
```

`commit-message` 스킬(`.claude/skills/commit-message/SKILL.md`)이 트리거되어 아래 순서로 동작한다.

1. **precondition** — `git diff --cached --quiet` 로 스테이지된 변경 유무 확인.
2. **`commit-msg-author`** — `git diff --cached` + `git log -10 --oneline` 를 읽고 초안을 `_workspace/commit-draft.md` 에 작성.
3. **`commit-msg-reviewer`** — 초안의 형식·범위·diff 사실 일치를 검증해 `_workspace/review-report.md` 에 PASS/REDO 기록.
4. PASS면 초안 제시 후 종료, REDO면 수정 지시를 포함해 author를 최대 2회까지 재호출.

## 폴더 구조

```
ex-02-02-my-first-harness/
├── CLAUDE.md                          ← 규칙 + 아키텍처 설명
├── .claude/
│   ├── agents/
│   │   ├── commit-msg-author.md       (초안 작성)
│   │   └── commit-msg-reviewer.md     (PASS/REDO 검증)
│   └── skills/
│       └── commit-message/SKILL.md    (author↔reviewer 오케스트레이션)
└── _workspace/
    ├── commit-draft.md                ← 산출물 예시
    └── review-report.md               ← 산출물 예시
```

## 보기

`_workspace/commit-draft.md`(author 초안 예시), `_workspace/review-report.md`(reviewer PASS 판정 예시).
