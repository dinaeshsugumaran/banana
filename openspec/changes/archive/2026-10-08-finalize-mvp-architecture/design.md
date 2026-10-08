## Context

See `proposal.md` — Why. The repository has no application code yet (`frontend/`, `backend/`, `routing/`, `shape-engine/`, `tests/` are empty), no specs exist in `openspec/specs/`, and the GitHub repository `dinaeshsugumaran/banana` is **public**. Development happens on a Mac.

Constraints that shape every decision (from `docs/product-requirements.md` §10, BAN-5, BAN-6, and the $0 target set for BAN-7):

- **Cost:** **$0/month recurring infrastructure** for development and a small pilot. A small unavoidable cost is allowed only when no credible free alternative exists, and must be named with its cause.
- **Not fragile:** free choices must still meet the Banana requirements and have a credible path to paid, production-grade hosting later.
- **Team:** small or solo. Few services, little to operate, one main language if possible.
- **Platform:** native iOS + Android from one codebase. No web/PWA.
- **Privacy:** location data is private, collect only what is needed, recorded walks belong to the user. Pilot users are in Québec (Law 25) and Ontario.
- **Future flexibility:** cycling, on-demand route generation, offline maps, turn-by-turn, progressive shape completion, and elevation must be addable without a rewrite.
- **Pilot scope:** two cities, six shapes each, up to 12 precomputed walking routes, online-only, a small number of pilot users.

Facts were checked in October 2026. Free-tier limits change without notice; BAN-9 and BAN-19 re-check them.

## Goals / Non-Goals

**Goals:**

- A $0/month recurring architecture for each of the seven BAN-7 decisions, with rationale, constraint fit, alternatives, trade-offs, and the conditions that would force a paid service.
- Clear integration boundaries between the mobile app, backend, map tiles, and the not-yet-chosen routing engine and shape engine.
- An explicit list of what is deferred and to which issue.

**Non-Goals:**

- Choosing the routing engine (BAN-8).
- Designing the shape model, normalization, scoring, or shape-to-road matching, or deciding where scoring runs (BAN-10).
- Database schemas, API shapes, or route-generation pipeline design (BAN-11, BAN-13, BAN-14).
- Any code, scaffolding, dependency installation, or infrastructure provisioning (BAN-9 onward).
- French-language requirements (not decided).

## Cost summary

| Phase | Recurring monthly infrastructure | Other costs |
|---|---|---|
| Development | **$0** (local Supabase in Docker, OpenFreeMap, local builds, GitHub Actions on a public repo) | None. A free Apple ID can install development builds on the developer's own iPhone (7-day signing). |
| Small pilot | **$0** (Supabase Free, OpenFreeMap, GitHub Actions, EAS Free optional, Firebase App Distribution) | **Apple Developer Program, US$99/year**, unavoidable for iOS pilot users (see decision 4). |

## Decisions

Each decision below is **confirmed by this change once approved**. Items marked **Deferred** are not decided here.

### 1. Mobile framework — React Native with Expo (TypeScript), local builds by default

**Decision:** React Native using Expo and TypeScript, with Expo **development builds** (not Expo Go, which cannot load the native map and background-location modules). Use the current stable Expo SDK at scaffolding time (SDK 57 / React Native 0.86 as of October 2026; BAN-9 pins versions).

**Builds at $0:**

- **Default:** build on the developer's Mac. Use `npx expo run:ios` / `run:android` for development, and local release builds (Xcode / Gradle, or `eas build --local`) for pilot binaries. Local builds are free and unlimited.
- **Optional:** the Expo EAS **Free** plan (15 iOS + 15 Android cloud builds per month as of October 2026; no overage charges) when cloud builds are convenient. Nothing depends on EAS. When the quota runs out, local builds continue.
- EAS Update (over-the-air updates) is not used in the MVP.

**Why:**

- One TypeScript codebase for iOS and Android. The same language serves backend functions (Supabase Edge Functions run TypeScript).
- Expo modules cover location (`expo-location`, including background tracking via a config plugin and TaskManager), secure storage, and native sign-in. Expo's prebuild generates the native projects, so the team does not maintain them by hand.
- A Mac is already available, so local iOS builds are practical. A paid build service is not needed.

**$0 evaluation:** fully free. Expo and React Native are open source. Local builds are free.

**Alternatives considered:**

| Option | Why not chosen |
|---|---|
| Flutter (Dart) | Also free and capable, with a MapLibre plugin. Adds a second language next to TypeScript backend code, and its MapLibre plugin is community-maintained. Reasonable runner-up. |
| Kotlin / Compose Multiplatform | Shared iOS UI and map integration need more native work than a solo team should take on. |
| Two native apps | Violates the single-codebase constraint. |
| Bare React Native (no Expo) | Same cost ($0). More native tooling to maintain. Expo prebuild keeps this path open. |

**Trade-offs:** local release builds need Xcode, Android Studio/SDK, and signing set up on the Mac (BAN-9 documents this). React Native adds a layer over native maps and GPS; acceptable for drawing one route and one live position.

### 2. Map rendering — MapLibre Native + OpenFreeMap, with a self-hosted PMTiles exit path

**Decision (renderer):** MapLibre Native through `@maplibre/maplibre-react-native` (MIT licence; v11+ supports only the New Architecture, React Native ≥ 0.80, and has an Expo setup path). Free and open source.

**Decision (tiles):** the **OpenFreeMap public instance** (OpenStreetMap data, OpenMapTiles schema; no API key, no registration, no usage caps, no cost). The style URL is **configuration, not code**.

**Exit path (planned, not built now), cheapest first:**

1. **Self-hosted PMTiles extract** of Montréal and Toronto, produced with the open-source `pmtiles` tool from the free Protomaps basemap build, served from free object storage. Cloudflare R2's free tier offers 10 GB storage, 10 million read operations per month, and no egress fees. Cloudflare may require a payment method on file even within the free tier. MapLibre Native reads PMTiles over HTTP range requests (`pmtiles://https://…`; documented for Android from 11.7, with iOS support to be confirmed in BAN-9).
2. A paid hosted tile provider (MapTiler, Stadia Maps) only if self-hosting proves unsuitable. That would need a new OpenSpec change.

**Attribution:** every map shows OpenFreeMap / OpenMapTiles / "© OpenStreetMap contributors" attribution (spec: Open map rendering with visible attribution).

**Future features:** MapLibre Native supports offline regions and local PMTiles files (`pmtiles://file://…`), runtime styling of route and progress layers (progressive shape completion), user location with heading, and raster-dem terrain sources (elevation). Turn-by-turn guidance is app logic on top of route data from the routing engine. Nothing in this choice blocks those features.

**$0 evaluation:**

- **Renderer:** free.
- **OpenFreeMap:** free, but donation-funded and without an SLA.
- **Self-hosted PMTiles:** free at pilot scale.
- **Privacy:** tile requests reveal to the tile host which map area a device is viewing (normal for any online map). OpenFreeMap states it uses no cookies or user database. The privacy policy (BAN-19) should mention the tile host.

**Alternatives considered:**

| Option | Why not chosen |
|---|---|
| Mapbox SDK + tiles | Proprietary SDK licence and usage-based pricing. |
| Google Maps SDK | Proprietary, usage-based billing, no OSM data, limited base-map styling. |
| MapTiler / Stadia Maps free tiers | Free tiers are limited or non-commercial; commercial use needs a paid plan. Kept only as a paid fallback. |
| Protomaps hosted API | Commercial use is limited to GitHub sponsors; self-hosting PMTiles is the intended route. |
| `tile.openstreetmap.org` | OSM Foundation tile usage policy does not allow apps to rely on it. |
| Self-hosting from day one | Adds setup, a storage account, and an update process before the pilot proves the concept. Kept as the exit path. |
| Bundling city PMTiles inside the app | Large app download, and Android cannot read PMTiles from bundled assets (no byte-range reads), so files would need copying to device storage first. Better suited to the future offline-maps feature. |

### 3. Backend and database — Supabase Free plan (PostgreSQL + PostGIS)

**Decision:** Supabase on the **Free plan**, with **no planned paid upgrade** for the MVP:

- **PostgreSQL + PostGIS** as the single system of record for shapes, routes, recorded walk paths, users, and walk history.
- **PostgREST** (auto-generated API) and **Postgres functions** for catalog reads and walk storage, protected by **Row Level Security (RLS)**.
- **Edge Functions** (TypeScript) only for privileged server logic, such as account deletion.
- **Storage:** not used by the MVP.

**Can it stay at $0 for the whole MVP?** Yes, for a small pilot, with the mitigations below. Free plan limits (Supabase pricing page, October 2026) compared with expected MVP usage:

| Free-plan limit | Expected MVP use | Fits? |
|---|---|---|
| Database 500 MB, shared CPU, 500 MB RAM | Catalog: 12 routes + 6 shapes, under 5 MB. Walks: one GPS fix every 2–5 s for a 45–90 min walk is about 600–2,700 points. Stored as one PostGIS line with timestamps, that is about 30–100 KB per walk, so roughly 3,000+ walks fit after system overhead. | Yes, if walks are stored as one geometry per walk, not one row per GPS point. BAN-14 must follow this. |
| Egress 5 GB/month | Catalog and route geometry per browse: tens of KB. Walk uploads are ingress. | Yes |
| 50,000 monthly active users (Auth) | Pilot: tens of users | Yes |
| 500,000 Edge Function invocations/month | Account deletion and occasional privileged calls | Yes |
| File storage 1 GB | Not used | Yes |
| 2 active projects | One production project. Development runs locally. | Yes |
| **Pauses after 1 week of low activity** | Pilot use may be sporadic | **Mitigated** (below) |
| **No automatic backups** | Pilot data must not be lost | **Mitigated** (below) |
| No SLA, shared compute | Small pilot | Accepted |

**Mitigations:**

- **Pausing:**
  - A scheduled GitHub Actions workflow makes a real read request to the production API once a day. Supabase's documentation says a few user requests each day are typically enough to prevent pausing.
  - If the project is paused anyway, data is kept and the project can be restored from the dashboard within 90 days. The team is alerted by Supabase's warning email.
  - GitHub disables scheduled workflows in public repositories after 60 days without repository activity; regular commits during the MVP avoid this. BAN-9 adds a check.
- **Backups:**
  - A scheduled job on the developer's Mac (launchd) runs `pg_dump` against the production database weekly, encrypts the dump (for example with `age`), and keeps it on the developer's disk, with a second encrypted copy on a separate disk or drive the developer controls.
  - Backups are **never** uploaded as GitHub Actions artifacts, because artifacts of a public repository are downloadable by any GitHub user.
  - A restore drill into the local Supabase stack happens before the pilot (spec: Project-controlled encrypted backups).
- **Region:** the production project is created in `ca-central-1`. BAN-9 confirms that Free-plan projects can select this region. If they cannot, the decision returns for approval before any pilot data is stored.

**Future cycling and on-demand routing:** travel mode is stored as data on routes (spec: Travel-mode-agnostic route data). On-demand route generation would need a hosted routing engine (BAN-8 territory) and is the most likely future cause of a paid server. Nothing in Supabase prevents calling such a service later.

**Path to production:**

- Supabase Pro removes pausing, adds daily backups, raises limits, and has the same API and data, so the move is a plan change, not a migration.
- Because the database is standard Postgres + PostGIS, the system can also move to self-hosted Supabase or any managed Postgres.

**Alternatives considered:**

| Option | $0? | Why not chosen |
|---|---|---|
| Self-hosted Postgres + PostGIS (plus self-built auth/API) on Oracle Cloud Always Free (OCI has Toronto and Montréal regions) | Yes, with a card on file | Idle instances can be reclaimed (7-day low-usage rule), 2026 reports of reduced free ARM capacity, and the team must run OS patching, TLS, backups, auth, and the API for private location data. Too much operational and security risk for a solo team. Kept as a long-term self-hosting option. |
| Self-hosted Supabase (Docker) on the developer's machine or a home server | Yes | Not reachable or reliable enough for pilot users, and exposes a residential network. Fine for local development only, which is what this design uses. |
| Neon Free (PostGIS supported, 0.5 GB, scales to zero instead of pausing) | Yes | Database only: auth, API, and server functions would be separate services, more pieces to integrate. Good database-only migration target. |
| Firebase Spark | Yes | No real geospatial queries, NoSQL modelling for relational data, stronger lock-in. |
| PocketBase / SQLite on a free VM | Yes | No PostGIS-class spatial support; same free-VM reliability problems. |
| Supabase Pro | No (US$25/month) | Not needed for a small pilot given the mitigations. It becomes the trigger-based upgrade path (see "What would cause paying"). |

### 4. Hosting — local development, Supabase Free in `ca-central-1`, free distribution channels

**Decision:**

- **Development:** everything runs locally. The Supabase CLI runs Postgres + PostGIS, Auth, the API, and Edge Functions in Docker on the Mac, and the app runs in simulators or on the developer's own devices. Cost: $0.
- **Pilot backend:** **one Supabase Free project** in **`ca-central-1` (Canada Central, Montréal)**. The second Free project slot stays unused, or is used as a temporary staging copy when needed.
- **Map tiles:** OpenFreeMap (decision 2). Banana hosts nothing.
- **Routing engine:** **Deferred to BAN-8.** For the MVP it runs offline to precompute routes (BAN-11), for example on the developer's Mac, and needs no hosting.
- **Pilot distribution:**
  - **iOS:** TestFlight. Requires the **Apple Developer Program (US$99/year)**, the one unavoidable cost. A free Apple ID only installs builds on the developer's own devices and they expire after 7 days. There is no free way to give an iOS build to other pilot testers, and Sign in with Apple also needs the paid membership.
  - **Android:** **Firebase App Distribution** (free, Spark plan) sends signed APKs to invited testers by email without a Google Play listing. The **Google Play registration fee (US$25 one-time)** is deferred until public launch.
- **Privacy policy page** (required for store distribution): hosted free on GitHub Pages from this public repository (BAN-19).

**Why:** No paid server anywhere. Production personal data stays in Canada, physically in Québec. That simplifies Québec Law 25 obligations about communicating personal information outside Québec, but does not settle them by itself: Supabase support/sub-processors, Firebase tester invitations, and the tile host are all outside Québec. The privacy assessment stays in BAN-19.

**Alternatives considered:**

| Option | Why not chosen |
|---|---|
| Oracle Cloud Always Free VM for everything | See decision 3: reclaim risk and operational burden. |
| Fly.io / Render / Railway free or trial tiers | Free allowances are small, time-limited, or need sleeping services; not more reliable than Supabase Free and adds a service. |
| Google Play internal testing for Android | Needs the US$25 Play registration fee now. Firebase App Distribution is free for the pilot. |
| Running a pilot backend on the developer's Mac | Not reachable or reliable for testers; security risk. |
| Supabase US region | Adds cross-border transfer of Canadian users' location data. |

### 5. Authentication — Supabase Auth with Sign in with Apple and Sign in with Google

**Decision:**

- **Supabase Auth** (included in the Free plan, 50,000 MAU) with **native Sign in with Apple** (iOS) and **native Sign in with Google** (iOS and Android). The app obtains an ID token from the platform and exchanges it with Supabase Auth.
- **No email/password and no magic links in the MVP**, so the system never sends email.
- Sessions are stored on the device in secure storage (Keychain / Keystore).
- **Account deletion:** an Edge Function verifies the caller's session, deletes the user's data (app tables reference the auth user with cascading deletes), **revokes the user's Sign in with Apple token** through Apple's REST API (Apple requires this for apps offering account deletion), and deletes the auth user with the service-role key, which stays server-side.
- RLS enforces per-user access on every table holding personal data.
- Google sign-in uses only basic, non-sensitive scopes (OpenID, email, profile), so it needs no paid or lengthy OAuth verification.

**Why this changed from email + password:** Supabase's built-in email sender only delivers to the project's team members and is limited to about 2 messages per hour, so it cannot serve pilot sign-ups or password resets. Free email providers (for example Resend's or Brevo's free tiers) need a verified sending domain for reliable delivery, which costs about US$10–20/year. Apple and Google sign-in need no email at all. The Apple Developer Program they rely on is already required for TestFlight, so they add no cost. Offering Sign in with Apple also satisfies App Store guideline 4.8, which applies as soon as Google sign-in is offered.

**$0 evaluation:** free. Supabase Auth Free, Apple sign-in (covered by the already-required membership), and Google OAuth client (free).

**Alternatives considered:**

| Option | Why not chosen |
|---|---|
| Email + password with a free SMTP tier | Needs a domain (~US$10–20/year) and email deliverability setup; password resets add support burden. Deferred until a domain exists. |
| Email + password without email (no confirmation, no reset) | Users who forget their password lose their walks; unsuitable. |
| Magic link / email OTP | Same email-delivery problem, plus deep-link handling. |
| Google sign-in only | Violates guideline 4.8 on iOS (Sign in with Apple must also be offered). |
| Clerk / Auth0 / Firebase Auth | Separate identity system to map into Postgres RLS; free tiers exist but add a vendor for no benefit. |

**Trade-offs:**

- Users must have an Apple ID (iOS) or a Google account (Android or iOS). Nearly all smartphone users do, but an Android user without a Google account cannot sign up in the MVP.
- More one-time setup (Apple Services ID and keys, Google OAuth clients) than email + password.
- Apple "Hide My Email" users have relay addresses. This doesn't matter for the MVP, because no email is sent.

### 6. CI/CD — GitHub Actions (public repo, Linux runners), no paid build or CI service

**Decision:**

- **GitHub Actions** on standard **Linux** runners. These are free and unmetered for public repositories.
  - **PR checks:** type checking, linting, unit tests, and database tests against the local Supabase stack in Docker. Branch protection (required checks, squash merges) is available free on public repositories.
  - **Production migrations:** SQL migrations in the repo are applied to the production Supabase project after merge to `main`, from a protected GitHub environment holding the access token as an encrypted secret.
  - **Keep-alive:** a daily scheduled request to the production API (decision 3).
- **Mobile binaries are not built in CI.** They are built locally (or with EAS Free when convenient) and distributed through TestFlight and Firebase App Distribution. No macOS runners are used.
- **Public-repository hygiene:** no secrets, personal data, or database dumps in the repository, workflow logs, or artifacts. CI uses encrypted secrets only.

**$0 evaluation:** free. Public-repository Actions usage on standard runners is free, and no other CI service is used. If the repository is made private later, it gets 2,000 free Linux minutes per month, which is enough for this workflow. Branch protection on a private repository would then need GitHub Pro (about US$4/month), so keeping the repository public is part of the $0 plan.

**Alternatives considered:**

| Option | Why not chosen |
|---|---|
| EAS Build in CI on every PR | Uses the Free quota quickly; not needed to validate PRs. |
| GitHub macOS runners for iOS builds | Unnecessary; local builds are free. |
| Bitrise / Codemagic / CircleCI | Extra vendor; free tiers are limited. |
| Automated store submission | Not needed for a small pilot; manual upload keeps setup small. |

### 7. City-selection UX — manual selection, remembered on device (unchanged)

**Decision:** The user **chooses Montréal or Toronto manually** from a simple city picker shown the first time they browse. The choice is **stored only on the device**, remembered across restarts, and changeable at any time. **Browsing never requires location permission**; it is first requested when the user starts a walk (BAN-17). The list of cities comes from the catalog (BAN-13). See `specs/city-selection/spec.md`.

**$0 re-check:** manual selection costs nothing and needs no geocoding, reverse-geocoding, or geofencing service. Location-based detection could also be done for free on the device, but would still require location permission before browsing. The $0 analysis gives no reason to change the decision.

**Why:** privacy (no location to browse), simplicity (one tap for two cities), works when permission is denied, lets users plan walks in a city they are not in yet, and new cities appear by adding them to the catalog.

**Alternatives considered:** location-based automatic selection (needs permission first; fails outside both cities); hybrid auto-suggest (unneeded complexity for two cities; possible later enhancement); one combined list (does not scale).

### Integration boundaries

```
┌──────────────── Mobile app (Expo / React Native, TS) ────────────────┐
│  City picker (local)  ·  Browse/preview  ·  Live walk + GPS record   │
│  Apple / Google native sign-in                                       │
│        │                     │                        │              │
│   MapLibre Native ◄── style/tiles ── OpenFreeMap (configurable URL)  │
└────────┼─────────────────────┼────────────────────────┼──────────────┘
         │ public anon key + user session (HTTPS)        │
         ▼                     ▼                        ▼
┌──────────────── Supabase Free (ca-central-1) ────────────────────────┐
│  Auth (Apple, Google)  ·  PostgREST + Postgres functions (RLS)       │
│  Edge Functions (account deletion, Apple token revocation)           │
│  PostgreSQL + PostGIS: shapes, routes (per city, versioned,          │
│  travel mode), users, recorded walks (one geometry per walk)         │
└───────▲──────────────────────▲──────────────────────────▲────────────┘
        │ daily keep-alive      │ load routes               │ weekly encrypted
        │ + migrations          │ (BAN-11 → BAN-13)         │ pg_dump
┌───────┴────────┐   ┌─────────┴──────── Offline (dev Mac) ┐  ┌──────┴───────┐
│ GitHub Actions │   │ Route generation (BAN-11)           │  │ Developer's  │
│ (public repo)  │   │ Shape engine (BAN-10)               │  │ Mac, encrypted│
└────────────────┘   │ Routing engine (BAN-8, TBD)         │  │ local copies │
                     └──────────────────────────────────────┘  └──────────────┘
```

- **Mobile app ↔ backend:** only through Supabase's public client API with the user's session. All access rules are enforced server-side (RLS, Edge Functions). No privileged keys ship in the app or the public repository.
- **Mobile app ↔ map tiles:** directly to the tile host via a configurable style URL.
- **Mobile app ↔ routing engine:** **none in the MVP.** Routes are precomputed.
- **Route generation ↔ backend:** the offline process writes finished, scored, versioned routes into the database. How and where it runs is BAN-8/BAN-11.
- **Shape engine:** used by route generation (planned-route score) and the post-walk score. Whether it runs on the device, in an Edge Function, in PostGIS, or as a shared TypeScript module is **deferred to BAN-10**. All options are free under this architecture.

### What would cause paying

| Trigger | Likely response | Approximate cost |
|---|---|---|
| Pilot growth beyond Supabase Free limits (500 MB DB, 5 GB egress), need for guaranteed uptime, no pausing, managed backups, or point-in-time recovery | Upgrade production to Supabase Pro (same API and data) | ~US$25/month |
| OpenFreeMap becomes unavailable or unsuitable | Self-host a PMTiles extract on free object storage; paid tiles only if that fails | $0 within R2's free tier; reads beyond 10 M/month are low cost |
| Wanting email/password or magic-link sign-in | Register a domain and use a free email-provider tier | ~US$10–20/year |
| More cloud builds than EAS Free allows, and local builds are not wanted | EAS Starter | ~US$19/month (avoidable with local builds) |
| Public launch on Google Play | Google Play developer registration | US$25 one-time |
| Making the repository private while keeping branch protection | GitHub Pro | ~US$4/month |
| On-demand route generation, or hosting the routing engine (future) | A hosted routing server (decided with BAN-8 / later) | Depends on engine and city coverage |
| Monitoring needs beyond free tiers (BAN-19) | Paid monitoring plan | Avoided while free tiers suffice |

Each of these requires an approved OpenSpec change (spec: Zero recurring infrastructure cost for the MVP).

### Deferred decisions

| Item | Owner | Notes |
|---|---|---|
| Routing engine (Valhalla, GraphHopper, OSRM, others), where it runs, OSM extracts for both cities | BAN-8 | Must keep travel mode configurable; MVP use is offline only. |
| Shape model, normalization, scoring, shape-to-road matching, and where scoring runs | BAN-10 | All placement options are free under this architecture. |
| Route-generation pipeline runtime and loading | BAN-11 | Runs offline; writes to the Supabase database. |
| Catalog, walk, and user schemas | BAN-11, BAN-13, BAN-14 | Must store each recorded walk as one geometry to fit the 500 MB limit. |
| Background location during walks | BAN-17 | Expo supports it via config plugin and TaskManager. |
| Monitoring/crash-reporting tool (free tier) | BAN-19 | |
| Law 25 privacy assessment, privacy policy, store compliance | BAN-19 | |
| French-language requirements | Not assigned | Not decided. Expo/React Native does not prevent localisation later. |
| Email/password sign-in, OTA updates, self-hosted tiles, offline maps | Later | Exit paths described above. |

## Risks / Trade-offs

- [Supabase Free pauses after a week of low activity; the keep-alive may stop counting if Supabase changes its policy] → Daily keep-alive; Supabase warning emails; manual restore keeps data. Upgrade to Pro is a planned, trigger-based response, not a default.
- [Supabase Free has no backups] → Weekly encrypted `pg_dump` to project-controlled storage and a restore drill before the pilot. Losing up to a week of pilot walks is accepted for the MVP.
- [Weekly backups depend on the developer's Mac being on] → The job retries at the next wake; BAN-9 documents a manual fallback. A missed week is visible in the backup folder.
- [OpenFreeMap is donation-funded with no SLA] → Configurable style URL and a documented free self-hosting exit path.
- [Free tiers change without notice (Supabase, EAS, Firebase, Cloudflare, GitHub)] → BAN-9 and BAN-19 re-check limits; any paid response needs an approved change.
- [Apple/Google sign-in excludes users without those accounts] → Acceptable for a small pilot; email sign-in can be added once a domain exists.
- [Public repository exposes code and CI logs] → Secret hygiene rules in spec and CI; no personal data in logs or artifacts.
- [`@maplibre/maplibre-react-native` + Expo SDK compatibility; PMTiles on iOS] → BAN-9 verifies the map library in a development build on both platforms first.
- [Hosting in Québec does not on its own satisfy Law 25] → Privacy assessment in BAN-19. Not legal advice.
- [Firebase App Distribution and TestFlight share tester emails with Google and Apple] → Disclosed in the privacy policy (BAN-19).

## Migration Plan

Not applicable: there is no existing system. After approval, the decisions are recorded in the repository and Linear (see `tasks.md`). BAN-9 is the first change that provisions anything. Reversing a decision later goes through a new OpenSpec change that modifies the `platform-architecture` or `city-selection` spec.

## Open Questions

- Exact Expo SDK, React Native, and MapLibre React Native versions → pinned in BAN-9.
- Which free monitoring tool to use → BAN-19.
