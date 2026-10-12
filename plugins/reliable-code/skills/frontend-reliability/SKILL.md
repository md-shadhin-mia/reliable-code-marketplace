---
name: frontend-reliability
description: Use when writing, changing, or reviewing frontend/UI code (React, Vue, Svelte, plain JS, etc.) to prevent state, async, rendering, and accessibility bugs.
---

# Frontend Reliability

Goal: ship UI code with as few defects as possible. Most frontend bugs come from unhandled states, async races, bad state management, and untested edge-case data. This skill makes those explicit, then verifies by running the app. No process guarantees zero bugs; report honestly what was checked.

Follow the steps in order. Do not skip verification.

## 1. Understand before writing

- Read the component(s) you'll touch, their parents, shared hooks/stores, and existing tests.
- Identify the stack and conventions: framework and version, TypeScript or not, state management, data-fetching library, styling approach, component library, test tools.
- Find the commands to type-check, lint, test, build, and run the dev server.
- Reuse existing components and patterns instead of inventing parallel ones.

## 2. Enumerate every UI state

For every piece of UI that depends on data or user action, design and implement all of these:

- **Loading** (and avoid layout jump when it resolves)
- **Error** (human-readable message and a way to retry)
- **Empty** (zero items, no results)
- **Success** with typical data
- **Success with extreme data**: one item, hundreds of items, very long names, missing optional fields, null values, emoji/RTL text
- **Disabled / permission-denied** states where relevant

If a state isn't handled, the UI will show it anyway, just badly.

## 3. Implementation rules

**Data and types**
- Type API responses. Where data crosses a trust boundary, validate it at runtime (e.g. Zod) rather than assuming the shape.
- Use null-safe access for optional fields; never assume an array exists before mapping.
- Format dates, numbers, and currency with `Intl` APIs; be explicit about time zones.

**State**
- Keep one source of truth. Derive values instead of copying them into extra state that can drift.
- Update state immutably.
- Keep state as local as possible; lift only when shared.
- Don't store derived or server data in local state when the data-fetching layer already caches it.

**Async and effects**
- Cancel or ignore stale requests (AbortController, or the data library's built-in handling) so a slow old response can't overwrite a newer one.
- Prevent double submission: disable the button and guard the handler while a request is in flight.
- Effects must declare correct dependencies and clean up everything they start: listeners, timers, intervals, subscriptions, observers.
- Don't set state after unmount.
- Debounce search-as-you-type and other high-frequency inputs.

**Rendering**
- Use stable, unique keys for lists (IDs, not array index, when items can reorder, insert, or delete).
- Avoid expensive work on every render; memoize only where measured or obviously needed.
- No hydration mismatches in SSR: don't render `Date.now()`, `Math.random()`, or `window`-dependent values on the server.

**Forms**
- Validate on the client for UX, but always assume the server is the authority and display its errors.
- Preserve user input when submission fails.
- Show errors next to the relevant field and associate them for screen readers.

**Accessibility**
- Use semantic elements: `button` for actions, `a` for navigation, real `label`s for inputs.
- Everything must work with keyboard alone; focus must be visible and managed in modals/dialogs (trap focus, return it on close).
- Images need meaningful `alt` (or empty `alt` if decorative). Sufficient color contrast.

**Layout**
- Check at narrow (320px) and wide widths. No horizontal scroll from long words or URLs (use wrapping/truncation).

**Security**
- Never inject user-controlled HTML (`dangerouslySetInnerHTML`, `v-html`, `innerHTML`) without sanitizing.
- Don't put secrets in frontend code or env vars shipped to the browser.

## 4. Test

- Component tests that interact the way a user does (e.g. Testing Library: find by role/label, click, type, assert visible result). Avoid testing implementation details.
- Cover each state from step 2, especially error and empty.
- For critical flows (sign-in, checkout, forms that save data), add or update an end-to-end test (Playwright/Cypress) if the project has them.

## 5. Verify by actually running it

Run, in this order, and fix everything that fails:

1. Type checker
2. Linter
3. Test suite
4. Production build (catches errors dev mode hides)
5. Open the app in a browser and use the changed feature:
   - Browser console has no new errors or warnings
   - Try the happy path, an error (e.g. block the request or go offline), and empty data
   - Try keyboard-only navigation
   - Check a narrow viewport

If something can't be run in this environment, say so explicitly rather than assuming it works.

## 6. Self-review the diff

Re-read the full diff skeptically:

- Is every loading/error/empty state handled?
- Can a fast double-click or a slow response cause wrong UI?
- Does every effect clean up after itself?
- Any leftover `console.log`, debug styles, TODOs, or hardcoded test data?
- Did I break other places that use this component or hook?

## 7. Report honestly

Finish with a short summary:
- What changed
- What was verified and how (commands run, browser checks done)
- What was NOT verified, and any known risks or assumptions

Never claim code is bug-free. Claim only what was tested.
