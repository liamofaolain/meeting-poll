# Meeting Poll

A simple, no-nonsense meeting scheduler built by William Whelan-Curtin. No accounts required for participants — they enter only their name.

## What it does

- Organiser creates a poll with any number of time slots (variable duration)
- A shareable link is generated for participants
- Participants see all slots converted to their own timezone
- Each participant votes Yes / If necessary / No for each slot
- Results show totals and highlight the best slot
- Anyone can export results as a `.csv` spreadsheet at any time
- The create page is password-protected — participants cannot create polls

---

## Setup

### Step 1 — Create a Supabase account

1. Go to [supabase.com](https://supabase.com) and sign up
2. Click **New project**
3. Give it a name (e.g. `meeting-poll`)
4. Set a database password and save it somewhere safe
5. Choose a region close to you (e.g. **West Europe**)
6. Click **Create new project** — takes about 30 seconds

---

### Step 2 — Create the database tables

1. In your Supabase dashboard, click **SQL Editor** in the left sidebar
2. Paste and run the following SQL:

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
create policy "public delete votes" on votes for delete using (true);
```

3. Run this second statement to prevent duplicate votes:

```sql
ALTER TABLE votes ADD CONSTRAINT unique_vote UNIQUE (poll_id, name);
```

This setup is done **once only**. All future polls use the same tables.

---

### Step 3 — Get your Supabase credentials

1. In your Supabase dashboard go to **Project Settings → Data API**
2. Copy your **Project URL** (e.g. `https://xxxxxxxxxxxx.supabase.co`)
3. Copy your **anon public key** (long string starting with `eyJ…`)

The anon key is safe to use in public frontend code. It only allows the operations defined by the RLS policies above. Never use the `service_role` key in frontend code.

---

### Step 4 — Create a GitHub repository

1. Go to [github.com](https://github.com) and sign in
2. Click **New repository**
3. Name it (e.g. `meeting-poll`)
4. Set it to **Private** (recommended, since `index.html` contains your create password)
5. Check **Add a README**
6. Click **Create repository**

---

### Step 5 — Enable GitHub Pages

1. In your repo go to **Settings → Pages**
2. Under Source, select **Deploy from a branch**
3. Branch: `main`, folder: `/ (root)`
4. Click **Save**
5. Your site will be live at `https://yourusername.github.io/meeting-poll`

Note: GitHub Pages serves the site publicly even if the repo is private. The private repo just means the source code (including the password) is not publicly visible.

---

### Step 6 — Upload the files

1. In your repo click **Add file → Upload files**
2. Upload `index.html` and `README.md`
3. Click **Commit changes**
4. Wait about 1 minute for GitHub Pages to deploy
5. Check **Actions** tab — wait for the green tick

---

## How poll links work

When you create a poll you receive two links:

| Link | Purpose |
|---|---|
| **Participant link** — `?poll=abc123` | Share this with everyone |
| **Organiser link** — `?poll=abc123&org=xyz789` | Keep this private — gives you delete access |

Participants who follow a poll link see only the voting view. The create page is not accessible from a poll link.

Bookmark your organiser link. If you lose it, you can recover it: take the participant link and append `&org=x` (any value works). You can also find the poll ID directly in Supabase under **Table Editor → polls**.

---

## The create password

The create page is protected by a password so that only the organiser can create polls. The password is stored as a SHA-256 hash in `index.html`:

```javascript
const CREATE_PW_HASH = 'your-hash-here';
```

To change the password:
1. Go to [SHA-256 hash generator](https://emn178.github.io/online-tools/sha256.html)
2. Type your new password and copy the hash
3. Replace the hash value in `index.html`
4. Commit and push — takes about 1 minute to deploy

The password is case-sensitive. Avoid leading or trailing spaces.

---

## Backing up

Every commit to GitHub is permanently stored in history. To take a manual snapshot before making changes:

- Go to your repo → click the green **Code** button → **Download ZIP**

To restore a previous version:
- Go to your repo → click **commits** → find the version → **Browse files** → restore the file you need

---

## Security notes

- Participant data is limited to a display name — no email, no account, no tracking
- All data is transmitted over HTTPS
- The anon key in the source code is intentional and safe — it is designed to be public
- The `service_role` key (in Supabase Project Settings) must never be put in frontend code
- The create password is stored as a hash in `index.html` — keep the repo private
- The organiser link contains a secret code in the URL — do not share it with participants
- Enable two-factor authentication on your GitHub account (Settings → Password and authentication)

---

## Timezone handling

All times are stored as UTC. The organiser enters times in their local timezone and the app converts to UTC on save. Participants see all slots converted to their own timezone automatically, using their browser's timezone setting. Daylight saving time is handled correctly.

---

## Troubleshooting

**Poll page is blank after following a link**
- Check the browser console (F12 → Console) for errors
- Confirm the SQL in Step 2 was run successfully in Supabase
- Try a hard refresh: Ctrl+Shift+R (Windows) or Cmd+Shift+R (Mac)

**Create password not working**
- The password is case-sensitive — check for accidental capitalisation or spaces
- Confirm the hash in `index.html` was generated from exactly the password you are typing
- Test the hash at [SHA-256 hash generator](https://emn178.github.io/online-tools/sha256.html)

**Vote not saving**
- Check Supabase Table Editor → votes for the row
- If you see a duplicate name error, the organiser needs to delete the existing vote first

**GitHub Pages showing README instead of the app**
- Confirm `index.html` is in the root of the repo (not inside a folder)
- Check Settings → Pages → source is set to main / root
