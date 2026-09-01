# CLAUDE.md

Guidance for Claude Code (and any AI agent) working in this repository.

## Project

**Banana** — a shape-based walking app that generates walkable routes matching a
user-selected shape. Indicative component layout (currently empty scaffolding;
the structure is flexible and may evolve as specs are written):

| Directory       | Purpose                                             |
| --------------- | -------------------------------------------------- |
| `frontend/`     | User-facing app (shape input, map, route display)  |
| `backend/`      | API, persistence, auth                             |
| `routing/`      | Walkable-route generation and path search          |
| `shape-engine/` | Shape definition, matching, and scoring            |
| `docs/`         | Project documentation                              |
| `tests/`        | Cross-cutting / integration tests                  |

Directories may be added, removed, renamed, or restructured when an OpenSpec
change calls for it. No application code exists yet. Do not create application
code until a change has gone through the workflow below **and** the user has
explicitly approved the OpenSpec implementation plan.

## Systems of record

Three systems govern this project. Each is authoritative for its domain; do not
duplicate that authority elsewhere.

### Linear — project-management source of truth

- Workspace: `banana` (https://linear.app/bananarun) · Team: **Banana** (`BAN`)
- Linear is the **only** authoritative place for what work exists, its priority,
  its status, and who owns it. Markdown TODO lists, code comments, and OpenSpec
  files are not substitutes for a Linear issue.
- **All project-management actions must go through the Linear MCP integration** —
  creating and updating issues, changing status, posting status-update comments,
  managing projects and labels. Do not track or mutate project state by any other
  means (manual notes, local files, ad-hoc scripts).
- Every unit of tracked work is a Linear issue with identifier `BAN-<n>`.
- Workflow states: `Backlog` -> `Todo` -> `In Progress` -> `In Review` -> `Done`
  (plus `Canceled`, `Duplicate`).
- Issue labels: `Feature`, `Bug`, `Improvement`.
- Project: **Shape-Based Walking App**. New issues belong to a project.

### OpenSpec — required specification workflow

- OpenSpec (`@fission-ai/openspec`, schema `spec-driven`) is the required path
  for specifying any significant change **before** implementation.
- Config: `openspec/config.yaml`. Commands live in `.claude/commands/opsx/`;
  skills in `.claude/skills/openspec-*/`.
- Artifact flow: `proposal` -> `specs` -> `design` -> `tasks`.
- Slash commands:
  - `/opsx:explore` — investigate options before committing to a proposal
  - `/opsx:propose "<idea>"` — create a change and its planning artifacts
  - `/opsx:apply` — implement an approved change's tasks
  - `/opsx:archive` — fold a completed change into the permanent specs
- Planning artifacts authorize **planning only**. Stop after they are produced
  and wait for explicit approval before running `/opsx:apply`.
- **Every OpenSpec change must be linked to a Linear issue.** Record the
  `BAN-<n>` identifier in the change's `proposal.md`, and record the OpenSpec
  change name/identifier on the Linear issue (description or a comment). Neither
  a change without a Linear issue nor a significant Linear issue without a change
  should proceed to implementation.

### GitHub — source-control system

- Git/GitHub is authoritative for code, history, and review.
- `main` is the integration branch; it must stay releasable. No direct pushes to
  `main` — all changes land via pull request.
- Branch naming: `ban-<issue-number>-<short-slug>` (e.g.
  `ban-12-shape-matcher`). This lets Linear auto-link the branch to the issue.
- Reference the Linear issue in every PR description and in the merge commit
  (e.g. `BAN-12`) so Linear and GitHub stay cross-linked.
- **Merge via squash** only; the squash commit message must carry the `BAN-<n>`
  reference.
- Conventional-commit style for messages (`feat:`, `fix:`, `test:`, `docs:`,
  `refactor:`, `chore:`).
- A remote is not yet configured; add `origin` before the first PR.

## What counts as a "significant" change

Use the full workflow when a change is any of:

- A new feature, capability, or user-visible behavior
- A new service, module, package, or public API/interface
- A change to an existing contract (API shape, schema, route format, shape-engine
  scoring) that affects other components or stored data
- A dependency addition or a cross-cutting architectural change

**Not** significant (Linear issue still required, OpenSpec optional): typo and
comment fixes, formatting, small localized bug fixes with no contract change,
test-only additions for existing behavior, doc edits.

If in doubt, treat it as significant.

## Workflow

### Before implementation — every significant change

1. **Linear issue.** Confirm a `BAN-<n>` issue exists (create one via the Linear
   MCP if not) in the Shape-Based Walking App project, with a clear title,
   description, label, and priority. This issue is the anchor for everything that
   follows.
2. **OpenSpec change.** Run `/opsx:propose "<what and why>"` and produce the
   planning artifacts (`proposal`, `specs`, `design`, `tasks`). Put the
   `BAN-<n>` identifier in `proposal.md`, and add the OpenSpec change
   name/identifier back onto the Linear issue.
3. **Implementation gate.** Present the Linear issue and the OpenSpec artifacts
   to the user. **Do not write application code, and do not run `/opsx:apply`,
   until the user has explicitly approved the OpenSpec implementation plan
   (`tasks.md`).** Once approved, set the Linear issue to `Todo`.

### During implementation

4. Move the Linear issue to **In Progress** (via the Linear MCP).
5. Create a branch `ban-<n>-<slug>` off `main`.
6. Implement strictly against the approved OpenSpec `tasks.md` (run
   `/opsx:apply`). If the plan must change, update the OpenSpec artifacts and
   note it on the Linear issue, then get the user's approval again before
   diverging.

### After implementation — required

7. **Tests.** Add or update automated tests covering the new behavior. All tests
   must pass locally. No change is "done" without tests; a deliberate exception
   must be recorded on the Linear issue with a reason.
8. **Linear status update.** Post a comment on the `BAN-<n>` issue (via the
   Linear MCP) summarizing what changed, test results, and any follow-ups. Move
   the issue to **In Review** when the PR is open, and to **Done** only after
   merge.
9. **Pull request.** Open a PR to `main` that links `BAN-<n>` and names the
   OpenSpec change it implements. Merge via squash with the issue reference in
   the commit message.
10. **Archive the spec.** After merge, run `/opsx:archive` so the change folds
    into the permanent OpenSpec specs.

## Guardrails

- Never start implementation from a bare request — route it through Linear +
  OpenSpec first.
- Never perform project-management actions outside the Linear MCP integration.
- Never run `/opsx:apply` or write application code before the user has
  explicitly approved the OpenSpec implementation plan.
- Every OpenSpec change identifier must be linked to a Linear issue, and vice
  versa, before implementation.
- Never mark a Linear issue `Done`, or claim work is complete, without passing
  tests and a status update on the issue.
- Never push to `main` directly, and never merge a PR that lacks a Linear
  reference or uses a non-squash merge.
- Keep the three systems consistent: if scope changes, update the Linear issue
  and the OpenSpec artifacts in the same session as the code.
- Report test failures and skipped steps plainly; do not describe unverified
  work as finished.
