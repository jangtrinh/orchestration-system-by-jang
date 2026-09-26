---
name: orchestration-system-by-jang
description: "Orchestration System by Jang. Use when spawning, supervising, handing off, or coordinating AI agents/subagents for any kind of work. Enforces an explicit dispatch contract, capability routing, independent verification, no-op/refusal detection, and resource cleanup."
license: MIT
metadata:
  author: Jang
  version: 2.0.0
  category: orchestration
  tags: [agents, orchestration, delegation, verification]
---

# Orchestration System by Jang

Policy for delegating work to other agents, independent of runtime and domain.
Local choices (runtime, models, paths, approval gates) live in
`orchestration-config.md` next to this file — read it first. If it is missing,
run `SETUP.md`. Command syntax lives in `references/runtime-adapters.md`.

## When NOT to orchestrate

Trivial answers, small reversible edits, and anything faster to do than to brief
stay direct. Never force delegation, a minimum worker count, a new plan file, or a
repeated approval gate just because orchestration was mentioned.

## The loop

```text
FRAME → ACCEPTANCE → ROUTE → DISPATCH → EXECUTE → VERIFY → REPORT
```

1. **FRAME** — outcome, constraints, non-goals, reversibility.
2. **ACCEPTANCE** — number every requirement `A1..`, constraint/edge case `C1..`,
   independent check `V1..` in `<work dir>/acceptance.md`. Every task, report, and
   finding cites these IDs.
3. **ROUTE** — role + tier (+ effort if supported) per task, with a one-line reason
   (`references/dispatch-contract.md` §2).
4. **DISPATCH** — fill the task spec (§1) completely; pin state (§3) *before*
   launch; create the whole ready wave before starting any worker.
5. **EXECUTE** — wait on completion signals, not guesses. Answer worker questions
   only from the approved plan; approval gates go to the human.
6. **VERIFY** — someone other than the last editor re-checks the real result after
   the final change (§3 post-checks, §5 verifier brief).
7. **REPORT** — unverified items first, then result, then open questions.

## Choose exactly one operating mode

| Mode | Use when | Tracking state? |
|---|---|---|
| **Full handoff** | Ownership transfers; you stop supervising | None — send a self-contained brief |
| **Lightweight prompt** | One-off input, nothing to track | None |
| **Supervised** | You must monitor, coordinate dependencies, answer questions, verify, retry, or clean up | run → tasks → attempts → verify → release |

Never mix modes (e.g. a "handoff" you secretly poll).

## Binding invariants

1. **One control plane.** Only the configured runtime schedules workers. No
   side-channel agent processes or a second task ledger competing with it.
2. **Names are labels; IDs route.** Route by runtime-issued IDs (run, task,
   attempt, terminal/agent). A retry is a new attempt ID.
3. **Explicit dispatch contract.** Before dispatch declare: controller, worker
   role, independent verifier, permissions, dependencies, approval gates,
   ownership of each file/record/document, isolation boundary.
4. **Disjoint ownership for parallel writers**, or separate copies/branches/workspaces.
5. **Routing is recorded**, never hard-coded forever. Re-verify model IDs after
   runtime or provider upgrades.
6. **Clean-context counsel at commitment boundaries** (architecture, data shape,
   public interface, security, irreversible action, wide rewrite, 3rd attempt after
   2 failures). Counsel advises; it never implements or approves its own advice.
7. **Builder ≠ verifier.** The last editor cannot be the final verifier. Reviewers
   are read-only. A worker's self-report is a claim, not evidence.
8. **"Success" is not done.** Missing change/artifact/state ⇒ `NO-OP`; refusal or
   access error ⇒ `REFUSED` (quote verbatim). Never silently swap model, effort,
   permissions, or runtime.
9. **Bounded fix loop.** At most the configured number of verify→fix rounds
   (default 3). Same finding twice = wrong model of the problem → counsel, then
   the human decides.
10. **Approval gates stay with the human.** Everything listed in the config, plus
    any one-way door (delete, publish, deploy, merge, migrate, spend, message people).
11. **Account for every worker.** After an accepted completion: reuse it
    immediately, release it, or keep it only because the human asked. Never
    release on timeout, idle, heartbeat, or an open question.
12. **Idle ≠ finished.** Timeouts, heartbeats, "running", and idle are not
    completion. Don't kill or restart a worker for being slow.

## Before the first dispatch

- Read `orchestration-config.md` and `references/dispatch-contract.md`; fill
  every spec section (or `n/a`).
- Paste `references/worker-preamble.md` into every non-trivial worker prompt.
- Check the worker launcher works in a **fresh** shell/session — a live session
  proves nothing (auto-updaters can break new launches).
- Check shared resources (same files, database, document) for other live writers.
  A worker's promise not to touch something is not a lock.

## Completion checklist

- [ ] One operating mode chosen and not mixed
- [ ] Acceptance IDs exist and every task/report/finding cites them
- [ ] Routing recorded per task (role · tier · effort-if-any · reason)
- [ ] State pinned before dispatch with the configured pin method
- [ ] Every attempt classified `DONE | DONE_WITH_CONCERNS | NO-OP | REFUSED`
- [ ] Final result verified by someone other than the last editor, after the last change
- [ ] Every worker reused, released, or kept on the human's request
- [ ] Report lists what could NOT be verified first, open questions last
