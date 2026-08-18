# Holiday Packing

A private web app for two parents to pack for family trips together, in real time, from two phones.

Built because paper lists and shared Google Tasks couldn't hold what mattered: which room something comes from, which bag it goes in, who's packing it, what's prescription and needs reordering after, and what worked or didn't on the last trip so the next one is better informed.

**Live at:** https://sophiecee.github.io/packing-app/

## What it does

Every item belongs to a room (where it lives at home) and a bag (where it travels). Three views show the same items differently:

- **Gather**, grouped by room — walk the house once, collecting.
- **Pack**, grouped by bag — load each bag correctly.
- **In bags**, packed items only, grouped by bag — "which bag is X actually in right now."

Ticking an item off in any view updates it everywhere. Progress bars show percent packed overall and per group.

## Who can do what

No login. Anyone with the Project URL and publishable key can read and write everything. That's a deliberate simplification for a two-person household tool — see **Secrets** below for what that does and doesn't expose.

## The three tags on an item

| Tag | Icon | What it means |
|---|---|---|
| Flag | 🚩 amber | Something's wrong with this one specifically — can't find it, only one left. Carries a free-text note. Shows a stripe on the row so it can't be missed. |
| Prescription | 💊 teal | This is medication. Appears in **After → Reorder** so nothing runs out the week after you're home. |
| Last minute | ❄️ blue | Can't be packed until the morning — fridge, freezer, in daily use, still charging. |

All three are set from the Add-item form or by opening any item afterwards.

## The other tabs

- **Jobs** — three checklists per trip: *Sort before we go* (bookings, insurance, medical letters, pet care), *Buy before we go*, *As we leave the house* (the last ten minutes, in order — cat fed, bins out, phones and keys). Each has a "Suggest jobs for this trip" button. Ticked jobs stay ticked; nothing here affects the packing list itself.
- **Info** — contacts and bookings for the trip: where you're staying, transport, nearest A&E, pet sitter. Phone numbers are tappable.
- **After** — two halves. **Reorder** lists everything tagged Prescription on this trip so you can tick each off as requested. **Trip notes** has five prompts (bags actually used, what came home unused, what you wished you'd had, what worked, anything else) — autosaves as you type, no button to remember. This is what makes the third trip better than the first: the AI reads these notes when drafting the next one.

## Rooms, bags and packers

Cog → **Rooms, bags and packers**. Add, rename, delete, drag to reorder (hold the grip handle on the left of each row; arrows are the fallback if a drag misfires on a fussy screen). Renaming sweeps through every item on every trip at once — nothing needs re-tagging by hand. Deleting is blocked while items still use that name, and tells you how many.

New rooms and bags can also be added inline, mid-task, from any "which room / which bag" dropdown — tap **+ Add new…** at the bottom of the list.

Packers work the same way. Assign a packer to a single item, or tap the person icon on any group heading to set the whole room or bag at once. The packer filter chips at the top of the List tab show only one person's items, or everyone's.

## Getting a new trip started

Three ways in: the sparkles icon in the header, the sparkles button inside **Your trips** (clock icon), or **Repeat as a new trip** on any past trip's card, which clones its whole list, unticked, ready to edit.

New-trip questions beyond the basics — accommodation type, international or not, nights in temporary accommodation en route — exist because they change what should be suggested, not just what's asked. See **How the AI drafts a list** below.

## Pasting in an old list

Add tab → **Paste in a whole list instead**. Works with Google Tasks exports and similar: headings (with or without a colon, including `**bold:**` and `*italic:*` markdown styles, and checkbox-prefixed ones like `[ ] BUGGIES:`) become groups, quantities written as "x3", "X2" or "7 bottles" are picked up automatically including when glued onto the item name, and anything in brackets becomes a note rather than part of the name.

**The first line is checked for a trip title** before anything else — something like "Efteling christmas, 4 days, driving" is read for name, month (including "christmas" → December, "easter" → April, and similar), day count, transport, and whether it looks international, rather than being imported as an item. A teal box shows what it understood and lets you create a proper new trip from it, or turn that off to import into whichever trip is already active instead.

After that, **map each group to a current room and bag** — not each item individually. The app makes its own best guess first: a direct name match, or "Upstairs"/"Downstairs" mapped against which of your rooms are upstairs (bedrooms, our bathroom, landing, utility cupboard) versus everything else. Where it genuinely can't tell — "Meds", "Purple bag", anything that doesn't match a known room or bag — it says so honestly with an amber "needs a choice" flag rather than silently defaulting to whatever's first in the list. Those groups float to the top so you see them before the ones it got right. The import button stays disabled until every included group has both a room and a bag chosen, so nothing can land in the wrong place by accident.

Old bag names like "blue bag" come in fine under whatever heading the old list used; rename them properly afterwards from Rooms and bags, which will sweep every item that used that name.

Strip real names out before pasting. Use S, F, Mum, Dad.

## How the AI drafts a list

The "Draft the list for me" button sends Claude your past trips — including their trip notes — plus this trip's month, length, transport, accommodation, international status, and nights en route. It's told explicitly how to weigh that history, because different categories of item should be informed by different things:

- **Clothing and weather gear** — weighted by season above all. A trip in a similar month beats a more recent one in the wrong season.
- **Logistics and bulk** — weighted by transport and accommodation. A car trip two years ago is more useful than a flight last month; self-catering needs food-prep kit a hotel doesn't.
- **Medication and feeding equipment** — weighted by recency above all, because needs change. The most recent trip wins even if the season and transport don't match.
- **Consumables scale with days**; equipment doesn't.
- Anything a past trip's notes said came home unused isn't suggested again unless season or transport makes it newly relevant. Anything the notes said was wished for, is.

The grab bag, cabin bag and nappy bag are always suggested regardless of history. International trips pull in passports, birth certificates, insurance and adaptors.

This is a prompt, not a formula — it's guidance to a language model, not a hard rule, so it's worth glancing at what it suggests rather than trusting it blind, especially the first few times.

If the AI reply comes back garbled, it's usually because it got cut off before finishing — the app salvages whatever complete items arrived rather than discarding the whole response, so a partial list is more likely than a hard failure.

## Offline

Every write saves to the phone first, then tries Supabase. If that fails — no signal, a tunnel — it queues instead of being dropped. A "waiting to sync" badge in the header shows how many changes haven't landed yet; it clears itself once they do, retrying automatically every few seconds and the moment the phone notices it's back online.

Don't force-close the tab while that badge is showing if you can avoid it — leave it open, even in the background, so the retry loop gets to run. It's fine if you do close it anyway: the queue is saved to the phone itself, not memory, so reopening the app later picks up exactly where it left off.

A completely cold start still needs a connection once — the app loads React, Tailwind and Babel from a CDN, so if none of that is cached and there's no signal at all, the page won't render. Once it's loaded once, it works offline from then on.

## Nothing is unrecoverable

- Wrong room, bag, quantity, note, packer, or any tag — tap the item, edit, save.
- Item added by mistake — tap it, **Remove from list**.
- Ticked the wrong thing — just tap it again.
- Whole trip needs redoing after a bad AI draft — the trip stays; just add or remove items, or start a fresh trip and abandon it.
- Renamed a room or bag by mistake — rename it back; the sweep runs both ways.

There's no undo for a genuine delete of an item or a trip. There's also no per-record history — items don't remember old values the way `reviews` does in the observation log app. If that granularity ever matters here, it isn't built.

## Backups

The free Supabase tier has no automated backups.

- Photos of packed bags and the trip notes are the only things without an easy re-creation path if lost — everything else can be rebuilt from memory or a past list.
- Free projects pause after 7 days with no activity. Resumable from the Supabase dashboard, not lost — but don't leave it untouched for months across a quiet season.
- Full data dump: Supabase dashboard → Table Editor → each table → Export as CSV.

## Secrets

- The Supabase URL and key sitting at the top of `index.html` are meant to be public — that's what a **publishable** key (`sb_publishable_...`) is for.
- If a key starting `sb_secret_` or a `service_role` key ever ends up in this file, it bypasses every permission and must be rotated immediately in the Supabase dashboard.
- There is no login and no Row Level Security here — anyone with the URL and key can read and write everything. That's fine for what this is: a two-person household tool with no clinical or identifying data that isn't also on a fridge door. It would not be fine for the observation log app, which is why that one has real access control and this one doesn't.
- An Anthropic API key can be added in Settings for the AI features. It's stored on that device only, in the browser's local storage, and is sent only to Anthropic's API — never anywhere else, never committed to the repo.

## Working on this from a phone

- The whole app is one file, `index.html`. No build step. To update it: delete the old file in GitHub and upload the new one, rather than trying to edit lines directly — deleting and pasting inside the GitHub mobile editor is unreliable and can truncate long pastes.
- The file must be named `index.html` before uploading, not renamed afterwards — renaming in the mobile interface is fiddly.
- The GitHub mobile file browser's **"..."** menu sometimes needs a moment or a different tap point to reveal Delete — it isn't gone, just awkward to find, particularly if you're mid-edit rather than viewing the file normally.
- The Supabase SQL editor on mobile can add a stray closing bracket after a paste, giving `ERROR: 42601: syntax error at or near ")"`. If that happens, delete the last line — it's the extra bracket, nothing else is wrong.
- Run SQL blocks one at a time.
- If the app breaks, a red panel appears on screen with the exact failure rather than a blank page. Screenshot or copy that — it's the thing worth sending to whoever's helping debug.
- After any update, load the site with a changed value on the end, like `?v=7`, once — phones cache aggressively and will otherwise keep showing the old version.

## Stack

Single-file HTML. React 18 and Tailwind via CDN, Babel standalone compiling in the browser, lucide for icons, Supabase (REST API, polled every 4 seconds — not websockets) for the database, GitHub Pages for hosting. No build step, no package manager, no dependencies to install.

The repo is public, which GitHub Pages requires on the free tier. That's fine: the repo contains code, not personal data — trip names, medical notes, contacts and photos all live in Supabase, not here. Keep it that way.

## Setup SQL

Run once, in the Supabase SQL editor, one block at a time.

```sql
create table trips (
  id uuid primary key, name text, month text, days int,
  transport text, notes text, review text,
  accommodation text, international boolean default false,
  nights_en_route int default 0,
  created_at timestamptz default now()
);
```

```sql
create table items (
  id uuid primary key,
  trip_id uuid references trips(id) on delete cascade,
  name text not null, room_source text, assigned_bag text,
  is_packed boolean default false, quantity int default 1,
  notes_or_special_instructions text, packer text,
  is_flagged boolean default false, flag_note text,
  is_prescription boolean default false, last_minute boolean default false,
  reordered boolean default false
);
```

```sql
create table tasks (
  id uuid primary key,
  trip_id uuid references trips(id) on delete cascade,
  kind text, title text, notes text,
  done boolean default false, packer text, position int default 0
);
```

```sql
create table contacts (
  id uuid primary key,
  trip_id uuid references trips(id) on delete cascade,
  label text, category text, phone text, reference text, notes text
);
```

```sql
create table photos (
  id uuid primary key,
  trip_id uuid references trips(id) on delete cascade,
  bag text, caption text, data_url text
);
```

```sql
alter table trips enable row level security;
alter table items enable row level security;
alter table tasks enable row level security;
alter table contacts enable row level security;
alter table photos enable row level security;
create policy "open" on trips for all using (true) with check (true);
create policy "open" on items for all using (true) with check (true);
create policy "open" on tasks for all using (true) with check (true);
create policy "open" on contacts for all using (true) with check (true);
create policy "open" on photos for all using (true) with check (true);
```

```sql
grant select, insert, update, delete on public.trips to anon;
grant select, insert, update, delete on public.items to anon;
grant select, insert, update, delete on public.tasks to anon;
grant select, insert, update, delete on public.contacts to anon;
grant select, insert, update, delete on public.photos to anon;
```

If you set this project up before the trip-similarity fields existed, run just this against an existing database rather than the whole block above:

```sql
alter table trips add column if not exists accommodation text;
alter table trips add column if not exists international boolean default false;
alter table trips add column if not exists nights_en_route int default 0;
alter table items add column if not exists reordered boolean default false;
```

## Possible future changes

- **A service worker**, so a cold start works with no signal at all — the biggest real gap left.
- **Per-item history**, if flags or prescription reordering ever need "when did this change" rather than just current state.
- **Smarter group-heading detection** in the importer, if a very differently-formatted old list doesn't parse well — the heading matcher currently looks for room/bag words and colons, which won't catch everything.
- **Real access control**, only if this app ever needs to be shown to someone outside the household — nursery, a professional, anyone. Nothing here currently stops that person from editing anything.
