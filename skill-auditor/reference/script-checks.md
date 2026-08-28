# Script checks

Apply only when the audited skill ships a `scripts/` folder. Fold the results into the
report's Fixes and Safety sections. From Anthropic's "Skills with executable code".

## Checklist

- **Solve, don't defer.** Scripts handle their own error conditions (missing file,
  permission, bad input) instead of failing and leaving it to the model.
- **No voodoo constants.** Every magic number (timeout, retries, limits) is named and
  justified in a comment. If the author can't justify it, the model can't either.
- **Declared dependencies.** Required packages are listed in SKILL.md and the script does
  not assume they are installed.
- **Non-interactive.** No prompts that block automation - read args/env, never `input()`.
- **Structured output.** Emit JSON or CSV the model can parse, not free prose.
- **Idempotent.** Re-running does not double-apply or corrupt state.
- **Cross-platform.** Forward-slash paths, no OS-specific commands assumed.
- **Execute vs read is stated.** SKILL.md says whether to run the script or read it as
  reference - "Run `analyze.py`" not an ambiguous mention.

## Safety overlaps (also feed the Safety hard flag)

- No hardcoded secrets in the script.
- No unguarded destructive operations (`rm -rf`, force push, `DROP`) without confirm/backup.
- No fetch-and-execute (`curl ... | bash`, piping remote content into a shell/eval).

## How to score

Script problems land in the report as normal Fixes (severity by impact). A secret or an
unguarded destructive op is a **Safety red flag** and caps the gate at RISKY, same as in
the main scan.
