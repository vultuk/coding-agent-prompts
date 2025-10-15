You are a senior full-stack engineer. Set up a fresh, production-ready monorepo that lets us start building a mobile app immediately.

STACK (latest stable; pin exact versions in package.json):
- Nx (monorepo) • Bun (pkg mgr/runtime)
- Expo (mobile via @nx/expo) • Next.js 15 (web) • Hono (API)
- TypeScript (strict) • Tailwind CSS (via Bun)
- Postgres on Supabase (host) via Drizzle ORM (no supabase-js) using postgres.js driver on Bun
- Clerk (auth) • Redis (ioredis) • Resend (email)
- Docker • Railway
- Tiny helpers allowed: zod, @hono/zod-validator, hono/secure-headers, @hono-rate-limiter (+ redis store)

RUNTIME CLARITY
- Use Bun as the package manager across the monorepo.
- Next.js dev/build/runtime runs on Node (per Next.js system requirements). API (Hono) runs on Bun.
- Document this in README to avoid “Bun runtime for Next” surprises.

INSTRUCTIONS
1) Git bootstrap
   - Initialise repo in current folder: `git init`
   - Create origin: `https://github.com/$GIT_ORGANIZATION/$APPLICATION_NAME.git`
   - First commit “Initial commit for $APPLICATION_NAME” and push to `main`.

2) Nx workspace at repository root (not a subfolder)
   - Name: "$APPLICATION_NAME", layout: integrated (`--preset=apps`).
   - Ensure nx.json, tsconfig.base.json at root.
   - ESLint + Prettier + Husky (pre-commit: lint, typecheck).
   - Enable `enforce-module-boundaries` with tags: ["type:app","type:lib","scope:shared","scope:web","scope:mobile","scope:api"].

3) Apps
   - apps/mobile: Expo (RN + TS) scaffolded with **@nx/expo**; Clerk Expo integrated.
   - apps/web: Next.js (App Router + TS) with @clerk/nextjs; include `/api/health` and a protected page.
   - apps/api: Hono (Bun + TS). Routes: `GET /health`, `GET /version`, `POST /emails/test` (Resend), 
     `GET /realtime/ping` (Redis roundtrip), `GET /user/me` (Clerk-protected).
     Middleware: CORS, logger, error handler, **secure-headers**, **Zod validation (@hono/zod-validator)**,
     **JWK auth (pointed at Clerk JWKS)**, rate‑limit (Redis-backed). `PORT` default 8787.
     Export route types and generate a typed client in `libs/shared/sdk` via `hono/client`.


4) Libraries (Nx libs)
   - Shared libs in **libs/shared/**:
     • libs/shared/ui — cross-platform components (React Native Web + NativeWind/Tailwind).  
     • libs/shared/utils — logger, **env validation (zod)**, helpers. 
     • libs/shared/config — eslint, prettier, tailwind config, postcss config.  
     • libs/shared/db — **Drizzle** setup (postgres.js driver on Bun), schema, migration helpers.  
     • libs/shared/sdk — Redis adapter (ioredis by default; swappable to Bun.redis),
       Resend client, **React Email** templates, typed Hono API client, shared types.
   - App-specific libs in **libs/APP_NAME/** (e.g. libs/web/feature-*, libs/mobile/feature-*, libs/api/feature-*).
   - TS path aliases: `@shared/*`, `@web/*`, `@mobile/*`, `@api/*`.

5) **Database policy (Drizzle Migrations = source of truth)**
   - All database additions/changes MUST be implemented via **Drizzle migrations**. No ad-hoc SQL, no direct edits to the live DB.  
   - Workflow:
     1) Edit schema in `libs/shared/db/schema/*.ts`.  
     2) Generate migration: `bun db:generate` (drizzle-kit).  
     3) Review SQL, commit migration files.  
     4) Apply: `bun db:migrate`.
   - Scripts (root `package.json`):
     • `bun db:generate` → `drizzle-kit generate --config libs/shared/db/drizzle.config.ts`  
     • `bun db:migrate` → run migrations against `DATABASE_URL_MIGRATE` if present, else `DATABASE_URL`
     • `bun db:studio` (optional) → drizzle-kit studio if installed
   - Env: `DATABASE_URL` (pooled for API), `DATABASE_URL_MIGRATE` (non-pooled for migrations), `DRIZZLE_LOG?`.
   - Driver: use `drizzle-orm/postgres-js` for Bun runtime (document in drizzle.config).

   - Include a short CONTRIBUTING.md section that restates the above policy.

6) Clerk auth
   - Web: protect `/app/*` via `clerkMiddleware()` + `createRouteMatcher` in `middleware.ts`; sample protected page.
   - Mobile: Clerk Expo with **SecureStore token cache configured**; sign-in/out + “Me” screens.
   - API: **JWK middleware** verifies Bearer tokens against `CLERK_JWT_ISSUER` (JWKS URL).
     `requireAuth` attaches `ctx.var.user` (sub, email, roles if present).
   - Env: `CLERK_PUBLISHABLE_KEY`, `CLERK_SECRET_KEY`, `CLERK_JWT_ISSUER` (or JWKS URL), `CLERK_WEBHOOK_SECRET?`.

7) Redis
   - ioredis singleton; pub/sub helpers `publish(topic,payload)`, `subscribe(topic,handler)`.
   - Redis adapter interface with default **ioredis** client; pub/sub helpers `publish(topic,payload)`, `subscribe(topic,handler)`.
     Optional: swap to **Bun.redis** via adapter flag for Bun-only deployments.
   - Env: `REDIS_URL`. Demo heartbeat from `/realtime/ping` consumed in web + mobile.

8) Resend
   - `sendEmail({ to, subject, reactBody })` using **React Email** components for templates.
   - Env: `RESEND_API_KEY`, `MAIL_FROM`. API `/emails/test?to=` sends sample.

9) Tailwind CSS (via Bun)
   - Install `tailwindcss`, `postcss`, `autoprefixer`.
   - libs/shared/config exports `tailwind.config.ts` (tokens, plugins: typography, forms) and PostCSS config.
   - Content globs: `apps/web/**/*.{ts,tsx}`, `apps/mobile/**/*.{ts,tsx}`, `libs/shared/ui/**/*.{ts,tsx}`, plus app-specific libs.
   - Web: `globals.css` with `@tailwind base; @tailwind components; @tailwind utilities;`.
   - Mobile: **nativewind**; add `"nativewind/babel"` in `babel.config.js`. Ensure RNW config for shared UI.

10) Tooling & scripts
   - Strict TS everywhere.
   - Scripts:
     • `bun dev:all` → `nx run-many --target=serve --all --parallel` (api + web + mobile)  
     • `bun dev:api`, `bun dev:web`, `bun dev:mobile`  
     • `bun typecheck`, `bun lint`, `bun test`  
     • `bun db:generate`, `bun db:migrate`
   - `.env.example` at root and app-level; load via expo-constants/app.config (mobile), **process.env (Next.js with NEXT_PUBLIC_*)** (web), process.env (api).

11) **Docker Compose for human-friendly local testing**
   - Root `docker-compose.yml` for one-command startup with healthchecks + dependencies:
     • **db**: `postgres:16-alpine` (vol `postgres-data`, ports `5432:5432`).  
     • **redis**: `redis:7-alpine` (vol `redis-data`, ports `6379:6379`).  
     • **migrate**: lightweight container that waits for `db` healthy then runs `bun db:migrate` once; exits 0 on success.  
     • **api**: build `apps/api` (depends on `db`, `redis`, `migrate`; health `/health`; port `8787:8787`).  
     • **web**: build `apps/web` (depends on `api`; health `/api/health`; port `3000:3000`).  
     • **mailpit (optional)**: dev email sink (`1025`, `8025`).
   - Named volumes: `postgres-data`, `redis-data`. Network: `appnet`.
   - README shows: `docker compose up -d` and `docker compose down -v`. Include healthcheck commands and wait‑for scripts.

12) Docker (per app images)
   - Root `.dockerignore`.
   - Multi-stage **Node** image for `apps/web` (Next) and **Bun** image for `apps/api` (Hono) with matching healthchecks.

13) Railway
   - `railway.json` and README deploy steps for web + api.
   - Document external services (Supabase Postgres, Redis) and env wiring.

14) DX & CI
   - GitHub Action on PR: install (bun with `--frozen-lockfile`), typecheck, lint, **generate migrations to ensure schema compiles**, and build web + api.
   - Add CI steps:
     • Nx **affected**: run lint/typecheck/build only on changed projects.
     • Fail if there are un-applied migrations in a throwaway Postgres service:  
     • Start ephemeral Postgres, run `bun db:migrate`, then run `bun db:generate` and ensure it produces **no new changes** (i.e. schema and DB are in sync).  
     • This enforces the “migrations only” policy.

ENV VARS (.env.example)
- CLERK_PUBLISHABLE_KEY, CLERK_SECRET_KEY, CLERK_JWT_ISSUER, CLERK_WEBHOOK_SECRET?
- DATABASE_URL, DATABASE_URL_MIGRATE?
- REDIS_URL
- RESEND_API_KEY, MAIL_FROM
- PORT (api), NEXT_PUBLIC_API_BASE_URL, EXPO_PUBLIC_API_BASE_URL
- (Dev email optional) MAILPIT_SMTP_URL, MAILPIT_FROM?

DELIVERABLES
- Complete file tree and key files with concise comments.
- Exact commands: install, env setup, dev (api/web/mobile), **migrations (generate/apply)**, `docker compose up -d`, Railway deploy.
- Root README: stack overview, env vars, DB migration policy & workflow, Tailwind usage, “compose up” workflow, deployment, “first hour” checklist.
- Proof:
  • `curl :8787/health` ok; Next.js `/api/health` ok.  
  • Expo runs with Clerk sign-in; “Me” hits `/user/me`; Redis heartbeat visible.  
  • `bun db:generate && bun db:migrate` runs cleanly.  
  • `docker compose ps` shows services healthy.

CONSTRAINTS
- Minimal, production-lean code; avoid sample cruft.
- Only listed libs unless a tiny helper is indispensable.
- Maximise shared code in `libs/shared`; app-only code lives in `libs/APP_NAME`.
- All secrets via env; never commit keys.

SECURITY
- Enable `secure-headers`.
- Add a reusable rate-limit middleware (Redis-backed) and apply to `/emails/*` + auth routes.
- Document CORS policy (dev: localhost origins incl. Expo; prod: env-based allowlist).

OUTPUT
1) Brief summary (≤12 lines).
2) Repo tree (collapsed where obvious).
3) Key snippets (non-boilerplate): Hono routes + `requireAuth`, Drizzle setup/schema, migration scripts, Clerk usage (web/mobile/api), Tailwind config, shared UI, docker-compose.yml, Dockerfiles, railway.json, scripts.
4) One-liners to run dev for api/web/mobile, generate/apply migrations, and build/run Docker or Compose locally.
