# Vue 3 — справочник

Официальная документация: https://vuejs.org/

---

## Composition API — структура компонента

```vue
<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue'

// props
const props = defineProps<{
  userId: number
  label?: string
}>()

// emits
const emit = defineEmits<{
  select: [id: number]
  update: [value: string]
}>()

// state
const items   = ref<Item[]>([])
const loading = ref(false)

// computed
const activeItems = computed(() => items.value.filter(i => i.active))

// watch
watch(() => props.userId, (id) => loadItems(id))

// lifecycle
onMounted(() => loadItems(props.userId))

// methods
async function loadItems(id: number) {
  loading.value = true
  items.value = await api.get(`/users/${id}/items`)
  loading.value = false
}
</script>
```

---

## ref vs reactive

```ts
// ref — примитивы и объекты, доступ через .value
const count = ref(0)
count.value++

// reactive — только объекты, доступ без .value
const state = reactive({ count: 0, name: '' })
state.count++

// ⚠️ деструктуризация reactive теряет реактивность
const { count } = state         // ❌ не реактивно
const { count } = toRefs(state) // ✅ реактивно
```

---

## computed

```ts
// только чтение
const double = computed(() => count.value * 2)

// с сеттером
const fullName = computed({
  get: () => `${first.value} ${last.value}`,
  set: (val) => {
    [first.value, last.value] = val.split(' ')
  },
})
```

---

## watch / watchEffect

```ts
// watch — явная зависимость, доступ к old/new
watch(count, (newVal, oldVal) => { ... })
watch([a, b], ([newA, newB]) => { ... })
watch(() => props.id, (id) => load(id), { immediate: true })

// watchEffect — авто-трекинг зависимостей
watchEffect(() => {
  console.log(count.value) // трекается автоматически
})
```

---

## Директивы

```html
v-if / v-else-if / v-else
v-show
v-for="item in items" :key="item.id"
v-model                          — двусторонняя привязка
v-model:title="val"              — именованный v-model
v-bind: / :
v-on: / @
v-slot / #
v-memo="[dep]"                   — мемоизация поддерева
```

---

## defineModel (Vue 3.4+)

```ts
// вместо props + emit для v-model
const value = defineModel<string>()
// <Input v-model="text" /> → value.value = 'text'

const title = defineModel<string>('title')
// <Modal v-model:title="t" />
```

---

## Слоты

```html
<!-- дефолтный -->
<slot />

<!-- именованный -->
<slot name="header" />
<template #header>...</template>

<!-- scoped slot -->
<slot :item="item" name="item" />
<template #item="{ item }">{{ item.name }}</template>
```

---

## provide / inject

```ts
// родитель
provide('key', value)
provide('service', readonly(ref(state)))

// потомок (любой глубины)
const value = inject<Type>('key')
const value = inject('key', defaultValue) // с дефолтом
```

---

## Lifecycle

```
setup → onBeforeMount → onMounted
      → onBeforeUpdate → onUpdated
      → onBeforeUnmount → onUnmounted
```

```ts
onMounted(() => { /* DOM готов */ })
onUnmounted(() => { /* очистка: timers, listeners */ })
```

---

## Teleport

```html
<!-- рендерит содержимое вне текущего дерева компонентов -->
<Teleport to="body">
  <Modal v-if="open" />
</Teleport>
```

---

## Suspense

```html
<Suspense>
  <template #default>
    <AsyncComponent />
  </template>
  <template #fallback>
    <Spinner />
  </template>
</Suspense>
```

---

## defineOptions / defineExpose

```ts
// задать имя компонента в <script setup>
defineOptions({ name: 'UserCard', inheritAttrs: false })

// открыть методы/свойства родителю через ref
defineExpose({ reset, validate })
```

---

## Composables

```ts
// composables/useCounter.ts
export function useCounter(initial = 0) {
  const count = ref(initial)
  const increment = () => count.value++
  const reset = () => { count.value = initial }
  return { count, increment, reset }
}

// использование
const { count, increment } = useCounter(10)
```

---

## Vue Router 4

```ts
// навигация
const router = useRouter()
router.push({ name: 'user-detail', params: { id: 1 } })
router.push({ path: '/users', query: { page: '2' } })
router.replace(...)
router.back()

// текущий маршрут
const route = useRoute()
route.params.id
route.query.page
route.meta.requiresAuth
```

---

## Полезные ссылки

- Composition API FAQ: https://vuejs.org/guide/extras/composition-api-faq
- Vue Router 4: https://router.vuejs.org/
- Reactivity in Depth: https://vuejs.org/guide/extras/reactivity-in-depth
- Performance: https://vuejs.org/guide/best-practices/performance
