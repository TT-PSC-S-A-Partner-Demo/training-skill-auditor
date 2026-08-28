# skill-auditor

A skill that audits other skills against Anthropic's authoring best practices, then
gives a score, a readiness gate, and a **split verdict** (when to break a skill up).

Built for the workshop: run it live on the three flawed fixtures and watch it catch a
missing eval set, a god-skill, and a vague description.

## What it checks

- **Description** - third person, WHAT + WHEN + 3 trigger phrases, right length (hard
  ceiling 1024 chars, not so short it is vague)
- **Naming / frontmatter** - kebab-case, ≤64 chars, no reserved words, no XML tags
- **Single responsibility** - one job, not "and also... and also..."
- **Body & structure** - under 500 lines, concise, references one level deep
- **Evaluations** - 3+ eval scenarios with expected_behavior (no evals caps the gate)
- **Instruction quality** - imperative, specific, concrete examples

## Files

This README lives **outside** `skill-auditor/` on purpose - a skill folder should not
contain a README.md (it confuses discovery). The loadable folder holds only:

- `SKILL.md` - the auditor itself
- `evals.json` - its own evals (it passes its own check)
- `fixtures/` - three deliberately flawed skills to audit live:
  - `no-evals-skill/` - solid otherwise, ships zero evals
  - `god-skill/` - one skill doing six jobs → SPLIT
  - `vague-description-skill/` - `description: Helps with projects.`

## Install (any agent)

It follows the open Agent Skills standard (SKILL.md folder), so it drops into any
standard-compliant agent. **What else is needed: nothing** - it is markdown-only, zero
runtime dependencies. The `evals.json` and `fixtures/` travel with it but cost nothing
until read.

Copy the folder into a skills directory, then it loads by `name`:

```bash
cp -r skill-auditor .agents/skills/skill-auditor    # universal bus - every standard agent
cp -r skill-auditor ~/.claude/skills/skill-auditor  # Claude Code (or .claude/skills/ per repo)
cp -r skill-auditor ~/.codex/skills/skill-auditor   # Codex
cp -r skill-auditor .cursor/skills/skill-auditor    # Cursor
cp -r skill-auditor .gemini/skills/skill-auditor    # Gemini CLI
```

Or install from a git repo with the universal loader (works across Codex, Claude Code,
Cursor, Windsurf, Aider, Copilot and more):

```bash
npx openskills install <owner>/<repo>
```

> To use `npx openskills`, push this folder to a git repo first. For a quick local demo,
> the `cp` lines above are enough - no repo, no network.

Then, in the agent:

```
Audit skill-auditor/fixtures/no-evals-skill/SKILL.md
Is skill-auditor/fixtures/god-skill/SKILL.md too big? Should I split it?
Review the description in skill-auditor/fixtures/vague-description-skill/SKILL.md
```

## Thresholds, and where they come from

All anchored in Anthropic's skill-authoring best practices: body under 500 lines,
description max 1024 chars, references one level deep, reference files over 100 lines get
a table of contents, and at least three evaluations per skill.
