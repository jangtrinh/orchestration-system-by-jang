# Provider notes — guidance that depends on your models

Treat everything here as starting defaults, not measured facts for your setup.
Replace with your own measurements when you have them.

## Multi-tier mode (≥2 models)

- Default workhorse for bounded building; cheap tier for mechanical, reversible
  work with an automatic checker.
- For bounded *judgment* (choosing between designs, reviewing another model's
  output), a top model at low effort often beats a cheaper model at high effort.
  Observed on Anthropic models; unverified elsewhere — A/B it on a real task.
- Weak tiers produce format-perfect reports even when they failed. Always grade
  against ground truth.

## Single-model mode (1 model)

Tiers collapse, but the discipline still works:
- **Escalate by prompt, not model:** on failure, re-dispatch with a narrower task,
  more context, or the counsel role (a fresh session asked only for advice).
- **Effort** (if supported) becomes the only knob: low for mechanical, high for
  judgment and verification.
- **Independence comes from context, not model:** the verifier must be a fresh
  session that never saw the builder's reasoning — same model is acceptable,
  same context is not.

## No effort parameter

Ignore every "effort" field (write `n/a`). Escalation goes straight to the next
tier (multi-tier) or to prompt changes (single-model).

## Re-verification

Model IDs, context limits, and effort values change with provider releases.
Re-check them after upgrades and update `orchestration-config.md` with the date.
