# Schema-driven формы

Форма описывается декларативной схемой — массивом полей с типом, лейблом, валидацией и опциями. Компонент-рендерер читает схему и отрисовывает нужный input. Добавить или убрать поле — значит изменить схему, а не шаблон.

Подходит когда:
- Много однотипных форм (CRUD-сущности в админке)
- Форма генерируется динамически (поля зависят от роли, конфига с бэкенда)
- Нужно переиспользовать логику валидации и сабмита

## Типы схемы

```ts
// types/FormSchema.ts
export type FieldType = 'text' | 'number' | 'select' | 'multiselect' | 'date' | 'checkbox' | 'textarea'

export interface FieldOption {
  label: string
  value: string | number
}

export interface FormField {
  field:        string
  label:        string
  type:         FieldType
  required?:    boolean
  options?:     FieldOption[]
  placeholder?: string
  disabled?:    boolean | ((values: Record<string, unknown>) => boolean)
}
```

## Схема формы (декларация)

```ts
// pages/users/formSchema.ts
import { UserStatus, UserStatusLabel } from '@/models/UserStatus'

export const userFormSchema: FormField[] = [
  {
    field:    'name',
    label:    'Имя',
    type:     'text',
    required: true,
  },
  {
    field:    'email',
    label:    'Email',
    type:     'text',
    required: true,
  },
  {
    field:    'status',
    label:    'Статус',
    type:     'select',
    required: true,
    options:  Object.values(UserStatus).map(s => ({ value: s, label: UserStatusLabel[s] })),
  },
  {
    field:    'comment',
    label:    'Комментарий',
    type:     'textarea',
    disabled: (values) => values.status !== UserStatus.Blocked,
  },
]
```

## Composable `useSchemaForm`

```ts
// composables/useSchemaForm.ts
export function useSchemaForm<T extends Record<string, unknown>>(
  schema: FormField[],
  initial: Partial<T> = {}
) {
  const values = ref<Record<string, unknown>>(
    Object.fromEntries(schema.map(f => [f.field, initial[f.field] ?? null]))
  )
  const errors = ref<Record<string, string>>({})

  function validate(): boolean {
    errors.value = {}
    for (const field of schema) {
      const val = values.value[field.field]
      if (field.required && (val === null || val === '' || val === undefined)) {
        errors.value[field.field] = `${field.label} обязательно`
      }
    }
    return Object.keys(errors.value).length === 0
  }

  function isDisabled(field: FormField): boolean {
    if (typeof field.disabled === 'function') return field.disabled(values.value)
    return field.disabled ?? false
  }

  function reset() {
    values.value = Object.fromEntries(schema.map(f => [f.field, initial[f.field] ?? null]))
    errors.value = {}
  }

  return { values, errors, validate, isDisabled, reset }
}
```

## Компонент-рендерер

```vue
<!-- components/SchemaForm.vue -->
<script setup lang="ts">
import type { FormField } from '@/types/FormSchema'

const props = defineProps<{
  schema:   FormField[]
  values:   Record<string, unknown>
  errors:   Record<string, string>
  disabled?: (field: FormField) => boolean
}>()

const emit = defineEmits<{
  'update:values': [values: Record<string, unknown>]
}>()

function update(field: string, value: unknown) {
  emit('update:values', { ...props.values, [field]: value })
}
</script>

<template>
  <div class="schema-form">
    <div
      v-for="field in schema"
      :key="field.field"
      class="schema-form__field"
    >
      <label class="schema-form__label">
        {{ field.label }}
        <span v-if="field.required" class="schema-form__required">*</span>
      </label>

      <textarea
        v-if="field.type === 'textarea'"
        class="schema-form__input"
        :value="String(values[field.field] ?? '')"
        :disabled="disabled?.(field)"
        :placeholder="field.placeholder"
        @input="update(field.field, ($event.target as HTMLTextAreaElement).value)"
      />

      <select
        v-else-if="field.type === 'select'"
        class="schema-form__input"
        :value="values[field.field]"
        :disabled="disabled?.(field)"
        @change="update(field.field, ($event.target as HTMLSelectElement).value)"
      >
        <option v-for="opt in field.options" :key="opt.value" :value="opt.value">
          {{ opt.label }}
        </option>
      </select>

      <input
        v-else
        class="schema-form__input"
        :type="field.type"
        :value="String(values[field.field] ?? '')"
        :disabled="disabled?.(field)"
        :placeholder="field.placeholder"
        @input="update(field.field, ($event.target as HTMLInputElement).value)"
      />

      <span v-if="errors[field.field]" class="schema-form__error">
        {{ errors[field.field] }}
      </span>
    </div>
  </div>
</template>
```

## Страница редактирования

```vue
<!-- pages/users/UserEditPage.vue -->
<script setup lang="ts">
import { useUserStore }   from '@/stores/useUserStore'
import { useSchemaForm }  from '@/composables/useSchemaForm'
import { userFormSchema } from './formSchema'
import { User }           from '@/models/User'

const store  = useUserStore()
const route  = useRoute()
const router = useRouter()

const { values, errors, validate, isDisabled } = useSchemaForm(
  userFormSchema,
  await store.fetchById(Number(route.params.id))
)

async function onSubmit() {
  if (!validate()) return
  await store.update(User.fromRaw(values.value))
  router.push({ name: 'users' })
}
</script>

<template>
  <SchemaForm
    :schema="userFormSchema"
    :values="values"
    :errors="errors"
    :disabled="isDisabled"
    @update:values="values = $event"
  />
  <button @click="onSubmit">Сохранить</button>
</template>
```

**Плохо** — каждая форма руками описывает поля в шаблоне, дублирует валидацию:

```vue
<!-- ❌ -->
<template>
  <input v-model="form.name"  :class="{ error: errors.name }" />
  <span>{{ errors.name }}</span>

  <input v-model="form.email" :class="{ error: errors.email }" />
  <span>{{ errors.email }}</span>

  <select v-model="form.status">
    <option value="active">Активен</option>
    <option value="blocked">Заблокирован</option>
  </select>
</template>

<script setup lang="ts">
function validate() {
  if (!form.value.name)  errors.value.name  = 'Обязательно'
  if (!form.value.email) errors.value.email = 'Обязательно'
}
</script>
```
