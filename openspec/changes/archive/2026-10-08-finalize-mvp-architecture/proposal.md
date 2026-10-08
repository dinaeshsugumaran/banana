## Linear

- Issue: **BAN-7 — Finalize technical architecture** (child of BAN-5, Banana MVP)
- OpenSpec change: `finalize-mvp-architecture`

## Why

The Banana MVP cannot be scaffolded (BAN-9), and the shape engine (BAN-10) and routing engine evaluation (BAN-8) cannot start, until the core technical stack is decided. The PRD (`docs/product-requirements.md` §12) leaves the mobile framework, map stack, backend, hosting, and CI/CD open, and BAN-7 also owns the city-selection UX for the two pilot cities (Montréal and Toronto, decided in BAN-6).

The MVP must also run at **$0/month recurring infrastructure cost** through development and a small pilot, while keeping a credible path to paid, production-grade hosting later. Deciding the stack together, once, keeps the system small, free to run, and coherent for a solo team.

## What Changes

This change records architecture decisions and the constraints every later MVP change must follow. It adds no application code, schemas, scaffolding, or infrastructure.

- **Cost target:** no paid recurring infrastructure for the MVP. Every service runs on a free tier, open-source software, or the developer's own machine. Adopting a paid service needs an approved OpenSpec change.
- **Mobile framework:** React Native with Expo (TypeScript), one codebase for iOS and Android, using development builds. Binaries are built **locally on the developer's Mac** by default; the Expo EAS Free plan is optional convenience, never a requirement.
- **Map rendering:** MapLibre Native through `@maplibre/maplibre-react-native`, with OpenStreetMap-based vector tiles from the free OpenFreeMap public instance. The tile source is configurable. Exit path: a self-hosted PMTiles extract of Montréal and Toronto on free object storage.
- **Backend and database:** Supabase **Free plan**, used without a paid upgrade (PostgreSQL + PostGIS, Row Level Security, auto-generated API, Edge Functions). Inactivity pausing and missing backups on the Free plan are handled with a scheduled keep-alive request and encrypted backups the project controls.
- **Hosting:** local Supabase stack (Docker) for development; one Supabase Free cloud project in `ca-central-1` (Canada Central, Montréal) for the pilot. No other servers.
- **Authentication:** Supabase Auth with **Sign in with Apple and Sign in with Google** (native sign-in). There is no email/password, so the MVP never needs to send email, which avoids a paid domain and email provider. Account deletion is server-side and revokes Apple tokens.
- **CI/CD:** GitHub Actions on standard Linux runners (free for this public repository) for PR checks, production migrations, and the keep-alive job. Mobile binaries are not built in CI.
- **Pilot distribution:** TestFlight for iOS (needs the paid Apple Developer Program, the one unavoidable cost); Firebase App Distribution (free) for Android, deferring the Google Play registration fee to public launch.
- **City-selection UX:** manual city selection (Montréal or Toronto), remembered on the device, with no location permission needed to browse. Unchanged by the $0 analysis.
- **Planning artifacts:** add `docs/architecture.md`, add the tech-stack context to `openspec/config.yaml`, update the PRD's open questions and pilot-city wording, and update affected Linear issues (see `tasks.md`).
- **Deferred, not decided here:** routing engine (BAN-8), shape engine design and where it runs (BAN-10), route-generation pipeline runtime (BAN-11), database schemas (BAN-11/13/14), background location (BAN-17), monitoring tool (BAN-19), French-language requirements.

## Capabilities

### New Capabilities

- `platform-architecture`: cross-cutting platform requirements every MVP component must meet: zero recurring infrastructure cost, single cross-platform mobile codebase, MapLibre/OSM map rendering with attribution and a replaceable tile source, PostgreSQL + PostGIS as the system of record, sign-in that does not depend on sending email, per-user data isolation, complete account deletion, Canadian hosting, encrypted backups under project control, travel-mode-agnostic data, and CI checks on every pull request.
- `city-selection`: how a user chooses which pilot city's shapes and routes to browse, and the privacy constraint that browsing does not require location permission.

### Modified Capabilities

None. There are no existing specs in `openspec/specs/`.

## Impact

- **Code:** none in this change. BAN-9 (Project scaffolding) implements the stack described here.
- **New dependencies chosen (installed later, in BAN-9 and onward):** Expo / React Native, `@maplibre/maplibre-react-native`, `@supabase/supabase-js`, native Apple and Google sign-in modules, Supabase CLI, Docker (local development).
- **External services (all free tiers):** Supabase Free, OpenFreeMap, GitHub Actions, Expo EAS Free (optional), Firebase App Distribution (Android pilot), Google Cloud OAuth client (free).
- **Unavoidable cost:** Apple Developer Program (annual fee) for iOS pilot distribution through TestFlight and for Sign in with Apple. Not an infrastructure service, and no free alternative exists for iOS testers.
- **Downstream issues whose descriptions change (via Linear, in `tasks.md`):**
  - BAN-9: local builds, keep-alive and backup jobs.
  - BAN-12: Apple/Google sign-in, Apple token revocation, no SMTP.
  - BAN-16: manual city selection.
  - BAN-19: Firebase App Distribution instead of Google Play internal testing, no paid Supabase plan.
- **Documents:** `docs/product-requirements.md` (open questions, pilot cities), `openspec/config.yaml` (project context), new `docs/architecture.md`.
