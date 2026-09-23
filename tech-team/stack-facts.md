# Stack facts

What claude-mem knows about Manuel's repos. The lead pastes only the current repo's section into spawn prompts. Verify each fact against the repo before relying on it, and update this file when a run proves one stale.

## grapeseed (Bucare)

- Bun + Nx monorepo. `packages/bucare` is Next.js 16 on :6969 (each worktree's dev server on its own port), `app/` routes-only.
- Feature packages in `packages/bucare/packages/<domain>/{actions/<verbNoun>.ts, view/, types.ts}` (alias `bucare/*`), actions re-exported from `actions/index.ts`, plus `bucare-design-system`, `bucare-common` and `packages/supabase` (config.toml, migrations, seed.sql).
- Event-sourced on Supabase: `record_events` RPC, trigger projections, workspace-scoped RLS through `current_user_workspaces()` and `has_any_role()`.
- Server actions are `'use server'` with `(input, ctx?: CommandContext)`, static imports, `revalidatePath` for every route that shows the data, and throw `ActionError`.
- Storage: private buckets, path `{workspace}/{entity}/{file}/{filename}`, a typed `validateFile`, short-lived signed URLs from a Server Action, the storage object deleted before the deletion is recorded. `storage.objects` policies are wrapped in `SET ROLE supabase_storage_admin; ... RESET ROLE;`. Migrations applied through make.
- Make targets: start, stop, status, gen-types, supabase-reset, supabase-seed, test, test-unit, test-e2e.
- Jest colocated `*.spec.ts`; E2E in `e2e/{features,pages,helpers}` with a `BasePage`, run with `bun run e2e` / `e2e:ui`.
- Request interception in `proxy.ts` (Next.js 16); never add `middleware.ts`. UI text goes through i18next, English first.
- No Sentry SDK. `.entire/` enabled. Project skills: impeccable, apple-design, nextjs-server-actions, next-dev-loop, next-cache-components-*, supabase, supabase-postgres-best-practices.
- Drift: CLAUDE.md still says E2E runs "in mock mode" and `playwright.config.ts` sets `NEXT_PUBLIC_USE_MOCKS=true`, against Manuel's "no more mocks".

## capital-circles

- Next.js 16 + Payload 3 (Postgres on local Supabase :54322) + Stripe (iDEAL, SEPA) + `@sentry/nextjs` (org grapeseed-pp) on Vercel.
- `lib/tryCatch.ts`, `packages/{registration,payments,stripe,membership,checkout,auth}`, `lib/payload-cache.ts`, `_actions.ts` files for Server Actions, nuqs.
- Tests: `bun test`, `bun run test:e2e`, `test:e2e:ui`, `seed:e2e`; `e2e/pages/*.page.ts` (with a `BasePage`) and `e2e/specs/*.e2e.ts`.
- `make start` = supabase start + dev + `stripe listen`. Migrations: `bun run migrate:create|migrate|migrate:status`; production migrations run on merge through GitHub Actions.
- Squash merges. `.entire/` enabled. Project skills: impeccable, next-cache-components-adoption.

## DRE-Eugenia-todo-app

- Next.js + Supabase + Stripe (iDEAL to SEPA). `lib/try-catch.ts`, `packages/{stripe,supabase,subscriptions,ui}`, shadcn in `components/ui`, nuqs.
- Jest via `bun run test`; Playwright via `bun run test:e2e` (`e2e/{tests,pages,helpers}`, no `BasePage`), seeding Supabase and Stripe test mode.

## whatsapp-webhook

- Bun + Express, strict TS, Zod Env in `packages/env.ts`, `{ data, error }` / `Error | undefined`, Spanish user messages.
- Firebase Admin SDK with deny-all client rules; `@sentry/bun` via `--preload instrument.ts`.
- Jest; Testcontainers (Gotenberg) for PDF tests. Required CI "Run tests" on PRs; Cloud Build to Cloud Run on main. Merge commits.

## gentle-frame-milano

- Next.js + Payload (Postgres, S3, en/nl), `lib/tryCatch.ts`, nuqs. DESIGN.md, PRODUCT.md and `.impeccable` checked in.
- `make` runs `supabase start && bun run dev` on :3000. Project skill: impeccable.

## rekenen-doe-je-zo

- Next.js + Strapi + `@sentry/nextjs` (org grapeseed-pp) on Vercel. `app/lib/try-catch.ts`, `app/packages/{schedule,forms,emails,util}`, `fetchWithRetry`. No CI. Merge commits.

## Tools

- MCPs: Sentry, Stripe, Playwright (`bunx @playwright/mcp@latest`), Supabase, Vercel. Most are project-scoped, so check which ones the session actually has.
- CLIs: gh, entire, supabase, stripe, vercel.
