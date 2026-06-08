# Enums

Используй `enum` для именованных наборов значений — это даёт автодополнение, защиту от опечаток и единую точку изменения. Всегда используй строковые значения (`string enum`), числовые энумы плохо читаются в API-ответах и логах.

Лейблы и маппинги храни рядом с enum в том же файле.

**Хорошо** — строковый `enum` + лейблы рядом, итерация через `Object.values`:

```ts
// models/UserStatus.ts
export enum UserStatus {
  Active  = 'active',
  Blocked = 'blocked',
  Pending = 'pending',
}

export const UserStatusLabel: Record<UserStatus, string> = {
  [UserStatus.Active]:  'Активен',
  [UserStatus.Blocked]: 'Заблокирован',
  [UserStatus.Pending]: 'Ожидает',
}
```

```vue
<script setup lang="ts">
import { UserStatus, UserStatusLabel } from '@/models/UserStatus'
import type { User } from '@/models/User'

const props = defineProps<{ user: User }>()

const statuses = Object.values(UserStatus)
</script>

<template>
  <!-- ✅ итерация по значениям без хардкода -->
  <select>
    <option v-for="status in statuses" :key="status" :value="status">
      {{ UserStatusLabel[status] }}
    </option>
  </select>

  <!-- ✅ сравнение через enum, не строку -->
  <span v-if="user.status === UserStatus.Blocked" class="user-card__status--blocked">
    Заблокирован
  </span>
</template>
```

**Плохо** — числовой enum или строки вручную:

```ts
// ❌ числовой enum — значения нечитаемы в API и логах
enum UserStatus {
  Active,   // 0
  Blocked,  // 1
  Pending,  // 2
}
```

```vue
<!-- ❌ строки вручную — опечатки не поймает компилятор -->
<template>
  <span v-if="user.status === 'actve'">...</span>  <!-- опечатка, TS не заметит -->

  <option v-for="s in ['active', 'blocked', 'pending']" :value="s">
    {{ s }}
  </option>
</template>
```
