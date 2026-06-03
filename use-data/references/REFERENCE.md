# DataLoader Reference

Use this file only when the main `SKILL.md` is not enough for the current task.

## Source of truth

- Component: `src/DataLoader.vue`
- Export: `src/index.js`

## Contract

- Props:
  - `hash`: required, `String | Boolean`
  - `loadData`: required function
  - `loadDataArgs`: optional, forwarded as one argument
  - `filter`: optional transform for derived output
  - `reloaderName`: optional, defaults to `reload`
- Slot props:
  - `data`: raw result from `loadData`
  - `filtered-data`: `filter(data)` or `data`
  - `loaded`: true only when internal `loaded === hash`
  - `reload`: internal `setLoaded` function
- Event:
  - `loaded(data, hash, filteredData)`

## Runtime behavior

`DataLoader` keeps an internal `loaded` ref.

- If `loaded !== hash`, the computed `loaded` slot prop becomes `false`.
- That mismatch schedules `loadData()` with `nextTick`.
- After success, the component stores `data`, sets internal `loaded` to `hash`, and emits `loaded`.
- Calling `reload()` resets internal `loaded` to `false`, which causes the next render cycle to load again.
- `onBeforeUnmount` also resets internal `loaded` to `false`.

## Example pattern

```vue
<script setup>
const hash = 'users-1'
const query = { keyword: '', page: 1 }
const loadUsers = ({ keyword, page }) => api.user.list({ keyword, page })
</script>

<template>
  <data-loader :hash="hash" :load-data="loadUsers" :load-data-args="query" #="{ data, loaded, reload }">
    <user-list v-if="loaded" :users="data?.records || []" @refresh="reload()" />
    <div v-else>加载中...</div>
  </data-loader>
</template>
```

## Descendant reload

If a child component must trigger reload:

```vue
<script setup>
import { inject } from 'vue'

const reload = inject('reload')
</script>

<template>
  <save-button @success="reload?.()" />
</template>
```

If the parent uses a custom `reloaderName`, inject that exact name.

## Gotchas

- `hash=false` may suppress the first auto-load because the internal default is also `false`.
- `filter` does not overwrite raw `data`; it only affects `filtered-data` and the third `loaded` event argument.
- `loadDataArgs` is not spread into multiple parameters.
- `DataLoader` already renders `数据加载出错了` plus the error details on failure.
- Avoid combining `DataLoader` with duplicate `onMounted` or `watch` requests for the same data.
