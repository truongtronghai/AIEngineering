# React Application Scaffold Standards

## Stack

Unless the user specifies otherwise, use:

- React 19
- TypeScript
- Vite
- React Router v7
- Tailwind CSS v4
- ESLint
- Prettier
- Husky
- lint-staged
- Vitest
- React Testing Library
- MSW
- Playwright
- React Hook Form
- Zod
- TanStack Query
- clsx
- tailwind-merge
- ShadCn
- echarts ([text](https://echarts.apache.org/))

Use npm unless instructed otherwise.

---

# Folder Structure

```

src/
app/
components/
features/
hooks/
layouts/
lib/
providers/
routes/
services/
styles/
types/
utils/

public/

```

Avoid dumping everything into components/.

Group business logic inside feature folders.

---

# TypeScript

Enable strict mode.

Avoid:

- any
- unknown casts
- non-null assertions

Prefer:

- discriminated unions
- type inference
- utility types
- using "type" rather than "interface" for declaration of type

---

# React

Use:

- Functional components
- Hooks
- Composition
- Context only when appropriate

Avoid:

- Class components
- Prop drilling
- Massive components
- Nested ternaries

---

# Component Guidelines

Each component should:

- Have a single responsibility
- Be reusable
- Receive typed props
- Keep rendering logic simple

Prefer:

```

Component/
Component.tsx
Component.test.tsx
index.ts

```

---

# Styling

Use Tailwind CSS.

Avoid inline styles unless dynamic.

Use clsx + tailwind-merge for conditional classes.

Never use !important.

Implement keyboard focus using:

- outline
- outline-offset

Never implement focus rings using box-shadow.

---

# State Management

Use:

- local state for UI
- Context for application state
- TanStack Query for server state

Do not introduce Redux unless requested.

---

# Forms

Always use:

- React Hook Form
- Zod validation

Do not use uncontrolled forms.

---

# API Layer

Separate API logic from UI.

```

services/
api/
hooks/

```

Components should never call fetch() directly.

---

# Routing

Use React Router v7.

Keep routes centralized.

Support lazy loading for pages.

---

# Error Handling

Provide:

- Error Boundary
- Not Found page
- Loading UI

Never leave blank screens.

---

# Accessibility

Every component should support:

- keyboard navigation
- screen readers
- visible focus
- semantic HTML

Buttons must use `<button>`.

Links must use `<a>` or `<Link>`.

---

# Testing

Create:

- Component tests
- Hook tests
- Utility tests

Use:

- Vitest
- React Testing Library

Configure:

- MSW

Provide:

```

npm test

```

without additional setup.

---

# Performance

Use:

- React.lazy
- Suspense
- memo only when profiling indicates benefit

Avoid premature optimization.

---

# Environment

Support:

```

.env
.env.local
.env.production

```

Never hardcode secrets.

---

# Import Style

Prefer:

```

import { Button } from "@/components/Button";

```

Configure @ alias.

Avoid deep relative imports.

---

# Linting

Configure:

- ESLint
- Prettier
- Husky
- lint-staged

Pre-commit should run:

- eslint
- typecheck
- tests

---

# Documentation

Generate if not existed or keep the current files:

- README.md
- .gitignore

---

# Build

Setup rollupOptions.output.manualChunks to make package smaller.

# Output

When scaffolding a project:

1. Create the directory structure.
2. Install all required dependencies.
3. Configure TypeScript.
4. Configure Tailwind.
5. Configure routing.
6. Configure testing.
7. Configure linting.
8. Configure formatting.
9. Configure Git hooks.
10. Generate documentation.

Do not stop after running create-vite.

The project should be immediately ready for feature development.
