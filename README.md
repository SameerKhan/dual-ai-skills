# dual-ai — two models, one codebase

Two [Claude Code](https://claude.com/claude-code) skills that put **Claude and
OpenAI's Codex CLI in an adversarial loop** on the same work, instead of
trusting either model alone:

- **`/dual-plan`** — Claude drafts an implementation plan, Codex critiques it
  against the actual codebase (read-only), Claude adjudicates each point, and
  they loop until Codex signs off **SOUND** (max 3 rounds — after that,
  unresolved disagreements are surfaced to you as a decision point, never
  papered over). You approve the converged plan before any code is written.
- **`/dual-review`** — before a merge, Claude's `/code-review` and
  `codex exec review` run **in parallel** on the same diff. The findings are
  merged: agreement between two different models is a strong signal;
  Codex-only findings are verified against the code before being reported
  (rejected ones get one rebuttal round); disputes are shown with both
  positions. Every finding is tagged `[both]`, `[claude]`, `[codex]`, or
  `[disputed]`.

Why bother? Two different models trained by two different labs make
*different* mistakes. In practice the merge step catches real bugs that
either reviewer alone waves through — and the forced "verify before you
relay / concede or defend" rules stop one model from rubber-stamping the
other's hallucinations.

## Prerequisites

1. **Claude Code** (CLI, desktop, or IDE extension) — runs the skills.
2. **OpenAI Codex CLI** — `codex` on your PATH, authenticated
   (`codex login` or via the Codex desktop app).

Both are needed; the whole point is two independent vendors.

## Install

**Option A — as a plugin (recommended):** in Claude Code, run

```
/plugin marketplace add SameerKhan/dual-ai-skills
/plugin install dual-ai@dual-ai-skills
```

**Option B — plain copy:**

```bash
git clone https://github.com/SameerKhan/dual-ai-skills
cp -r dual-ai-skills/plugins/dual-ai/skills/* ~/.claude/skills/
```

(Or into a repo's `.claude/skills/` to share it with just that team/project.)

Then just say **"dual plan this feature"** or **"dual review this branch"**
in Claude Code.

## Not on Claude Code? (Cursor, Antigravity, etc.)

The SKILL.md files are plain-markdown playbooks — there's no code in them.
Two options:

- **Easiest:** install the Claude Code CLI and run it inside your IDE's
  terminal. The skills work anywhere the CLI runs.
- **Adapt:** paste the SKILL.md contents into your agent's custom
  instructions / workflow mechanism. Any agent that can run shell commands
  can drive the `codex exec` half; the merge-and-verify rules are
  model-agnostic. (You can also invert it — have Codex or Gemini drive and
  use `claude -p` as the second reviewer.)

## Tips learned the hard way

- **Always force Codex's reasoning effort to `high`** for reviews
  (`-c model_reasoning_effort="high"`). Default/low effort produces
  confident-sounding but shallow reviews.
- **Never let both models write to the same working tree.** One drives, the
  other critiques. The optional co-coder mode in `/dual-plan` uses an
  isolated `git worktree` for exactly this reason.
- **Cap the argument loops** (3 rounds for plans, 1 rebuttal for reviews).
  Two LLMs will trade nits forever if you let them.
- **Keep an `AGENTS.md` in your repos** (Codex reads it automatically). A
  copy of your CLAUDE.md works — both models should see the same ground
  rules.
- macOS Codex desktop app: if you symlink `codex` out of
  `/Applications/Codex.app`, symlink its sibling `codex-code-mode-host` into
  the same directory too, or every run dies with
  "failed to spawn code-mode host".

## License

MIT
