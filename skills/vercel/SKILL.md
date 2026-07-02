---
name: vercel
description: >-
  Deploy and manage applications on Vercel via the atelier-vercel CLI (a thin
  wrapper over the official Vercel CLI). Use when the operator asks to deploy a
  project to Vercel, ship to production, check a deployment, read deploy logs,
  list or set environment variables, link a project, or roll back.
---

# Vercel

`atelier-vercel` wraps the official Vercel CLI. It reads `VERCEL_TOKEN` (and
optional `VERCEL_ORG_ID` / `VERCEL_PROJECT_ID`) from the current project's
`.env`. Run `atelier-vercel --help` for the full reference.

## Commands

| Command | Purpose |
| --- | --- |
| `atelier-vercel whoami` | Current scope/user |
| `atelier-vercel list [app]` | List deployments |
| `atelier-vercel inspect <url\|id>` | Deployment details |
| `atelier-vercel logs <url\|id>` | Deployment logs |
| `atelier-vercel env-ls` | List env var names |
| `atelier-vercel deploy [path]` | Preview deployment |
| `atelier-vercel deploy-prod [path]` | Production deployment (prompts the operator) |
| `atelier-vercel env-add <name> [target]` | Add an env var, value via stdin (prompts the operator) |
| `atelier-vercel redeploy <url\|id>` | Rebuild + redeploy (prompts the operator) |
| `atelier-vercel rollback [url\|id]` | Revert to a previous deployment (prompts the operator) |
| `atelier-vercel promote <url\|id>` | Promote a deployment to current (prompts the operator) |
| `atelier-vercel link [path]` | Link directory to a project |
| `atelier-vercel remove [--yes] <url\|id>` | Delete a deployment (in-CLI confirmation) |
| `atelier-vercel env-rm [--yes] <name> [target]` | Remove an env var (in-CLI confirmation) |
| `atelier-vercel project-rm [--yes] <name>` | Delete a project (in-CLI confirmation) |

## Deploy-and-verify flow

1. `deploy` for a preview; read the returned URL.
2. `inspect <url>` / `logs <url>` to confirm it built and is healthy.
3. When the operator approves, `deploy-prod` (or `promote <url>`).
4. If production breaks, `rollback` to the previous good deployment.

## Rules

- Resolve the target with `list` first; never guess a deployment URL/id.
- Only read ops and preview `deploy` are in the agent allowlist. Production
  writes (`deploy-prod`, `promote`, `rollback`, `redeploy`, `env-add`) are not:
  Claude Code prompts the operator before each one, because an autonomous agent
  must never ship to production without an explicit human go-ahead.
- `remove`, `env-rm`, and `project-rm` additionally require in-CLI confirmation:
  they die without `--yes` when there is no interactive terminal. Only pass
  `--yes` after the operator explicitly approved that exact operation.
- Only run a production deploy (`deploy-prod`/`promote`) when the operator asked
  to ship to production.
