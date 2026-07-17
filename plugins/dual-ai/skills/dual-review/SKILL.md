---
name: dual-review
description: Dual code review — run Claude's /code-review AND OpenAI Codex CLI review on the same diff, then merge both into one consolidated report. Use when the user says "dual review", "review with both", "Claude + ChatGPT review", or before a merge when they want two independent reviewers.
---

# Dual review (Claude + Codex/ChatGPT)

Two independent reviews of the same diff, merged into one report. The value is
in the merge: agreement between two different models is a strong signal;
disagreement tells the user where to look manually.

## Steps

1. **Determine diff scope.**
   - Branch has commits vs the main branch (check the repo's CLAUDE.md for
     which branch is the trunk — it is not always `main`): scope =
     `origin/<main>...HEAD`, Codex flag = `--base <main>`.
   - Only uncommitted working-tree changes: Codex flag = `--uncommitted`,
     and review the working-tree diff on the Claude side.

2. **Start the Codex review in the background** (it takes several minutes):

   ```bash
   codex exec review --base main -c model_reasoning_effort="high"
   # or: codex exec review --uncommitted -c model_reasoning_effort="high"
   ```

   Run via Bash with `run_in_background: true`. Always override reasoning
   effort to `high` — a low default in the user's `~/.codex/config.toml` is
   too weak for review. If the sandbox blocks Codex's network access, retry
   with the sandbox disabled.

3. **While Codex runs, invoke `/code-review` at high effort** on the same scope.

4. **Merge the findings** once both are done:
   - **Agreed** (both flagged the same issue): report first — highest confidence.
   - **Claude-only**: report as normal /code-review findings.
   - **Codex-only**: verify each against the actual code before reporting;
     mark CONFIRMED or REJECTED (with the reason). Never relay a Codex finding
     unverified.
   - Dedupe by file + line + issue.

5. **Rebuttal round — a refuted finding gets a defense.** If any Codex
   findings were REJECTED in step 4, send Codex ONE follow-up
   (`codex exec`, read-only) containing each rejected finding plus the
   refutation evidence, asking it to CONCEDE or DEFEND each with code
   citations. Concede → drop silently. Defend → re-examine once; if you still
   disagree, include it in the report as **[disputed]** with both positions
   and your recommendation — the user rules, same as dual-plan's deadlock
   rule. One rebuttal round only (reviews are expensive; the diff, unlike a
   plan, doesn't change mid-review). Skip this step entirely when nothing
   was rejected.

6. **Report one consolidated list**, most severe first, tagging each finding
   with who found it: `[both]`, `[claude]`, `[codex]`, or `[disputed]`.
   Never omit a disputed finding.

## Notes

- Requires the OpenAI Codex CLI (`codex`) installed and authenticated
  (`codex login`). Don't change the user's global `~/.codex/config.toml`;
  use `-c` overrides only.
- macOS + Codex desktop app: if `codex` is a symlink into
  `/Applications/Codex.app`, the sibling helper `codex-code-mode-host` must
  be symlinked into the same directory too — codex resolves that helper next
  to the invoked binary, and without it every run fails with "failed to
  spawn code-mode host" and returns a useless provisional verdict.
- Codex reads `AGENTS.md` for project context automatically. If the repo
  only has a CLAUDE.md, consider keeping an `AGENTS.md` copy so both models
  see the same ground rules.
- If Codex reports an auth error, tell the user to re-login (app or
  `codex login`) — do not attempt to fix auth yourself.
