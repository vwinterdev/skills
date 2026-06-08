# Описание роутов отдельно от инициализации

Определяй конфигурацию маршрутов в отдельных файлах по модулям, а в `router/index.ts` только собирай их и создавай роутер. Это позволяет масштабировать маршрутизацию без превращения `index.ts` в свалку.

```ts
// router/routes/users.ts
import type { RouteRecordRaw } from 'vue-router'

export const usersRoutes: RouteRecordRaw[] = [
  {
    path: '/users',
    name: 'users',
    component: () => import('@/pages/users/UsersPage.vue'),
    meta: { requiresAuth: true },
  },
  {
    path: '/users/:id',
    name: 'user-detail',
    component: () => import('@/pages/users/UserDetailPage.vue'),
    meta: { requiresAuth: true },
  },
]
```

```ts
// router/routes/auth.ts
import type { RouteRecordRaw } from 'vue-router'

export const authRoutes: RouteRecordRaw[] = [
  {
    path: '/login',
    name: 'login',
    component: () => import('@/pages/auth/LoginPage.vue'),
    meta: { requiresAuth: false },
  },
]
```

```ts
// router/index.ts — только сборка и инициализация
import { createRouter, createWebHistory } from 'vue-router'
import { usersRoutes } from './routes/users'
import { authRoutes } from './routes/auth'

export const router = createRouter({
  history: createWebHistory(),
  routes: [
    ...authRoutes,
    ...usersRoutes,
  ],
})

router.beforeEach((to) => {
  const isAuthed = useAuthStore().isAuthenticated
  if (to.meta.requiresAuth && !isAuthed) {
    return { name: 'login' }
  }
})
```

**Плохо** — все маршруты в одном файле вперемешку с гардами и инициализацией:

```ts
// ❌ router/index.ts
export const router = createRouter({
  history: createWebHistory(),
  routes: [
    { path: '/login', component: () => import('@/pages/LoginPage.vue') },
    { path: '/users', component: () => import('@/pages/UsersPage.vue'), meta: { requiresAuth: true } },
    { path: '/users/:id', component: () => import('@/pages/UserDetailPage.vue'), meta: { requiresAuth: true } },
    { path: '/settings', component: () => import('@/pages/SettingsPage.vue'), meta: { requiresAuth: true } },
    // ... ещё 40 маршрутов
  ],
})

router.beforeEach(...)
```
