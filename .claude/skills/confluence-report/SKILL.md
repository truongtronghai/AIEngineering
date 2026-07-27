---
name: confluence-report
description: Generate and publish standardized test reports to Confluence after successful test execution.
disable-model-invocation: true
---

# Confluence Report

## Purpose

This skill is responsible for publishing test results to Confluence.

It is used after implementation has completed and all required tests have been executed.

---

## When to Use

Use this skill whenever:

-   Tests have just completed.
-   Coverage has been generated.
-   A user requests a test report.
-   A release report is requested.

Do not use this skill before testing.

---

## Preconditions

Before publishing, ensure:

-   All required tests have passed.
-   Coverage has been generated.
-   Test artifacts are available.
-   Atlassian MCP is connected.

If any precondition is not satisfied:

-   Do not publish.
-   Explain why.
-   Return control to the user.

---

## Report Contents

Generate a concise report including:

# Test Report Summary

Before creating a test report, ensure the following information is available.

If any item is missing, ask the user before continuing.

Required fields:

-   Date & time
-   Feature / task
-   Branch
-   Commit SHA

Rules:

1. Never invent values.
2. If the current conversation, Git repository, or another trusted source already contains a value, use it.
3. Otherwise, ask the user.
4. Do not generate the report until all required fields are known.

Example:

Missing information:

-   Branch
-   Commit SHA

Ask:

> I need two details before I can generate the report:
>
> -   Branch name
> -   Commit SHA

### Test Results

-   Unit tests
-   Component tests
-   Integration tests
-   Playwright / E2E tests

Display:

-   Passed
-   Failed
-   Skipped
-   Duration

### Coverage

Include available metrics:

-   Statements
-   Branches
-   Functions
-   Lines

### Failed Tests

If failures exist:

-   Test name
-   Error summary

Do not include full stack traces unless requested.

### Artifacts

When available include references to:

-   Coverage report
-   Playwright HTML report
-   Screenshots
-   Videos

---

## Publishing

Use the Atlassian MCP server.

Prefer updating an existing Test Report page for the current feature.

Only create a new page if one does not already exist.

---

## Formatting

Use:

-   Headings
-   Tables
-   Bullet lists

Keep reports concise.

Do not paste raw terminal output.

Summarize instead.

---

## Completion Message

After publishing, tell the user:

-   Confluence page updated.
-   Page title.
-   Page URL.

Do not include the full report in chat unless requested.
