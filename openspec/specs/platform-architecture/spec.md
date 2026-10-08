# platform-architecture Specification

## Purpose
Defines the cross-cutting platform requirements that every Banana MVP component must meet: zero recurring infrastructure cost, one mobile codebase for iOS and Android, open map rendering with attribution, a geospatial system of record, sign-in without email delivery, per-user data isolation and deletion, Canadian hosting, project-controlled backups, travel-mode-agnostic route data, and automated checks on every change.

## Requirements

### Requirement: Zero recurring infrastructure cost for the MVP
The MVP SHALL be developable, testable, and operable for a small pilot without any paid recurring infrastructure service. Every hosted service it depends on SHALL be used within a free tier, and adopting a paid recurring service SHALL require an approved OpenSpec change that states the cost and why no free alternative is suitable. App-store developer program fees are not infrastructure and are outside this requirement.

#### Scenario: Monthly infrastructure bill during the pilot
- **WHEN** the MVP is running for pilot users in Montréal and Toronto
- **THEN** the recurring monthly charge for hosting, database, authentication, map tiles, CI, and builds is $0

#### Scenario: A free-tier limit is reached
- **WHEN** usage exceeds a free-tier limit or a free service becomes unavailable
- **THEN** the system does not start incurring charges automatically, and moving to a paid tier happens only through an approved OpenSpec change

### Requirement: Single cross-platform mobile codebase
The MVP mobile app SHALL be built from one shared codebase that produces both the iOS and the Android app. The MVP SHALL NOT ship a web or PWA client.

#### Scenario: One change ships to both platforms
- **WHEN** a user-facing feature is merged to `main`
- **THEN** the iOS and Android builds both include it, without a separate platform-specific implementation of the feature logic

#### Scenario: No web client
- **WHEN** the MVP is released
- **THEN** it is available only as iOS and Android apps

### Requirement: Builds without a paid build service
iOS and Android binaries for development and the pilot SHALL be producible on the developer's own machine without a paid build service.

#### Scenario: Build quota exhausted
- **WHEN** a hosted build service's free monthly quota is used up
- **THEN** a local build still produces installable iOS and Android binaries

### Requirement: Open map rendering with visible attribution
Every map shown in the app SHALL be rendered from OpenStreetMap-based data with an open-source map renderer, and SHALL display the attribution required by the map data and tile licences (including OpenStreetMap contributors) whenever the map is visible.

#### Scenario: Attribution on every map
- **WHEN** any screen displays a map (route preview, live walk, walk history)
- **THEN** the required map-data and tile attribution is visible or reachable from a visible control on that map

### Requirement: Replaceable map tile source
The map tile and style source SHALL be configurable, so the app can switch tile providers (for example from a public tile service to self-hosted tiles) without changing map features or screens.

#### Scenario: Switching tile provider
- **WHEN** the project moves to a different OpenStreetMap-based tile source
- **THEN** only the configured style or tile endpoint changes, and route preview and live-walk map features keep working unchanged

### Requirement: Geospatial system of record
Shapes, routes, recorded walk paths, and user accounts SHALL be stored in a single relational system of record that supports geospatial types and spatial queries (for example distance, containment, and bounding-box queries).

#### Scenario: Spatial query on routes
- **WHEN** a component needs routes whose geometry falls within a city area
- **THEN** it can obtain them with a spatial query against the system of record, without exporting data to another store

### Requirement: Sign-in without email delivery
Users SHALL create an account and sign in through a platform identity provider: Sign in with Apple on iOS, and Sign in with Google on iOS and Android. The MVP's sign-up, sign-in, and account flows SHALL NOT depend on the system sending email.

#### Scenario: New user on Android
- **WHEN** a new user on Android chooses Sign in with Google and approves
- **THEN** an account is created and the user is signed in, without any email being sent by the system

#### Scenario: New user on iOS
- **WHEN** a new user on iOS chooses Sign in with Apple, including with a hidden (relay) email address
- **THEN** an account is created and the user is signed in

### Requirement: Per-user data isolation
A signed-in user SHALL be able to read and change only their own personal data (account details and recorded walks). Access rules SHALL be enforced on the server side, not only in the mobile app.

#### Scenario: Reading another user's walk is refused
- **WHEN** a signed-in user requests a recorded walk that belongs to a different user
- **THEN** the request returns no data or an authorization error

#### Scenario: Unauthenticated access to personal data is refused
- **WHEN** a request without a valid session asks for any user's personal data
- **THEN** the request is refused

### Requirement: Privileged credentials stay server-side
Credentials that bypass per-user access rules (such as database service-role keys, identity-provider private keys, and backup encryption keys) SHALL NOT be included in the mobile app, in any client-side code, or in the public source repository.

#### Scenario: Mobile build contains no privileged credentials
- **WHEN** a mobile app build is produced
- **THEN** it contains only public, access-rule-restricted client credentials

#### Scenario: Public repository contains no secrets
- **WHEN** the public repository is inspected
- **THEN** it contains no privileged credentials; CI uses encrypted repository secrets instead

### Requirement: Complete account deletion
When a user deletes their account, the system SHALL remove the user's authentication identity and all personal data tied to it, including recorded walks and GPS paths, and SHALL revoke the user's Sign in with Apple authorization where one exists. Deletion SHALL be performed on the server side.

#### Scenario: Deleted account cannot sign in to the same account
- **WHEN** a user deletes their account and then signs in again with the same Apple or Google identity
- **THEN** they get a new, empty account, and none of their previous walks are visible

#### Scenario: Deleted account leaves no personal data
- **WHEN** a user deletes their account
- **THEN** none of that user's recorded walks, GPS paths, or account details remain in the system of record

#### Scenario: Apple authorization revoked
- **WHEN** a user who signed in with Apple deletes their account
- **THEN** the app's Sign in with Apple authorization for that user is revoked

### Requirement: Canadian hosting region
Personal data (accounts and recorded walks, including GPS paths) SHALL be stored in a Canadian data-centre region for the MVP.

#### Scenario: Production data location
- **WHEN** the production backend is provisioned
- **THEN** its database is located in a Canadian region

### Requirement: Project-controlled encrypted backups
Production personal data SHALL be backed up automatically at least once a week to storage controlled by the project, independent of the hosting provider. Backups SHALL be encrypted, SHALL NOT be stored in the public repository or its CI artifacts, and a restore SHALL be tested before pilot users are invited.

#### Scenario: Weekly backup exists
- **WHEN** the pilot has been running for a week
- **THEN** at least one encrypted backup of the production database from that week exists outside the hosting provider

#### Scenario: Restore drill
- **WHEN** the restore procedure is run against a backup before the pilot
- **THEN** a working database with the backed-up data is produced in a non-production environment

### Requirement: Backend stays available during pilot quiet periods
The production backend SHALL remain reachable for pilot users even when no pilot user has used the app for several days.

#### Scenario: Quiet week
- **WHEN** no pilot user opens the app for seven days
- **THEN** the next user to open the app can still browse shapes and sign in without manual intervention by the team

### Requirement: Travel-mode-agnostic route data
Route data SHALL record the travel mode it was generated for, and components SHALL NOT assume walking is the only possible mode. The MVP SHALL generate and serve walking routes only.

#### Scenario: MVP routes are walking routes
- **WHEN** an MVP route is stored
- **THEN** it is identified as a walking route

#### Scenario: Adding a travel mode later
- **WHEN** cycling routes are added in a later release
- **THEN** existing walking routes and their records remain valid without migration of their meaning

### Requirement: Automated checks on every pull request
Every pull request to `main` SHALL run automated checks (at least type checking, linting, and the automated test suite for the parts of the system it touches), and a pull request with failing checks SHALL NOT be merged.

#### Scenario: Failing check blocks merge
- **WHEN** a pull request's automated checks fail
- **THEN** the pull request cannot be merged into `main` until the checks pass
