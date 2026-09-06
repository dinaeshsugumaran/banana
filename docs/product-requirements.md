# Banana — Product Requirements

Status: Product-discovery output. Not yet linked to a Linear issue or an OpenSpec
change. This document records only the decisions explicitly approved during the
discovery interview on 2026-09-03. Items the user deferred are listed under
"Open questions".

---

## 1. Product vision

The shape is the point. Banana generates a walkable route that forms a
user-selected shape on the map. Unlike normal walking/running apps — where the
route is about reaching a destination or tracking fitness — Banana turns the walk
itself into a way of drawing a recognizable shape.

---

## 2. Target users

- Primarily casual walkers and GPS-art enthusiasts who want to turn a walk into a
  recognizable shape.
- Users are comfortable using a smartphone and following a map / turn-by-turn
  directions.
- Routes are designed for normal walking, not advanced athletic ability.
- V1 is **not** designed for serious runners, competitive athletes, or users
  requiring accessibility-specific routing.

---

## 3. Core user experience

### MVP flow

Login → browse available shapes/routes → view the complete proposed route →
start the walk → see live GPS position → record the walk → save it to history.

### Details

- The user selects from a catalog of pre-determined shapes available for their
  city.
- Before starting, the app shows the proposed walk with relevant information:
  distance, estimated time, route details, and how well the route matches the
  intended shape (a shape-match / accuracy score or equivalent).
- The shape is pre-determined, so the user does not draw the final shape
  themselves.
- Route parameters (target distance, time, difficulty) are **not** required for
  the MVP; they are a future capability.

---

## 4. Shape system

- MVP shapes are **hand-curated / pre-determined per city** — not user-submitted
  and not automatically generated.
- The catalog per city starts small and manageable. Categories include
  recognizable objects, animals, symbols, and other interesting shapes. The exact
  number and category mix are decided during product planning.
- A shape is stored as an **abstract geometric form that is fitted to the local
  street network**, not a permanently fixed route. The same conceptual shape can
  therefore yield different walkable routes over time.
- MVP provides **one best route per shape**. Multiple route options per shape is a
  future enhancement.
- A shape-match / accuracy score (or equivalent information) is shown before and
  after the walk. The bar is "clearly recognizable as the intended shape," not
  mathematical perfection — real street networks cannot always reproduce an
  abstract shape precisely.

---

## 5. Route generation

- MVP uses **precomputed, stored routes** per city/shape — not on-demand
  generation.
- Each shape is anchored to a specific area of the city. The user's current
  location does **not** move or reshape the route; the user travels to the
  designated starting point.
- Route constraints:
  - Walkable and pedestrian-legal paths only.
  - Avoid highways, private/inaccessible areas, water, and other unsuitable
    areas.
  - Prefer a closed loop so the user finishes near the starting point.
- Routing accounts for real pedestrian constraints: one-way restrictions where
  applicable, crossings, pedestrian paths, parks, trails, and stairs where
  appropriate. Routing is **not** restricted to roads only.
- Use a **third-party routing engine based on OpenStreetMap data** for the MVP
  rather than building a custom engine. The specific engine is selected later
  based on route quality, cost, licensing, and API capabilities.
- MVP is **walking only**. The routing layer must not be hard-coded as
  permanently pedestrian-specific; the architecture must support adding cycling as
  a future travel mode without major restructuring. Cycling is not implemented
  now.

---

## 6. Maps and visualization

- **Map provider:** cost is an important constraint and the app should stay
  lightweight. Prefer an open-source / low-cost solution (e.g. MapLibre with
  OpenStreetMap-based data). Avoid committing to an expensive proprietary map
  provider for the MVP unless there is a strong technical reason. Respect required
  attribution and licensing obligations.
- **Offline maps:** not required for the MVP; not implemented now. The
  architecture leaves room for offline map support later.
- **MVP map rendering** focuses on displaying the precomputed route and the
  target shape, plus the user's live GPS position during the walk.
- **Pre-walk view:** top-down view showing the entire route and shape so the user
  understands the complete artwork and route, along with basic route information
  such as distance.
- **During the walk:** on start, the map automatically zooms into the user's
  current location and transitions toward a navigation-style map view (similar to
  Google Maps).
- The mapping architecture must allow the following to be added later **without
  redesigning the core mapping system**: live position, heading, turn-by-turn
  directions, progressive shape completion / colour changes, elevation, and
  offline maps.

---

## 7. During-walk experience and GPS tracking

### During-walk experience (MVP)

- The MVP is about displaying and validating the available shape routes, not
  providing a full navigation experience.
- MVP: browse precomputed shapes/routes; selecting a shape displays the complete
  proposed route on a top-down map with the shape and basic route info (e.g.
  distance); on start, the map follows the user's live GPS position in a
  navigation-style view.
- **Not in the MVP:** turn-by-turn navigation / detailed walking directions, live
  navigation guidance, progressive shape colouring / completion, heading display,
  and elevation information. These are future phases.
- MVP core objective: prove that we can create and display walkable routes that
  successfully form recognizable shapes.

### GPS tracking (MVP)

- GPS tracking is part of the MVP.
- The app requests the user's location permission and displays their live GPS
  position on the map while they are walking.
- The user's position updates continuously so the app can determine where they
  are relative to the proposed shape route.
- The MVP collects basic movement data available from GPS: distance travelled,
  speed, duration, and actual path taken.
- The actual GPS path is recorded during the walk and saved so it can later be
  compared with the planned route and target shape.
- Future: a full walk-history system, detailed post-walk analytics, and advanced
  shape-completion analysis.

---

## 8. MVP scope and boundaries

### In scope

- **Platform:** native mobile application for both iOS and Android. No web / PWA
  version. Prefer a cross-platform framework to maintain a single codebase for
  both platforms.
- **Launch city:** one pilot city (specific city not yet finalized).
- **Accounts / authentication:** users must have accounts and log in. Walks and
  user data are associated with the user's account.
- **Backend and database:** required. The backend serves the shape catalog and
  stores users' recorded walks and history.
- **Walk history:** basic history is part of the MVP — users can see the walks
  they have completed.
- **Connectivity:** online-only is acceptable for the MVP.
- **Core experience:** user login → browse available shapes/routes → view the
  complete proposed route → start the walk → see live GPS position → record the
  walk → save it to the user's history.

### Out of scope for the MVP

- Web / PWA version.
- Offline maps.
- Turn-by-turn navigation.
- Advanced elevation data.
- Advanced post-walk analytics / shape analysis.
- Route parameters (target distance, time, difficulty).
- Multiple routes per shape.
- Progressive shape colouring / completion, heading display.
- Cycling travel mode.
- Additional cities beyond the pilot.

---

## 9. Future features and product direction

Not part of the MVP; recorded as intended direction.

- Social sharing of completed shape artwork / walks.
- User-submitted custom shapes.
- Automatic shape generation based on the user's location and the available
  street network.
- On-demand route generation near the user's current location.
- Challenges, streaks, achievements, and other gamification features.
- Leaderboards and community rankings.
- Export completed walks as images and GPX files.
- Ability to discover and browse shapes created by other users.
- Route parameters: target distance, time, difficulty.
- Multiple route options per shape.
- Turn-by-turn navigation and detailed walking directions.
- Progressive shape completion / colour changes, heading display.
- Elevation information.
- Offline maps.
- Cycling as an additional travel mode.
- Full walk-history system and advanced post-walk analytics.
- Expansion to additional cities.

---

## 10. Technical constraints and non-functional requirements

- **Cost:** keep the entire system cost-conscious, with a strong preference for
  free tiers and open-source technologies where practical. Avoid unnecessary paid
  services.
- **Mapping:** prefer open-source / low-cost mapping technologies and
  OpenStreetMap-based data where practical. Respect required attribution and
  licensing obligations.
- **Architecture:** keep the application lightweight and avoid unnecessary
  complexity. Prefer a small number of well-integrated services that are easy to
  maintain.
- **Platform:** cross-platform mobile application supporting both iOS and Android.
  No web / PWA version for the foreseeable future.
- **Backend / cloud:** no specific cloud provider is required at this stage.
  Choose based on cost, simplicity, scalability, and suitability for GPS /
  location data.
- **Privacy:** user location data is private. The app must obtain appropriate
  location permission / consent and collect only the location data necessary for
  the application's functionality. Recorded walks belong to the user's account.
- **Performance:** GPS tracking should be reasonably battery-efficient and the
  application should remain lightweight. Avoid unnecessary background processing
  and data collection.
- **Maintainability:** assume a small / solo development team. The architecture,
  codebase, deployment, and infrastructure should be simple to understand,
  maintain, and operate.
- **Future flexibility:** the architecture should allow future additions such as
  cycling, offline maps, additional navigation capabilities, and on-demand route
  generation without requiring a complete rewrite.
- **CI/CD and deployment:** no specific provider is required yet. Use a
  straightforward development and deployment workflow appropriate for a small
  project.

---

## 11. Success criteria

The primary measure of success is whether the core concept works in the real
world:

- A user can create an account, select a shape in the pilot city, and clearly see
  the proposed route.
- The user can walk the route while the app tracks their live GPS position.
- The actual GPS path is recorded and saved to their account / history.
- The recorded path is sufficiently close to the intended route that the
  completed walk is clearly recognizable as the selected shape.
- The experience is simple enough that a normal smartphone user can understand the
  route and complete it without turn-by-turn navigation.
- Pilot users find the concept enjoyable and feel the resulting GPS artwork is
  worth sharing or repeating.

### Measurable targets (kept deliberately simple for the MVP)

- Start with one pilot city.
- Launch with a small curated set of recognizable shapes; the exact number is
  determined during implementation planning.
- Establish a practical route / shape-match threshold during route validation
  rather than requiring mathematical perfection.
- Test with real pilot users and measure whether they can successfully complete
  the routes and produce recognizable shapes.
- Track basic technical metrics — crashes, GPS tracking reliability, and battery
  usage — but do not impose strict numerical targets until real-world testing
  provides a baseline.

### Overall

MVP success = proving that the core loop works: select a shape → view the route →
walk it with GPS tracking → record the actual path → produce recognizable shape
artwork → save it to history.

---

## 12. Open questions (deferred during discovery)

- Pilot city not yet finalized.
- Exact number and category mix of shapes in the pilot catalog — decided during
  product planning.
- Specific third-party OpenStreetMap-based routing engine — selected later on
  route quality, cost, licensing, and API capabilities.
- Specific map rendering stack (MapLibre named as a preference, not a final
  decision).
- Cross-platform framework choice.
- Cloud / backend provider.
- CI/CD and deployment provider and workflow.
- The practical route / shape-match threshold value — set during route
  validation.
- Numerical targets for crash rate, GPS reliability, and battery usage — set
  after a real-world baseline exists.
