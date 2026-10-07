# Contact Manager

A full-stack contact management app. Add, edit, delete, and browse contacts,
all persisted in a real PostgreSQL database.

## What it does

- Add a contact with a name and phone number
- View all saved contacts in a list (table on desktop, cards on mobile)
- Edit a contact's name or phone number in place
- Delete a contact, with a confirmation dialog
- All data is stored in PostgreSQL and survives page refreshes, browser
  restarts, and new sessions — nothing is kept only in React state or
  localStorage

## Technology used

- **Next.js 16** (App Router) + **React 19**
- **TypeScript** throughout
- **Tailwind CSS 4** for styling
- **PostgreSQL** for persistence, accessed with the [`pg`](https://node-postgres.com/)
  driver (no ORM) via server-side API routes
- Deployed on **Vercel**

## Project structure

```
app/
  page.tsx                 # Main page: header + <ContactManager />
  layout.tsx                # Root layout
  globals.css                # Tailwind + theme tokens
  api/
    contacts/
      route.ts              # GET (list), POST (create)
      [id]/route.ts          # PUT/PATCH (update), DELETE

components/
  ContactManager.tsx         # Client component: owns state, wires everything together
  ContactForm.tsx            # "Add contact" form
  ContactList.tsx             # Table (desktop) / card list (mobile)
  ContactItem.tsx              # One row/card, switches to EditContactForm when editing
  EditContactForm.tsx           # Inline edit form (Save / Cancel)
  DeleteConfirmation.tsx         # Accessible confirmation dialog
  Toast.tsx                       # Success/error toast notifications

lib/
  db.ts                       # PostgreSQL connection pool + schema bootstrap
  contacts.ts                  # Data access functions (list/get/create/update/delete)
  validation.ts                  # Shared client + server validation

types/
  contact.ts                    # Shared TypeScript types

.env.example                    # Required environment variable names (no secrets)
```

## 1. Install dependencies

```bash
npm install
```

## 2. Set up the database

You need a PostgreSQL database. The app will automatically create its
`contacts` table (and required `pgcrypto` extension) the first time it
connects — there's no separate migration step to run.

**Option A — local Postgres**

```bash
createdb contact_manager
```

**Option B — a free hosted Postgres (recommended for Vercel)**

Any Postgres-compatible provider works. Good free options:
- [Neon](https://neon.tech) — serverless Postgres, generous free tier, works great with Vercel
- [Vercel Postgres](https://vercel.com/storage/postgres) (powered by Neon) — one-click setup from your Vercel project
- [Supabase](https://supabase.com)

Create a database/project with any of these and copy the connection string
they give you.

## 3. Configure environment variables

Copy the example file and fill in your connection string:

```bash
cp .env.example .env.local
```

```
DATABASE_URL=postgresql://user:password@host:port/dbname?sslmode=require
```

(For a local database without SSL, add `?sslmode=disable` instead, e.g.
`postgresql://postgres:postgres@localhost:5432/contact_manager?sslmode=disable`.)

`lib/db.ts` also accepts `POSTGRES_URL` as an alternative name, since that's
what some providers' Vercel integrations set automatically.

## 4. Run locally

```bash
npm run dev
```

Open http://localhost:3000.

## 5. Build for production

```bash
npm run build
npm run start
```

## Push to GitHub

```bash
git init
git add .
git commit -m "Initial commit: Contact Manager"
git branch -M main
git remote add origin https://github.com/<your-username>/<your-repo>.git
git push -u origin main
```

`.env` / `.env.local` are already excluded via `.gitignore`, so your database
credentials won't be committed. `.env.example` (safe placeholder names only)
is committed intentionally.

## Deploy to Vercel

1. Go to [vercel.com/new](https://vercel.com/new) and import your GitHub repository.
2. Vercel will auto-detect Next.js — no build settings need to change.
3. Before the first deploy (or in Project Settings → Environment Variables
   afterward), add:
   - `DATABASE_URL` — your production Postgres connection string
     (if you used the Vercel Postgres/Neon integration from the Vercel
     dashboard, this — or `POSTGRES_URL` — is set for you automatically)
4. Deploy. On first request, the app creates its `contacts` table
   automatically if it doesn't already exist.

### Environment variables to configure on Vercel

| Variable       | Required | Notes                                                             |
| -------------- | -------- | ------------------------------------------------------------------ |
| `DATABASE_URL` | Yes      | PostgreSQL connection string. `POSTGRES_URL` also works as a name. |

## Notes

- Validation (required name, required + format-checked phone) runs both in
  the browser and again on the server — the server never trusts client input.
- Database credentials are only ever read on the server (`lib/db.ts`, used by
  API routes); nothing is exposed to the browser.
- Errors are caught and translated into user-friendly messages; raw database
  errors are logged server-side but never shown to the user.
