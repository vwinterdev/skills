# PrimeVue — справочник

Официальная документация: https://primevue.org/
Версия: PrimeVue 4 (для Vue 3)

---

## Установка и настройка

```ts
// main.ts
import { createApp } from 'vue'
import PrimeVue from 'primevue/config'
import Aura from '@primevue/themes/aura' // встроенная тема
import ToastService from 'primevue/toastservice'
import ConfirmationService from 'primevue/confirmationservice'

import 'primeicons/primeicons.css'

createApp(App)
  .use(PrimeVue, {
    theme: {
      preset: Aura,
      options: { darkModeSelector: '.dark' },
    },
  })
  .use(ToastService)
  .use(ConfirmationService)
  .mount('#app')
```

---

## Часто используемые компоненты

### Кнопки

```html
<Button label="Сохранить" icon="pi pi-save" @click="save" />
<Button label="Удалить" severity="danger" variant="outlined" />
<Button icon="pi pi-plus" rounded aria-label="Добавить" />
<Button label="Отправить" :loading="loading" />

<!-- severity: secondary | success | info | warn | danger | contrast -->
<!-- variant: outlined | text | link -->
```

### Поля ввода

```html
<InputText v-model="name" placeholder="Имя" />

<Select
  v-model="status"
  :options="statuses"
  option-label="label"
  option-value="value"
  placeholder="Статус"
/>

<AutoComplete
  v-model="selected"
  :suggestions="suggestions"
  @complete="search"
  field="name"
/>

<Textarea v-model="comment" rows="3" />

<InputNumber v-model="amount" :min-fraction-digits="2" />

<DatePicker v-model="date" date-format="dd.mm.yy" />

<MultiSelect
  v-model="selectedItems"
  :options="items"
  option-label="name"
  placeholder="Выберите..."
/>
```

### Таблицы

```html
<DataTable
  :value="items"
  :loading="loading"
  paginator
  :rows="20"
  :total-records="total"
  lazy
  @page="onPage"
  @sort="onSort"
  sort-mode="single"
  row-hover
  @row-click="onRowClick"
>
  <Column field="id"     header="ID"     sortable />
  <Column field="name"   header="Имя"    sortable />
  <Column field="status" header="Статус">
    <template #body="{ data }">
      <Tag :value="data.status" :severity="statusSeverity[data.status]" />
    </template>
  </Column>
  <Column header="">
    <template #body="{ data }">
      <Button icon="pi pi-pencil" text @click="edit(data)" />
      <Button icon="pi pi-trash" text severity="danger" @click="remove(data)" />
    </template>
  </Column>
</DataTable>
```

### Диалоги

```html
<Dialog v-model:visible="visible" header="Заголовок" :style="{ width: '500px' }" modal>
  <p>Содержимое</p>
  <template #footer>
    <Button label="Отмена"      text @click="visible = false" />
    <Button label="Подтвердить" @click="confirm" />
  </template>
</Dialog>
```

### ConfirmDialog (встроенный диалог подтверждения)

```html
<!-- в шаблоне — один раз на приложение -->
<ConfirmDialog />

<script setup>
import { useConfirm } from 'primevue/useconfirm'

const confirm = useConfirm()

function removeItem(item) {
  confirm.require({
    message: `Удалить ${item.name}?`,
    header:  'Подтверждение',
    icon:    'pi pi-exclamation-triangle',
    accept:  () => doRemove(item.id),
  })
}
</script>
```

### Toast (уведомления)

```html
<!-- в шаблоне — один раз на приложение -->
<Toast />

<script setup>
import { useToast } from 'primevue/usetoast'

const toast = useToast()

toast.add({ severity: 'success', summary: 'Сохранено', life: 3000 })
toast.add({ severity: 'error',   summary: 'Ошибка', detail: err.message, life: 5000 })
// severity: success | info | warn | error
```

### Формы и валидация (с @primevue/forms)

```html
<Form :resolver="resolver" @submit="onSubmit">
  <FormField v-slot="$field" name="email">
    <InputText v-bind="$field" placeholder="Email" />
    <Message v-if="$field.invalid" severity="error">{{ $field.error.message }}</Message>
  </FormField>
  <Button type="submit" label="Отправить" />
</Form>

<script setup>
import { zodResolver } from '@primevue/forms/resolvers/zod'
import { z } from 'zod'

const resolver = zodResolver(z.object({
  email: z.string().email('Неверный email'),
}))

function onSubmit({ valid, values }) {
  if (!valid) return
  // values.email
}
</script>
```

---

## Оверлеи

```html
<!-- Popover -->
<Button label="Инфо" @click="pop.toggle($event)" />
<Popover ref="pop">
  <p>Дополнительная информация</p>
</Popover>

<!-- Drawer (боковая панель) -->
<Drawer v-model:visible="drawerOpen" position="right" header="Фильтры">
  ...
</Drawer>
```

---

## Иконки (PrimeIcons)

```html
<i class="pi pi-check" />
<i class="pi pi-times" style="font-size: 1.5rem" />

<!-- часто используемые -->
pi-plus / pi-minus
pi-pencil / pi-trash
pi-check / pi-times
pi-search
pi-filter
pi-sort-alt
pi-chevron-down / pi-chevron-right
pi-exclamation-triangle
pi-info-circle
pi-save / pi-upload / pi-download
```

Все иконки: https://primevue.org/icons/

---

## PassThrough (кастомизация классов)

```html
<!-- переопределить внутренние классы компонента -->
<DataTable
  :pt="{
    root:        { class: 'my-table' },
    loadingIcon: { class: 'text-primary' },
  }"
/>
```

---

## Полезные ссылки

- Все компоненты: https://primevue.org/components/
- Темы и токены: https://primevue.org/theming/styled/
- PrimeIcons: https://primevue.org/icons/
- PrimeFlex (утилиты): https://primeflex.org/
- Формы (@primevue/forms): https://primevue.org/forms/
