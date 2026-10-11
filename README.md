# FlashSpark AI — Hackathon MVP

Turn class notes into colorful flashcards, mind maps, and quick quizzes.

## What works
- Paste notes or upload `.txt` / `.md` files (PDF upload UI is included; PDF text extraction can be added with `pdf-parse`).
- Generate question/answer cards with a deterministic fallback, or an LLM when configured.
- Colorful 3D flip cards, shuffle, previous/next, and "I know it" tracking.
- Interactive concept mind map with SVG nodes and connecting lines.
- Quick quiz mode.
- Saves decks in browser local storage for an easy no-account demo.
- Express API, with optional OpenAI-compatible API integration.
- Optional Supabase and MongoDB integration points are documented below.

## Quick start
Requirements: Node.js 20+

### 1. Install
```bash
npm install
```

### 2. Start both apps
```bash
npm run dev
```
Open http://localhost:3000

The Next.js UI proxies `/api/*` requests to Express at `http://localhost:4000`.

## AI setup (optional)
Copy `.env.example` to `.env` and set:
- `LLM_API_KEY`
- `LLM_BASE_URL` (OpenAI-compatible endpoint, defaults to OpenAI)
- `LLM_MODEL` (defaults to `gpt-4o-mini`)

Without an API key, FlashSpark uses its built-in note-to-card generator so the demo still works.

## Supabase setup (optional)
Create a Supabase project and add:
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`

The app can be extended to use Supabase Auth and persist decks. Browser-local storage is the default so judges can try it immediately without credentials. Suggested table:
```sql
create table public.decks (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users(id) on delete cascade,
  title text not null,
  cards jsonb not null default '[]',
  mindmap jsonb not null default '{}',
  created_at timestamptz not null default now()
);
alter table public.decks enable row level security;
create policy "Users manage their own decks" on public.decks
  for all using (auth.uid() = user_id) with check (auth.uid() = user_id);
```

## MongoDB
`MONGODB_URI` is supported as an optional persistence/analytics connection point. The current MVP intentionally uses local storage as the no-setup demo path. For a judged production deployment, pick one primary deck database (Supabase or MongoDB) rather than duplicating user data.

## Project structure
- `app/page.jsx` — complete student dashboard
- `app/globals.css` — responsive, colorful design system
- `app/layout.jsx` — app shell
- `app/api/[...path]/route.js` — Next.js API proxy
- `server/index.js` — Express API for flashcards and mind maps
- `lib/demoData.js` — sample notes
