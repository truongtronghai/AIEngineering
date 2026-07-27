---
name: component-test-suite
description: Run the complete test suite for a React component after implementation or modification. Validate behavior, accessibility, styling, and regressions without changing application code.
---

# Component Test Suite

## Goal

Verify that a React component is production-ready by executing all applicable tests and reporting the results.

This skill **must never modify production code**. It only runs tests, analyzes failures, and reports findings.

---

# When to Use

Use this skill whenever:

-   a React component is created
-   a component is modified
-   component behavior changes
-   UI styling changes
-   accessibility changes
-   bug fixes affect a component

---

# Test Order

Run tests from fastest to slowest.

## 1. Static Analysis

Run:

-   TypeScript type checking
-   ESLint

Stop immediately if these fail.

---

## 2. Unit / Component Tests

Run the component's test suite.

Verify:

-   rendering
-   props
-   events
-   callbacks
-   state updates
-   edge cases
-   error handling

---

## 3. Accessibility Tests

Verify:

-   keyboard navigation
-   focus management
-   ARIA attributes
-   accessible names
-   semantic HTML

Report every accessibility violation.

---

## 4. Visual Regression

If snapshot or screenshot testing exists:

-   generate screenshots
-   compare against baseline
-   report visual differences

Do not automatically update snapshots.

---

## 5. Integration Tests

Run integration tests affected by the component.

Examples:

-   forms
-   routing
-   API mocking
-   context providers
-   dialogs
-   tables

---

## 6. End-to-End Tests

Run only the E2E tests impacted by this component.

Avoid executing the full E2E suite unless explicitly requested.

---

# Failure Handling

If a test fails:

Do NOT edit code.

Instead:

1. identify the failing test
2. identify the reason
3. explain the likely root cause
4. include relevant stack traces
5. suggest possible fixes

Never suppress or ignore failures.

---

# Reporting

Produce a concise report.

Example:

## Component Test Report

Component:
Button

### Results

✅ Type Check

✅ ESLint

✅ Component Tests (18 passed)

✅ Accessibility

✅ Visual Regression

✅ Integration Tests

❌ E2E

### Failed Tests

-   should close dialog after Escape key

Reason:

Escape handler was never registered after the refactor.

Likely Cause:

The `useEscape()` hook is no longer called.

Recommendation:

Restore the Escape handler and rerun the suite.

---

# Success Report

If everything passes:

## Component Test Report

Component:
Button

### Results

✅ Type Check

✅ ESLint

✅ Component Tests

✅ Accessibility

✅ Visual Regression

✅ Integration Tests

✅ E2E

Result:

All tests passed successfully.

No regressions detected.

---

# Rules

-   Never modify application code.
-   Never modify tests.
-   Never update snapshots automatically.
-   Never skip failing tests.
-   Run only the minimum tests necessary.
-   Report every failure clearly.
-   Keep reports concise.
-   Prefer deterministic tests.
-   Stop immediately if type checking or linting fails.

---

# Philosophy

Testing exists to verify software quality, not to make tests pass.

Always report the truth about the current state of the component.
