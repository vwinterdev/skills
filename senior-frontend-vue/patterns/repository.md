# Repository Pattern

Разделяй ответственность на два слоя:
- **Repository** — класс, который знает только про HTTP: эндпоинты, адаптеры, сериализацию.
- **Store (Pinia)** — управляет состоянием, вызывает репозиторий, хранит `loading` / `error`.

Компонент работает только со стором, не знает ни про API, ни про адаптеры.

```ts
// repositories/UserRepository.ts
import { api } from '@/lib/api'
import { User } from '@/models/User'

export class UserRepository {
  async fetchAll(): Promise<User[]> {
    const { data } = await api.get('/users')
    return data.map(User.fromRaw)
  }

  async fetchById(id: number): Promise<User> {
    const { data } = await api.get(`/users/${id}`)
    return User.fromRaw(data)
  }

  async update(user: User): Promise<User> {
    const { data } = await api.put(`/users/${user.id}`, User.toRaw(user))
    return User.fromRaw(data)
  }

  async remove(id: number): Promise<void> {
    await api.delete(`/users/${id}`)
  }
}

export const userRepository = new UserRepository()
```

```ts
// stores/useUserStore.ts
import { defineStore } from 'pinia'
import { userRepository } from '@/repositories/UserRepository'
import type { User } from '@/models/User'

export const useUserStore = defineStore('user', () => {
  const users = ref<User[]>([])
  const loading = ref(false)
  const error = ref<string | null>(null)

  async function loadUsers() {
    loading.value = true
    error.value = null
    try {
      users.value = await userRepository.fetchAll()
    } catch (e) {
      error.value = 'Не удалось загрузить пользователей'
    } finally {
      loading.value = false
    }
  }

  async function removeUser(id: number) {
    await userRepository.remove(id)
    users.value = users.value.filter(u => u.id !== id)
  }

  return { users, loading, error, loadUsers, removeUser }
})
```

```vue
<!-- components/UserList.vue — компонент не знает про HTTP -->
<script setup lang="ts">
import { useUserStore } from '@/stores/useUserStore'

const store = useUserStore()
onMounted(store.loadUsers)
</script>

<template>
  <div v-if="store.loading">Загрузка...</div>
  <div v-else-if="store.error">{{ store.error }}</div>
  <ul v-else>
    <li v-for="user in store.users" :key="user.id">
      {{ user.name }}
      <button @click="store.removeUser(user.id)">Удалить</button>
    </li>
  </ul>
</template>
```

**Плохо** — компонент сам делает запросы, мешает HTTP, адаптацию и состояние:

```vue
<!-- ❌ -->
<script setup lang="ts">
import { api } from '@/lib/api'

const users = ref([])
const loading = ref(false)

onMounted(async () => {
  loading.value = true
  const { data } = await api.get('/users')          // HTTP в компоненте
  users.value = data.map(u => ({                    // адаптация в компоненте
    id: Number(u['id']),
    name: String(u['full_name']),
  }))
  loading.value = false
})
</script>
```
