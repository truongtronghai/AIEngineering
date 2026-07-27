# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Frontend for **Dela Security** ("Dela IEM" / "CDNA frontend"), an identity threat detection & response
security product. React + TypeScript + Vite SPA. Users authenticate via AWS Cognito and view/manage
identity hygiene, threat detection, attack-path analysis, M365/Entra ID hygiene, disaster recovery, and
reporting for on-prem Active Directory and Azure/Entra tenants.

## Commands

Package manager is **yarn** (pinned via `packageManager` in package.json — don't use npm to install).

```bash
yarn install

yarn dev            # local dev server on :3000, loads .env.dev
yarn dev:vinh       # same, but loads .env.dev-vinh (per-developer env file)
yarn dev:duy        # loads .env.dev-duy
yarn dev:sky        # loads .env.dev-sky
yarn dev:prod       # loads .env.prod

yarn build:stg      # tsc -b && vite build against .env.stg
yarn build:prod     # tsc -b && vite build (uses ambient/CI env vars, no env-cmd)
yarn preview        # preview a production build locally

yarn lint           # eslint . --cache --max-warnings=0
yarn type-check     # tsc --noEmit
```

Git hooks (husky): `pre-commit` runs `lint-staged` (eslint on staged ts/tsx/js/jsx), `pre-push` runs
`yarn type-check`. Both must pass for a normal commit/push flow.

### Environment

Each `yarn dev*` script points at a different `.env.dev-*` file (per-developer AWS Cognito / API
endpoints) via `env-cmd`. See `.env.sample` for the required variables: `VITE_APP_NAME`, `VITE_APP_URL`,
`VITE_API_URL`, `VITE_API_WS_URL`, `VITE_USER_POOL_ID`, `VITE_CLIENT_ID`. CI (`.github/workflows/main.yml`)
injects these per-branch (`dev`/`staging`/`main`) from GitHub vars/secrets, builds with `yarn build:prod`,
and syncs the `build/` output to a per-environment S3 bucket.

## Architecture

### Path alias

`@/*` maps to `src/*` (configured in both `vite.config.ts` and `tsconfig.app.json`). Existing code mixes
`@/...` and relative imports (`../../`) inconsistently — prefer `@/...` for new code but don't rewrite
existing imports as a side effect of unrelated changes.

### App bootstrap and provider stack

`src/main.tsx` → `ThemeProvider` → `src/AppProvider.tsx` (the actual `App` component, despite the
filename). Provider nesting order in `AppProvider.tsx` matters:

```
ConfigProvider (antd theme, built from CustomTheme)
  └─ AuthProvider (Cognito session/token state)
      └─ WebSocketProvider (live WS connection, agent status etc.)
          └─ NotificationProvider (antd message/notification wrapper)
              └─ QueryClientProvider (@tanstack/react-query)
                  └─ RouterProvider (react-router routes from core/routes/routes.tsx)
```

### Routing & authorization

Routes are a flat array of `RouteObject` in `src/core/routes/routes.tsx`, each wrapped in either
`ProtectedLayout`, `UnprotectedLayout`, or `StaticPage` layout. Heavier feature pages are
`React.lazy`-loaded and wrapped individually in `<Suspense>` at the route level (not centrally).

Authorization is role-based, not JWT-scope-based: `src/core/config/authorization.ts` defines four roles
(`ROLE_KEYS`: `security_analyst`, `security_manager`, `security_engineer`, `trial_user`), each mapped to a
flat list of permission strings, including `route:<pathSegment>` entries that gate whole pages. The role
comes out of the decoded Cognito access token (`getRole()` in `src/core/utils/utils.ts`). `ProtectedLayout`
(`src/core/layout/Protected/Layout.tsx`) checks `checkPermission(role, "route:" + firstPathSegment)` on
every navigation and redirects to `/panel` if not permitted, redirects to `/login` if not authenticated,
and also runs an idle-timeout watcher (`config.SESSION_EXPIRATION_TIME`, 1hr) that force-signs-out on
inactivity. When adding a new protected route, add both the route entry and a matching `route:<segment>`
permission per role in `authorization.ts` — otherwise the layout will bounce users back to `/panel`.

### Auth

Cognito auth via `aws-amplify/auth`, configured in `src/core/config/awsAmplify.ts`. `AuthProvider`
(`src/pages/auth/context/AuthProvider.tsx`) exposes `authInfo` (token/email/etc.) through
`useAuthContext()`. Access tokens are read from Cognito's local storage via helpers in
`src/core/utils/awsAmplifyStorage.ts` (`getToken`, `getLoginId`). There's an explicit
`AUTH_STATE.LOADING` sentinel — children of `AuthProvider` don't render until the initial token load
resolves. `handleRefreshToken()` force-refreshes the Cognito session and is called both proactively (on
layout mount) and reactively (mutation hooks call it before/after requests when the API reports an
expired token — see below).

### Data fetching pattern

No axios/central API client — mutations and queries use the native `fetch` API directly against
`config.API_URL` (see `src/core/config/config.ts`), wrapped in `@tanstack/react-query`'s `useMutation` /
`useQuery`. The established pattern (e.g. `src/core/hooks/useAgentMutation.ts`) is:

-   Manually attach `Authorization: Bearer ${authInfo.token}`.
-   Call `handleRefreshToken()` before the request (or detect an expired-token error in the response body
    and retry once after refreshing).
-   Inspect `data.detail` on success: a string containing `"Token expired"` triggers a refresh + retry; a
    string otherwise, or an array (FastAPI-style validation errors, `data.detail[0].msg`), is shown via
    `message.error(...)` from `useNotification()`. This implies the backend is a Python/FastAPI-style API
    returning `{ detail: ... }` error bodies.
-   Report success/failure via the `useNotification()` hook (antd message wrapper), not local UI state.

New data-fetching hooks should follow this same shape rather than introducing a new HTTP client or error
convention.

### Feature module structure

Each top-level feature lives under `src/pages/<feature>/` and follows a consistent internal layout:

```
src/pages/<feature>/
  <Feature>.tsx           # page entry component
  components/             # feature-local UI components
  hooks/                  # feature-local hooks (data fetching, derived state)
  services/                # feature-local fetch/service functions
  utils/                   # feature-local helpers
  NestedPages/             # sub-routes/sub-views specific to this feature
```

`src/core/` holds cross-feature shared code: `core/ui/` (generic components — charts, modals, badges),
`core/hooks/`, `core/utils/`, `core/types/`, `core/config/`, `core/context/` + `core/provider/` (app-wide
contexts: theme, notifications, websocket), and `core/layout/` (the three layout shells: `Protected`,
`Unprotected`, `StaticPage`).

Some pages have multiple parallel versions of the same view (e.g. `dashboard/DashboardV2.tsx`,
`DashboardV3.tsx`, `dashboard.tsx`) — check `routes.tsx` to see which one is actually wired up (currently
`DashboardV3`) before editing; older versions may be dead code kept for reference rather than in use.

### Styling

TailwindCSS is primary (`tailwind.config.js`), supplemented by CSS Modules for a handful of complex
feature-specific components (`src/core/css/*.module.css`). Ant Design (`antd`) supplies most interactive
components (tables, forms, modals, dropdowns); its theme tokens are set centrally in `AppProvider.tsx`
via `CustomTheme` (from `useThemeContext`) rather than per-component — prefer extending the central
`antTheme` object over inline overrides when a design change should apply app-wide.

### Real-time updates

`WebSocketProvider`/`WebSocketContext` (`src/core/provider/`, `src/core/context/WebSocketContext.tsx`)
maintain a single WS connection (`config.API_WS_URL`) exposed via `useWebsocketContext()`, used for things
like live agent-online status and disaster-recovery progress push updates rather than polling.

## Important

-   Let use meaningful words for variable names, function names, css class names instead of abbreviation.
-   Using Chart.js or echarts.apache.org for charts and gauge.
-   Using react-hook-form for form with zod.
-   Using yup for validate schema.
-   Using type instead of interface for declaration of types.
-   Just using React.memo for complex components such as: table, data grid, large chart, markdown render, map, code editor . Not using React.memo for simple components.
