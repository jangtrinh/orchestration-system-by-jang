# Runtime adapters — map abstract operations to your runtime

`SKILL.md` speaks in abstract operations. Use the section named in
`orchestration-config.md`. Commands are examples: confirm against your installed
version's `--help`/docs before relying on them.

## Abstract operations

| Operation | Meaning |
|---|---|
| `run` | Namespace for one objective; groups tasks and messages |
| `task` | Durable work item with a spec and dependencies |
| `attempt` | One launch of a task; retries get a new attempt ID |
| `launch` | Start a worker on a task with a chosen model (+ effort, placement) |
| `wait` | Block on completion / question / escalation |
| `ask/reply` | Worker question → controller answer |
| `complete` | Worker's single completion message (contract §4) |
| `release` | Free the worker after accepted completion |
| `isolate` | Separate copy/branch/workspace for parallel writers |

## Claude Code

| Operation | Mapping |
|---|---|
| run / task | Plan folder + acceptance.md (task-list tools if available) |
| attempt | Each sub-agent launch; record the returned agent ID |
| launch | Agent/Task tool with an agent type and optional model. If your existing rules mandate an orchestrator (e.g. Orca), use that section instead |
| isolate | git worktree isolation if offered; without git, a copied folder per writer |
| wait | Background completion notification; idle ≠ done |
| ask/reply | Message tool to the agent ID, only if that agent type has one |
| complete | Final report text, or a file at the declared report path |
| release | Automatic when the sub-agent ends |

Check each agent type's tool list before dispatch: types without a messaging tool
must write their report to an exact file path you then read.

## Codex CLI (externally orchestrated)

| Operation | Mapping |
|---|---|
| launch | `codex exec -m <model> -c model_reasoning_effort=<level> -s <sandbox> -C <dir> "<spec>"` |
| isolate | One `git worktree add` (or copied folder) per writer; point `-C` at it |
| complete | Exit code **plus** report file at the declared path (exit 0 ≠ done) |
| release | Process exit; remove the worktree/copy after verification |

Sandbox: reviews `read-only`; building `workspace-write`; full access only inside a
disposable, isolated environment.

## Orca (visible terminals, worktrees, Run/Task/Dispatch lifecycle)

Load current syntax first: `orca status --json`, `orca skills get orca-cli`,
`orca skills get orchestration`.

| Operation | Mapping |
|---|---|
| run | `orca orchestration run-create --objective "<product · outcome>" --json` |
| task | `orca orchestration task-create --task-title … --display-name … --spec … [--deps '["task_x"]'] --json` |
| launch | `orca orchestration worker-start --task <id> --worktree current\|new-child\|new-top-level --agent <claude\|codex> --model <id> --effort <lvl> --json` |
| custom agent profile | `orca terminal create --command "claude --agent <profile>" --json` → `orca terminal wait --for tui-idle` → `worker-start --task <id> --terminal <handle>` |
| wait | `orca orchestration check --wait --types worker_done,escalation,question --timeout-ms 900000 --json` |
| ask/reply | `orca orchestration reply --id <msg-id> --body … --json` |
| complete | `orca orchestration send --type worker_done --task-id … --dispatch-id … --outcome succeeded\|failed --report-path … --json` |
| release | `orca orchestration worker-release --dispatch <id>`; terminal-attached dispatches return `external_terminal` → read tail, then `orca terminal close` |

A valid `worker_done` settles task + dispatch; don't also mark the task completed.

## Cursor / Gemini CLI / other IDE agents

If the tool can launch background or parallel agents, map `launch` to that feature
and require every worker to write its report to the declared path. If it cannot,
use `single-session` below.

## Generic runtime

Minimum: launch with a chosen model, a unique attempt ID, a completion signal
you can wait on, and a report file path. Without a question channel, workers
must assume-and-state instead of asking (see the preamble).

## single-session (no sub-agents)

You play every role in sequence. What still applies: acceptance IDs, the spec
template (as your own brief), state pin, post-checks, approval gates, the report
format. What breaks: builder ≠ verifier. Mitigate: ask the human to open a
**fresh session** with only the verifier brief (§5) and the result; if that is
impossible, state plainly in the report that verification was not independent.
