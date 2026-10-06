# CleanUI #160 — Dropdown Keyboard & Focus Accessibility

## Overview

This case study compares two AI coding agents solving the same accessibility issue in the CleanUI Vue component library from an identical frozen repository state.

The task involved keyboard interaction, focus management, ARIA semantics, and menu behavior across dropdown and context-menu components. Both agents produced substantial implementations, but their approaches differed significantly in scope and accessibility design.

## Evaluation Setup

- **Repository:** CleanUI
- **Issue:** #160
- **Frozen commit:** `3638e888b6fb5bc7d80821b7c77856d71196f870`
- **Models evaluated:** Astra and Gemini
- **Evaluation focus:** correctness, accessibility behavior, implementation scope, regression risk, and browser-observable behavior

Both agents received the same task against independent copies of the same frozen codebase.

## Problem

The existing dropdown and context-menu implementation had several keyboard and focus-management gaps.

The evaluation examined whether each agent could correctly handle:

- Keyboard activation of dropdown triggers
- Focusable trigger semantics
- Initial menu-item focus
- Checkbox and radio menu items
- Escape-to-close behavior
- Focus restoration after closing
- Disabled triggers
- Interactions between teleported menus and their owning dropdown
- Avoidance of duplicate or incorrect tab stops

## Astra Approach

Astra implemented a focused change across five component files.

Key decisions included:

- Centralizing the selector for enabled menu items
- Supporting `menuitem`, `menuitemcheckbox`, and `menuitemradio`
- Delaying browser focus until the menu becomes visible
- Restoring focus only when focus still belongs to the closing menu
- Preventing hover/focus logic from reopening a menu while it is closing
- Supporting Enter and Space activation
- Propagating disabled state through dropdown context
- Preserving an existing native interactive child as the single tab stop
- Making the wrapper keyboard-focusable only when no interactive child exists

### Patch Scope

```text
5 files changed
69 insertions
20 deletions
```

## Gemini Approach

Gemini addressed many of the same behaviors but used a substantially broader implementation.

Its changes included:

- Expanded focus-history and restoration logic
- Checkbox and radio menu-item support
- Escape handling
- Disabled-state propagation
- Keyboard activation
- Additional ARIA state synchronization
- Explicit focus styling
- A large set of new component tests

### Patch Scope

```text
7 files changed
412 insertions
8 deletions
```

## Comparative Analysis

Both implementations identified important accessibility requirements, but Astra produced the more focused solution.

A particularly important distinction was trigger semantics. Astra detects whether the dropdown slot already contains an interactive element such as a button or link. When it does, that element remains the single interactive target. When it does not, the wrapper receives the required button semantics and keyboard behavior.

This avoids unnecessarily introducing another focusable control around an already-interactive child.

Astra also consolidated enabled-menu-item selection into a shared selector and guarded focus restoration so that closing a dropdown does not steal focus from another control intentionally focused by application behavior.

Gemini provided considerably more test coverage, which is valuable, but also introduced substantially more implementation surface area. For this task, the additional code increased complexity without producing a correspondingly stronger interaction model.

## Verification

The implementations were evaluated using:

- Source-code inspection
- Git patch comparison
- Browser behavior
- Keyboard interaction
- Focus behavior
- Accessibility semantics
- Regression-oriented reasoning

Browser screenshots were preserved for menu behavior involving leading checkbox and radio items.

## Result

**Preferred implementation: Astra**

The deciding factor was not code size alone. Astra provided the cleaner interaction model while addressing the accessibility requirements with a smaller and more targeted change.

This evaluation demonstrates an important principle when assessing AI-generated code:

> A plausible or comprehensive patch is not automatically the strongest patch. Correct behavior, semantic accuracy, regression risk, and implementation scope must be evaluated against the actual system.

## Evidence

- [`issue.png`](./issue.png) — original public issue
- [`astra.patch`](./astra.patch) — Astra implementation
- [`gemini.patch`](./gemini.patch) — Gemini implementation
- [`astra-diff-stat.txt`](./astra-diff-stat.txt) — Astra patch scope
- [`gemini-diff-stat.txt`](./gemini-diff-stat.txt) — Gemini patch scope
- [`screenshots/`](./screenshots/) — preserved browser verification

## Skills Demonstrated

AI evaluation · software debugging · accessibility · Vue · TypeScript · code review · regression analysis · Git · browser verification · comparative model evaluation