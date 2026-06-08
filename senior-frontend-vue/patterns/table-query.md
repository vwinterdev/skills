# Листинг с фильтрами и пагинацией (TableQuery pattern)

Унифицированный механизм для любых страниц-таблиц в админке. Вся логика состояния — в `useListPage`, компонент только описывает поля фильтра и подключает стор.

Три ключевых объекта:
- **`filters`** — конфигурация полей формы фильтрации
- **`pageMeta` / `Pagination`** — текущая страница, `perPage`, `total`
- **`TableQuery`** — объект, который собирается перед каждым запросом: `{ filter, sorter, pagination }`

## Типы

```ts
// types/TableQuery.ts
export interface TableQuery {
  filter:     FilterDTO
  sorter?:    Sorter
  pagination: Pagination
}

export type FilterDTO = Record<string, string | number | Array<string | number>>
```

```ts
// adapters/Pagination.ts
export class Pagination {
  total:       number
  perPage:     number
  currentPage: number
  lastPage:    number

  constructor(raw: Raw) {
    this.total       = Number(raw['total']        ?? 0)
    this.perPage     = Number(raw['per_page']     ?? 20)
    this.currentPage = Number(raw['current_page'] ?? 1)
    this.lastPage    = Number(raw['last_page']    ?? 0)
  }

  static fromRaw(raw: Raw) { return new Pagination(raw) }

  static toRaw(p: Pagination): Raw {
    return { current_page: p.currentPage, per_page: p.perPage }
  }
}
```

```ts
// adapters/Sorter.ts
export class Sorter {
  sortBy:  string | null
  sortDir: 'asc' | 'desc' | null

  constructor(raw: Raw) {
    this.sortBy  = raw['sort_by']  ? String(raw['sort_by'])           : null
    this.sortDir = raw['sort_dir'] ? raw['sort_dir'] as 'asc'|'desc' : null
  }

  static fromRaw(raw: Raw) { return new Sorter(raw) }

  static toRaw(s: Sorter): Raw {
    return { sort_by: s.sortBy ?? undefined, sort_dir: s.sortDir ?? undefined }
  }
}
```

## Composable `useListPage`

```ts
// composables/useListPage.ts
export interface ListPageStore<T> {
  loadList(query: TableQuery): Promise<{ items: T[]; pagination: Pagination }>
}

export function useListPage<T>(store: ListPageStore<T>) {
  const items      = ref<T[]>([])
  const pagination = ref(Pagination.fromRaw({}))
  const sorter     = ref(Sorter.fromRaw({}))
  const filter     = ref<FilterDTO>({})
  const loading    = ref(false)

  async function load(query: TableQuery) {
    loading.value = true
    try {
      const result = await store.loadList(query)
      items.value      = result.items
      pagination.value = result.pagination
    } finally {
      loading.value = false
    }
  }

  function onFilter(newFilter: FilterDTO) {
    filter.value                 = newFilter
    pagination.value.currentPage = 1
    load({ filter: filter.value, sorter: sorter.value, pagination: pagination.value })
  }

  function onSort(newSorter: Sorter) {
    sorter.value                 = newSorter
    pagination.value.currentPage = 1
    load({ filter: filter.value, sorter: sorter.value, pagination: pagination.value })
  }

  function onPaginate(newPagination: Pagination) {
    pagination.value = newPagination
    load({ filter: filter.value, sorter: sorter.value, pagination: pagination.value })
  }

  function reload() {
    load({ filter: filter.value, sorter: sorter.value, pagination: pagination.value })
  }

  onMounted(reload)

  return { items, pagination, sorter, filter, loading, onFilter, onSort, onPaginate, reload }
}
```

## Стор

```ts
// stores/useUserStore.ts
export const useUserStore = defineStore('users', () => {
  async function loadList({ filter, sorter, pagination }: TableQuery) {
    const { data } = await api.get('/users', {
      params: {
        ...filter,
        ...Sorter.toRaw(sorter ?? Sorter.fromRaw({})),
        ...Pagination.toRaw(pagination),
      },
    })
    return {
      items:      data.data.map(User.fromRaw),
      pagination: Pagination.fromRaw(data.meta),
    }
  }

  return { loadList }
})
```

## Конфигурация фильтров

```ts
// pages/users/filters.ts
export const userFilters: FilterField[] = [
  { field: 'id',     label: 'ID',     type: 'number' },
  { field: 'name',   label: 'Имя',    type: 'string' },
  { field: 'status', label: 'Статус', type: 'select', options: Object.values(UserStatus) },
]
```

## Страница

```vue
<!-- pages/users/UsersPage.vue -->
<script setup lang="ts">
import { useUserStore } from '@/stores/useUserStore'
import { useListPage }  from '@/composables/useListPage'
import { userFilters }  from './filters'

const store = useUserStore()
const { items, pagination, loading, onFilter, onSort, onPaginate } = useListPage(store)
</script>

<template>
  <FilterForm :fields="userFilters" @filter="onFilter" />

  <DataTable
    :rows="items"
    :loading="loading"
    @sort="onSort"
  />

  <Pagination
    :meta="pagination"
    @paginate="onPaginate"
  />
</template>
```

**Плохо** — каждая страница-таблица управляет фильтрами, пагинацией и запросами по-своему:

```vue
<!-- ❌ разные страницы — разный код, нет единого механизма -->
<script setup lang="ts">
const page    = ref(1)
const perPage = ref(20)
const search  = ref('')
const items   = ref([])

async function load() {
  const { data } = await api.get('/users', {
    params: { page: page.value, per_page: perPage.value, search: search.value }
  })
  items.value = data.items
}

watch([page, search], load)
onMounted(load)
</script>
```
