# SETUP — instructions for the AI agent installing this system

You are installing **Orchestration System by Jang** for your human. Follow the
steps in order. Detect before you ask; ask at most 4 questions, all in one message.
Do not modify `SKILL.md` or `references/` — local choices go only in
`orchestration-config.md`.

Success test for this setup:
- S1 `orchestration-config.md` exists, every field filled or `n/a`, no `<...>` left.
- S2 The system folder sits where your runtime loads it (§2), or a pointer line
  exists in an always-loaded instruction file.
- S3 A fresh session (or re-read of the instruction file) can find `SKILL.md`.
- S4 The dry-run task spec in §6 is complete and cites acceptance IDs.
- S5 You reported S1–S4 to your human with evidence, including anything NOT verified.

## 1. Detect (read-only; no questions yet)

Record each finding as FACT (observed) or ASSUMPTION.

| Detect | How |
|---|---|
| Runtime | Which one you are running in; also look for `~/.claude/`, `.claude/`, `~/.codex/`, `AGENTS.md`, `.cursor/rules/`, `GEMINI.md`, `~/.gemini/`, an `orca` binary |
| Can spawn sub-agents? | Do you have a tool/command that launches another agent with its own context? |
| Worker models | Models you can choose at spawn time (tool schema, CLI `--help`, config files) |
| Effort control | Does launch accept a reasoning-effort/thinking-budget parameter? |
| Work domain | Git repo present? Code files? Or documents/data/design assets? |
| State-change evidence | git (diff) · files (mtime/snapshot) · external system (API/DB/doc versions) |
| OS shell | POSIX (macOS/Linux) or Windows (PowerShell) |
| Existing rules | Existing orchestration/delegation rules in instruction files — never overwrite them |

## 2. Choose install location

| Runtime | Put the folder at | Pointer line goes in |
|---|---|---|
| Claude Code | `~/.claude/skills/orchestration-system-by-jang/` (global) or `.claude/skills/…` (project) | optional: `CLAUDE.md` |
| Codex CLI | `~/.codex/skills/orchestration-system-by-jang/` if skills are supported, else any path | `AGENTS.md` |
| Cursor | `.cursor/orchestration-system-by-jang/` | a rule in `.cursor/rules/` (always apply) |
| Gemini CLI | `~/.gemini/orchestration-system-by-jang/` or project | `GEMINI.md` |
| Orca | same as the agent runtime Orca launches | that runtime's file |
| Other / chat-only | any stable path, or paste `SKILL.md` into the system prompt | system prompt / custom instructions |

Pointer line (adapt path):
`Before delegating to, spawning, or supervising any agent, read <path>/SKILL.md and <path>/orchestration-config.md and follow them.`

## 3. Ask your human (one message, ≤4 questions, skip any you detected)

1. Global (all projects) or this project only?
2. Which models may workers use, cheapest to strongest? (Offer what you detected.)
3. Where should plans and reports live? (Default: `plans/` and `plans/reports/`.)
4. Which actions always need their approval? (Default: delete, publish, deploy,
   merge, migrate, spend money, send messages to people.)

If they don't answer, use the defaults and mark them ASSUMPTION in the config.

## 4. Write `orchestration-config.md`

Copy `orchestration-config.template.md` to `orchestration-config.md` in the same
folder and fill every field from §1 and §3. Pick:

- **Routing mode:** `multi-tier` (≥2 models) or `single-model` (1 model; see
  `references/provider-notes.md`).
- **Pin method:** `git` · `file-snapshot` · `external-state` (can list several).
- **Adapter:** the matching section name from `references/runtime-adapters.md`.
- **Mode for "no sub-agents":** if you cannot spawn agents, set adapter to
  `single-session` and read its limits.

## 5. Install

1. Put the whole folder at the §2 location (create parent dirs as needed). With
   git, prefer cloning straight there so `git pull` updates it later:
   `git clone https://github.com/jangtrinh/orchestration-system-by-jang.git <location>`.
   Otherwise copy the downloaded folder. If a folder with that name exists,
   compare first; never overwrite silently — ask your human.
2. Add the pointer line to the instruction file, appended, not replacing anything.
   Skip if the runtime auto-loads skills by description and the human chose global.

## 6. Verify (dry run — no workers launched)

1. Re-open the installed `SKILL.md` and `orchestration-config.md` from their final
   paths (proves S2/S3).
2. Search the config for unfilled `<` placeholders; there must be none (S1).
3. Draft, but do not dispatch, one task spec for a harmless sample task
   ("summarize the README of this project") using `references/dispatch-contract.md`
   §1, with acceptance IDs A1/C1/V1 and a routing line from the config (S4).
4. If your runtime can spawn agents and the human agrees, optionally dispatch that
   sample to the cheapest tier and run the §3 post-checks on the result.

## 7. Report to your human

Lead with what was NOT verified. Then: install path, pointer file changed (with the
exact line added), config summary (runtime, routing mode, pin method, approval
gates), the dry-run spec, and open questions. Keep the report short.
