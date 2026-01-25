# DataLoader Component

[简体中文](./README.zh-CN.md)

A generic Vue 3 component designed to simplify asynchronous data loading scenarios. It handles the loading state, error presentation, data filtering, and automatic reloading based on hash changes.

## Features

- **Automatic Data Loading**: Triggers the `loadData` function automatically when the component mounts or when the `hash` prop changes.
- **Error Handling**: Captures exceptions during data loading and displays an error alert (using `<a-alert>`).
- **Data Filtering**: Supports an optional `filter` function to process the loaded data before rendering.
- **State Management**: Manages internal `loaded` and `data` states, while allowing external control via props.
- **Scoped Slots**: Exposes the loaded data, filtered data, loading status, and reload trigger to the parent component via scoped slots.
- **Dependency Injection**: Provides the `reload` method to descendant components via `provide/inject`.

## Props

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `hash` | Any | **Yes** | - | A unique identifier or version hash for the data state. If `hash` changes (and does not match the loaded state), a reload is triggered. |
| `loadData` | Function | **Yes** | - | An asynchronous function that fetches and returns the data. |
| `loadDataArgs` | Any | No | `undefined` | Arguments to be passed to the `loadData` function. |
| `loaded` | Any | No | `undefined` | Optional external loaded state. The component checks if `loaded === hash` to determine if data is fresh. |
| `reload` | Function | No | `undefined` | Optional external function to update the loaded state. If provided, it is used instead of the internal state mutation. |
| `filter` | Function | No | `undefined` | A function that takes the raw `data` and returns processed/filtered data. |

## Events

| Event | Payload | Description |
|-------|---------|-------------|
| `loaded` | `(data, hash, filteredData)` | Emitted when data is successfully loaded. |

## Slots

### Default Slot

The component renders its default slot content when no error has occurred.

**Slot Props:**

| Prop | Type | Description |
|------|------|-------------|
| `data` | Any | The raw data returned by `loadData`. |
| `filteredData` | Any | The result of applying the `filter` prop to `data`. If no filter is provided, it equals `data`. |
| `loaded` | Boolean | Indicates if the current data corresponds to the provided `hash`. |
| `reload` | Function | A function to manually trigger a reload or update the loaded state. Accepts a boolean or hash value (default `false`). |

## Dependencies

- **Ant Design Vue**: The component uses `<a-alert>` to display error messages. Ensure Ant Design Vue is installed and configured in your project.

## Usage Example

```vue
<template>
  <DataLoader
    :hash="params.page"
    :load-data="fetchItems"
    :load-data-args="{ category: 'books' }"
    :filter="itemFilter"
    @loaded="handleLoaded"
  >
    <template #default="{ filteredData, reload }">
      <!-- Custom Loading Indicator if needed, though DataLoader handles logic -->
      <div v-if="!filteredData">Loading...</div>
      
      <div v-else>
        <ul>
          <li v-for="item in filteredData" :key="item.id">
            {{ item.name }}
          </li>
        </ul>
        <button @click="reload(false)">Force Reload</button>
      </div>
    </template>
  </DataLoader>
</template>

<script setup>
import { ref } from 'vue';
import DataLoader from './DataLoader.vue';

const params = ref({ page: 1 });

// Async data fetching function
const fetchItems = async (args) => {
  const response = await fetch(`/api/items?category=${args.category}`);
  if (!response.ok) throw new Error('Network response was not ok');
  return response.json();
};

// Optional filter function
const itemFilter = (data) => {
  return data.filter(item => item.active);
};

const handleLoaded = (data, hash, filtered) => {
  console.log('Data loaded for hash:', hash);
};
</script>
```

## Logic Details

1. **Initialization**: On mount, the component checks if the data for the current `hash` is loaded. If not, it calls `loadData`.
2. **Data Loading**: 
   - `loadData` is called with `loadDataArgs`.
   - On success: `data` is updated, internal/external loaded state is set to `hash`, and the `loaded` event is emitted.
   - On failure: `error` state is set, displaying the `<a-alert>`.
3. **Reactivity**:
   - The `dataLoaded` computed property monitors `props.hash` vs `loaded` state. If they mismatch, it triggers `loadData` on the next tick.
   - This allows the component to automatically refetch when `hash` changes.
4. **Cleanup**: `onBeforeUnmount`, the loaded state is reset to `false`.
