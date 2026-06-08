# Vuetify — справочник

Официальная документация: https://vuetifyjs.com/
Версия: Vuetify 3 (для Vue 3)

---

## Установка и настройка

```ts
// main.ts
import { createApp } from 'vue'
import { createVuetify } from 'vuetify'
import 'vuetify/styles'
import { aliases, mdi } from 'vuetify/iconsets/mdi'

const vuetify = createVuetify({
  icons: { defaultSet: 'mdi', aliases, sets: { mdi } },
  theme: {
    defaultTheme: 'light',
    themes: {
      light: {
        colors: {
          primary:   '#1976D2',
          secondary: '#424242',
          error:     '#FF5252',
        },
      },
    },
  },
})

createApp(App).use(vuetify).mount('#app')
```

---

## Сетка (Grid)

```html
<!-- 12-колоночная сетка -->
<v-container>
  <v-row>
    <v-col cols="12" sm="6" md="4">...</v-col>
    <v-col cols="12" sm="6" md="8">...</v-col>
  </v-row>
</v-container>

<!-- выравнивание -->
<v-row align="center" justify="space-between">
<v-row no-gutters>   <!-- без отступов между колонками -->
```

---

## Часто используемые компоненты

### Кнопки

```html
<v-btn color="primary" @click="save">Сохранить</v-btn>
<v-btn variant="outlined" color="error">Удалить</v-btn>
<v-btn icon="mdi-plus" />
<v-btn :loading="loading" :disabled="loading">Отправить</v-btn>

<!-- variant: elevated | flat | tonal | outlined | text | plain -->
```

### Поля ввода

```html
<v-text-field
  v-model="name"
  label="Имя"
  :rules="[v => !!v || 'Обязательно']"
  variant="outlined"
  density="comfortable"
/>

<v-select
  v-model="status"
  :items="statuses"
  item-title="label"
  item-value="value"
  label="Статус"
/>

<v-autocomplete
  v-model="selected"
  :items="options"
  :loading="searching"
  @update:search="onSearch"
/>

<v-textarea v-model="comment" label="Комментарий" rows="3" />

<!-- density: default | comfortable | compact -->
<!-- variant: filled | outlined | underlined | plain | solo -->
```

### Таблицы

```html
<v-data-table
  :headers="headers"
  :items="items"
  :loading="loading"
  items-per-page="20"
  @click:row="(_, { item }) => openItem(item)"
>
  <template #item.status="{ item }">
    <v-chip :color="statusColor[item.status]">{{ item.status }}</v-chip>
  </template>
</v-data-table>

<!-- headers -->
<script setup>
const headers = [
  { title: 'ID',    key: 'id',    sortable: true  },
  { title: 'Имя',   key: 'name',  sortable: true  },
  { title: 'Статус',key: 'status',sortable: false },
  { title: '',      key: 'actions',sortable: false },
]
</script>
```

### Диалоги

```html
<v-dialog v-model="dialog" max-width="600">
  <v-card>
    <v-card-title>Заголовок</v-card-title>
    <v-card-text>Содержимое</v-card-text>
    <v-card-actions>
      <v-spacer />
      <v-btn @click="dialog = false">Отмена</v-btn>
      <v-btn color="primary" @click="confirm">Подтвердить</v-btn>
    </v-card-actions>
  </v-card>
</v-dialog>
```

### Снэкбары (уведомления)

```html
<v-snackbar v-model="snackbar" :color="snackColor" :timeout="3000">
  {{ message }}
  <template #actions>
    <v-btn icon="mdi-close" @click="snackbar = false" />
  </template>
</v-snackbar>
```

### Карточки

```html
<v-card>
  <v-card-title>Заголовок</v-card-title>
  <v-card-subtitle>Подзаголовок</v-card-subtitle>
  <v-card-text>Содержимое</v-card-text>
  <v-card-actions>
    <v-btn>Действие</v-btn>
  </v-card-actions>
</v-card>
```

### Формы и валидация

```html
<v-form ref="formRef" @submit.prevent="onSubmit">
  <v-text-field
    v-model="email"
    :rules="emailRules"
    label="Email"
  />
  <v-btn type="submit">Отправить</v-btn>
</v-form>

<script setup>
const formRef = ref()

const emailRules = [
  v => !!v           || 'Обязательно',
  v => /.+@.+/.test(v) || 'Неверный email',
]

async function onSubmit() {
  const { valid } = await formRef.value.validate()
  if (!valid) return
  // ...
}
</script>
```

---

## Утилити-классы

```html
<!-- отступы: ma/pa/mt/pt/mx/px/my/py + 0-16 -->
<div class="ma-4 px-2 mt-0">...</div>

<!-- flex -->
<div class="d-flex align-center justify-space-between ga-3">...</div>

<!-- текст -->
<p class="text-h5 font-weight-bold text-primary">...</p>
<p class="text-body-2 text-medium-emphasis">...</p>

<!-- display -->
<div class="d-none d-sm-flex">показывать только от sm</div>
```

---

## Темизация в компоненте

```html
<!-- переключить тему для поддерева -->
<v-theme-provider theme="dark">
  <v-card>...</v-card>
</v-theme-provider>
```

```ts
// программно
import { useTheme } from 'vuetify'
const theme = useTheme()
theme.global.name.value = 'dark'
```

---

## useDisplay — брейкпоинты

```ts
import { useDisplay } from 'vuetify'

const { mobile, smAndUp, lgAndDown, width } = useDisplay()

// в шаблоне
// v-if="mobile"
```

---

## Полезные ссылки

- Все компоненты: https://vuetifyjs.com/en/components/all/
- Grid system: https://vuetifyjs.com/en/components/grids/
- Theming: https://vuetifyjs.com/en/features/theme/
- Icons (MDI): https://pictogrammers.com/library/mdi/
- Утилити-классы: https://vuetifyjs.com/en/styles/spacing/
