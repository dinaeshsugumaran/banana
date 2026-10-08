# Banana — MVP Architecture

Status: approved architecture for the Banana MVP (Linear **BAN-7**, OpenSpec
change `finalize-mvp-architecture`). This document summarizes the decisions.
The full rationale, alternatives, and risks are in the OpenSpec design
(`openspec/changes/finalize-mvp-architecture/design.md`, archived under
`openspec/changes/archive/` after merge). The binding requirements are in the
`platform-architecture` and `city-selection` specs under `openspec/specs/`.

Facts and free-tier limits were checked in October 2026 and change without
notice. BAN-9 and BAN-19 re-check them.

---

## Guiding constraint: $0/month recurring infrastructure

The MVP is developed, tested, and run for a small pilot (Montréal and Toronto)
with **no paid recurring infrastructure**. Every hosted service is used within a
free tier. Adopting a paid service requires an approved OpenSpec change that
names the cost and why no free alternative fits.

## Cost summary

| Phase | Recurring monthly infrastructure | Other costs |
|---|---|---|
| Development | **$0** (local Supabase in Docker, OpenFreeMap, local builds, GitHub Actions on a public repo) | None. A free Apple ID can install development builds on the developer's own iPhone (7-day signing). |
| Small pilot | **$0** (Supabase Free, OpenFreeMap, GitHub Actions, EAS Free optional, Firebase App Distribution) | **Apple Developer Program, US$99/year**, unavoidable for iOS pilot testers (TestFlight) and Sign in with Apple. |

---

## 1. Mobile framework

- **React Native with Expo, in TypeScript.** One codebase for iOS and Android. No web/PWA.
- Expo **development builds**, not Expo Go (Expo Go cannot load the native map and background-location modules).
- **Builds run locally on the developer's Mac by default** (`npx expo run:ios` / `run:android`, Xcode/Gradle release builds, or `eas build --local`). The Expo EAS **Free** plan is optional; nothing depends on it.
- No over-the-air updates in the MVP.
- Versions are pinned in BAN-9 (current stable as of October 2026: Expo SDK 57 / React Native 0.86).

## 2. Map rendering

- **MapLibre Native** via `@maplibre/maplibre-react-native` (open source, MIT).
- **Tiles: OpenFreeMap public instance** (OpenStreetMap data, OpenMapTiles schema, free, no API key).
- The style/tile URL is **configuration, not code**.
- **Attribution** (OpenFreeMap, OpenMapTiles, "© OpenStreetMap contributors") is shown on every map.
- **Exit path:** a self-hosted PMTiles extract of Montréal and Toronto on free object storage (e.g. Cloudflare R2's free tier). A paid tile provider only through a new OpenSpec change.
- The renderer supports the planned future features: offline regions and local PMTiles, heading, runtime-styled progress layers, and terrain/elevation sources.

## 3. Backend and database

- **Supabase Free plan**, with no planned paid upgrade for the MVP:
  - **PostgreSQL + PostGIS**: single system of record for shapes, routes, recorded walks, and users.
  - **PostgREST + Postgres functions** for the catalog and walk storage, protected by **Row Level Security**.
  - **Edge Functions** (TypeScript) only for privileged logic such as account deletion.
  - Supabase Storage is not used.
- **Fits the Free plan** (500 MB database, 5 GB egress, 50,000 MAU, 500,000 function calls) provided each recorded walk is stored as **one geometry per walk**, not one row per GPS point. That allows roughly 3,000+ walks.
- **Free-plan gaps and how they are covered:**
  - *Pauses after a week of low activity* → a daily keep-alive read request from GitHub Actions. If paused anyway, data is kept and can be restored from the dashboard.
  - *No automatic backups* → a weekly encrypted `pg_dump` run on the developer's Mac, stored on developer-controlled disks, never in GitHub artifacts. A restore drill happens before the pilot.
- **Path to production:** Supabase Pro (same API and data), self-hosted Supabase, or any Postgres + PostGIS host.

## 4. Hosting

- **Development:** fully local. The Supabase CLI runs Postgres/PostGIS, Auth, API, and Edge Functions in Docker.
- **Pilot backend:** one Supabase Free project in **`ca-central-1` (Canada Central, Montréal)**. BAN-9 confirms the region is selectable on the Free plan.
- **Map tiles:** OpenFreeMap. Banana hosts nothing.
- **Routing engine:** deferred to BAN-8. For the MVP it runs offline (e.g. on the developer's Mac) to precompute routes and needs no hosting.
- **Pilot distribution:**
  - iOS: **TestFlight** (needs the Apple Developer Program).
  - Android: **Firebase App Distribution** (free). The Google Play fee (US$25 one-time) is deferred to public launch.
- **Privacy policy page:** GitHub Pages (free).
- Personal data stays in Canada (in Québec). This helps with, but does not settle, Québec Law 25. The privacy assessment is BAN-19.

## 5. Authentication

- **Supabase Auth** with native **Sign in with Apple** (iOS) and **Sign in with Google** (iOS and Android).
- **No email/password and no magic links.** The MVP never sends email. Supabase's built-in sender only reaches team members, and reliable free email delivery needs a paid domain.
- Sessions are stored in device secure storage (Keychain / Keystore).
- **Account deletion** is an Edge Function. It verifies the session, deletes the user's data (cascading from the auth user), **revokes the Sign in with Apple token**, and deletes the auth user. The service-role key stays server-side.
- Google sign-in uses only basic, non-sensitive scopes.

## 6. CI/CD

- **GitHub Actions on standard Linux runners.** These are free for this public repository, and branch protection is free on public repositories. The workflows are:
  - **PR checks:** type check, lint, unit tests, and database tests against the local Supabase stack. These are required to merge; merges are squash only.
  - **Production migrations:** applied after merge to `main` from a protected environment with encrypted secrets.
  - **Daily keep-alive** request to the production API.
- **Mobile binaries are not built in CI.** They are built locally (or with EAS Free) and distributed through TestFlight and Firebase App Distribution.
- **Public-repository hygiene:** no secrets, personal data, or database dumps in the repository, CI logs, or artifacts.

## 7. City-selection UX

- The user **chooses Montréal or Toronto manually** the first time they browse.
- The choice is **stored only on the device**, is remembered, and can be changed any time.
- **Browsing never needs location permission.** It is first requested when starting a walk.
- The city list comes from the catalog; a city appears only when it has at least one shipped route.

---

## Integration boundaries

```
Mobile app (Expo/RN)
  ├─ MapLibre ── tiles ──► OpenFreeMap (configurable URL)
  ├─ Apple / Google native sign-in
  └─ HTTPS, public key + user session ──► Supabase Free (ca-central-1)
                                            Auth · PostgREST/RLS · Edge Functions
                                            Postgres + PostGIS
GitHub Actions (public repo) ── PR checks · migrations · daily keep-alive ──► Supabase
Developer's Mac ── weekly encrypted pg_dump ◄── Supabase
Offline route generation (BAN-11: shape engine BAN-10 + routing engine BAN-8)
  └─ writes scored, versioned routes ──► Supabase
```

- The app talks only to Supabase's public API with the user's session. All access rules are enforced server-side, and no privileged keys are in the app or the repository.
- The app never calls the routing engine in the MVP; routes are precomputed.
- Where the shape engine runs is decided in BAN-10. Every option (on device, Edge Function, PostGIS, shared TypeScript) is free under this architecture.

## What would cause paying

| Trigger | Likely response | Approximate cost |
|---|---|---|
| Pilot outgrows Supabase Free, or needs guaranteed uptime, no pausing, managed backups, or point-in-time recovery | Supabase Pro | ~US$25/month |
| OpenFreeMap unavailable or unsuitable | Self-hosted PMTiles on free object storage; paid tiles only if that fails | $0 within free tier |
| Email/password or magic-link sign-in wanted | Domain + free email-provider tier | ~US$10–20/year |
| More cloud builds than EAS Free, and local builds not wanted | EAS Starter | ~US$19/month (avoidable) |
| Public launch on Google Play | Play developer registration | US$25 one-time |
| Repository made private but branch protection kept | GitHub Pro | ~US$4/month |
| On-demand route generation / hosted routing engine (future) | Hosted routing server | Depends on BAN-8 |
| Monitoring beyond free tiers | Paid monitoring plan | Avoided while free tiers suffice |

Each requires an approved OpenSpec change.

## Deferred decisions

| Item | Owner |
|---|---|
| Routing engine (Valhalla, GraphHopper, OSRM, others), where it runs, OSM extracts | BAN-8 |
| Shape model, normalization, scoring, shape-to-road matching, where scoring runs | BAN-10 |
| Route-generation pipeline runtime and loading | BAN-11 |
| Catalog, walk, and user schemas (one geometry per recorded walk) | BAN-11, BAN-13, BAN-14 |
| Background location during walks | BAN-17 |
| Free monitoring/crash-reporting tool; Law 25 assessment; privacy policy; store compliance | BAN-19 |
| French-language requirements | Not decided |
| Email sign-in, OTA updates, self-hosted tiles, offline maps | Later |
