# CLAUDE.md

vercel-integration is an optional, standalone Claude Code plugin that lets atelier agents deploy and manage apps on Vercel via the official Vercel CLI. It supports preview and production deployments, deployment inspection, log reading, env var management, project linking, and rollback. It is independent of atelier — install it only when deploying with Vercel.

## Stack

- **Language**: Bash (single script — `scripts/atelier-vercel`)
- **Framework**: Claude Code plugin (`/.claude-plugin/plugin.json` manifest v0.1.1)
- **Package manager**: pnpm (used as a fallback: `pnpm dlx vercel@latest` when `vercel` is not on PATH)
- **Test runner**: TBD — no test suite present
- **Linter / formatter**: TBD — no linter config present

## Architecture

The repo has no `src/` tree. All logic lives in two places:

- `scripts/atelier-vercel` — the CLI wrapper: loads per-project `.env` for `VERCEL_TOKEN`, dispatches to `vercel` (or `pnpm dlx vercel@latest`), exposes read/write subcommands, and manages user-level `settings.json` permission merging.
- `skills/vercel/SKILL.md` — the Claude Code skill definition: describes the deploy-and-verify flow and the rules agents follow (resolve with `list` first, confirm destructive ops).
- `commands/setup.md` — the `/vercel-integration:setup` slash command that runs `configure --non-interactive` then wires the project `.env`.

## Conventions

- **CLI invocation**: `atelier-vercel <command>` (or `${CLAUDE_PLUGIN_ROOT}/scripts/atelier-vercel <command>` from inside a session). Run `atelier-vercel --help` for the full reference.
- **Auth**: per-project `VERCEL_TOKEN` in `.env` (gitignored). Machine-wide PATH link + allowlist merge via `atelier-vercel configure`.
- **Destructive ops** (`remove`, `env-rm`, `project-rm`) are intentionally absent from the agent allowlist and require operator confirmation.
- **CI**: no GitHub Actions workflows present.

## What this project is NOT

- TBD

## Out of scope for AI agents

- Never `git add .env` or any file containing `VERCEL_TOKEN`.
- Never run destructive subcommands (`remove`, `env-rm`, `project-rm`) without explicit operator confirmation — they are gated outside the allowlist for this reason.
- Never edit `atelier`'s shipped `settings.template.json`; use `enable-permissions` / `disable-permissions` which target the user-level `settings.json` only.
