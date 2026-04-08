# VHO Scholarship Management System
## Setup Guide: Supabase + Vercel

---

## File Structure
```
vho-sms/
├── index.html              ← Login page
├── admin-dashboard.html    ← Admin portal (all admin features)
├── scholar-dashboard.html  ← Scholar portal (all scholar features)
├── shared.css              ← Shared design system
└── README.md               ← This file
```

---

## Demo Credentials (Frontend Demo)
| Role    | Email               | Password    |
|---------|---------------------|-------------|
| Admin   | admin@vho.org       | admin123    |
| Scholar | scholar@vho.org     | scholar123  |

---

## Features Implemented

### Admin
- ✅ Dashboard with stats (total scholars, passing GWA, pending docs, active polls)
- ✅ View scholars by year level (1st–4th)
- ✅ Scholar information & detail view
- ✅ Message scholars (per-scholar chat)
- ✅ Create polls
- ✅ Create/delete announcements (with categories)
- ✅ View GWA records — passing threshold: 2.00 (85+)
- ✅ Export scholar list (PDF/TXT download)
- ✅ Audit logs (who did what, when)

### Scholar
- ✅ Notifications (new announcements, polls, messages)
- ✅ Answer polls
- ✅ Message admin (chat)
- ✅ Upload Certificate / Registration (PDF)
- ✅ Upload Report Card (PDF)
- ✅ Input / update GWA
- ✅ GWA history tracking

---

## Supabase Setup

### 1. Create a Supabase project
Go to https://supabase.com → New Project

### 2. Create these tables in the SQL Editor:

```sql
-- Scholars/Users
create table profiles (
  id uuid references auth.users primary key,
  full_name text,
  email text,
  role text default 'scholar', -- 'admin' | 'scholar'
  year_level text,
  course text,
  school text,
  created_at timestamp default now()
);

-- GWA Records
create table gwa_records (
  id uuid default gen_random_uuid() primary key,
  scholar_id uuid references profiles(id),
  gwa numeric(4,2),
  semester text,
  recorded_at timestamp default now()
);

-- Documents
create table documents (
  id uuid default gen_random_uuid() primary key,
  scholar_id uuid references profiles(id),
  type text, -- 'certificate' | 'report_card'
  file_name text,
  file_url text,
  status text default 'pending', -- 'pending' | 'accepted' | 'rejected'
  uploaded_at timestamp default now()
);

-- Announcements
create table announcements (
  id uuid default gen_random_uuid() primary key,
  title text,
  body text,
  category text default 'General',
  created_by uuid references profiles(id),
  created_at timestamp default now()
);

-- Polls
create table polls (
  id uuid default gen_random_uuid() primary key,
  question text,
  options jsonb,
  active boolean default true,
  created_by uuid references profiles(id),
  created_at timestamp default now()
);

-- Poll Answers
create table poll_answers (
  id uuid default gen_random_uuid() primary key,
  poll_id uuid references polls(id),
  scholar_id uuid references profiles(id),
  answer text,
  answered_at timestamp default now(),
  unique(poll_id, scholar_id)
);

-- Messages
create table messages (
  id uuid default gen_random_uuid() primary key,
  sender_id uuid references profiles(id),
  recipient_id uuid references profiles(id),
  body text,
  read boolean default false,
  sent_at timestamp default now()
);

-- Audit Logs
create table audit_logs (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references profiles(id),
  user_name text,
  action text,
  detail text,
  created_at timestamp default now()
);
```

### 3. Enable Row Level Security (RLS)
In Supabase → Authentication → Policies, add policies so:
- Scholars can only see their own data
- Admins can see all data
- Anyone can read announcements/polls

### 4. Add Supabase Storage bucket
- Create a bucket named `documents`
- Set policy: authenticated users can upload to their own folder

### 5. Add Supabase JS to your HTML files

Add before `</body>` in each HTML file:
```html
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<script>
  const supabase = window.supabase.createClient(
    'YOUR_SUPABASE_URL',
    'YOUR_SUPABASE_ANON_KEY'
  );
</script>
```

Replace the demo login in `index.html` with:
```javascript
async function doLogin() {
  const { data, error } = await supabase.auth.signInWithPassword({
    email: document.getElementById('loginEmail').value,
    password: document.getElementById('loginPassword').value,
  });
  if (error) { /* show error */ return; }
  const { data: profile } = await supabase
    .from('profiles').select('role').eq('id', data.user.id).single();
  window.location.href = profile.role === 'admin' 
    ? 'admin-dashboard.html' : 'scholar-dashboard.html';
}
```

---

## Vercel Deployment

### 1. Push to GitHub
```bash
git init
git add .
git commit -m "Initial VHO SMS"
git remote add origin https://github.com/yourusername/vho-sms.git
git push -u origin main
```

### 2. Deploy on Vercel
1. Go to https://vercel.com → New Project
2. Import your GitHub repo
3. No build settings needed (static HTML)
4. Click Deploy

### 3. Add Environment Variables in Vercel
- `SUPABASE_URL` = your project URL
- `SUPABASE_ANON_KEY` = your anon key

---

## Color Palette (from VHO Brand)
| Name        | Hex       | Usage                    |
|-------------|-----------|--------------------------|
| Pink BG     | #e8d5d5   | Backgrounds, accents     |
| Blue Main   | #6b7fad   | Primary buttons, sidebar |
| Blue Dark   | #4a5a8a   | Sidebar background       |
| Red Heart   | #e05065   | Accent, heart logo       |
| Red Dark    | #a83248   | Heart shadow             |

---

Built for Victoria Heartstrong Organization © 2026
