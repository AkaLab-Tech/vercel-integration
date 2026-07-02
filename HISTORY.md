# History

Completed work log. Newest first. Each entry references the PR(s) that delivered the work.

## 2026-07

### Self-resolving CLI launcher — `link-cli` survives plugin updates — 2026-07-02
**PR:** _pending_ (branch `fix/self-resolving-cli-launcher`)

`atelier-vercel link-cli` symlinked `~/.local/bin/atelier-vercel` to the
script's resolved path. Run from Claude Code's plugin cache that path is
versioned (`.../vercel-integration/0.2.0/scripts/atelier-vercel`), so every
`claude plugin update` left the symlink dangling.

**Delivered:**
- `cmd_link_cli` now writes a small executable launcher (regular file, not a
  symlink) that resolves the ACTIVE install on every invocation via Claude
  Code's `installed_plugins.json`
  (`.plugins["vercel-integration@akalab-tech"][0].installPath`), falling back
  to the link-time path for git-clone / dev installs or a missing manifest.
- The launcher is emitted to a temp file and `mv`ed into place after `rm -f`
  of the old target — a bare `>` redirect onto the existing symlink would
  follow it and clobber the plugin-cache script (this exact incident motivated
  the guard).
- Docs aligned: `usage()` (`link-cli` / `configure`), `commands/setup.md`.
- Version 0.2.0 → 0.3.0 (`plugin.json` + CLI `VERSION`).

**Tests:** `bash -n` + `shellcheck -S warning` clean; hermetic scratch-HOME
suite: `link-cli` run from a fake plugin cache (9.9.9) installs a regular
executable file whose invocation runs the 9.9.9 copy; repointing the fake
manifest at a 10.0.0 copy makes the same launcher run 10.0.0 **without
re-linking**; a pre-existing symlink target is replaced without clobbering the
cached script (byte-identical before/after); clone/dev fallback works with no
manifest; `configure --non-interactive` end-to-end against a scratch
`settings.json`.

### Release 0.2.0 — 2026-07-02
**PR:** [#6](https://github.com/AkaLab-Tech/vercel-integration/pull/6) — branch `chore/release-0.2.0`

Version bump for the PR #5 contract change: `remove`, `env-rm` and `project-rm`
now require `--yes` non-interactively, and production writes left the unattended
allowlist. Also realigns the CLI `VERSION` (was 0.1.0) with `plugin.json` (0.1.1),
a pre-existing drift.

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
