# Upstream

- Upstream: https://github.com/cursor/plugins (subdirectory `pstack/`)
- Fork: https://github.com/Dhruv2mars/plugins
- Branch: `feat/pstack-zcode-port` (sync work: `feat/pstack-sync-0.15.2`)
- Synced against upstream `main` at 889ec4b (v0.15.2, 2026-09-11).
- Keep `.cursor-plugin/` and skill prose untouched where possible so future
  upstream merges apply cleanly. Every intentional divergence is listed in
  CHANGES.md. On sync, re-apply: the global substitutions (see CHANGES.md),
  the vendored skills wiring, and the attic moves.
