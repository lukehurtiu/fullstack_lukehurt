# Community Classes Full-Stack App

This repository is a full-stack starter configured for a **community classes website**.

It includes:
- Frontend: React + Vite (`apps/web`)
- Backend: Express + TypeScript (`apps/api`)
- Auth + Database: Supabase

## Product Behavior

Two user roles are supported:

- `admin`
  - can log in (after being assigned admin role in DB)
  - can create classes
  - can view all existing classes
- `member`
  - can sign up / log in
  - can view all existing classes
  - can register for classes

Role permissions are enforced in the backend API, not only in the frontend.
Signup always creates `member` users. Promote users to `admin` directly in `users` when needed.

## Data Model (Supabase)

The SQL file at `apps/api/supabase/schema.sql` creates:

- `users`
  - links each auth user (`auth.users.id`) to a role (`admin` or `member`)
- `community_classes`
  - class records created by admins
- `class_registrations`
  - member registrations to classes
  - unique `(class_id, member_id)` to prevent duplicate registration

It also configures row-level security policies aligned with roles.

`apps/api/prisma/schema.prisma` mirrors these tables for reference, but the API uses Supabase client calls directly.

## 1. Create a Supabase Project

Create a project in Supabase and collect:
- Project URL
- Publishable key
- Service role key

## 2. Run Database Schema

In the Supabase SQL editor, run:

- `apps/api/supabase/schema.sql`

The script also seeds 3 sample classes for testing when at least one row exists in `public.users`.

## 3. Configure Backend Environment

From repo root:

```powershell
copy apps\api\.env.example apps\api\.env
```

Set values in `apps/api/.env`:

```env
SUPABASE_URL="https://YOUR-PROJECT-ID.supabase.co"
SUPABASE_PUBLISHABLE_KEY="YOUR_SUPABASE_PUBLISHABLE_KEY"
SUPABASE_SERVICE_ROLE_KEY="YOUR_SUPABASE_SERVICE_ROLE_KEY"
CORS_ORIGINS="http://localhost:5173,https://YOUR-VERCEL-DOMAIN.vercel.app"
PORT=4000
```

## 4. Configure Frontend Environment

```powershell
copy apps\web\.env.example apps\web\.env.local
```

For local development:

```env
VITE_API_BASE_URL="http://localhost:4000"
```

For deployed frontend, set this to your deployed API base URL (no trailing slash).

## 5. Install Dependencies

```bash
npm install
```

## 6. Run Locally

```bash
npm run dev
```

- Web: `http://localhost:5173`
- API health: `http://localhost:4000/health`

## API Endpoints

### Auth

- `POST /api/auth/signup`
  - body: `{ email, password }`
  - always creates a `member` user row
- `POST /api/auth/login`
  - body: `{ email, password }`
- `GET /api/auth/me`
  - header: `Authorization: Bearer <token>`

### Admin

- `GET /api/admin/classes`
- `POST /api/admin/classes`
  - body: `{ title, description, instructorName, location, startsAt, capacity }`

To promote a user to admin in Supabase SQL:

```sql
update public.users
set role = 'admin'
where id = 'USER_UUID_HERE';
```

### Member

- `GET /api/member/classes`
  - includes registration status for the logged-in member
- `POST /api/member/registrations`
  - body: `{ classId }`

## Deploy Notes

### API service
Set:
- `SUPABASE_URL`
- `SUPABASE_PUBLISHABLE_KEY`
- `SUPABASE_SERVICE_ROLE_KEY`
- `CORS_ORIGINS` including your frontend URL

### Frontend service
Set:
- `VITE_API_BASE_URL` to deployed API URL (no trailing slash)
