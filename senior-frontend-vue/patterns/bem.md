# БЭМ и написание стилей

Правила:
- блок — имя компонента (`user-card`)
- элемент — двойное подчёркивание (`user-card__avatar`)
- модификатор — двойной дефис (`user-card--highlighted`, `user-card__status--active`)
- один блок = один компонент; вложенные блоки не повторяют префикс родителя
- используй `scoped` + `:deep()` только для переопределения сторонних компонентов
- переменные (цвет, отступ, размер) — через CSS custom properties или токены дизайн-системы

**Хорошо:**

```vue
<!-- components/UserCard.vue -->
<template>
  <article
    class="user-card"
    :class="{ 'user-card--highlighted': highlighted }"
  >
    <img
      class="user-card__avatar"
      :src="user.avatarUrl"
      :alt="user.name"
    />

    <div class="user-card__body">
      <h3 class="user-card__name">{{ user.name }}</h3>

      <span
        class="user-card__status"
        :class="`user-card__status--${user.status}`"
      >
        {{ user.status }}
      </span>
    </div>

    <button
      class="user-card__action"
      type="button"
      @click="emit('select', user)"
    >
      Выбрать
    </button>
  </article>
</template>

<script setup lang="ts">
import type { User } from '@/models/User'

const props = defineProps<{
  user: User
  highlighted?: boolean
}>()

const emit = defineEmits<{
  select: [user: User]
}>()
</script>

<style>
.user-card {
  display: flex;
  align-items: center;
  gap: var(--spacing-3);
  padding: var(--spacing-4);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  background: var(--color-surface);
}

.user-card--highlighted {
  border-color: var(--color-primary);
  background: var(--color-primary-subtle);
}

.user-card__avatar {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  object-fit: cover;
  flex-shrink: 0;
}

.user-card__body {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-1);
  flex: 1;
  min-width: 0;
}

.user-card__name {
  margin: 0;
  font-size: var(--text-md);
  font-weight: 600;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.user-card__status {
  font-size: var(--text-sm);
  color: var(--color-text-secondary);
}

.user-card__status--active  { color: var(--color-success); }
.user-card__status--blocked { color: var(--color-danger);  }
.user-card__status--pending { color: var(--color-warning); }

.user-card__action {
  padding: var(--spacing-2) var(--spacing-3);
  border: none;
  border-radius: var(--radius-sm);
  background: var(--color-primary);
  color: var(--color-on-primary);
  cursor: pointer;
  flex-shrink: 0;
}
</style>
```

**Плохо** — смешанные стили, произвольные имена, магические значения:

```vue
<!-- ❌ -->
<template>
  <div class="card highlighted-card">            <!-- два несвязанных класса -->
    <img class="avatar-img" />
    <div class="cardBody">                        <!-- camelCase вместо BEM -->
      <h3 class="cardBody__text">...</h3>
      <span class="statusSpan active">...</span>  <!-- модификатор без блока -->
    </div>
  </div>
</template>

<style scoped>
.card { padding: 16px; border: 1px solid #e2e8f0; } /* магические значения */
.highlighted-card { border-color: blue; }            /* отдельный класс вместо модификатора */
.cardBody {
    display: flex;
    flex-direction: column;
    &__text{
        color: grey;
    }
}
.active { color: green; }                            /* глобальный модификатор без контекста */
</style>
```
