<!--
Sync Impact Report
==================
Version change: [unversioned template] → 1.0.0
Bump rationale: Initial ratification — first concrete population of the
constitution template with project-specific principles, constraints, and
governance for the RSS Feed Reader POC.

Modified principles (template placeholder → concrete name):
- [PRINCIPLE_1_NAME] → I. MVP-First Simplicity (YAGNI)
- [PRINCIPLE_2_NAME] → II. Code Quality & Maintainability
- [PRINCIPLE_3_NAME] → III. Security by Default
- [PRINCIPLE_4_NAME] → IV. Verified Behavior (Manual MVP, Automated as it Grows)
- [PRINCIPLE_5_NAME] → V. Incremental, Future-Ready Architecture

Added sections:
- Technology & Architecture Constraints (replaces [SECTION_2_NAME])
- Development Workflow & Quality Gates (replaces [SECTION_3_NAME])
- Governance (concrete amendment, versioning, and compliance rules)

Removed sections: none

Templates requiring updates:
- .specify/templates/plan-template.md ⚠ pending — verify Constitution Check
  gate names match the five principles above
- .specify/templates/spec-template.md ⚠ pending — confirm scope statement
  references MVP-first boundaries
- .specify/templates/tasks-template.md ⚠ pending — ensure task categories
  cover security review, manual verification checklist, and refactor/cleanup
- .specify/templates/checklist-template.md ⚠ pending — align quality
  checklist headings with principles II, III, IV
- .github/copilot-instructions.md ⚠ pending — ensure agent guidance points
  to this constitution

Follow-up TODOs: none — RATIFICATION_DATE set to today (initial adoption).
-->

# RSS Feed Reader Constitution

## Core Principles

### I. MVP-First Simplicity (YAGNI)

The project MUST deliver the smallest end-to-end vertical slice before any
additional feature is started. For this codebase the MVP scope is fixed:
add a subscription by URL and display the list of subscriptions, using
in-memory storage, with no feed fetching, parsing, validation, or
persistence. Features outside this scope MUST NOT be implemented until the
MVP is demonstrably working end-to-end. Extended-MVP and post-MVP features
are deferred and MUST be tracked separately. Rationale: scope creep is the
primary risk to a proof-of-concept; explicit boundaries keep delivery fast
and reviewable.

### II. Code Quality & Maintainability

All code MUST follow clean separation of concerns between the ASP.NET Core
backend (data and, later, feed operations) and the Blazor WebAssembly
frontend (UI and user interaction). The following are NON-NEGOTIABLE:

- No hardcoded environment-specific values (URLs, ports, origins). They
  MUST be read from configuration files (`appsettings.json`,
  `launchSettings.json`).
- Public types, methods, and components MUST have intention-revealing
  names; dead code, unused template demo pages, and commented-out code
  MUST be removed before merge.
- Each Razor page MUST have a unique `@page` route; the template demo
  pages (`Home.razor`, `Counter.razor`, `Weather.razor`) MUST be deleted
  during foundational setup and the cleanup MUST be verified before any
  feature work begins.
- Backend and frontend MUST build with zero warnings under the default
  analyzers shipped with the .NET SDK in use.

Rationale: a clean, predictable layout keeps the POC reviewable and
prevents runtime defects (e.g., ambiguous routes) that are costly to
diagnose later.

### III. Security by Default

Even as a local proof-of-concept, the application MUST observe baseline
security practices:

- CORS on the backend MUST explicitly allow only the configured frontend
  origin(s); wildcard origins (`*`) are PROHIBITED.
- Secrets, API keys, tokens, and personal data MUST NOT be committed to
  the repository. Configuration that varies by environment MUST use the
  ASP.NET Core configuration system (not source-embedded constants).
- User-supplied input (notably subscription URLs) MUST be treated as
  untrusted. For Extended-MVP and beyond, any rendered feed content MUST
  be sanitized (e.g., via `HtmlSanitizer`) before display; raw HTML
  injection into the DOM is PROHIBITED.
- External HTTP calls (Extended-MVP onward) MUST use a configured
  `HttpClient` with explicit timeouts; indefinite waits are PROHIBITED.
- Dependency additions MUST come from trusted NuGet sources and MUST be
  justified in the implementation plan.

Rationale: security shortcuts in a POC routinely survive into production;
enforcing minimal hygiene now prevents predictable OWASP-class defects
(injection, misconfiguration, supply-chain risk).

### IV. Verified Behavior (Manual MVP, Automated as it Grows)

Every feature MUST be verifiable before it is considered done.

- For the MVP, verification MUST follow the local development checklist
  from `StakeholderDocuments/ProjectGoals.md` (backend up, frontend up,
  configured base URL, CORS allows origin, no console errors) plus a
  manual scenario: add a subscription URL and confirm it appears in the
  list.
- For Extended-MVP and beyond, any new backend behavior MUST be covered
  by automated tests (xUnit). New integration points (HTTP endpoints,
  feed parsing, persistence) MUST have at least one integration or
  contract test before merge.
- Defects discovered during verification MUST be fixed or explicitly
  documented as known limitations in the spec before the feature is
  marked complete.

Rationale: the project will outgrow purely manual testing; introducing a
test discipline at the Extended-MVP boundary keeps regression cost low
without slowing the initial POC.

### V. Incremental, Future-Ready Architecture

Architectural choices MUST keep the path to production open without
demanding speculative work today.

- The split between `backend/` (ASP.NET Core Web API) and `frontend/`
  (Blazor WebAssembly) MUST be preserved; UI logic MUST NOT bypass the
  API to access storage directly.
- In-memory storage is the MVP default; the storage abstraction MUST be
  shaped so that swapping in a persistence layer (EF Core + SQLite) is a
  localized change, not a rewrite.
- Background work, persistence, polling, and HTML rendering MUST be
  introduced only when the corresponding Extended-MVP / post-MVP feature
  is scheduled — but interfaces MUST NOT actively block them.
- Breaking changes to the API contract MUST be reflected in the frontend
  in the same change set; the system MUST be in a working state at the
  end of every merge.

Rationale: incremental architecture protects delivery speed today while
preserving the technology-selection promise made in
`StakeholderDocuments/TechStack.md`.

## Technology & Architecture Constraints

The following constraints are binding for all contributions:

- **Runtime & frameworks**: ASP.NET Core Web API (backend) and Blazor
  WebAssembly (frontend), both targeting a cross-platform .NET SDK
  (Windows, macOS, Linux). Alternative stacks are out of scope.
- **MVP libraries**: no feed-parsing, no HTTP client on the backend, no
  database — in-memory `List`-backed storage only.
- **Extended-MVP libraries**: feed parsing MUST use
  `System.ServiceModel.Syndication`; fetching MUST use `HttpClient` with
  explicit timeouts; refresh MUST be manual (no background scheduler).
- **Configuration**: backend port, frontend port, API base URL, and CORS
  origins MUST be coordinated across `launchSettings.json` (backend and
  frontend), `wwwroot/appsettings.json` (frontend), and `Program.cs`
  (backend) per `StakeholderDocuments/TechStack.md`. Drift between these
  files is a blocking defect.
- **Project hygiene**: Blazor template demonstration pages MUST be
  removed before MVP feature work (per Principle II).
- **No premature features**: persistence, background polling, read/
  unread tracking, OPML, notifications, mobile clients, and any item in
  the "future enhancements" lists in
  `StakeholderDocuments/ProjectGoals.md` and
  `StakeholderDocuments/AppFeatures.md` MUST NOT be implemented until
  formally scheduled in a spec.

## Development Workflow & Quality Gates

All changes flow through the Spec Kit workflow
(`/speckit.specify` → `/speckit.plan` → `/speckit.tasks` →
`/speckit.implement`) and MUST pass the following gates before merge:

1. **Constitution Check**: the plan MUST explicitly confirm compliance
   with each of the five Core Principles, or document a justified
   exception.
2. **Scope Check**: the spec MUST identify whether the change is MVP,
   Extended-MVP, or post-MVP, and changes outside the currently active
   scope MUST be rejected.
3. **Build & Run**: backend and frontend MUST build with zero warnings
   and start cleanly on the configured ports.
4. **Verification**: the MVP manual checklist (Principle IV) MUST pass;
   Extended-MVP and beyond also require new/updated automated tests to
   pass.
5. **Security review**: any change touching CORS, configuration, user
   input handling, external HTTP, or rendered content MUST be reviewed
   against Principle III before merge.
6. **Cleanup**: no dead code, no unused template pages, no committed
   secrets, no TODO comments without a tracked follow-up.

Code review is mandatory; reviewers MUST cite the principle(s) any
requested change is based on.

## Governance

This constitution supersedes ad-hoc practices and informal conventions
for this repository. In any conflict between this document and other
guidance (READMEs, comments, prior plans), this document wins until
amended.

**Amendments** MUST be proposed via a change to
`.specify/memory/constitution.md` together with a Sync Impact Report at
the top of the file describing the version change, the affected
principles/sections, and any dependent templates that need updating.

**Versioning policy** (semantic):

- **MAJOR**: a principle is removed, renamed in a way that changes its
  meaning, or redefined to be backward incompatible with prior plans.
- **MINOR**: a new principle or section is added, or existing guidance
  is materially expanded.
- **PATCH**: clarifications, wording, typo fixes, or non-semantic
  refinements.

**Compliance review**: every `/speckit.plan` run MUST execute the
Constitution Check; every pull request MUST verify that the listed gates
in *Development Workflow & Quality Gates* are satisfied. Unjustified
deviations are a blocking review comment.

**Runtime agent guidance** lives in `.github/copilot-instructions.md`
and in the per-command prompts under `.github/prompts/`. Those files
MUST remain consistent with this constitution; when this document is
amended, they MUST be reviewed for drift in the same change set.

**Version**: 1.0.0 | **Ratified**: 2026-05-19 | **Last Amended**: 2026-05-19
