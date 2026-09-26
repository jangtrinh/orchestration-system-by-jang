# Orchestration config — local choices for Orchestration System by Jang

Copy to `orchestration-config.md` and fill every `<...>`. Mark guessed values
`(ASSUMPTION)`. This is the only file you edit; core files stay untouched.

## Scope
- Scope: <global | project: path>
- Installed at: <absolute path of this folder>
- Pointer line added to: <file path | none (auto-loaded)>

## Runtime
- Runtime: <Claude Code | Codex CLI | Cursor | Gemini CLI | Orca | other: name>
- Adapter section: <name from references/runtime-adapters.md>
- Can spawn sub-agents: <yes | no → adapter single-session>
- Shell: <POSIX | PowerShell>

## Models and routing
- Routing mode: <multi-tier | single-model>
- cheap tier: <model id | n/a>
- workhorse tier: <model id>
- deep tier: <model id | n/a>
- strongest tier (counsel/judgment): <model id | n/a>
- Effort control: <parameter name and allowed values | not supported>
- Model IDs last verified: <YYYY-MM-DD, how>

## Work
- Domain: <code | research | content | data | design | notes/knowledge-base | mixed | other: describe>
- Pin method(s): <git | file-snapshot | external-state: which system>
- Work dir for plans: <path, default plans/>
- Reports dir: <path, default plans/reports/>
- Max verify→fix rounds: <3>

## Precedence
- Existing orchestration rules found: <paths | none>
- On conflict: <existing rules win (default) | this system wins | merged: how>

## Approval gates (always ask the human first)
- <delete, publish, deploy, merge, migrate, spend money, message people, …>

## Human-specific notes
- <language, reporting style, anything the human asked for>
