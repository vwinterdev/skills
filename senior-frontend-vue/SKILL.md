---
name: "senior-frontend-vue"
description: Code review, create new components, create js/ts/vue code, refactoring and architecture guidance for Vue 3 / Nuxt / TypeScript front‑end projects.
when_to_use: When you are working on a Vue/TS front‑end feature, fixing bugs, refactoring components, or designing new pages and flows.
scope: project
disable_model_invocation: false
---

# Назначение скила

## Этот скил помогает тебе как сеньер вью разработчик:
    - Делать code review файлов Vue/TS.
    - Предлагать архитектурные решения (куда вынести логику, как организовать state, когда использовать composables и pinia).
    - Проверять код на стайл‑гайд, типобезопасность, переиспользуемость и производительность.
    - Помогать с рефакторингом устаревших Vue‑компонентов под современный Vue 3 / Nuxt / Composition API.
    - Помогать с поддержкой устаревших Vue‑компонентов.


# Когда использовать

## Вызывайте этот скил, когда:
- вы пишете новый компонент или страницу на Vue2 / Vue 3 / Nuxt / TypeScript / JavaScript / css / scss;
- хотите, чтобы код был «senior‑level» (чистый, модульный, типобезопасный);
- рефакторите legacy‑код, мигрируете с Options API на Composition API при архитектуре проекта готовой к переходу;
- настраиваете state‑менеджмент (Pinia), async-state-state‑менеджмент (TanStackQuery), композаблы, маршрутизацию или API‑слои.
- пишите разметку, стили.


# Логика и шаги
## Когда скил срабатывает, Claude должен:
1. Понять контекст
    - Определить, что за проект: Vue 2 / Vue 3 / Nuxt, Pinia или Vuex, Vue Router, TypeScript версии.
    - Определить возможность имплементации установленных библиотек и плагинов, найти существующие решения используемые в проекте.
    - Понять, что за компонент: страница, layout, компонент UI, composables, API‑слой.
2. Проверить качество кода
    - Типы: интерфейсы, пропсы, параметры, возвращаемые значения.
    - Композиция: повторяющийся код, большие компоненты, вынести в composables / хелперы.
    - Избегать миксинов.
    - Ссылки: ref, reactive, computed, watch — правильно ли используются.
3. Предложить архитектуру (если не смог определить)
    - Где вынести бизнес‑логику (composables, services, stores).
    - Как разделить слой UI и поведения (отдельные компоненты, части, слой API‑клиента).
    - Как структурировать каталоги (components/, composables/, stores/, pages/, layouts/, lib/).
4. Предложить улучшения
    - Рефакторинг сложных условий, больших методов, хардкод‐данных.
    - Использовать defineOptions, defineProps, defineEmits и т.п. если поддерживается.
    - Проверить обработку ошибок, загрузку данных, условия лоадинга/ошибки.
5. Проверить UI‑соответствие
    - Повторяющиеся UI‑блоки, которые можно вынести в BaseButton, BaseInput, BaseCard и т.п.
    - Семантика: правильные теги, доступность (aria‑атрибуты, role, tabIndex).


# Паттерны

Подробные примеры и разбор паттернов — в папке `patterns/`:

| Паттерн | Файл |
|---|---|
| Адаптер (fromRaw / toRaw) | [patterns/adapter.md](patterns/adapter.md) |
| БЭМ и написание стилей | [patterns/bem.md](patterns/bem.md) |
| Принцип инверсии зависимостей (DIP) | [patterns/dip.md](patterns/dip.md) |
| Enums | [patterns/enums.md](patterns/enums.md) |
| Repository Pattern | [patterns/repository.md](patterns/repository.md) |
| Роуты отдельно от инициализации | [patterns/routes.md](patterns/routes.md) |
| Листинг с фильтрами и пагинацией (TableQuery) | [patterns/table-query.md](patterns/table-query.md) |
| Schema-driven формы | [patterns/schema-forms.md](patterns/schema-forms.md) |

# Документация

Справочники по библиотекам — в папке `docs/`:

| Библиотека | Файл |
|---|---|
| Vue 2 | [docs/vue2.md](docs/vue2.md) |
| Vue 3 | [docs/vue3.md](docs/vue3.md) |
| Pinia | [docs/pinia.md](docs/pinia.md) |
| Vuetify 3 | [docs/vuetify.md](docs/vuetify.md) |
| PrimeVue 4 | [docs/primevue.md](docs/primevue.md) |
