# History

Completed work log. Newest first. Each entry references the PR(s) that delivered the work.

## 2026-07

### Gate production writes behind operator confirmation — 2026-07-01
**PR:** [#5](https://github.com/AkaLab-Tech/vercel-integration/pull/5) — branch `fix/gate-production-writes`

Audit hardening: the agent allowlist granted production-affecting writes, and
the destructive wrappers passed `--yes` to the vendor CLI with no confirmation
of their own.

**Delivered:**
- `VERCEL_ALLOW` re-cut to read ops + preview `deploy` + `link`/`pull`; removed `deploy-prod`, `promote`, `rollback`, `redeploy`, `env-add` so they fall through to Claude Code's default "ask" prompt.
- New `VERCEL_REVOKED` list: `enable-permissions` strips the previously granted production entries on upgrade, and `disable-permissions` removes them from existing `settings.json` files too.
- In-CLI confirmation for `remove`, `env-rm`, `project-rm` (`_confirm_destructive`): explicit `--yes` flag or interactive `y/N` required; non-TTY without `--yes` dies before the vendor CLI is invoked.
- Docs aligned: `usage()`, `skills/vercel/SKILL.md`, `commands/setup.md`, `README.md`, `CLAUDE.md`.

**Tests:** `bash -n` + shellcheck clean; `--help` runs; gated commands die at the guard pre-network without `--yes` on non-TTY stdin; `enable-permissions`/`disable-permissions` verified idempotent and valid-JSON against a sandbox `settings.json` (including stale-grant cleanup).
