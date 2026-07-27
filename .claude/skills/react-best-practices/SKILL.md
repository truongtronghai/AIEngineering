---
name: react-best-practices
description: Apply modern React best practices whenever creating or modifying React components, hooks, or pages. Prefer simple, idiomatic React over unnecessary abstractions.
---

# React Best Practices

## Goal

Write React code that is simple, maintainable, performant, and follows modern React best practices.

---

## General Principles

-   Prefer readability over cleverness.
-   Keep components small and focused.
-   Avoid premature optimization.
-   Prefer composition over inheritance.
-   Avoid unnecessary abstractions.
-   Delete dead code instead of commenting it out.
-   Minimize component complexity.

---

## Components

-   Prefer arrow function components with derived type props with FC when possible.
-   Keep components responsible for one thing.
-   Extract repeated UI into reusable components.
-   Avoid deeply nested JSX.
-   Prefer early returns.
-   Use semantic HTML whenever possible.
-   Keep render logic easy to follow.

Good:

```tsx
if (!user) {
    return null;
}

return <Profile user={user} />;
```

Avoid:

```tsx
return <>{user && <>...</>}</T>;
```

---

## Props

-   Keep props minimal.
-   Pass only data that is actually needed.
-   Prefer explicit props over generic objects.
-   Avoid prop drilling when Context is appropriate.
-   Prefer children over render props unless flexibility is required.

---

## State

Use the simplest state that solves the problem.

Prefer:

-   local state
-   derived values
-   lifting state up

Avoid:

-   duplicated state
-   storing derived state
-   unnecessary global state

Good:

```tsx
const completedCount = todos.filter((t) => t.done).length;
```

Avoid:

```tsx
const [completedCount, setCompletedCount] = useState(0);
```

---

## Effects

Treat `useEffect` as synchronization with external systems.

Do NOT use `useEffect` for:

-   derived values
-   filtering
-   sorting
-   mapping
-   event handlers

Prefer computing values during render.

Always include proper dependencies.

Clean up subscriptions.

Avoid infinite loops.

---

## Hooks

Follow the Rules of Hooks.

Custom hooks should:

-   solve one concern
-   have clear names
-   expose a small API
-   hide implementation details

Prefer:

```tsx
const { isLoading, user } = useUser();
```

instead of exposing many unrelated values.

---

## Performance

Do NOT use:

-   useMemo
-   useCallback
-   React.memo

unless profiling shows a measurable benefit.

Avoid unnecessary re-renders by improving component design first.

---

## Lists

Always use stable keys.

Prefer:

```tsx
key={user.id}
```

or

```tsx
key={user.id + index}
```

Avoid:

```tsx
key = { index };
```

unless the list is static.

---

## Forms

Keep form state predictable.

Avoid duplicating validation logic.

Use React Hook Form with Yup for schema validation.

Use controlled components for React Hook Form components.

---

## Event Handlers

Prefer named handlers.

Good:

```tsx
const handleSubmit = () => {
    ...
};

<Button onClick={handleSubmit} />
```

Avoid large inline functions inside JSX.

---

## Conditional Rendering

Prefer:

```tsx
if (!data) {
    return <Loading />;
}
```

instead of nested ternaries.

Avoid multiple nested ternary expressions.

---

## Styling

Keep styling separate from business logic.

Prefer utility classes or design system components.

Avoid inline styles unless dynamic values require them.

---

## Accessibility

Always:

-   provide button labels
-   label form controls
-   preserve keyboard navigation
-   provide alt text
-   use semantic HTML

---

## Error Handling

Handle:

-   loading
-   empty
-   error

states explicitly.

---

## Code Organization

Component structure:

1. imports
2. constants
3. component
4. hooks
5. derived values
6. handlers
7. effects
8. JSX
9. helper components (if any)
