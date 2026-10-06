# PrimeReact #8476 — Dynamic Column Reordering State

## Overview

This case study compares two AI coding agents solving the same state-management defect in PrimeReact's DataTable component from an identical repository state.

The issue involved the interaction between user-defined column reordering and dynamic changes to the table's columns. Once a user established an order by dragging columns, changes to the rendered column set could leave the DataTable working from stale ordering state.

## Evaluation Setup

- **Repository:** PrimeReact
- **Issue:** #8476
- **Models evaluated:** Astra and Gemini
- **Evaluation focus:** state synchronization, dynamic React children, preservation of user-established state, implementation scope, and regression risk

Both agents received the same task against independent copies of the same codebase.

## Problem

PrimeReact DataTable maintains internal column-order state when column reordering is enabled.

That creates a synchronization problem when the application's rendered columns later change.

A correct solution needs to distinguish between:

- A genuine change to the available columns
- The same columns being rendered in a different order
- User-established ordering created through drag-and-drop

Resetting internal order too aggressively can discard valid user state. Failing to reset it when the column set actually changes can leave stale ordering information behind.

## Astra Approach

Astra derived the current column identity from each child's `columnKey` or `field` and tracked that order separately.

When the rendered columns changed, it compared the previous and current sets while accounting for columns that still existed.

The implementation was designed so that adding or removing columns could invalidate stale internal ordering without treating every render-order difference as a reason to discard the order established by the user.

The key intent was explicitly documented in the patch:

> Adding or removing columns should preserve the order established by dragging.

## Gemini Approach

Gemini also tracked the current rendered columns and detected changes using a ref.

When the current column ordering differed from the previously rendered ordering, it cleared the existing internal `columnOrderState` and used the newly rendered order.

It also modified column retrieval and reorder completion behavior so the component could fall back to the current child order after resetting stale state.

## Comparative Analysis

Both implementations recognized the central problem: internal reorder state can become stale when the DataTable's children change.

The important difference is **what counts as a meaningful column change**.

Gemini responds to a difference in the rendered column-order array by clearing the stored reorder state. That is straightforward, but it risks treating a reordered version of the same column set as equivalent to columns actually being added or removed.

Astra performs a more targeted comparison. It filters the old and new identities against one another before deciding whether the stored ordering should be invalidated.

That distinction matters because DataTable has two sources of ordering information:

1. The order supplied by the application through React children.
2. The order established interactively by the user.

A robust fix must synchronize changing columns without unnecessarily destroying the second source of state.

## Verification

The implementations were evaluated through:

- Source-code inspection
- Git patch comparison
- React state-management reasoning
- Dynamic-child identity analysis
- Column-reorder behavior
- Regression-risk analysis

Particular attention was given to whether an implementation distinguished structural changes to the column set from ordering changes involving the same columns.

## Result

**Preferred implementation: Astra**

Both agents addressed stale column-order state, but Astra's solution more explicitly protects user-established drag ordering while reacting to actual changes in the available columns.

The case demonstrates a recurring challenge in AI-generated frontend fixes:

> Synchronizing derived state requires understanding what changed semantically, not merely detecting that two arrays are different.

## Evidence

- [`astra.patch`](./astra.patch) — Astra implementation
- [`gemini.patch`](./gemini.patch) — Gemini implementation

## Skills Demonstrated

AI evaluation · React · JavaScript · state synchronization · dynamic component trees · UI state management · code review · regression analysis · Git · comparative model evaluation