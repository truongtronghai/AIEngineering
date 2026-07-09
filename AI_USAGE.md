# AI Usage Notes

This project used AI as a coding and design-review collaborator. The final architecture, trade-offs, code review, and acceptance decisions were human-owned. I used AI to generate options, identify risks, draft implementation slices, and speed up repetitive code changes, but every shipped change was reviewed against the assignment requirements and the running project.

## Prompting Strategy

I worked in small, reviewable slices instead of asking for one large implementation. The main slices were quality gates, app shell, data layer, dashboard UI, filters, drawer actions, realtime simulation, keyboard navigation, charts, deployment, and documentation.

Prompts included the assignment constraints explicitly: strict TypeScript, TanStack React Query, React Hook Form + Yup, Ant Design, Tailwind CSS, no unnecessary major dependencies, lint with zero warnings, and meaningful commit history. I also used lightweight feature specs and engineering standards, now consolidated in `docs/`, as review checklists for AI output. When architecture trade-offs mattered, I asked for alternatives before choosing an approach.

## Accepted AI Contributions

- Quality-gate setup with lint, typecheck, build, and GitHub Actions.
- Feature decomposition into `pages/alerts` components, hooks, data helpers, realtime helpers, and utilities.
- Mock API shape, React Query query keys, and optimistic mutation patterns.
- UI polish iterations for loading, error, empty, table, drawer, and responsive states.
- Optional stretch implementations: mocked realtime emitter, keyboard row navigation, and trend charts.
- README and deployment documentation drafts.
- Consolidated feature-spec and engineering-standard documentation.

## Rejected Or Revised Suggestions

- I avoided overusing React Router Data Router for this small single-page assignment.
- I kept i18n lightweight instead of adding a full localization library.
- I revised dashboard layouts that made the table less usable or too compressed.
- I simplified or reverted visual ideas that made skeletons, detail panels, or chart areas too heavy.
- I avoided IndexedDB persistence late in the submission because it would add dependency and reconciliation complexity after the core feature was already complete.
- I split the optional chart and vendor output into explicit chunks instead of hiding Vite bundle warnings with arbitrary warning limits.

## Critical Review Process

Every generated change was checked against:

- assignment requirements
- strict TypeScript and ESLint rules
- React Query data ownership
- component boundaries and hook responsibilities
- table-first dashboard usability
- accessibility behavior
- local `yarn quality`

Specific review decisions included separating drawer-specific mutations from dashboard state, using a mocked event emitter rather than pretending to have a real WebSocket backend, keeping chart data derived from filtered alerts, and keeping mock alert content as backend-like data instead of translating it as UI copy.
The chart bundle split keeps source imports readable and uses a matching skeleton fallback so the lazy-loaded chart does not shift the dashboard after data loads.

## Ownership And Explainability

I can explain the shipped implementation choices in review, including:

- query key structure
- stale time and gc time choices
- optimistic mutation rollback
- typed i18n lookup
- filter derivation and active filter counting
- realtime cache insertion
- keyboard focus management for rows
- chart derivation from filtered alert data
- why some optional stretch goals were deferred

## Presentation Notes

- How I used AI: as a planning, implementation, review, and documentation assistant for small scoped changes.
- Where I challenged AI output: routing strategy, dashboard layout density, drawer action ownership, skeleton fidelity, chart scope, and dependency choices.
- What I would improve next: add focused tests around filtering/mutations, inspect deeper Ant Design dependency cost if this became a larger production app, and replace the mock API with a typed backend contract.
