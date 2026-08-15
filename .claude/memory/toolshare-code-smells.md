---
name: toolshare-code-smells
description: Known code smells, duplication, tech debt and risk spots in ToolShare to consider before extending it
metadata:
  type: project
---

Snapshot as of 2026-08-15 (HEAD a1ecdf8). Update as items are fixed.

**Duplication**
- Auth-guard `useEffect` + "Loading..." block copy-pasted in ~8 pages; no `useUser`/AuthProvider. Redirects happen client-side only.
- `list-item/page.tsx` and `item/[id]/edit/page.tsx` are near-identical forms (same fields, same `selectClass`, same upload code) — candidate for a shared `ItemForm`.
- Two `Item` types: `components/ItemCard.tsx` exports one, `item/[id]/page.tsx` redefines a wider one; `ConversationRow`/`ConversationInfo` duplicated between sidebar and thread page.
- Back-button and container markup repeated per page; native `<select>` styled by string const rather than a ui component.

**Correctness / robustness**
- Check-then-insert conversation race (no unique index) — see [[toolshare-data-model-rls]].
- Photo replace/delete leaves orphans in storage; upload path uses raw `file.name` (spaces/unicode not sanitised); no size/type validation beyond `accept`.
- Edit page always writes `photo_url` even when unchanged; request-type edits null it out (intended).
- Errors from `.single()` swallowed in several loaders (`if (!error && data)`), leaving "Item not found" for both 404 and RLS/network failures. No `not-found.tsx`/`error.tsx`.
- `formatRelativeTime` rounds (`Math.round`) at every step, so 89 min → "1h", 36 h → "2d"; approximate by design, and untested.
- Sidebar fetches every message of every conversation for previews.
- Thread doesn't scroll to newest message; no realtime; sending clears input only on success and shows no error on failure.

**Type safety**
- Pervasive `data as unknown as X` casts because no `supabase gen types`; joined relations typed by hand.
- `process.env.NEXT_PUBLIC_*!` non-null assertions — missing env fails at runtime, not build.

**Tooling / repo hygiene**
- Package manager: `package-lock.json` is tracked and CI runs `npm ci`, so npm is canonical; no `.npmrc`/`packageManager` field pins it, so a pnpm/yarn install works locally but produces a lockfile the repo doesn't track. Consider a `packageManager` field or `.gitignore` entries for other lockfiles.
- Only one unit test (`filterItems`); RLS test needs live Supabase + seed data (Jane/Mark/Tom) + `SUPABASE_SECRET_KEY`.
- No `proxy.ts`, no rate limiting, open signup (README lists as deliberate v1 limitation).
- README credentials table (seed accounts) is public — fine for demo, note before any real deployment.

Related: [[toolshare-architecture]], [[toolshare-overview]].
