# Prompt: HTML system documentation for a legacy Django + AngularJS repo

Inputs: `REPO_PATH` = {{REPO_PATH}} · `OUTPUT_DIR` = {{OUTPUT_DIR}} (outside the repo) · `PROJECT_NAME` = {{PROJECT_NAME}}

## Goal
You are a software archaeologist. Analyze the full repo at `REPO_PATH` (Django backend, AngularJS 1.x frontend, many customizations) and produce ONE self-contained, offline HTML file, `OUTPUT_DIR/PROJECT_NAME_documentation.html`, that is the single source of truth for how the system works. It must cover screens and user inputs, every frontend and backend layer, database reads and writes, background jobs, and external systems. It must read from simple to deep: newcomer, then developer, then architect.

## Hard rules
1. **Read-only.**
   - Don't create, edit, or delete anything in the repo.
   - Don't run `manage.py`, tests, builds, or installs, and don't import project code.
   - Don't connect to databases or call external URLs.
   - Use only read-only git commands.
   - Put tools and scratch files in `OUTPUT_DIR/_work`.
   - Capture `git status --porcelain` at the start and at the end; the two must match.
2. **Evidence only.**
   - Every statement cites `path:Lstart-Lend`. If you can't cite it, don't write it.
   - Don't guess from names or describe "typical Django behavior".
   - Don't invent data, examples, hostnames, counts, personas, or mockups.
   - Use exact identifiers and UI strings from the code.
   - Unwired or commented-out code is not active behavior.
3. **Label every claim** as one of:
   - VERIFIED: directly visible at the cited lines.
   - FRAMEWORK: library behavior switched on by cited config; give the library version.
   - INFERRED: show the reasoning and all citations; never use it for business intent.
   - UNRESOLVED: can't be known statically; list every candidate value.
4. **Nothing new.** No recommendations, improvements, ratings, best-practice comments, or TODOs.
5. **Empty sections** say "Not present in this codebase", followed by the searches you ran.
6. **Redact secrets** (keep their names and locations). Never copy personal data from fixtures.
7. **Code beats README, comments, and tests.** Record contradictions under Anomalies.

## Method
Work in phases. Store every fact with its citation as JSON in `_work/kb/`, and keep `_work/PROGRESS.md` current so you can resume. Build the HTML from the knowledge base with a script, not from memory. Parse both Python 2 and Python 3, and handle every AngularJS dependency-injection syntax.

1. **Census.** Record the commit hash, classify every file, and find versions (with their sources), Django apps, AngularJS modules, settings modules, and entry points. **Stop and report.**
2. **Backend.**
   - Settings and environment variables.
   - The full URL tree, including includes and DRF routers.
   - Views: decorators, inputs, validation, permissions, ORM operations, raw SQL, transactions, side effects, responses.
   - Forms, serializers, models, signals, middleware.
   - Templates, template tags, context processors.
   - Admin, management commands, authentication.
3. **Frontend.**
   - How Django serves AngularJS: interpolation symbols, injected globals, build pipeline.
   - Modules, routes/states and their guards, `$http` interceptors.
   - Controllers, services and `$resource`, directives, filters, events, browser storage.
   - Templates: every field, validation, button, and visibility condition.
   - jQuery and other non-Angular code, and server-rendered forms.
4. **Stitch.**
   - Map every frontend HTTP call to its URL pattern and view, and mark it MATCHED, AMBIGUOUS, or UNMATCHED.
   - List endpoints with no consumer in the repo.
   - Compare payload fields in both directions.
5. **Workflows.**
   - Enumerate every entry point: UI actions, form posts, APIs, admin actions, management commands, cron/Celery, tasks, signals, webhooks, middleware, startup code, authentication.
   - Group entry points into workflows and justify each grouping.
   - Trace every branch, validation, database operation, side effect, transaction, and error path.
   - Turn status fields into state machines.
   - **Stop and show the workflow index.**
6. **Data.**
   - Every model and table, including unmanaged ones and built-in tables the code uses.
   - Columns, keys, constraints, indexes, choices, M2M tables.
   - Migrations and data migrations, raw SQL, a CRUD matrix, caching, file storage.
   - Never query a database.
7. **Integrations.**
   - Cover everything outbound and inbound: HTTP, SOAP, SDKs, email, SFTP, queues, LDAP/SSO, third-party JS, webhooks, shared tables.
   - For each: call sites, triggers, data sent and received, auth type, timeouts, retries, error handling.
8. **Async jobs and customizations.**
   - Celery tasks and beat schedules, cron, management commands.
   - Monkey patches, `$provide.decorator`, vendored or edited libraries, overridden templates, custom backends, fields, and middleware.

**Legacy traps to search for explicitly:**
- Old Django idioms: `MIDDLEWARE_CLASSES`, `patterns()`, South migrations.
- JSON returned through `HttpResponse` instead of a framework.
- Logic hidden in `save()`, signals, template tags, or context processors.
- Thread-local "current user" helpers, generic foreign keys, multiple databases, `csrf_exempt` views.
- `$rootScope` and `$broadcast` event patterns, URLs built from strings.
- Pages that mix AngularJS and server rendering, duplicate registrations.

## Document sections
- **0. Start here:** commit stamp, legend, and a guided tour of one workflow.
- **1. System at a glance**
- **2. Actors and roles**
- **3. Tech stack**
- **4. Architecture:** context, runtime, layers, dependency graphs.
- **5. Request lifecycle** (animated).
- **6. Repo map**
- **7. Screens:** a navigation map, plus a page per screen with route, access, files, loaded data, field table, action table, and conditions.
- **8. Workflows:** an index, plus a page per workflow with summary, user, frontend, server, database, and what happens afterwards; a step player, flowchart, rules, transactions, and error paths.
- **9. Modules**
- **10. Endpoint catalog**
- **11. Data model:** ER diagrams, table pages, CRUD matrix, raw SQL, migrations, state diagrams.
- **12. Business rules**
- **13. Security and access:** authentication flows and a role × action matrix.
- **14. Background jobs and signals**
- **15. Integrations**
- **16. Frontend architecture**
- **17. Cross-cutting:** logging, errors, cache, emails, files, reports.
- **18. Customizations**
- **19. Config and environment variables**
- **20. Build and deploy** (only what is in the repo).
- **21. Traceability matrix and indexes**
- **22. Anomalies** (facts only).
- **23. Coverage and the full UNRESOLVED list**

## Diagrams and animations
- **Diagrams:**
  - Use Mermaid (pinned version, pre-rendered to inline SVG where possible) for flowcharts, sequences, ER, state, and dependency graphs. Keep each diagram's source in a collapsible block.
  - Every node and edge must be backed by evidence. Keep each diagram to 25 nodes or fewer and split larger ones.
  - Use consistent layer colors (frontend, backend, data, async, external) with a legend.
  - Nodes are clickable, diagrams support zoom and pan, and each has a text equivalent below it.
- **Animations** exist only to show the order of flow:
  - An animated request lifecycle.
  - A step player per workflow (play, pause, step, branch selector, evidence per step), built from the same knowledge-base step list as the diagrams.
  - Flowing edges, and hover highlighting in graphs.
  - Step order must match the code exactly.
  - Provide a motion toggle and respect `prefers-reduced-motion`.

## HTML
- **File:** a single file with everything inlined that works offline from `file://`.
- **Navigation:**
  - Sidebar tree navigation.
  - Search (focus with `/`), and a light/dark theme.
  - A depth switch: Overview, Functional, Technical, Code.
  - Stable anchors, cross-links, and backlinks.
- **Evidence:** citation chips that expand to show the embedded source lines.
- **Tables and performance:** sortable and filterable tables, and lazy rendering.
- **Accessibility and print:** keyboard accessible, with a print stylesheet.
- **Design:** a calm, legible reference design; use color only where it carries meaning.
- **Stamp:** "Generated from commit X on DATE; describes source code only."

## Verify before delivery
Write scripts that confirm each of these, and report the results in the Coverage section:
- Every identifier, path, table, and URL in the HTML exists in the repo.
- Every citation's lines exist and contain the thing the claim is about.
- Every inventoried entity has an anchor, and every file is either documented or listed as excluded.
- Diagrams render and match the knowledge base, and step players follow the knowledge base's step order.
- No forbidden words (should, recommend, improve, probably, likely, typically, TODO) appear outside quoted code or INFERRED/UNRESOLVED blocks.
- No secret values appear anywhere.
- The file opens offline with no console errors.
- `git status` is unchanged.

Start with step 1.