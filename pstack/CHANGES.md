# Changes vs upstream pstack

ZCode + GLM-5.3-Flash port of `cursor/plugins/pstack`. Compatibility port only —
behavior is preserved except where the Cursor harness had no equivalent. All
roles inherit the session model; there is no per-role model routing.

## Packaging

- Added `.zcode-plugin/plugin.json` and root `marketplace.json` (local ZCode
  marketplace). `.cursor-plugin/` kept untouched so the fork still installs in
  Cursor and upstream merges stay diffable.

## Dropped from the plugin (parked in `attic/`, kept for reference)

- `skills/setup-pstack` — its whole job was writing Cursor's
  `~/.cursor/rules/pstack-models.mdc`. With inherit-default everywhere the
  config has no job. Reintroduce a role-mapping version if cross-model CLIs
  get wired in.
- `skills/make-bot-ui` — Grok-Bot webhook/Tailscale niche skill.
- `skills/automate-me` — builds a personal -mode skill; redundant while
  poteto-mode defines the style.
- `skills/tdd`, `skills/unslop` — name collisions with the operator's existing
  user-level skills. References inside playbooks resolve to those copies.
- `automations/benny` — Cursor event-triggered automation pack.
- `docs/guide` — Cursor UI tutorial and screenshots.

## Vendored imports (not upstream)

- `skills/frontend-design`, `skills/code-review`,
  `skills/thermo-nuclear-review`, `skills/thermo-nuclear-code-quality-review`
  — from the operator's personal stack (`~/.zcode/skills`). Attributed there.
  Wired into poteto-mode triggers and the feature, visual-parity, bug-fix,
  shipping, and opening-a-pr playbooks.

## Harness substitutions (repo-wide)

- `Task` tool → `Agent` tool; `subagent_type: generalPurpose` →
  `"general-purpose"`; `AskQuestion` → `AskUserQuestion`.
- All model slugs removed. Every subagent omits `model` and inherits the
  session model (GLM-5.3-Flash). Slug-fallback paragraphs deleted; there is no
  `pstack-models.mdc` lookup anywhere.
- Cursor cloud agents → local background subagents (`run_in_background: true`)
  with one git worktree per writing worker.
- Cursor's `/loop` → ZCode Goal Mode or scheduled automations (CronCreate).
- `environment: "cloud"` / `cloud_base_branch` → local worktrees.
- Transcript paths `~/.cursor/projects/…` → genericized: the harness names the
  active workspace transcript directory. `worktree-audit.sh` now reads
  `PSTACK_TRANSCRIPTS` and fails fast when unset.
- `.cursor/skills/verify-*/` → `.zcode/skills/verify-*/`.
- `cursor-team-kit` references → bundled or browser-use equivalents:
  `control-ui` → browser-use `control-browser`; `control-cli` → repo-local
  control skill; `deslop` → unslop-style slop-strip; Bugbot triage generalized
  to review-bot-comment triage.

## Skill rewrites

- `poteto-mode`: Cursor-only frontmatter (`mode`, `icon`, `color`, `reminder`)
  stripped. The per-turn nudge now lives as a routing line in the operator's
  `~/.zcode/AGENTS.md`. Subagent defaults rewritten: omit `model`, worktree
  isolation for writers, second opinions via fresh isolated contexts
  (thermo-nuclear-code-quality-review carries the harsher-pass role).
- `interrogate`: multi-model panel → four independent lenses (correctness,
  security, maintainability, spec/verification) with differentiated prompts.
  Consensus language rewritten around lens agreement; verdict notes that the
  panel is single-model and correlated blind spots are possible.
- `arena`: model runner pool → N session-model candidates in isolated
  worktrees; judge inherits.
- `swarm`: cloud workers → local background workers, worktree per writer,
  race arms differentiated by approach not model.
- `no-comments`: spawns `subagent_type: "comment-sicko"`.
- `recall`, `reflect`, `show-me-your-work`, `why`: transcript/MCP environment
  paths and phrasing genericized.
- `babysit`, `pause-safely`, `session-pickup`, `orchestrate`, `shipping`,
  `multi-phase-plan`, `autopilot-*`, `worktree-cleanup`, `visual-parity`,
  `feature`, `bug-fix`, `opening-a-pr`: cloud-agent, dashboard, restart, and
  control-skill references adapted.

## Subagents

- `agents/poteto-agent.md`: `is_background` removed, description updated.
- `agents/comment-sicko.md`: renamed `Comment Sicko` → `comment-sicko` for
  valid subagent_type syntax.
- Both installed to `~/.zcode/agents/` by the operator (not packaged; ZCode
  loads agent definitions from the user dir).
