---
name: toolshare-overview
description: What the ToolShare (neighborhoodapp) project is, its stack, deployment, repo layout, and where things live — entry point of the project knowledge base
metadata:
  type: project
---

**ToolShare** (repo dir `neighborhoodapp`, package name `toolshare`) — a neighborhood tool-sharing marketplace: residents list tools to lend ("offer") or things they want ("request"), browse/filter, and message each other. Portfolio/demo-scale app, ~22 commits (2026-07-10 → 2026-07-22). Live demo: https://neighborhoodapp-xi.vercel.app/ . GitHub remote: `emilmanninen/neighborhoodapp`.

**Stack:** Next.js 16.2.10 (App Router, TS, all pages `'use client'`), React 19.2, Tailwind v4 (CSS-first `@theme inline` in `src/app/globals.css`), shadcn "base-vega" style on **@base-ui/react** (not Radix), Phosphor icons (`@phosphor-icons/react/dist/ssr`), Supabase JS v2 (auth + Postgres + Storage, **no `@supabase/ssr`**), Vitest 4, ESLint 9 flat config, Vercel hosting, GitHub Actions CI (lint → `vitest run src` → build with placeholder env).

**Layout (47 tracked files):**
- `src/app/*/page.tsx` — routes: `/` browse, `/login`, `/signup`, `/complete-profile`, `/profile`, `/list-item`, `/item/[id]`, `/item/[id]/edit`, `/my-listings`, `/messages`, `/messages/[id]`. `layout.tsx` = Inter font + globals.css only.
- `src/components/` — `AppHeader` (nav+search+avatar initial), `ItemCard` (+ exported `Item` type), `CategoryIcon`, `ConversationSidebar`; `ui/` = shadcn button/badge/input/label/textarea.
- `src/lib/` — `supabaseClient.ts` (singleton browser client from `NEXT_PUBLIC_*` env), `categories.ts` (CATEGORIES/CONDITIONS consts + icon map), `filterItems.ts` (+ only unit test), `formatRelativeTime.ts`, `utils.ts` (`cn`).
- Root: `schema.sql` (tables, trigger, RLS), `storage_policies.sql`, `seed.mjs` (admin-key seeder), `tests/rls.test.ts` (integration), `.env.local.example`, `.github/workflows/ci.yml`. `next.config.ts` is empty; no `proxy.ts`/middleware.
- `AGENTS.md`/`CLAUDE.md`: Next 16 has breaking changes vs training data — read `node_modules/next/dist/docs/` first (e.g. `middleware.ts` is deprecated → `proxy.ts`).

Related: [[toolshare-architecture]], [[toolshare-data-model-rls]], [[toolshare-code-smells]], [[toolshare-dev-workflow]], [[toolshare-design-system]].
