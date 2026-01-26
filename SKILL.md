---
name: data-loader-vue3
description: Invoke this skill when the user mentions "/use loader", "DataLoader", "data loading", "async data", "loading state", or "error handling" in Vue components. It provides the standard implementation guidelines for the DataLoader component.
---

# DataLoader Component Skill

This skill provides instructions on how to correctly use the `DataLoader` component in Vue 3 applications.

## Overview

The `DataLoader` component abstracts away the boilerplate of:
- Tracking loading/error states
- Handling async data fetching
- Automatically reloading data when parameters (hash) change
- Filtering data before rendering

## Usage Pattern

When you need to load data from an API, use the `DataLoader` component instead of manually creating `loading`, `error`, and `data` refs.

### Basic Template Structure

```vue
<template>
  <DataLoader
    :hash="dependencyKey"
    :load-data="fetchDataFunction"
    :load-data-args="fetchDataArgs"
    @loaded="onLoaded"
  >
    <template #default="{ data, filteredData, reload, loaded }">
      <!-- Your Content Here -->
      <div v-if="filteredData">
        <!-- Render data -->
      </div>
      
      <!-- Optional: Manual Reload Button -->
      <button @click="reload">Reload</button>
    </template>
  </DataLoader>
</template>
```

## Configuration Rules

1.  **`hash` Prop (Required)**:
    -   Pass a reactive value (e.g., `page` number, `id`, or a unique key) to `hash`.
    -   When `hash` changes, `DataLoader` automatically re-runs `loadData`.
    -   If there is no specific dependency, pass a static string or symbol, but ensure it matches the initial state if needed.

2.  **`loadData` Prop (Required)**:
    -   Must be a function returning a Promise.
    -   Do **not** invoke the function in the prop (e.g., `:load-data="fetchData"`, NOT `:load-data="fetchData()"`).

3.  **`filter` Prop (Optional)**:
    -   Use this for client-side filtering.
    -   The `filteredData` slot prop will contain the result of this function.

## Slot Props

-   `data`: The raw result from `loadData`.
-   `filteredData`: The processed data (use this for rendering if `filter` is used).
-   `reload`: A function to manually trigger a reload.
-   `loaded`: Boolean indicating if the current data matches the `hash`.

## Examples

### Example 1: Loading a List with Pagination

```vue
<script setup>
import { ref } from 'vue'
import { getItems } from '@/api/items'
import DataLoader from '@/components/DataLoader.vue'

const currentPage = ref(1)

// Wrapper to match loadData signature if needed
const fetchItems = (args) => getItems({ page: args.page })
</script>

<template>
  <DataLoader
    :hash="currentPage"
    :load-data="fetchItems"
    :load-data-args="{ page: currentPage }"
  >
    <template #default="{ filteredData }">
      <div v-for="item in filteredData" :key="item.id">
        {{ item.name }}
      </div>
      <button @click="currentPage++">Next Page</button>
    </template>
  </DataLoader>
</template>
```

### Example 2: Error Handling

The component automatically handles errors. If `loadData` throws, an `<a-alert>` is displayed. You do not need to add manual error handling in the template unless you want to customize it beyond the default alert.

## Anti-Patterns

-   **Don't** manually create `isLoading` refs in the parent component; use the `DataLoader`'s internal state.
-   **Don't** call the API function in `onMounted`; `DataLoader` does this automatically.
