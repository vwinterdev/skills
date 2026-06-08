# Vue 2 — справочник

Официальная документация: https://v2.vuejs.org/

---

## Options API — структура компонента

```js
export default {
  name: 'ComponentName',
  components: { ChildComponent },
  props: {
    userId: { type: Number, required: true },
    label:  { type: String, default: '' },
  },
  data() {
    return {
      items: [],
      loading: false,
    }
  },
  computed: {
    filteredItems() {
      return this.items.filter(i => i.active)
    },
  },
  watch: {
    userId(newVal) {
      this.loadItems(newVal)
    },
  },
  created() {
    this.loadItems(this.userId)
  },
  methods: {
    async loadItems(id) {
      this.loading = true
      this.items = await api.get(`/users/${id}/items`)
      this.loading = false
    },
  },
}
```

---

## Реактивность

```js
// ❌ Vue 2 не отслеживает добавление/удаление свойств объекта
this.obj.newProp = 'value'        // не реактивно
delete this.obj.prop              // не реактивно

// ✅ используй Vue.set / this.$set
this.$set(this.obj, 'newProp', 'value')
this.$set(this.arr, 0, 'newValue')

// ✅ или замени объект целиком
this.obj = { ...this.obj, newProp: 'value' }
```

---

## Директивы

```html
v-if / v-else-if / v-else   — условный рендер (удаляет из DOM)
v-show                       — скрывает через display:none
v-for="item in items" :key   — итерация, key обязателен
v-model                      — двусторонняя привязка
v-bind: / :                  — привязка атрибута
v-on: / @                    — обработчик события
v-slot / #                   — именованные слоты
v-once                       — рендер один раз
```

---

## События

```js
// дочерний компонент
this.$emit('update', payload)

// родитель
<Child @update="handleUpdate" />

// .sync — сокращение для emit('update:prop')
<Child :value.sync="localValue" />
// эквивалент:
<Child :value="localValue" @update:value="localValue = $event" />
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
<slot :item="item" />
<template #default="{ item }">{{ item.name }}</template>
```

---

## Lifecycle

```
beforeCreate → created → beforeMount → mounted
→ beforeUpdate → updated
→ beforeDestroy → destroyed
```

Загружай данные в `created` (доступен `this`), работай с DOM в `mounted`.

---

## Миксины — избегай

```js
// ❌ миксины — неявные зависимости, конфликты имён
const myMixin = {
  data() { return { loading: false } },
  methods: { load() {} },
}

// ✅ вынеси логику в сервис или composable (если Vue 2.7+)
```

---

## Vue 2.7+ (Composition API backport)

Vue 2.7 включает `setup()`, `ref`, `computed`, `watch` без плагина.
Для более ранних версий — `@vue/composition-api` плагин.

```js
import { ref, computed, onMounted } from 'vue' // 2.7+

export default {
  setup(props) {
    const count = ref(0)
    const double = computed(() => count.value * 2)
    onMounted(() => console.log('mounted'))
    return { count, double }
  }
}
```

---

## Vuex (state management для Vue 2)

```js
// store
const store = new Vuex.Store({
  state:     { users: [] },
  getters:   { activeUsers: state => state.users.filter(u => u.active) },
  mutations: { SET_USERS(state, users) { state.users = users } },
  actions:   { async loadUsers({ commit }) {
    const users = await api.get('/users')
    commit('SET_USERS', users)
  }},
})

// в компоненте
computed: {
  ...mapState(['users']),
  ...mapGetters(['activeUsers']),
},
methods: {
  ...mapActions(['loadUsers']),
}
```

---

## Полезные ссылки

- Migration guide Vue 2 → Vue 3: https://v3-migration.vuejs.org/
- Vue Router 3 (для Vue 2): https://v3.router.vuejs.org/
- Vuex 3: https://v3.vuex.vuejs.org/
