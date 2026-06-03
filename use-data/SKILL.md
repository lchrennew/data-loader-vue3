---
name: use-data
description: Use this skill when editing this Vue 3 project and the task needs automatic data loading or reload behavior with the `DataLoader` component, including `hash`-driven refresh, scoped-slot rendering, `loaded` callbacks, and injected reload handlers.
---

# use-data

Use this skill only when data loading in this project should go through `DataLoader` rather than custom `onMounted`, `watch`, or duplicate fetch state.

## When to use

- Add automatic async data loading to a page, view, or child component.
- Replace manual fetch or refresh logic with `DataLoader`.
- Add reload behavior after search, save, delete, filter, tab, or page change.
- Expose loaded data through scoped slots.
- Trigger reload from descendants with `inject`.
- Handle post-load callbacks through the `loaded` event.

Read [REFERENCE.md](references/REFERENCE.md) only if you need prop contracts, slot details, example code, or gotchas.

## Default approach

1. Read the target component and identify which inputs should trigger a reload.
2. Move the fetch into a `loadData` function and pass request params as one `loadDataArgs` object.
3. Build a stable `hash` from every input that should invalidate the current result.
4. Render from slot props: `data`, `filtered-data`, `loaded`, and `reload`.
5. Reuse slot `reload()` or an injected reloader instead of adding a second fetch lifecycle.
6. Use `filter` only for derived display data, not as a replacement for raw `data`.
7. Verify the chosen `hash` changes whenever the effective query changes.

## Rules

- Prefer truthy string `hash` values; avoid `hash=false` because internal `loaded` also starts as `false`.
- `loadDataArgs` is forwarded as a single argument to `loadData`.
- `loaded` means internal state matches the current `hash`, not merely that a request once completed.
- If the parent customizes `reloaderName`, descendants must inject that exact name.
- Avoid parallel fetching in `onMounted`, `watch`, or local refresh handlers when `DataLoader` owns the request.
