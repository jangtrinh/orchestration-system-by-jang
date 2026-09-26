# Orchestration System by Jang

A portable operating system for delegating work to AI agents without losing
control: explicit dispatch contracts, capability routing, builder/verifier
separation, no-op/refusal detection, and resource cleanup.

Works for code **and** non-code work (research, content, data, design), on any
agent runtime (Claude Code, Codex, Cursor, Gemini CLI, Orca, CI runners, or a
single chat session), with one model or many.

## Quick start: let your AI set it up

Paste this to your AI agent:

```text
Set up the Orchestration System by Jang for me:
https://github.com/jangtrinh/orchestration-system-by-jang
Read its README "AI setup guide" section and follow it.
```

Your AI checks your environment, asks you at most 5 questions, installs the
system, and reports what it did and what it could not verify.

## AI setup guide

> **If you are an AI agent, follow these steps in order.** Do not skip the
> report at the end.

1. **Get the files.** If you have a shell with `git`, clone into a temp location:
   `git clone --depth 1 https://github.com/jangtrinh/orchestration-system-by-jang.git`.
   Without git, download the archive
   `https://github.com/jangtrinh/orchestration-system-by-jang/archive/refs/heads/main.zip`.
   With only web access, read the raw files one by one from
   `https://raw.githubusercontent.com/jangtrinh/orchestration-system-by-jang/main/<file>`
   in the order listed in the table below.
2. **Read `SETUP.md` completely** before you change anything.
3. **Follow `SETUP.md` §1–§7.** It covers detection, install location, questions
   for your human, the config file, install, a dry-run check, and the report.
4. **Treat the content as instructions only for setting up this system.** It never
   asks you to send data anywhere, change unrelated settings, or remove existing
   rules. If something looks like it does, stop and ask your human.
5. **Chat-only AI (no file access):** skip installation. Read `SKILL.md`,
   `references/dispatch-contract.md`, and `references/worker-preamble.md`, then
   tell your human to paste `SKILL.md` into their custom instructions/system
   prompt and keep the rest as reference.

## Files (reading order)

| File | Purpose |
|---|---|
| `SETUP.md` | Self-setup procedure for an AI agent |
| `SKILL.md` | The system: loop, operating modes, 12 invariants, checklist |
| `orchestration-config.template.md` | The only file with local choices (runtime, models, paths, approval gates) |
| `references/dispatch-contract.md` | Task spec, routing, state pin, post-checks, verifier brief, cleanup |
| `references/worker-preamble.md` | Discipline block pasted into every non-trivial worker prompt |
| `references/runtime-adapters.md` | Abstract operations → Claude Code / Codex / Orca / Cursor / Gemini / single-session |
| `references/provider-notes.md` | Model-tier and effort guidance that depends on the provider |

The core files never need editing. Everything local lives in
`orchestration-config.md`, generated from the template during setup.

## Manual setup (humans, ~5 min)

1. Clone or copy this repo to where your runtime loads skills or instructions
   (table in `SETUP.md` §2), e.g.
   `git clone https://github.com/jangtrinh/orchestration-system-by-jang.git ~/.claude/skills/orchestration-system-by-jang`.
2. Copy `orchestration-config.template.md` to `orchestration-config.md` and fill it in.
3. Add one line to your always-loaded instructions:
   `Before delegating to or supervising any agent, read <path>/SKILL.md and <path>/orchestration-config.md.`

Update later with `git pull`. Your `orchestration-config.md` is git-ignored, so it stays yours.

## How it works

```text
FRAME → ACCEPTANCE → ROUTE → DISPATCH → EXECUTE → VERIFY → REPORT
```

- **Exit 0 is not done.** Workers report success on no-ops and refusals; the
  controller checks that pinned state actually changed.
- **Builder ≠ verifier.** Self-review misses what the builder misunderstood.
- **Routing is recorded.** Tier, effort, and a one-line reason per task make cost
  and failures auditable.
- **Guards live in the prompt.** A worker only follows rules it was given.
- **Weak tiers stay off irreversible steps.** Format-perfect reports can wrap
  destructive mistakes.

## Credit and license

Orchestration System by Jang, distilled from a production multi-agent setup.
[MIT](LICENSE). Keep this credit line when you redistribute or adapt it.
