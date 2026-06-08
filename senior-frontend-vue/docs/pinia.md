# Pinia — справочник

Официальная документация: https://pinia.vuejs.org/

---

## Определение стора (Setup Store — предпочтительный стиль)

```ts
// stores/useUserStore.ts
import { defineStore } from 'pinia'

export const useUserStore = defineStore('users', () => {
  // state
  const users   = ref<User[]>([])
  const loading = ref(false)
  const error   = ref<string | null>(null)

  // getters
  const activeUsers = computed(() => users.value.filter(u => u.active))

  // actions
  async function loadUsers() {
    loading.value = true
    error.value   = null
    try {
      users.value = await userRepository.fetchAll()
    } catch {
      error.value = 'Ошибка загрузки'
    } finally {
      loading.value = false
    }
  }

  function reset() {
    users.value   = []
    error.value   = null
    loading.value = false
  }

  return { users, loading, error, activeUsers, loadUsers, reset }
})
```

---

## Options Store (альтернатива)

```ts
export const useCounterStore = defineStore('counter', {
  state:   () => ({ count: 0 }),
  getters: { double: (state) => state.count * 2 },
  actions: { increment() { this.count++ } },
})
```

---

## Использование в компоненте

```vue
<script setup lang="ts">
import { useUserStore } from '@/stores/useUserStore'
import { storeToRefs }  from 'pinia'

const store = useUserStore()

// ✅ storeToRefs — реактивная деструктуризация state/getters
const { users, loading, error, activeUsers } = storeToRefs(store)

// actions деструктурируются напрямую (не реактивны)
const { loadUsers, reset } = store

onMounted(loadUsers)
</script>
```

---

## storeToRefs vs toRefs

```ts
// storeToRefs — только для Pinia сторов, пропускает actions
const { count } = storeToRefs(store) // ✅

// toRefs — для reactive объектов, не для сторов
const { count } = toRefs(store)      // ⚠️ включает actions как refs
```

---

## $patch — массовое обновление state

```ts
// объект — поверхностное слияние
store.$patch({ loading: false, error: null })

// функция — мутация для сложных изменений
store.$patch((state) => {
  state.users.push(newUser)
  state.loading = false
})
```

---

## $subscribe — подписка на изменения state

```ts
store.$subscribe((mutation, state) => {
  localStorage.setItem('users', JSON.stringify(state.users))
})
```

---

## $onAction — подписка на вызовы actions

```ts
store.$onAction(({ name, args, after, onError }) => {
  after((result) => { console.log(`${name} завершён`, result) })
  onError((error) => { console.error(`${name} упал`, error) })
})
```

---

## $reset — сброс к начальному состоянию

```ts
// работает только для Options Store
store.$reset()

// для Setup Store — реализуй reset() вручную (см. выше)
```

---

## Вложенные сторы

```ts
// сторы могут использовать друг друга
export const useCartStore = defineStore('cart', () => {
  const auth = useAuthStore() // ✅ вызывай внутри defineStore

  const canCheckout = computed(() => auth.isAuthenticated && items.value.length > 0)

  return { canCheckout }
})
```

---

## Плагины Pinia

```ts
// persist state в localStorage
import { createPinia } from 'pinia'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

const pinia = createPinia()
pinia.use(piniaPluginPersistedstate)

// в сторе
export const useAuthStore = defineStore('auth', () => { ... }, {
  persist: true, // или { key: 'auth', storage: sessionStorage }
})
```

---

## Тестирование

```ts
import { setActivePinia, createPinia } from 'pinia'
import { useUserStore } from '@/stores/useUserStore'

beforeEach(() => {
  setActivePinia(createPinia())
})

it('loadUsers заполняет users', async () => {
  const store = useUserStore()
  vi.spyOn(userRepository, 'fetchAll').mockResolvedValue([mockUser])

  await store.loadUsers()

  expect(store.users).toEqual([mockUser])
  expect(store.loading).toBe(false)
})
```

---

## Полезные ссылки

- Setup Stores: https://pinia.vuejs.org/core-concepts/
- Plugins: https://pinia.vuejs.org/core-concepts/plugins
- Testing: https://pinia.vuejs.org/cookbook/testing
- pinia-plugin-persistedstate: https://prazdevs.github.io/pinia-plugin-persistedstate/
