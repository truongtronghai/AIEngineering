# AGENTS.md

This document defines the coding standards and architecture that AI coding assistants must follow when working in this repository.

These instructions apply to all generated code unless explicitly overridden.

---

# Tech Stack

- React 19
- TypeScript (Strict Mode)
- Vite
- Tailwind CSS v4
- Ant Design v5
- React Hook Form
- Zod
- TanStack Query

---

# General Principles

Always:

- Follow the existing project architecture.
- Produce production-ready code.
- Keep code simple, readable, and maintainable.
- Reuse existing code whenever possible.
- Prefer composition over duplication.
- Keep business logic separate from UI.

Never:

- Introduce new libraries unless requested.
- Rewrite existing code without a clear reason.
- Ignore existing project conventions.

---

# TypeScript

Always:

- Use strict typing.
- Never use `any`.
- Prefer `type` over `interface` unless extension is required.
- Use `import type` whenever possible.
- Handle `null` and `undefined` correctly.
- Prefer type inference when it improves readability.

Avoid:

- Non-null assertions (`!`)
- Unnecessary type assertions (`as`)

---

# React

Use:

- Functional Components
- React Hooks
- Composition

Avoid:

- Class Components
- Prop drilling
- Large components
- Deep nesting

Components should only render UI.

Business logic belongs in custom hooks.

---

# Folder Structure

Prefer feature-based architecture.

```
src/
    api/
    assets/
    components/
    features/
        alerts/
            api/
            components/
            hooks/
            pages/
            schemas/
            types/
            utils/
    hooks/
    layouts/
    pages/
    routes/
    services/
    types/
    utils/
```

---

# Components

Components should:

- Render UI
- Receive props
- Handle UI interactions

Components should NOT:

- Call APIs directly
- Contain business logic
- Transform API responses

---

# Custom Hooks

Business logic belongs in hooks.

Examples:

```
useAlerts()
useAlert()
useCreateAlert()
useDeleteAlert()
```

Hooks may:

- Fetch data
- Transform data
- Manage state

Hooks never render JSX.

---

# API

Never call fetch() inside components.

Use a dedicated API layer.

Example:

```
api/
    alertApi.ts
```

Query hooks belong inside feature modules.

Example:

```
features/alerts/api/useAlerts.ts
```

---

# TanStack Query

Always use TanStack Query for server state.

Preferred hooks:

- useQuery
- useMutation
- useInfiniteQuery

Every query must properly handle:

- Loading
- Empty
- Error
- Success

---

# Validation

Always validate external data using Zod.

Never trust backend responses.

Example:

```ts
const AlertSchema = z.object({
    id: z.string(),
    title: z.string(),
    severity: z.enum([
        "Critical",
        "High",
        "Medium",
        "Low",
    ]),
});
```

---

# React Hook Form

React Hook Form is the ONLY form state management library.

Always use:

- useForm()
- Controller
- FormProvider when appropriate
- Zod Resolver

Never use:

- useState for form values
- Ant Design Form state
- Custom validation logic when Zod can handle it

---

# Ant Design

Ant Design is the project's component library.

Always prefer existing Ant Design components before creating custom ones.

Use:

- Table
- Input
- InputNumber
- Select
- DatePicker
- TimePicker
- Checkbox
- Radio
- Switch
- Button
- Modal
- Drawer
- Tooltip
- Dropdown
- Tabs
- Collapse
- Card
- Tag
- Badge
- Alert
- Empty
- Result
- Spin
- Skeleton
- Pagination
- Upload

Avoid overriding Ant Design styles unless necessary.

---

# React Hook Form + Ant Design

React Hook Form is the single source of truth.

Ant Design components are presentation only.

Always connect Ant Design controls using `Controller`.

Example controls:

- Input
- InputNumber
- Select
- DatePicker
- TimePicker
- Checkbox
- Radio
- Switch
- Upload

Do NOT use:

- `<Form>`
- `Form.Item` validation
- `initialValues`
- `rules`
- `onFinish`

Validation must come from React Hook Form + Zod.

Use `Form.Item` only for layout and displaying validation errors if the project already uses it.

---

# Tailwind CSS

Tailwind is used for:

- Layout
- Grid
- Flex
- Spacing
- Responsive design
- Utility styling

Do NOT recreate Ant Design components using Tailwind.

Prefer CSS variables for colors and design tokens.

Avoid inline styles.

---

# Styling

Prefer:

- Tailwind utilities
- Existing design tokens
- CSS variables

Avoid:

- !important
- Large custom CSS files
- Inline styles

---

# Naming

Components

```
AlertTable.tsx
AlertCard.tsx
AlertFilter.tsx
```

Hooks

```
useAlerts.ts
useAlert.ts
```

Schemas

```
alert.schema.ts
```

Types

```
alert.types.ts
```

Utilities

```
formatDate.ts
```

---

# Imports

Order imports:

1. React
2. Third-party libraries
3. Absolute imports
4. Relative imports
5. Type imports

---

# Error Handling

Always display:

- Loading state
- Empty state
- Error state

Never silently ignore errors.

---

# Accessibility

Always:

- Use semantic HTML
- Label form controls
- Support keyboard navigation
- Preserve focus visibility
- Provide alt text where needed

---

# Performance

Optimize only when necessary.

Avoid unnecessary:

- useMemo
- useCallback
- memo

---

# Testing

Use:

- Vitest
- React Testing Library

Test behavior, not implementation.

---

# Code Style

Prefer early returns.

Keep functions small.

Target:

- Components < 200 lines
- Functions < 50 lines

Extract reusable logic whenever possible.

---

# Comments

Comment only when explaining:

- Business rules
- Complex logic
- Workarounds

Explain WHY, not WHAT.

---

# Do Not

- Use `any`
- Ignore ESLint
- Ignore TypeScript errors
- Mutate props
- Mutate React state
- Duplicate utilities
- Add dependencies without approval
- Recreate Ant Design components unnecessarily

---

# When Generating Code

Always:

- Follow this document.
- Follow existing project conventions.
- Use React 19.
- Use strict TypeScript.
- Use feature-based architecture.
- Use TanStack Query for server state.
- Use React Hook Form for all forms.
- Use Zod for validation.
- Connect Ant Design controls with `Controller`.
- Use Tailwind CSS for layout and spacing.
- Use Ant Design for UI components.
- Produce clean, reusable, maintainable code.
