# DotaGraph #87 — Escape-Key State Preservation

## Overview

This case study compares two AI coding agents solving the same keyboard-interaction bug in a React application from an identical frozen repository state.

The defect involved Escape-key handling while Search was active. Closing Search could allow the same keyboard event to propagate into application-level handlers, unintentionally changing the selected hero or matchup state.

## Evaluation Setup

- **Repository:** DotaGraph
- **Issue:** #87
- **Frozen commit:** `919bd985ee76df89e7011a5027f16c959eda8084`
- **Models evaluated:** Astra and Gemini
- **Evaluation focus:** event propagation, UI state preservation, regression coverage, implementation scope, and observable behavior

Both agents received the same task against independent copies of the same frozen codebase.

## Problem

Escape had different responsibilities depending on application state.

While Search was active, Escape needed to:

- Clear the search query
- Close search results
- Remove focus from Search
- Preserve the currently selected hero
- Preserve an active matchup
- Preserve the corresponding URL state

After Search closed, subsequent Escape presses should continue through the application's normal navigation hierarchy: matchup → selected hero → graph overview.

The bug was therefore not simply "Escape does not close Search." It involved ownership of a keyboard event across nested UI state.

## Astra Approach

Astra identified event propagation as the key failure mechanism.

The production fix added:

```js
event.stopPropagation();
```

to Search's Escape handling so the event responsible for closing Search could not continue into the window-level handler and clear graph state.

Astra then added end-to-end coverage across multiple combinations of:

- Selected-hero state
- Matchup state
- Populated search queries
- Queries with no matching hero
- Empty search queries
- Subsequent Escape presses after Search closes

It also updated the documented UX behavior to describe the intended Escape hierarchy.

### Patch Scope

```text
3 files changed
48 insertions
2 deletions
```

## Gemini Approach

Gemini also recognized that the Search Escape event needed to be distinguished from application-level Escape handling.

Its solution modified application-level keyboard handling and added extensive React tests covering preservation of selected-hero and matchup state.

The approach verified several important behaviors, including:

- Search closing without clearing the selected hero
- Matchup preservation
- Empty-search behavior
- Subsequent Escape navigation

### Patch Scope

```text
4 files changed
138 insertions
2 deletions
```

## Comparative Analysis

Both agents understood the state-management symptom, but Astra isolated the failure closer to its source.

Search owns the Escape event while Search is active. Consuming that event at the component boundary prevents an event used for one interaction from accidentally triggering a second, application-level interaction.

This produces a small production change while leaving the existing global Escape hierarchy intact.

Astra also exercised the behavior through end-to-end tests across multiple search and graph states. That was particularly valuable because the defect depended on interaction between focus, keyboard propagation, URL state, and rendered UI—not merely a single function's output.

Gemini provided useful regression coverage as well, but its solution reached further into application-level keyboard logic.

## Verification

The implementations were evaluated through:

- Source-code inspection
- Git patch comparison
- React behavior
- Keyboard-event propagation
- Search focus state
- Selected-hero preservation
- Matchup preservation
- URL-state preservation
- Regression tests
- End-to-end interaction tests

## Result

**Preferred implementation: Astra**

Astra traced the failure to the event boundary and fixed it with a minimal production change, then backed that change with broad end-to-end regression coverage.

The case demonstrates an important debugging principle for AI coding agents:

> Fixing the visible state change is not enough. The stronger solution identifies which component owns the event and prevents the unintended transition at its source.

## Evidence

- [`issue.png`](./issue.png) — original public issue
- [`astra.patch`](./astra.patch) — Astra implementation
- [`gemini.patch`](./gemini.patch) — Gemini implementation
- [`astra-diff-stat.txt`](./astra-diff-stat.txt) — Astra patch scope
- [`gemini-diff-stat.txt`](./gemini-diff-stat.txt) — Gemini patch scope

## Skills Demonstrated

AI evaluation · React · TypeScript · event propagation · UI state management · end-to-end testing · regression analysis · Git · comparative model evaluation