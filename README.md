# Meeting Poll 2

A simple, no-nonsense meeting scheduler. No accounts, no tracking — participants enter only their name.

## How it works

- Organiser creates a poll with any number of time slots
- A shareable link is generated: `https://yourusername.github.io/meeting-poll?poll=abc123`
- Participants open the link, vote Yes / If necessary / No for each slot, enter their name
- Anyone can export results as a `.csv` spreadsheet at any time

## Setup

### 1. Supabase — run once in SQL Editor

```sql
create table polls (
  id text primary key,
  title text not null,
  description text,
  slots jsonb not null,
  created_at timestamptz default now()
);

create table votes (
  id uuid primary key default gen_random_uuid(),
  poll_id text references polls(id),
  name text not null,
  choices jsonb not null,
  created_at timestamptz default now()
);

alter table polls enable row level security;
alter table votes enable row level security;

create policy "public read polls" on polls for select using (true);
create policy "public insert polls" on polls for insert with check (true);
create policy "public read votes" on votes for select using (true);
create policy "public insert votes" on votes for insert with check (true);
```

### 2. GitHub Pages

- Push `index.html` to a public repo
- Go to Settings → Pages → Deploy from branch → main / root
- Your site will be live at `https://yourusername.github.io/reponame`

## Privacy

- No email addresses or personal data collected beyond a display name
- Names are stored only to identify responses in the results
- No cookies, no tracking, no analytics
