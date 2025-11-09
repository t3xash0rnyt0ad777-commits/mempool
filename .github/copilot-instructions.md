## Purpose

Short, focused guidance for AI coding agents working in this repository. Use this to find the big-picture architecture, developer workflows, project conventions, and concrete examples so you can be productive without guessing.

## Big picture (what this project is)

- Mempool is a full-stack mempool visualizer + explorer:
  - `backend/` — Node.js + TypeScript API service (compiles to `dist/`). Uses a small Rust helper under `rust/gbt` for GBT parsing.
  - `frontend/` — Angular app (dev server at :4200, production build outputs `dist/`). Uses `generate-config.js` to produce site-specific config before builds.
  - `docker/` and `production/` — docker-compose and production deployment examples. Containers only package backend & frontend; Bitcoin Core and Electrum must be provided separately.

Major integrations: Bitcoin Core RPC, Electrum Server (or Esplora), MariaDB (MySQL compatible), Redis, MaxMind, optional Lightning backends (lnd/clightning).

## Where to look (key files & folders)

- `backend/README.md` — backend setup, build, run, regtest examples, reindexing flags.
- `backend/mempool-config.sample.json` — canonical backend config; many docker env vars map to this file.
- `backend/package.json` — build/test/start scripts (see examples below).
- `frontend/README.md` and `frontend/package.json` — frontend dev/prod commands, config scripts, and Cypress e2e workflow.
- `docker/docker-compose.yml` — environment overrides for running full stack locally with containers.

## Node / Tooling requirements

- Node 20.x and npm 9.x (mentioned in both backend and frontend READMEs).
- Backend build also requires Rust (for `rust/gbt`) — `preinstall` uses `rust/gbt` build.

## Common developer workflows (concrete commands)

- Backend (local dev):
  - Install and build: `cd backend && npm install && npm run build`
  - Run: `MEMPOOL_CONFIG_FILE=/path/to/mempool-config.json npm run start`
  - Production run: `npm run start-production`
  - Tests: `npm run test` (Jest)
  - Lint: `npm run lint` / `npm run lint:fix`

- Frontend (local dev):
  - Quick dev (proxy to production backend): `cd frontend && npm install && npm run serve:local-prod`
  - Connect to local backend: `npm run serve` or `npm run start`
  - Build for production: `npm run build` (runs `generate-config.js` then Angular build and asset sync)
  - Cypress e2e: `npm run cypress:run` (or `cypress:open` for interactive)

- Docker: `cd docker && docker-compose up` (ensure `bitcoind` and Electrum are available or configured via env overrides)

## Important runtime flags / config patterns

- `MEMPOOL_BACKEND` (in `mempool-config.json` / docker env): `electrum`, `esplora`, or `none` — determines address lookup backend.
- `MEMPOOL_CONFIG_FILE` env var: pass a custom config file to the backend process.
- `--reindex` and `--update-pools` CLI flags on startup (see `backend/README.md`) for maintenance/reindexing operations.

## Project-specific conventions and patterns

- Backend TypeScript is compiled to `dist/` and run from Node. Expect code changes to require a rebuild (or use `nodemon` + `ts-node` in dev).
- The backend has a `preinstall` script that builds `rust/gbt` and copies it into the backend — don’t skip Rust when setting up.
- Frontend requires running `npm run generate-config` prior to serve/build; many scripts call it automatically.
- Docker compose environment variables map directly to JSON keys in `mempool-config.json` (see `docker/README.md`) — prefer env overrides in CI/containers.

## Integration and external dependencies to be aware of

- Bitcoin Core RPC (txindex=1, server=1) — required for blocks/mempool data.
- Electrum server (optional but required for address lookups) or Esplora REST API.
- MariaDB >= 10.5 (backend stores indexed data). The project expects MySQL-compatible usage.
- Redis, MaxMind geolocation DBs, and optional Lightning nodes (lnd or c-lightning) — these are toggled via config.

## Examples to include in PR/code suggestions

- When proposing a change that touches runtime behavior, reference `backend/mempool-config.sample.json` and the corresponding `docker/docker-compose.yml` env name.
- For a backend change that must be validated end-to-end, suggest the exact commands to run locally (build + start + sample regtest commands from `backend/README.md`).
- For frontend UI changes, include the `npm run serve:local-prod` workflow and how to run Cypress: `npm run cypress:run`.

## Quick checklist for PRs and CI hints

- If the change touches TypeScript: ensure `npm run build` (backend) or `npm run build` (frontend) completes.
- Run `npm run lint` in the affected package and `npm run test` when applicable.
- If adding network/dependency assumptions, update `backend/README.md` or `docker/README.md` with exact env vars and sample `mempool-config.json` fragments.

## If anything is unclear

Ask for the missing context (e.g., the intended deployment environment, whether DB seed data is available, or if the change requires running regtest). I can then produce exact shell steps and small validation checks.

---
If you'd like, I can merge this into an existing `.github/copilot-instructions.md` (none was found) or tweak tone/length. What would you like changed? 
