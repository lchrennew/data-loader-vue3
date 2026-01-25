# DataLoader 组件

这是一个通用的 Vue 3 组件，旨在简化异步数据加载场景。它处理加载状态、错误展示、数据过滤以及基于 hash 变化的自动刷新。

## 功能特性

- **自动数据加载**：当组件挂载或 `hash` 属性变化时，自动触发 `loadData` 函数。
- **错误处理**：捕获数据加载过程中的异常，并显示错误提示（使用 `<a-alert>`）。
- **数据过滤**：支持可选的 `filter` 函数，在渲染前对加载的数据进行处理。
- **状态管理**：管理内部的 `loaded` 和 `data` 状态，同时也允许通过 props 进行外部控制。
- **作用域插槽**：通过作用域插槽将加载的数据、过滤后的数据、加载状态和重新加载触发器暴露给父组件。
- **依赖注入**：通过 `provide/inject` 将 `reload` 方法提供给后代组件。

## Props 属性

| 属性名 | 类型 | 是否必须 | 默认值 | 描述 |
|---|---|---|---|---|
| `hash` | Any | **是** | - | 数据状态的唯一标识符或版本哈希。如果 `hash` 发生变化（且与已加载状态不匹配），则触发重新加载。 |
| `loadData` | Function | **是** | - | 用于获取并返回数据的异步函数。 |
| `loadDataArgs` | Any | 否 | `undefined` | 传递给 `loadData` 函数的参数。 |
| `loaded` | Any | 否 | `undefined` | 可选的外部已加载状态。组件通过检查 `loaded === hash` 来确定数据是否是最新的。 |
| `reload` | Function | 否 | `undefined` | 可选的外部函数，用于更新已加载状态。如果提供，将使用此函数代替内部状态变更。 |
| `filter` | Function | 否 | `undefined` | 一个函数，接收原始 `data` 并返回处理/过滤后的数据。 |

## Events 事件

| 事件名 | 载荷 | 描述 |
|---|---|---|
| `loaded` | `(data, hash, filteredData)` | 当数据成功加载时触发。 |

## Slots 插槽

### 默认插槽

当没有发生错误时，组件渲染其默认插槽内容。

**插槽 Props:**

| 属性名 | 类型 | 描述 |
|---|---|---|
| `data` | Any | `loadData` 返回的原始数据。 |
| `filteredData` | Any | 对 `data` 应用 `filter` prop 后的结果。如果未提供过滤器，则等于 `data`。 |
| `loaded` | Boolean | 指示当前数据是否对应提供的 `hash`。 |
| `reload` | Function | 手动触发重新加载或更新已加载状态的函数。接受一个布尔值或 hash 值（默认为 `false`）。 |

## 依赖项

- **Ant Design Vue**：该组件使用 `<a-alert>` 显示错误消息。请确保你的项目中已安装并配置了 Ant Design Vue。

## 使用示例

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
      <!-- 如果需要自定义加载指示器（DataLoader 已处理逻辑） -->
      <div v-if="!filteredData">加载中...</div>
      
      <div v-else>
        <ul>
          <li v-for="item in filteredData" :key="item.id">
            {{ item.name }}
          </li>
        </ul>
        <button @click="reload(false)">强制刷新</button>
      </div>
    </template>
  </DataLoader>
</template>

<script setup>
import { ref } from 'vue';
import DataLoader from './DataLoader.vue';

const params = ref({ page: 1 });

// 异步数据获取函数
const fetchItems = async (args) => {
  const response = await fetch(`/api/items?category=${args.category}`);
  if (!response.ok) throw new Error('网络响应异常');
  return response.json();
};

// 可选的过滤器函数
const itemFilter = (data) => {
  return data.filter(item => item.active);
};

const handleLoaded = (data, hash, filtered) => {
  console.log('数据已加载，hash:', hash);
};
</script>
```

## 逻辑详情

1.  **初始化**：挂载时，组件检查当前 `hash` 的数据是否已加载。如果没有，则调用 `loadData`。
2.  **数据加载**：
    *   使用 `loadDataArgs` 调用 `loadData`。
    *   成功时：更新 `data`，将内部/外部已加载状态设置为 `hash`，并触发 `loaded` 事件。
    *   失败时：设置 `error` 状态，显示 `<a-alert>`。
3.  **响应式**：
    *   `dataLoaded` 计算属性监控 `props.hash` 与 `loaded` 状态的对比。如果不匹配，它会在下一个 tick 触发 `loadData`。
    *   这允许组件在 `hash` 变化时自动重新获取数据。
4.  **清理**：在 `onBeforeUnmount` 时，已加载状态重置为 `false`。
