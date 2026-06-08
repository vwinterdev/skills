# Принцип инверсии зависимостей (DIP)

Скрывай внешние библиотеки и реализации за фасадом — компоненты и composables зависят от своего сервиса, а не от конкретного npm-пакета. Замена библиотеки не затрагивает ни одного компонента.

**Хорошо** — библиотека инкапсулирована в сервис, вся работа с датами через него:

```ts
// lib/date.ts
import dayjs from 'dayjs'

export class DateService {
  static format(date: string | Date, pattern = 'DD.MM.YYYY'): string {
    return dayjs(date).format(pattern)
  }

  static parse(date: string | Date): Date {
    return dayjs(date).toDate()
  }

  static isAfter(a: string | Date, b: string | Date): boolean {
    return dayjs(a).isAfter(dayjs(b))
  }
}
```

```vue
<!-- ✅ компонент не знает про dayjs -->
<script setup lang="ts">
import { DateService } from '@/lib/date'

const formatted = computed(() => DateService.format(props.user.createdAt))
</script>
```

**Плохо** — `dayjs` импортируется напрямую в каждом файле:

```vue
<!-- ❌ -->
<script setup lang="ts">
import dayjs from 'dayjs' // жёсткая зависимость от библиотеки в компоненте

const formatted = computed(() => dayjs(props.user.createdAt).format('DD.MM.YYYY'))
</script>
```

> При замене `dayjs` на `date-fns` или `Temporal` придётся менять каждый компонент. С фасадом — только `lib/date.ts`.
