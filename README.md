# misc-nextjs-express

Personal multi-tool web app. pnpm monorepo with an Express 4 + Sequelize + SQLite backend and a Next.js 16 (App Router) + React 19 + Tailwind CSS v4 + shadcn/ui frontend.

## Features

- **Auth** - local (email/password + bcrypt) and Google OAuth2, session-based (24h cookie), email activation and password reset
- **User management** - admin panel to list, edit, activate/deactivate users and assign permissions
- **Feature flags** - toggle `registrationActive` and `forgotPasswordActive` from the admin UI
- **Blood pressure tracker** - log and view readings (systolic, diastolic, pulse, weight) with date range filtering and charts
- **Music sheets** - store and render ABC notation music sheets; public read, auth-required write
- **File storage** - upload files to Backblaze B2, track downloads via unique slug-based public links
- **Investment calculator** - convert EUR profit to PLN using live NBP exchange rate data

## Prerequisites

- Node.js 22+ and pnpm (local dev)
- Docker and Docker Compose (containerized usage)

## Local development

```bash
# Optional: run maildev to capture outgoing emails (activation, password reset)
docker run -d -p 1080:1080 -p 1025:1025 --name maildev maildev/maildev
# Web UI: http://localhost:1080

cp .env.example .env   # fill in SESSION_SECRET at minimum
pnpm install
pnpm seed              # interactive: creates admin user + default permissions
pnpm dev               # frontend :3000, backend :3001
```

## Docker

**Production:**

```bash
cp .env.example .env   # set SESSION_SECRET, WEBSITE_URL, API_URL
docker compose up --build -d

# First run: seed the database
docker compose exec backend pnpm seed
```

> `API_URL` is baked into the frontend image at build time. Rebuild when it changes.

**Dev mode (hot reload with source mounts):**

Add to `.env`:

```
COMPOSE_FILE=docker-compose.yml:docker-compose.dev.yml
```

Then `docker compose up --build`. Changes under `packages/backend/src` and `packages/frontend/src` are picked up automatically.

## Deployment

Push to `master` triggers a GitHub Actions workflow that deploys via SSH. Required repository secrets: `MISC2026_SSH_HOST`, `MISC2026_SSH_USER`, `MISC2026_SSH_PRIVATE_KEY`, `MISC2026_SSH_PORT`.

## Environment variables

See `.env.example` for the full list. Key variables:

| Variable | Description | Default |
|---|---|---|
| `SESSION_SECRET` | Express session secret (required) | - |
| `PORT` | Backend port | `3001` |
| `FRONTEND_PORT` | Frontend port | `3000` |
| `WEBSITE_URL` | Frontend URL - used for CORS and OAuth redirects | `http://localhost:3000` |
| `API_URL` | Backend URL - used by the frontend | `http://localhost:3001` |
| `DB_PATH` | SQLite file path | `<root>/database/database.sqlite` |
| `COOKIE_DOMAIN` | Session cookie domain | - |
| `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` | Google OAuth2 credentials | - |
| `GOOGLE_CALLBACK_URL` | OAuth2 callback URL | `http://localhost:3001/auth/google/callback` |
| `SMTP_HOST` / `SMTP_PORT` / `SMTP_USER` / `SMTP_PASS` / `SMTP_FROM` | SMTP settings | `localhost:1025` |
| `BACKBLAZE_KEY_ID` / `BACKBLAZE_APPLICATION_KEY` / `BACKBLAZE_BUCKET_ID` / `BACKBLAZE_PUBLIC_URL` | Backblaze B2 | - |

## Project structure

```
packages/
  backend/
    src/
      config/      Database (Sequelize + SQLite)
      middleware/  isAuthenticated, hasPermission
      models/      User, Permission, UserPermission, FeatureFlag,
                   BloodPressure, Music, B2File, B2FileDownload
      passport/    Local and Google OAuth2 strategies
      routes/      auth, users, feature-flags, blood-pressure,
                   music, b2files, public-downloads
      scripts/     seed CLI
  frontend/
    src/
      app/         Pages: login, register, profile, admin/*, blood-pressure/*,
                   music/*, b2files/*, investment/*
      components/  shadcn/ui components, navbar
      context/     AuthContext + useAuth()
      hooks/       useRequireAuth, useAbcjs, useMusicSheet
      lib/         api fetch wrapper, types, utils
```

## API routes

| Method | Path | Auth | Description |
|---|---|---|---|
| POST | `/auth/login` | - | Local login |
| POST | `/auth/logout` | - | Logout |
| GET | `/auth/google` | - | Google OAuth2 |
| GET | `/auth/me` | session | Current user |
| GET/POST | `/blood-pressure` | session | Blood pressure records |
| GET/PUT/DELETE | `/blood-pressure/:id` | session | Single record |
| GET/GET | `/music` `/music/:id` | - | Music sheets (public) |
| POST/PUT/DELETE | `/music` `/music/:id` | session | Manage sheets |
| GET/POST/DELETE | `/b2files` `/b2files/:id` | session | Backblaze file management |
| GET | `/public-downloads/:slug` | - | Public file download by slug |
| GET/POST/PATCH/DELETE | `/users` `/users/:id` | admin | User management |
| GET/PATCH | `/feature-flags` | admin | Feature flag management |

## Scripts

```bash
pnpm dev                          # start both packages in parallel
pnpm seed                         # interactive admin user seeder
pnpm --filter backend build       # compile backend TypeScript
pnpm --filter frontend build      # production frontend build
pnpm --filter frontend lint       # ESLint
```

## Tech stack

- **Frontend**: Next.js 16, React 19, Tailwind CSS v4, shadcn/ui, Recharts
- **Backend**: Express 4, Passport.js, Sequelize, SQLite, Nodemailer, Multer
- **Storage**: Backblaze B2
- **Package manager**: pnpm workspaces
