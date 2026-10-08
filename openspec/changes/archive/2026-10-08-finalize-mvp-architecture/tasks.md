## 1. Branch and tracking

- [x] 1.1 Move BAN-7 to In Progress via the Linear MCP, and verify its status reads "In Progress"
- [x] 1.2 Create branch `ban-7-finalize-mvp-architecture` off `main`, and verify with `git branch --show-current`

## 2. Record the architecture in the repository (documentation only)

- [x] 2.1 Create `docs/architecture.md` summarizing the seven decisions, the cost summary, "What would cause paying", integration boundaries, and deferred items from `design.md` (no new decisions), and verify every decision heading in `design.md` has a matching section
- [x] 2.2 Add a `context:` block to `openspec/config.yaml` describing the chosen stack and the $0 constraint (Expo/React Native + TypeScript with local builds, MapLibre + OpenFreeMap, Supabase Free Postgres/PostGIS in `ca-central-1`, Supabase Auth with Apple/Google sign-in, GitHub Actions on the public repo, no paid recurring services without an approved change) and the two pilot cities, and verify `openspec instructions proposal --change finalize-mvp-architecture --json` shows the new context
- [x] 2.3 Update `docs/product-requirements.md`: change the one-pilot-city wording to Montréal and Toronto, record the approved six-shape set (BAN-6), and mark the framework, map stack, backend, hosting, CI/CD, and city-selection open questions as resolved with a link to `docs/architecture.md`, leaving routing engine, shape-match threshold, numeric targets, and French-language as open. Verify with a diff that no other PRD scope changed
- [x] 2.4 Confirm no application code, dependencies, schemas, or infrastructure were added, by checking that `git diff --stat main` lists only files under `docs/` and `openspec/`

## 3. Validate the change

- [x] 3.1 Run `openspec validate finalize-mvp-architecture --strict` and verify it passes
- [x] 3.2 Record on BAN-7 (Linear comment) that this change is documentation-only and has no automated tests, as the deliberate exception required by `CLAUDE.md`, and verify the comment is posted

## 4. Linear updates (via the Linear MCP)

- [x] 4.1 Update BAN-5's open questions to mark the mobile framework, map stack, backend, hosting, and CI/CD as decided (linking to this change), and verify BAN-5 no longer lists them as open
- [x] 4.2 Update BAN-16's description to record that city selection is manual (decided in BAN-7), and verify the "City selection UX is not decided" text is replaced
- [x] 4.3 Update BAN-9's description to include local mobile builds (EAS optional), the daily Supabase keep-alive workflow, the weekly encrypted backup job with a documented restore procedure, confirming Free-plan `ca-central-1` region selection, and the public-repository secret rules, and verify each item appears in its scope or acceptance criteria
- [x] 4.4 Update BAN-12's description to replace email/password with native Sign in with Apple and Sign in with Google, add Apple token revocation on account deletion, and state that the MVP sends no email, and verify the new acceptance criteria are present
- [x] 4.5 Update BAN-19's description to use Firebase App Distribution for the Android pilot (Google Play deferred to public launch), keep TestFlight for iOS, require a free-tier monitoring tool, host the privacy policy on GitHub Pages, remove any paid Supabase plan assumption, and include the backup restore drill, and verify the updated scope and acceptance criteria are present
- [x] 4.6 Post a status-update comment on BAN-7 summarizing the decisions, the PR link, and the deferred items (BAN-8, BAN-10, BAN-11, BAN-12, BAN-17, BAN-19), and verify the comment is posted

## 5. Review, merge, and archive

- [x] 5.1 Open a PR to `main` that references BAN-7 and names the OpenSpec change `finalize-mvp-architecture`, move BAN-7 to In Review, and verify the PR shows the BAN-7 link
- [x] 5.2 After approval, squash-merge with a commit message containing `BAN-7`, and verify the squash commit on `main` carries the reference
- [x] 5.3 Run `/opsx:archive` for `finalize-mvp-architecture`, and verify `openspec/specs/platform-architecture/spec.md` and `openspec/specs/city-selection/spec.md` exist
- [x] 5.4 Move BAN-7 to Done only after the merge and the status update, and verify its status reads "Done"
