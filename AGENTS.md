# umami-api

[Umami](https://umami.is) analytics on Zerops — Next.js app with PostgreSQL (Prisma), built and migrated at deploy time against the sibling `db` service.

## Zerops service facts

- HTTP port: `3000`
- Siblings: `db` (PostgreSQL) — env: `DATABASE_URL` (`${db_connectionString}/${db_dbName}`)
- Runtime base: `nodejs@22`

## Zerops dev

`setup: dev` idles on `zsc noop --silent`; the agent starts the dev server.

- Dev command: `pnpm run dev -- --hostname 0.0.0.0`
- In-container rebuild without deploy: `pnpm run build`

**All platform operations (start/stop/status/logs of the dev server, deploy, env / scaling / storage / domains) go through the Zerops development workflow via `zcp` MCP tools. Don't shell out to `zcli`.**

## Notes

- **pnpm 9** is pinned via `packageManager` in `package.json` — Umami's lockfile is v9; pnpm 11's strict build-gate fails CI installs.
- Install with `--node-linker=hoisted` (see `zerops.yaml`) so flat `node_modules` survives the build→run container bridge.
- **Do not set `NODE_ENV=production` during build** — pnpm would skip devDependencies (rollup, next, tsup) required for the build.
- `DATABASE_URL` points at the real `db` service during build; `pnpm run build` runs Prisma migrations at build time (schema + default admin/umami user). No dummy URL or migrate-at-init step.
- Set `CYPRESS_INSTALL_BINARY=0` in build env to skip the ~280MB Cypress binary download.
- Health/readiness: `GET /api/heartbeat` on port 3000.
