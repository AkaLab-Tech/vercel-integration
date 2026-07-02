---
description: Set up vercel-integration — link the CLI onto PATH, grant atelier agents the Vercel allowlist (user-level), and wire this project's .env with VERCEL_TOKEN
allowed-tools: Bash(${CLAUDE_PLUGIN_ROOT}/scripts/atelier-vercel:*), Read(.env), Edit(.env), Write(.env)
---

Set up the vercel-integration plugin, then report a short summary. Do every
step in order:

1. Run the machine-wide part (PATH launcher + user-level allowlist merge — never
   atelier's per-task template):

   ```sh
   "${CLAUDE_PLUGIN_ROOT}/scripts/atelier-vercel" configure --non-interactive
   ```

2. **Per-project auth.** Vercel auth is per project. If the current directory is
   a project, wire its `.env`:
   - Read `.env` (create it if missing). If `VERCEL_TOKEN` is absent, ask the
     operator for a Vercel access token (https://vercel.com/account/tokens) and
     add `VERCEL_TOKEN=<token>`. Never echo the token back.
   - Optionally add `VERCEL_ORG_ID` / `VERCEL_PROJECT_ID` if the operator wants
     a fixed scope (otherwise `atelier-vercel link` sets up `.vercel/`).
   - `.env` stays gitignored by atelier's `.env*` guardrail — never `git add` it.

3. Verify: `atelier-vercel whoami`.

## Report

PATH link status, permissions merged, whether `.env` has `VERCEL_TOKEN` (token
presence only, never the value), whether `vercel` is on PATH (or runs via
`pnpm dlx`), and the `whoami` result. Mention that the allowlist covers read
ops and preview deploys only: production writes (`deploy-prod`, `promote`,
`rollback`, `redeploy`, `env-add`) prompt the operator via Claude Code, and
`remove` / `env-rm` / `project-rm` additionally require an in-CLI `--yes` or
interactive confirmation — so an agent can never ship to or mutate production
unattended.
