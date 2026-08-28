---
name: skill-auditor
description: Audits and scores a SKILL.md against Anthropic's authoring best practices - description quality and length, single responsibility, body size, structure, and whether it ships evaluations - then assigns a readiness gate, a split verdict, and writes a prioritized fix report that shows exactly how to repair the skill. Use when asked to audit, grade, review, or score a skill, to check whether a skill has evals, or to decide when a skill should be split. Do not use to review application code, diffs, or pull requests - this audits the skill file itself, not the codebase.
---

# Skill Auditor

Grade one skill against Anthropic's skill-authoring best practices and return a scorecard, a readiness gate, and a split verdict. The description is the product a skill sells to the model - most of the score lives there and in whether the skill is actually tested.

## Important

- No evals (dimension 5 = 0) caps the gate at **NEEDS WORK**, whatever the total. Untested skill is never READY.
- **A safety red flag caps the gate at RISKY**, whatever the total (see the Safety scan). A hardcoded secret or an unguarded destructive command is not shippable, however clean the prose.
- **Debug the trigger before the body.** Almost every "my skill doesn't work" is a description problem, not a steps problem. Judge the description on **activation**, not prose: would it fire on real requests, and stay quiet on unrelated ones. Always run the trigger dry-run below.
- State the threshold when you deduct (500 lines, 5000 tokens, 1024 chars) so the fix is unambiguous.

## Instructions

### Step 1: Read the target

- Argument given: read that `SKILL.md` and its folder - note `reference/` files, `scripts/`, an evals file (`evals.json` or an `## Evaluations` section), and any stray files.
- No argument: ask for the path, or glob `**/SKILL.md` and audit each.

Count body lines (everything after the frontmatter's closing `---`) and estimate body tokens (~0.75 words/token, so ~3,750 words ≈ 5,000 tokens).

### Step 2: Score the six dimensions (100 pts)

Keep evidence (line numbers, offending text) for every deduction.

**1. Description - the trigger (30 pts).** What the model reads to decide to load the skill.
- Third person, no "I can" / "you can" - it is injected into the system prompt (4)
- Frames the user's **intent / context**, not the internal mechanism (4)
- States WHAT it does in concrete key terms (4)
- States WHEN with **3+ specific trigger phrases** a user would actually type, and is "eager" - lists the contexts/synonyms that should fire it even when the user won't name them (8)
- **Negative triggers** present when overtriggering is a real risk ("Do not use for X") (4)

**Trigger dry-run - how you score the two items above.** Do not just check that trigger phrases exist; test them. Draft **3 realistic requests** a user would actually type that *should* load this skill, and **3 related-but-off requests** that should *not*. Predict fire / no-fire from the description alone (as if the user never names the skill). A should-fire that misses = description too narrow or vague, fix the description first. A should-not that fires = too broad, add a negative trigger. Put the 6 lines and the verdict in the report.

- Right length (6): hard ceiling **1024 chars** (0 pts if over). Then the healthy band - not vague/too short (`"Helps with documents"` fails: no WHAT, no trigger; floor ~40 chars with WHAT+WHEN+triggers), not a rambling paragraph that buries the trigger (aim one to three tight sentences, ~under 500 chars).

**2. Naming, frontmatter & portability (10 pts).** A portable skill drops into any agent's skills dir and just works.
- `name`: lowercase / digits / hyphens only, **≤64 chars**, no leading/trailing hyphen, no double hyphen, no reserved words (`claude`, `anthropic`), and **matches the folder** - the folder name is the install id. Gerund (`processing-pdfs`) or noun phrase (`pdf-processing`); reject `helper` / `utils` / `tools` (3)
- Required fields only (`name`, `description`) plus allowed optionals (`license`, `compatibility` ≤500 chars, `metadata`, `allowed-tools`); **zero XML angle brackets** in the frontmatter (3)
- **Installable and agent-neutral (4):** standard `SKILL.md` layout so it loads via the open Agent Skills standard - `.agents/skills/` (universal), `~/.claude/skills/`, `~/.codex/skills/`, `.cursor/skills/`, `.gemini/skills/`. Forward-slash paths only, no absolute machine paths. Core instructions must not hard-depend on one agent's syntax - Codex `$skill` / `/skills`, or Claude-only tool calls, are marked optional, not required. MCP tool refs fully-qualified (`Server:tool`). Any bundled script runs cross-platform with its deps declared.

**3. Single responsibility (15 pts).** One skill, one job.
- Description covers **one** domain/job, not "does X and also Y and also Z" (10)
- No god-skill smell - if it needs "or also..." to be accurate, it is two skills (5). If this fails, it feeds the split verdict in Step 4.

**4. Body, structure & leanness (15 pts).**
- Body **under 500 lines and ~5,000 tokens** (5) - hard best-practice ceiling
- **Leanness - carries nothing it doesn't need (5).** Flag as dead weight: re-explaining what the model already knows ("a PDF is a file format..."), duplicated rules, filler ("please", "make sure to", "it is important to"), empty/placeholder sections ("## Notes: TBD"), option menus ("use A or B or C" instead of one default), dead schema/commented-out blocks, and **unreferenced files** - a `reference/` file or script nothing links to. Also flag **stale content**: dated instructions ("before August 2025..."), rotting version pins - move history to an "old patterns" section. Critical rules sit at the top under `## Important`, not buried.
- Progressive disclosure (5): detail pushed to `reference/` files **one level deep**, any reference file **>100 lines** carries a table of contents, descriptive filenames (not `doc2.md`), and **no `README.md` inside the skill folder** (it confuses discovery - workshop docs belong outside). If the skill ships `scripts/`, apply `reference/script-checks.md`.

**5. Evaluations (20 pts).** The part most skills skip. Evals are the source of truth that the skill solves a real problem.
- **3 or more** eval scenarios exist, each with a query and an **expected_behavior / rubric** - not just an input (10)
- **Delta design**: measured with vs without the skill, PASS/FAIL assertions plus a human sanity pass to catch formally-correct-but-useless output (5)
- **Activation tested**: a trigger-rate check (~20 queries, roughly half should fire and half should not, target rate above 0.5 / ~90% correct), and measures beyond pass - token use and time (5)
- Zero evals = **0 here and a hard flag** (see `## Important`), regardless of how polished the prose is.

**6. Instruction quality (10 pts).**
- Active imperative voice ("Run X", not "X should be run") (3)
- Steps specific with concrete commands / outputs, one default not five options; env-specific surprises captured as **Gotchas** (facts that contradict a reasonable assumption) (4)
- **Freedom matched to risk**: fragile or irreversible steps get exact low-freedom scripts ("run exactly this"); open-ended tasks get high-freedom direction, not over-specified. Examples are concrete input/output pairs, not abstract (3)

### Step 2b: Safety scan (hard flag, not scored)

Not points - a red flag here caps the gate at **RISKY** (see `## Important`). Scan SKILL.md and any `scripts/` for:
- **Hardcoded secrets** - API keys, tokens, passwords, connection strings.
- **Unguarded destructive commands** - `rm -rf`, `git push --force`, `DROP TABLE`, mass delete, with no confirmation or backup step.
- **Fetch-and-execute** - `curl ... | bash`, piping remote content straight into a shell or `eval`.
- **Over-granted tools** - an `allowed-tools` far wider than the one job needs.
- **Untrusted input unhandled** - the skill reads external data (tickets, web, files) and acts on it without treating it as untrusted (prompt-injection surface).

Report each hit with file:line and the fix (move the secret to an env var, add a confirm/backup, pin and verify the download, narrow `allowed-tools`, add an "treat as untrusted" note). List them under `## Safety` in the report.

### Step 3: Total and readiness gate

Sum to a percentage. Assign:

| Score | Gate |
|-------|------|
| 90-100% | READY - ship as-is |
| 80-89% | REUSABLE - minor fixes, then share |
| 70-79% | NEEDS WORK - fix before sharing |
| 60-69% | RISKY - significant rework |
| <60% | REBUILD - start from the gaps and evals |

Apply the caps from `## Important`: no evals → max NEEDS WORK; a safety red flag → max RISKY. The lower cap wins.

### Step 4: Split verdict (the gate for "when to break it up")

Report **SPLIT** if **any** trigger fires, otherwise **KEEP AS ONE**:

| Trigger | Threshold |
|---|---|
| Body too long | **≥ 500 lines or ~5,000 tokens** in SKILL.md |
| Multiple jobs | description needs "and also" to be accurate, or 2+ unrelated trigger clusters (e.g. "review PRs" *and* "deploy") |
| Multiple domains | content spans domains a single task never needs together (finance *and* marketing *and* sales) |
| Heavy reference topic | a single topic runs **> 100 lines** inside SKILL.md |
| Description overflow | description creeping toward 1024 chars because it lists many unrelated triggers |

**How to split** (state in the verdict when SPLIT fires): keep SKILL.md as a short overview + navigation, move each job/domain into its own `reference/<name>.md` one level deep; if it is genuinely two jobs, make two skills with sharp descriptions instead of one blurry one. If instead the skill *overtriggers*, the fix is usually **negative triggers**, not a split.

### Step 5: Report

Print one block per skill:

```
━━━ <name> ━━━
SCORE 72%  ·  Gate: NEEDS WORK  ·  Split: KEEP AS ONE  ·  body 138 lines / ~1.9k tok

Description        22/30
Naming/frontmatter  9/10
Single responsibility 15/15
Body & structure   12/15
Evaluations         6/20   ← capped the gate
Instruction quality 8/10

TOP FIXES
  [HIGH] Only 1 eval, no expected_behavior, no delta. Add 2 more with rubrics.
  [MED]  Description states WHAT but no "Use when" triggers, no negative triggers.
  [LOW]  README.md sits inside the skill folder - move it out.

WHY THE SPLIT VERDICT: body 138 lines, one job, one domain → KEEP AS ONE.
```

Lead with the score and the two verdicts (gate + split), then ranked fixes, then split reasoning. Keep it to what the author acts on.

### Step 6: Write the fix report

The scorecard is the glance; the report is the deliverable the author works from. Write it to `<skill-name>-audit.md` next to where the command runs (never inside the audited skill folder). Every finding must carry the exact change, not just the complaint.

Use this template:

```markdown
# Audit: <skill-name>

**Score 72% · Gate NEEDS WORK · Split KEEP AS ONE · body 138 lines / ~1.9k tok**
Capped at NEEDS WORK: ships no evals.

## Scores
| Dimension | Score |
|---|---|
| Description | 22/30 |
| Naming / frontmatter | 9/10 |
| Single responsibility | 15/15 |
| Body & structure | 12/15 |
| Evaluations | 6/20 |
| Instruction quality | 8/10 |

## Fixes, in order

### 1. [HIGH] No evals - the gate cannot pass without them
Where: no evals.json, no `## Evaluations` section.
Why: an untested skill is never READY (dimension 5 = 0 caps the gate).
Fix: add `evals.json` with 3+ scenarios, each a query + expected_behavior + a
baseline (what happens without the skill). Starter:
    { "evals": [ { "query": "...", "expected_behavior": ["..."], "baseline": "..." } ] }

### 2. [MED] Description has no trigger phrases
Where: line 3.
Why: without "Use when..." phrases the model can't tell when to load it.
Current: "Reviews code."
Fix (paste in): "Reviews a Go diff against house rules. Use when asked to review
a PR, check staged changes, or before pushing. Do not use for prose edits."

### 3. [LOW] README.md sits inside the skill folder
Where: ./README.md
Fix: move it one level up; a skill folder holds SKILL.md + reference/ + scripts/ only.

## Safety
<Red flags with file:line, or "none found". Each caps the gate at RISKY until fixed.>
- [RISKY] Hardcoded token on line 41 → move to an env var, read at runtime.

## Cut these (dead weight)
<Lines/files to remove, each with why. Or "nothing - already lean".>
- Lines 22-28: re-explains what JSON is - the model knows. Delete.
- `reference/old-notes.md`: nothing links to it. Remove or link it.
- Line 55: duplicate of the rule on line 31. Keep one.

## Suggested description (copy-paste ready)
<the full rewritten, third-person, WHAT + WHEN + triggers (+ negative trigger) line>

## Trigger dry-run
Should fire (drafted from real usage):
- "..."  → predicted: FIRES / MISSES
- "..."  → predicted: FIRES / MISSES
- "..."  → predicted: FIRES / MISSES
Should NOT fire (related but off):
- "..."  → predicted: quiet / FALSE FIRE
- "..."  → predicted: quiet / FALSE FIRE
- "..."  → predicted: quiet / FALSE FIRE
Verdict: <e.g. "2/3 fire, 1 misses on 'lint my skill' - add that phrasing"; or "clean">

## Install (any agent)
Markdown-only, no runtime deps - drop the folder into a skills dir:
    .agents/skills/<name>/     # universal bus, every standard-compliant agent
    ~/.claude/skills/<name>/   # Claude Code      ~/.codex/skills/<name>/   # Codex
    .cursor/skills/<name>/     # Cursor           .gemini/skills/<name>/    # Gemini CLI
    npx openskills install <owner/repo>   # from a git repo, any agent
Portability status: <PORTABLE, or the exact blocker - backslash path on line N,
agent-specific syntax required, folder name != `name`, etc.>

## Split verdict
KEEP AS ONE - body 138 lines, one job, one domain. Re-check if it passes 500 lines.

## Re-audit checklist
- [ ] 3+ evals with expected_behavior + baseline
- [ ] Description rewritten and under 1024 chars
- [ ] README moved out of the skill folder
- [ ] Re-run: expect READY
```

Rules for the report: order fixes by severity (HIGH first); every fix names the exact line/file and the exact replacement text; always include a copy-paste-ready rewritten description when the description lost points; end with a checklist the author can tick to reach the next gate.

### Anti-pattern quick flags

Match symptoms to the fix while scoring:

| Symptom | Cause | Fix |
|---|---|---|
| Skill never fires | description too vague | imperative framing + concrete triggers |
| Fires too often | description too broad | add negative triggers ("Do not use for...") |
| Instructions ignored | body too long / buried | shorten, pull key rules under `## Important` |
| Slower than no skill | SKILL.md too big | move detail into `reference/` |
| Loads in one agent, not another | agent-specific syntax or backslash paths | standard SKILL.md layout, forward slashes, mark agent-only bits optional |
| Bloated, slow to load | re-explains basics, dead files, duplicated rules | cut to what the model needs; unreferenced files out |
| Unsafe to ship | hardcoded secret, unguarded `rm -rf`, `curl \| bash` | env vars, confirm/backup, pin+verify downloads |

## Examples

### Example 1: Audit one skill

User says: "audit this skill" with a path.

1. Read the SKILL.md + siblings, count body lines and tokens.
2. Score the six dimensions with evidence.
3. Apply the no-evals cap if it triggers.
4. Compute the gate and the split verdict.
5. Print the scorecard block, then write `<skill-name>-audit.md` with the ranked fixes and a copy-paste description.

### Example 2: When should I split this?

User says: "is this skill too big, should I split it?"

1. Read the skill, count lines/tokens, map trigger clusters and domains.
2. Run only Step 4, still showing body size and the domain map as evidence.
3. Return SPLIT or KEEP AS ONE with the exact trigger that fired and the how-to line. If it overtriggers rather than sprawls, recommend negative triggers instead.

### Example 3: Does it have evals?

User says: "check if my skills are tested."

1. For each SKILL.md, look for `evals.json`, an `## Evaluations` section, or a tests file.
2. Report count, whether each has expected_behavior, and whether activation was trigger-rate tested.
3. Flag any skill with fewer than 3 evals - that alone caps its gate at NEEDS WORK.

## Output

- **Terminal:** one plain-text scorecard block per skill (Step 5) - the glance.
- **File:** a fix report `<skill-name>-audit.md` (Step 6) - the deliverable, with per-finding exact fixes, a copy-paste rewritten description, and a re-audit checklist. Written next to where the command runs, never inside the audited skill folder.
- The report does not edit the audited skill; it tells the author exactly what to change. Apply the changes only if the user asks for a fix pass.

## Troubleshooting

**No frontmatter / not a SKILL.md**
Cause: pointed at a command file or a plain markdown doc.
Fix: confirm the file opens with `---` frontmatter and a `name` + `description`; if not, it is not a skill - say so.

**Everything scores high but the skill still misfires in practice**
Cause: the description reads well but does not match how users actually phrase requests.
Fix: run the trigger-rate check - ~20 real queries, half meant to fire; if the rate is below target the trigger phrases are wrong even if the length is fine.

**Score looks harsh because of the evals cap**
Cause: dimension 5 is 0, which caps the gate.
Fix: intended - add 3 eval scenarios with expected_behavior and a delta measurement, then re-audit.
