---
name: toolshare-data-model-rls
description: ToolShare Postgres schema (residents/items/conversations/messages), trigger, RLS and storage policies, and the gaps in them
metadata:
  type: project
---

Defined in `schema.sql` (run manually in Supabase SQL editor — no migrations tooling) and `storage_policies.sql` (run after manually creating public bucket `item-photos`).

**Tables:**
- `residents(id uuid PK → auth.users on delete cascade, name, apartment_no, phone, created_at)` — 1:1 with auth users, auto-created empty by trigger `on_auth_user_created` → `handle_new_user()` (security definer).
- `items(id, owner_id → residents cascade, title, description, category text, condition, photo_url, listing_type check in ('offer','request') default 'offer', created_at)`. Category is free text by design (app-validated via `CATEGORIES` const).
- `conversations(id, item_id → items **on delete cascade**, requester_id, owner_id → residents cascade, created_at)`. `owner_id` denormalised from item.
- `messages(id, conversation_id → conversations cascade, sender_id → residents cascade, body, created_at)`.

**RLS (all four tables enabled):**
- residents: SELECT any authenticated; UPDATE own. (No INSERT/DELETE policy — trigger bypasses.)
- items: SELECT any authenticated; INSERT/UPDATE/DELETE where `auth.uid() = owner_id`.
- conversations: SELECT participants; INSERT where `auth.uid() = requester_id`. **No UPDATE/DELETE policy.**
- messages: SELECT/INSERT participants of parent conversation (and sender = self). **No UPDATE/DELETE.**
- storage.objects `item-photos`: SELECT anyone; INSERT/DELETE only into own `{uid}/` folder.

**Gaps / caveats:**
- README claims conversation history is preserved if the listing is deleted, but `conversations.item_id` is `on delete cascade` → deleting an item deletes its conversations and messages. README and schema contradict; schema wins.
- No unique constraint on `conversations(item_id, requester_id)` → the check-then-insert in `/item/[id]` can create duplicates under a race.
- Conversation INSERT policy doesn't verify `owner_id` matches `items.owner_id` → a requester can open a thread with any resident as "owner".
- `residents.phone` is readable by every authenticated user (PII exposure; not shown in UI but selectable).
- Public bucket → photo URLs readable without auth.
- Frontend uses `residents!conversations_requester_id_fkey` / `_owner_id_fkey` hint names in PostgREST selects — depends on Postgres auto-generated FK constraint names; renaming FKs breaks the messages pages.
- No generated Supabase types; joins are cast `as unknown as T`.

Related: [[toolshare-architecture]], [[toolshare-dev-workflow]] (RLS tests).
