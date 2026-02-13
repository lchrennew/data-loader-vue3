<script setup>
import { computed, nextTick, onBeforeUnmount, provide, ref } from 'vue'

const props = defineProps({
    hash: { required: true, type: [String, Boolean] },
    loadDataArgs: { required: false },
    loadData: { required: true, type: Function },
    filter: { required: false, type: Function },
    reloaderName: { type: String, default: 'reload' }
})

const emit = defineEmits(['loaded'])

const loaded = ref(false)
const data = ref(null)
const filteredData = computed(() => props.filter?.(data.value) ?? data.value)
const error = ref(false)

const dataLoaded = computed(() => {
    const result = loaded.value === props.hash
    !result && nextTick(loadData)
    return result
})

const loadData = async () => {
    try {
        data.value = await props.loadData(props.loadDataArgs)
        setLoaded(props.hash)
        emit('loaded', data.value, props.hash, filteredData.value)
    } catch (e) {
        error.value = e.message ?? e
    }
}

const setLoaded = ($loaded = false) => loaded.value = $loaded

setLoaded(false)
onBeforeUnmount(() => setLoaded(false))

provide(props.reloaderName, setLoaded)
</script>

<template>
    <div v-if="error">
        数据加载出错了
        <pre>{{ error }}</pre>
    </div>
    <slot v-else :data="data" :loaded="dataLoaded" :reload="setLoaded" :filtered-data="filteredData" />
</template>

<style scoped></style>