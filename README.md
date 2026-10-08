# Smart Stair Guardian

An AI-assisted staircase safety dashboard built with Next.js, Supabase, and Gemini. It checks visible conditions in camera frames and uploaded media, then keeps saved reports in the signed-in user's incident history.

## What it can do

- Run an optional live camera check that reviews a compressed frame about every six seconds.
- Analyze a JPG, PNG, or WebP image.
- Review up to five sampled frames from a video in one Gemini request.
- Search and filter saved reports by risk level.
- Show risk summaries and suggested actions for a human to review.

Live checks sample images in the browser. They are not continuous video recording, a hardware sensor, or an emergency alert service. Treat AI assessments as visual guidance and confirm serious hazards in person.

## Start the app

1. Install the project dependencies with `npm install`.
2. Add these values to `.env.local`:

   ```env
   NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
   NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
   GEMINI_API_KEY=your_gemini_api_key
   ```

   The Gemini key is used only by the server route. Use the Supabase anon key here; do not put a Supabase service-role key in a browser-visible variable.

3. Create the Supabase `incidents` table described below, and enable email or Google sign-in in the Supabase project.
4. Start the development server with `npm run dev`, then open [http://localhost:3000](http://localhost:3000).

## Supabase incident table

The dashboard reads and writes these columns. If the table does not already exist, this SQL creates the expected shape and limits each signed-in user to their own rows:

```sql
create table public.incidents (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references auth.users(id) on delete cascade,
  risk_level text not null check (risk_level in ('LOW', 'MEDIUM', 'HIGH')),
  risk_score integer not null check (risk_score between 0 and 100),
  detected_hazards jsonb not null default '[]'::jsonb,
  recommendation text not null,
  confidence integer not null check (confidence between 0 and 100),
  people_detected boolean not null default false,
  immediate_action text not null,
  source text not null,
  created_at timestamptz not null default now()
);

create index incidents_user_created_at_idx
  on public.incidents (user_id, created_at desc);

alter table public.incidents enable row level security;

create policy "Users can read their own incidents"
  on public.incidents for select to authenticated
  using (auth.uid() = user_id);

create policy "Users can add their own incidents"
  on public.incidents for insert to authenticated
  with check (auth.uid() = user_id);
```

## Project commands

- `npm run dev` — start the local development server.
- `npm run lint` — check project code style.
- `npm run build` — create a production build.
