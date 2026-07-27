---
name: v2-figma-component
description: >-
    Implement a component from a `specs/v2-design-*.md` file into
    `src/core/ui/v2`, sourcing measurements from Figma and vendoring a shadcn/ui
    primitive where one exists. Use whenever the user asks to implement, build,
    or update a v2 component from spec (e.g. "implement the Radio Card spec",
    "build the v2 dropdown from Figma", "do the next spec in /specs"). All spec
    files share one template with a fixed set of hard-won project conventions —
    this skill applies them consistently instead of re-deriving them each time.
---

# Implement a v2 component from a Figma spec

## 1. Locate and read the spec

Specs live at `specs/v2-design-<kebab-name>.md` (e.g. `v2-design-radio-card.md`
for "Radio Card"). If the user names a component but not a file, match by
kebab-casing the name. If ambiguous or missing, list `specs/*.md` and ask.

Every spec follows the same template — read the whole file, but in practice
only two things vary per component: the **Name** and the **Figma Node** URL.
Everything else (Requirements, Public API process, Deliverables, Figma Review
Checklist, Implementation Rules) is the fixed contract below — don't skip it
because it "looks like boilerplate."

## 2. Check whether this is new work or a revision

`ls src/core/ui/v2/<Name>/` first. If the folder already exists, treat this as
a revision against the spec (diff current behavior vs. spec), not a rebuild —
check `src/core/ui/v2/index.ts` for the existing barrel export too.

## 3. Pull the design from Figma — don't estimate

Follow the spec's Figma Review Checklist literally: Auto Layout, all variants,
all interactive states, typography tokens, spacing tokens, colors,
constraints, icon assets, component properties, prototype interactions.

-   Load the `figma-design-to-code` skill/resource before calling
    `get_design_context` (it's a hard MCP requirement, not optional).
-   Use `get_variable_defs` for bound tokens (spacing/radius/typography), but be
    aware fill colors on "on/checked/active" states are frequently baked into
    exported SVG assets rather than exposed as Tailwind classes or variables —
    when `get_design_context`/`get_variable_defs` don't surface a color, fetch
    the exported asset URL directly (e.g. `curl` it) and read the literal
    `fill="#..."` out of the SVG. Don't guess a color from the screenshot.
-   Cross-check any color you extract against
    `src/core/ui/v2/shared/variantSurface.ts`, `focusRing.ts`, and sibling
    components' variant-class files (e.g. `Tag/tagVariants.ts`) — this project's
    v2 palette is small and reused verbatim across components (`#7c3aed`
    primary, `#6225d1` hover, `#4c1ca8`/`#22222c` active/disabled, etc.). A
    "new" color that doesn't match anything already in `shared/` is a signal to
    double check the read, not necessarily a real new token.

## 4. Reuse or vendor the shadcn/ui primitive

Check `src/components/ui/` for an existing vendored primitive matching this
element (`button.tsx`, `dropdown-menu.tsx`, `badge.tsx` exist already). If
none exists for this element, vendor the real component from
https://ui.shadcn.com into `src/components/ui/<name>.tsx` — don't hand-roll a
`cva` variant structure from scratch.

Only adapt the vendored file where it concretely conflicts with this project
(see the fixed conflict list in step 5), and put an inline comment on each
adaptation explaining why. Leave everything else in the vendored file intact
even if unused (e.g. `asChild`/`Slot` support, unused variant colors).

## 5. Apply the fixed project conventions (non-negotiable)

These were each discovered the hard way on Button/Tag/DropdownMenu — apply
them up front rather than rediscovering them:

-   **Focus ring**: CSS `outline` + `outline-offset`, never `box-shadow` (a
    transparent shadow under a colored one does not punch a gap — `box-shadow`
    paints solid from the element edge outward regardless). Use `focus:`, not
    `focus-visible:`, so the ring shows on any focus including mouse clicks.
    e.g. `focus:outline focus:outline-2 focus:outline-offset-[4px] focus:outline-[#7c3aed]`.
    Reuse `src/core/ui/v2/shared/focusRing.ts` if the ring is per-`Variant`;
    extend it rather than inlining a new one.
-   **Borders need `border-solid` explicitly.** `tailwind.config.js` sets
    `corePlugins.preflight: false`, so the base reset that sets
    `border-style: solid` by default is gone — a bare `border`/`border-t`/etc.
    utility only sets width/color and silently renders nothing without an
    explicit `border-solid` alongside it. No visual difference of any kind, easy
    to misdiagnose as a color problem. Every bordered element in this codebase
    follows `border border-solid border-[...]` — match it.
-   **Disabled state = exact replacement color, not opacity.** This project's
    disabled states use specific hex values per variant (surface `#22222c`,
    indicator/text `#4a4a56`), not a generic `disabled:opacity-*` wash — opacity
    and background/text-color are different Tailwind groups so an override
    class won't reliably drop shadcn's default, and it would wash out the exact
    Figma hex anyway. (A supplementary `opacity-40` on an icon-only slot, as
    seen in `Button.tsx`, is a separate visual layer and is fine — the rule is
    about the base surface/text color, not every opacity use.)
-   **Strip shadcn's broken ring-token classes at the source**:
    `focus-visible:border-ring` / `ring-[3px]` / `ring-ring` reference an
    undefined `ring` theme token — it silently fails to compile but still
    leaves a stray colorless ring, and with `important: true` set globally an
    override class can't reliably win the cascade tie. Remove these from the
    vendored file rather than fighting them with overrides.
-   **Strip auto-icon-sizing descendant selectors that conflict**, e.g.
    `[&_svg:not([class*='size-'])]:size-4` — the `_` (descendant) combinator
    reaches into the component's own icon-slot wrapper and overrides its
    explicit per-size icon dimensions.
-   **Use `mergeClassNames` from `@/lib/utils`**, never shadcn's own `cn` name
    (this project's meaningful-names convention).
-   **All Tailwind values must be literal strings in source.** The JIT scanner
    needs full literal utility strings to appear in source — never build a
    class name by template-interpolating a runtime hex/rgba value; write every
    color out in full in a `Record<Variant, string>`-style map (see
    `tagVariants.ts`).

## 6. File layout — match the existing pattern exactly

```
src/core/ui/v2/<Name>/
  <Name>.tsx              # component
  <Name>.stories.tsx       # Storybook
  <Name>.test.tsx          # tests
  index.ts                 # export { <Name> } from "./<Name>"; export type {...}
  <name>Variants.ts         # component-specific variant/color class maps, if any (optional, see Tag)
```

-   Register the new folder in `src/core/ui/v2/index.ts`: add
    `export * from "./<Name>";`.
-   If a color/state class map is genuinely shared across components (not just
    this one), put it in `src/core/ui/v2/shared/` instead (mirrors
    `variantSurface.ts`/`focusRing.ts`); if it's specific to this component,
    keep it alongside it (mirrors `Tag/tagVariants.ts`).
-   Use `data-variant`/`data-size`/etc. attributes on the root element for
    Storybook/test targeting, matching `Button.tsx`.

## 7. Deliverables checklist

Before calling it done, confirm each exists: Component, Props interface,
Styles, Example usage (stories), Tests, Storybook entry. If the Figma design
and the implementation diverge anywhere, state why in a comment or in your
summary to the user — per the spec's "if implementation differs from Figma,
explain why" rule.

## 8. Verify

-   `yarn lint` and `yarn type-check` must pass.
-   Render the Storybook story (or otherwise mount the component) and visually
    compare against the Figma screenshot for each variant/state actually built.
