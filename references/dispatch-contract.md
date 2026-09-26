# Dispatch contract — fill before every supervised dispatch

This file owns what goes INTO a task and what the controller checks AFTER.
Paths, models, and gates come from `orchestration-config.md`; command syntax from
`runtime-adapters.md`.

## 0. Pre-flight (controller)

- Runtime healthy; its current docs/help reloaded after any upgrade.
- Worker launcher starts in a fresh shell/session.
- Shared-resource check: other live workers or people on the same files,
  database, or document? If nothing can be locked, write a lock note in the work
  dir and have the verifier look for foreign changes during the window.

## 1. Task spec template (every section filled or `n/a`)

```markdown
# Task — <ROLE> · <capability · bounded action>
## Identity
Run: <run id> · Role: BUILD | VERIFY | SCOUT | COUNSEL · Controller: <session>
Independent verifier: <different attempt> (BUILD) | n/a (VERIFY)
Workspace: <path/branch/doc> · Work dir: <plan folder> · Report path: <file>
## Acceptance IDs  (from <work dir>/acceptance.md — cite everywhere)
A<n> requirements · C<n> constraints/edge cases · V<n> independent checks
## Read first (ordered)
## Allowed to change (exact files/records/sections; everything else read-only)
## Objective (bounded) + steps  (inspect → change → inspect → assert)
## Constraints / non-goals / forbidden
(deletions, publishing, other files, messaging people, spawning sub-workers, …)
## Routing
ROLE: · TIER: · EFFORT (if supported): · ROUTING REASON:
## Authority
permissions · isolation · dependencies · approval gates (STOP + ask conditions)
## Question policy
Controller answers only from the approved plan/acceptance.md (cites the line).
Scope changes and approval gates go to the human; their verbatim answer is relayed.
## Verification (worker's own — not final)
exact checks · expected result/state · checks run AFTER the last change
## Completion report
acceptance table → changes → findings → assumptions → unresolved → Status line (§4)
## Worker preamble
<paste references/worker-preamble.md; Success test = acceptance IDs verbatim>
```

Title: `<capability> · <bounded action>`. Display name: `<role> · <purpose> (<tier>)`.

## 2. Routing

Use the tiers from the config. In `single-model` mode, see `provider-notes.md`.

| Work | Tier | Why |
|---|---|---|
| Scout, search, plumbing, deterministic report | cheap | mechanical |
| Bounded build, known pattern | workhorse | default |
| Bounded judgment: unfamiliar domain, spec says "choose", workhorse failed once, very large context | strongest (low effort if supported) | judgment beats volume |
| Deep build, unfamiliar territory | deep | |
| Plan / review / verify / synthesis | deep or strongest | verdicts need a judgment tier |
| Counsel (hard fork, 2 failures) | strongest, advisory-only | advice, no changes |

Rules:
- The cheap tier never gets tasks containing irreversible steps or
  compliance/policy constraints — split those out or route up.
- Escalation: a failure an automatic checker (tests, validator, linter, schema
  check) catches → retry one notch higher (effort if supported, else next tier)
  as a new attempt with its own routing reason. A failure no checker can see
  (wrong approach, bad judgment) → next tier up, not more effort.
- Record the exact launch command/parameters; effort is usually not attested
  anywhere afterwards.

## 3. State pin + post-completion checks

**Pin before dispatch** — use the config's pin method(s); save under `<work dir>/pin/`:

| Method | Pin | "Something changed" check |
|---|---|---|
| git | current commit id | uncommitted changes exist and/or diff vs pinned commit is non-empty (workers often don't commit) |
| file-snapshot | copy or hash of each target file + a timestamp marker file per attempt | targets differ from pin in the expected direction; files newer than the marker exist; untouched targets equal the pin |
| external-state | version/revision id, export, or query result of each target (doc revision, DB row count/checksum, API resource version) | new revision exists and differs as expected; untouched targets unchanged |

Create a per-attempt marker (file with the attempt ID and timestamp) right after
launch so retry artifacts are distinguishable. Use whatever the OS provides
(`touch` on POSIX, `New-Item` on PowerShell, or write a file with the tool API).

**After each completion message** (a success status is NOT enough):
1. Report exists at the declared path, ends with a `Status:` line, has the 5 body
   fields (§4). Malformed → reject and re-dispatch as a new attempt.
2. The expected change exists per the pin table.
3. Model actually used: read it from the worker's transcript/logs where available;
   mismatch ⇒ `REFUSED`-class. Never swap silently.
4. Scan the worker output for refusals/access errors. Found ⇒ `REFUSED`, quote verbatim.

Classify: `DONE | DONE_WITH_CONCERNS | NO-OP | REFUSED`. `NO-OP`/`REFUSED` go to
the human with the original text — never reported as done.

## 4. Worker completion contract (paste into every BUILD/VERIFY spec)

Send exactly one completion message through the runtime's channel, then stop.
Body fields, in order:
1. **Acceptance IDs addressed**
2. **Summary**
3. **Evidence** (≤10 lines inline, else file paths)
4. **Confidence**: `confident` | `uncertain because <X>`
5. **Out-of-scope noticed**

End with:
```text
Status: DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
Summary: <1–2 sentences>
Concerns/Blockers: <if any>
```
Prefix the subject with `REFUSED:` / `NO-OP:` / `BLOCKED:` when applicable.
Blocking questions go through the runtime's ask channel; never improvise past a STOP.

## 5. Verifier brief (different attempt, fresh context, read-only)

Give: original request verbatim · acceptance.md · pin · worker report and outputs
**as claims** · worker spec (scope + forbidden list) · allowed read-only tools ·
known risks and excluded scope.

Require:
- no-op test first (output + report exist, else `NO-OP`)
- own acceptance table from direct inspection, not from the worker report
- untouched targets vs pin; foreign changes during the window
- checks re-run **after the last change** (earlier runs are stale evidence)
- omitted or unrequested work
- what could NOT be verified; assumptions with confidence

Output: one findings file per verifier attempt, never overwritten:
`<reports dir>/verify-<date>-<slug>-<attempt>.md`. Verdict `PASS (bounded) | FAIL | NO-OP`.
Verifiers never fix.

## 6. Bounded verify→fix loop

Findings become follow-up tasks in the same run. Fixes go to the builder;
re-verification goes to a verifier attempt. Stop at the configured round limit.
Identical repeated finding → counsel, then the human decides.

## 7. Resource accounting (before every wait and at close)

- After each accepted completion: reuse the same worker immediately, or release it.
  Keep it only on the human's request.
- A failed launch may leave a stray terminal/process: inspect it, fix the cause,
  retry into the same resource (linking the failed attempt). No orphans, no duplicates.
- If the human typed into a worker, their input outranks the dispatch — read the
  tail before closing; ask if an instruction is pending.
- Close: every task completed/failed, every worker accounted, a trace block in
  the plan (run id, task/attempt ids incl. failed ones and why).

## 8. What this contract does NOT prove

- Correctness or quality — only that declared state changed, the declared model
  ran, declared IDs were addressed, and someone else looked.
- It can't stop a controller doing delegated work itself; it only names it.
- Report shape ≠ approval. Format-perfect reports from weak tiers can hide
  failures; grade against ground truth (rerun the check, inspect the change).
- Pins are blind outside pinned targets.
