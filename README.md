# vercel-integration

Optional, standalone [Claude Code](https://docs.anthropic.com/claude/docs/claude-code)
plugin that lets [atelier](https://github.com/AkaLab-Tech/atelier) agents deploy
and manage apps on [Vercel](https://vercel.com) via the **official Vercel CLI**:
deploy previews and production, inspect deployments, read logs, manage env vars,
link projects, and roll back.

It is independent of atelier; install it only if you deploy with Vercel.

## Install

```
/plugin marketplace add AkaLab-Tech/claude-plugins
/plugin install vercel-integration@akalab-tech
```

## Setup

Get a Vercel access token at <https://vercel.com/account/tokens>. Setup has a
**machine-wide** part (link the CLI onto `PATH`, grant the
[allowlist](#how-permissions-work)) and a **per-project** part (the token), so
one operator can deploy different projects under different scopes.

**With atelier** — pick it during `install.sh`, or run any time:

```
/atelier:setup-vercel
```

**Standalone** — `configure` does the machine-wide part, then (from inside a
project) wires that project's `.env`:

```sh
atelier-vercel configure
# or, inside a Claude Code session:
/vercel-integration:setup
```

`atelier-vercel` uses an installed `vercel` if present, otherwise
`pnpm dlx vercel@latest` (zero global install).

### Per-project auth

Each project carries its own credentials in its `.env` (kept gitignored by
atelier's `.env*` guardrail):

```sh
VERCEL_TOKEN=<token>
# optional, for a fixed scope:
VERCEL_ORG_ID=team_xxx
VERCEL_PROJECT_ID=prj_xxx
```

## How permissions work

This plugin never edits atelier's shipped `settings.template.json`.
`/vercel-integration:setup` (or `atelier-vercel enable-permissions`) merges the
allowlist into your **user-level** `settings.json`
(`$ATELIER_CONFIG_DIR/settings.json`) — idempotent, order-preserving, no
clobber. The allowlist covers **read ops and preview deploys only** (`whoami`,
`list`, `inspect`, `logs`, `env-ls`, `deploy`, `link`, `pull`).

Production-affecting writes (`deploy-prod`, `promote`, `rollback`, `redeploy`,
`env-add`) are deliberately omitted, so Claude Code prompts the operator before
each one — an autonomous agent must never ship to production without an
explicit human go-ahead. (`enable-permissions` also strips these entries if an
earlier release granted them.)

The destructive commands (`remove`, `env-rm`, `project-rm`) are omitted too
**and** confirmed by the wrapper itself: pass `--yes`, or answer the
interactive `y/N` prompt; without a terminal and without `--yes` they refuse to
run. Undo the allowlist with `atelier-vercel disable-permissions`.

## Usage

Run `atelier-vercel --help`, or let the `vercel` skill drive it.

## License

MIT — see [LICENSE](LICENSE).
