# AGENTS.md

For architecture, conventions, and the authoritative command reference, read `CLAUDE.md` (root), `docs/ARCHITECTURE.md`, and `open-sse/AGENTS.md`. This file only adds Cloud-agent-specific setup/run caveats.

## Cursor Cloud specific instructions

### Services / product
There is one primary service: the **Next.js dashboard + gateway** (`9router-app`, root `package.json`). It serves the dashboard at `/dashboard` and the OpenAI-compatible gateway at `/v1`. The `cli/` package is only a launcher/tray wrapper and is not needed to develop or test the gateway. No external datastore is required — persistence is SQLite via `src/lib/db/` (in this environment the native `better-sqlite3` driver builds and is used; `sql.js` is the pure-JS fallback).

### Running the dev server
- Dependencies are installed by the startup update script (root deps + `tests/` deps). Copy env once with `cp .env.example .env` before first run; dev login password defaults to `123456` (`INITIAL_PASSWORD`).
- Start with `npm run dev` (Next.js Turbopack). It listens on **port 20127** — the `--port 20127` flag in the `dev` script overrides any `PORT` env var, so the dashboard is at `http://localhost:20127/dashboard` regardless of `PORT`.
- The SQLite DB is written under `DATA_DIR` (set it in `.env`, e.g. `/workspace/.9router-data`); otherwise it falls back to `~/.9router`.

### Gotchas
- `npm run dev` **regenerates `CLAUDE.md`** on startup (Next.js `agentRules` in `next.config.mjs`), which leaves an unstaged git diff. Revert it (`git checkout -- CLAUDE.md`) before committing; don't commit the regenerated copy.
- Lint (`npx eslint .`) reports many pre-existing errors/warnings on a clean checkout. These are baseline, not caused by your changes — judge only newly introduced issues.
- Tests (run `npx vitest run` from `tests/`, per `CLAUDE.md`) are **not** all-green on a plain checkout (mix of code-drift, snapshot, network, and cwd-relative AUDIT failures). The harness itself works (the majority of tests pass).
- The committed regression gate `tests/__baseline__/verify-no-regression.mjs` splits test filenames on `/app/` (the path where its baseline snapshot was recorded). In Cloud the repo lives at `/workspace`, so the gate flags every failure as a "regression" — this is a path artifact, not real regressions. To compare against the baseline here, normalize paths to the `tests/...` suffix before diffing against `known-fails.txt`. The baseline snapshot also appears stale relative to current `master`.
