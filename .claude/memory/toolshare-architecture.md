---
name: toolshare-architecture
description: How ToolShare is architected — BaaS pattern, client-only auth guards, data-access patterns per page, messaging flow, photo upload flow
metadata:
  type: project
---

**Pattern: pure BaaS.** No custom backend, no API routes, no server components doing data work. Every page is a `'use client'` component calling the shared `supabase` singleton (`src/lib/supabaseClient.ts`) directly from the browser. Authorization = Postgres RLS only (see [[toolshare-data-model-rls]]). CI builds with placeholder Supabase env vars, which works precisely because nothing runs server-side.

**Auth / session:** supabase-js default (session in localStorage). Guard pattern is copy-pasted per page: `useEffect` → `supabase.auth.getSession()`/`getUser()` → `router.push('/login')` if none, else `setCheckingAuth(false)`; renders "Loading..." meanwhile. No route-level protection (no `proxy.ts`), no shared auth context/provider — `AppHeader` refetches the resident row on every mount to show the avatar initial. Login checks `residents.name` empty → `/complete-profile`. `/complete-profile` only checks auth on submit, not on load.

**Signup flow:** `auth.signUp` → DB trigger `handle_new_user()` inserts empty `residents` row → client pushes to `/complete-profile` which `update`s the row. Assumes the Supabase project has **email confirmation disabled** (otherwise no session after signUp and the profile update fails with "You need to be logged in") — unverified assumption, flag if touching auth.

**Browse (`/`):** loads ALL items (`items` + joined `residents(name, apartment_no)`), filters client-side with `filterItems` (title substring + category), splits into "Looking for" (request) / "Available to borrow" (offer). No pagination, no URL state for filters.

**Messaging:** `conversations` keyed by (item, requester, owner). `/item/[id]` non-owner form: `maybeSingle()` lookup of existing conversation for (item, me) → insert if missing → insert message → redirect to `/messages/[id]`. Thread page loads convo + all messages ascending; send = insert + append to local state. **No realtime, no auto-scroll, no pagination.** `ConversationSidebar` loads all conversations then ALL messages for them (ordered desc) just to derive last-message preview — O(all messages).

**Photos:** Storage bucket `item-photos` (public), path `${userId}/${Date.now()}-${file.name}`; public URL stored in `items.photo_url`. Rendered with `<img>` (eslint-disabled) not `next/image` — no `images.remotePatterns` configured. Replaced/deleted photos are never removed from storage.

**Owner checks in UI** (edit page redirects non-owner, delete button only for owner) are UX-only; RLS is the real enforcement.

Related: [[toolshare-overview]], [[toolshare-code-smells]].
