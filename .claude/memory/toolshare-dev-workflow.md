---
name: toolshare-dev-workflow
description: How to run, test, seed and deploy ToolShare locally and in CI; env vars and Supabase setup steps
metadata:
  type: project
---

- **Env:** `.env.local` (gitignored) from `.env.local.example`: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`. Seeder and RLS tests additionally need `SUPABASE_SECRET_KEY` (service role; loaded via `dotenv` from `.env.local`).
- **Scripts:** `dev`, `build`, `start`, `lint` (eslint flat config, next core-web-vitals + typescript), `test` = `vitest run src` (unit only, runs in CI), `test:rls` = `vitest run tests/rls.test.ts` (integration, live DB, NOT in CI). `vitest.config.ts` includes both globs; the script filter decides which runs.
- **Supabase setup order:** run `schema.sql` → create public bucket `item-photos` in dashboard (not scriptable) → run `storage_policies.sql` → optional `node seed.mjs` (creates 4 confirmed users pw `password123` + 8 items; not idempotent — re-run logs "Failed to create user" and skips). Assumes email confirmation disabled in Auth settings.
- **CI (`.github/workflows/ci.yml`):** on push/PR to `main`: Node 20, `npm ci`, lint, `npm test`, `npm run build` with placeholder Supabase env. Deploy = Vercel Git integration only: every push to `main` → `vercel[bot]` Production deployment (confirmed via GitHub deployments API, 2026-08-15); no `vercel.json`/`.vercel`, env vars live only in Vercel project settings (owner `emilmanninen`), CI does **not** gate deploys, DB schema is applied by hand (not part of any pipeline).
- **Package manager:** repo says npm; local install is pnpm — see drift note in [[toolshare-code-smells]].
- **Next 16 rule (AGENTS.md):** consult `node_modules/next/dist/docs/` before writing framework code; conventions differ from older Next (e.g. `proxy.ts` replaces `middleware.ts`).
- Commit messages follow Conventional Commits with scope (`type(scope): description`) and changes go through feature branches + PRs — see the Git workflow section in `AGENTS.md`.

Related: [[toolshare-overview]].
