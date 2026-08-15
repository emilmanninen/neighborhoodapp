---
name: toolshare-design-system
description: ToolShare UI/design-system conventions — brand tokens, Tailwind v4 theme mapping, shadcn base-ui components, layout patterns
metadata:
  type: project
---

- **Tokens** live in `src/app/globals.css` `:root`: canvas `#fffefb`, canvas-soft `#f8f4f0`, ink `#201515`, ink-soft, body `#605d52`, body-mid `#939084`, mute `#c5c0b1`, border-soft `#e4ddd2`, danger `#c0392b`, brand-primary **orange `#ff4f00`** (+hover/active). Mapped to shadcn semantic vars (`--primary`, `--border` = ink, `--ring` = orange…) and exposed to Tailwind via `@theme inline` as `text-ink`, `bg-canvas-soft`, `text-body-mid`, `border-border-soft`, etc. Radii: sm 6px, md/lg 12px, xl 16px. Font: Inter via `next/font` (`--font-inter` → `font-sans`). Light theme only (no dark tokens).
- **Components:** shadcn `components.json` style `base-vega`, iconLibrary phosphor, primitives from `@base-ui/react` — Button uses `render={<Link/>}` + `nativeButton={false}` for link-styled buttons (not `asChild`). Variants: default (orange), outline, secondary, ghost, destructive, link.
- **Icons:** import from `@phosphor-icons/react/dist/ssr` with `Icon` suffix names (`PlusIcon`); category → icon map in `lib/categories.ts` (`categoryIcon()`, fallback `PackageIcon`).
- **Layout conventions:** `AppHeader` sticky top (nav pill group, optional search when `onSearchChange` passed, orange "List item" CTA, avatar initial → `/profile`); page container `mx-auto max-w-7xl px-8 py-10` (browse) or `max-w-lg`/`max-w-md` (forms); item grid `grid-cols-2 sm:grid-cols-3 md:grid-cols-4 gap-5`; request cards use `border-dashed border-mute` + "Looking for" badge. Global rule: every clickable gets `cursor-pointer` (globals.css adds hover brightness to `button.cursor-pointer, a.cursor-pointer`).
- Messages pages use fixed `h-screen` / `h-[calc(100vh-70px)]` with a 300px sidebar.

Related: [[toolshare-overview]].
