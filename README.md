# ceo — CEO orchestration skill for Claude Code + OpenCode

A [Claude Code](https://claude.com/claude-code) skill for environments where **Claude runs on a limited token budget** while **[OpenCode](https://opencode.ai) runs an unlimited (e.g. local/self-hosted) model**.

Claude acts as the **CEO**: it analyzes the request, writes a work order, and reads the final verdict. OpenCode does everything else — execution **and** review — so Claude's expensive tokens are spent only on judgment, never on reading documents or generating bulk output.

## How it works

```
User request
   │
   ▼
Claude (CEO)          writes  _ceo/NN-order.md   (goal, inputs as paths, checkable acceptance criteria)
   │
   ▼
opencode run          worker session: executes the order, writes _ceo/NN-report.md
   │
   ▼
opencode run          reviewer session (separate!): verifies against acceptance criteria,
   │                  writes _ceo/NN-review.md starting with VERDICT: APPROVED / REJECTED
   ▼
Claude (CEO)          reads only report + review (both small)
   ├─ REJECTED → appends 1–3 lines of fix directives, reruns worker with `opencode run -c` (max 3 rounds)
   └─ APPROVED → spot-checks one file (≤50 lines), reports to the user
```

Key design decisions:

- **The role boundary is fixed at design time, not judged at runtime.** Claude follows a closed whitelist of 6 allowed actions; everything not on the list is delegated by default (default-deny). This matters because the orchestrating model may be a small/low-effort one — it should pattern-match, not exercise discretion.
- **Author/reviewer separation.** The reviewer runs in a fresh OpenCode session so the worker never approves its own output.
- **Everything flows through files**, not the chat context: work orders, reports, verdicts. Claude never pastes document content into its own context.

## Install

```bash
git clone https://github.com/Judy-0509/ceo-skill ~/.claude/skills/ceo
```

The target folder must be named `ceo` (folder name = skill name = `/ceo` command).

Requirements:

- Claude Code with skills support
- OpenCode CLI on PATH — verify `opencode run "hello"` works non-interactively, and that your version supports `-c` (continue last session)
- OpenCode configured with your unlimited-budget model as default (or add `-m provider/model` to the commands in SKILL.md)

## Use

The skill auto-triggers on token-heavy requests (document summarization, wiki/DB building, multi-file code work) — no need to ask for delegation. You can also invoke it explicitly with `/ceo` or by saying "opencode로 해줘".

Smoke-test cases are in [`test-prompts.md`](test-prompts.md).

## Customize

- The skill body is written in Korean (our team's language). Model-facing instructions work as-is; translate freely if your team prefers English.
- To enforce a team style guide on deliverables, put its file path in the work-order template's "지켜야 할 규칙" (rules) section — the OpenCode worker will read and follow it.

---

### 한국어 요약

토큰 예산이 제한된 Claude가 CEO(요구 분석·작업지시서·최종 판정)만 맡고, 토큰 무제한 OpenCode가 실행과 검토를 모두 수행하는 오케스트레이션 스킬입니다. 역할 경계는 실행 시점 판단이 아니라 설계 시점에 고정되어 있고(허용 행동 6종 whitelist, 기본값 = 위임), 작성자와 검토자 세션을 분리해 셀프 승인을 막습니다. 설치: 위 `git clone` 명령으로 `~/.claude/skills/ceo`에 받으면 `/ceo` 명령어로 바로 사용할 수 있습니다.
