# Адаптер

Используй адаптер все случаи, когда в систему попадают объекты из внешних источников:
- API‑ответы

У адаптера должно быть только 2 статических метода: `fromRaw` (обязательный) и `toRaw` (опциональный).

```ts
class User {
    id: number | uuid
    name: string
    groups: Group[]
    createAt: Date

    constructor(raw: Raw){
        id: Number(raw['id'])
        name: String(raw['name'])
        groups: Array.isArray(raw['groups'] ? raw['groups'].map(Group.fromRaw) : [])
        createAt: Date(raw['create_at'])
    }

    static fromRaw(raw: Raw){
        return new User(raw)
    }

    static toRaw(user: User){
        return {
            id: user.id
            name: user.id
            groups: user.groups.map(Group.toRaw)
            create_at: user.createAt
        }
    }
}
```

**Хорошо** — данные из API всегда проходят через адаптер, компоненты работают с типизированными объектами:
```ts
// composables/useUsers.ts
const { data } = await api.get('/users')
const users = data.map(User.fromRaw) // ✅ внутри приложения — только User[]

// при отправке обратно
await api.put(`/users/${user.id}`, User.toRaw(user)) // ✅ сериализация в одном месте
```

**Плохо** — сырые данные разбросаны по компонентам, нет единой точки преобразования:
```ts
// components/UserCard.vue
const { data: users } = await api.get('/users')

// ❌ компонент сам разбирает raw-данные
const name = users[0]['full_name'] ?? users[0]['name']
const createdAt = new Date(users[0]['create_at'])

// ❌ при отправке формат повторяется руками в каждом месте
await api.put(`/users/${users[0].id}`, {
  full_name: form.name,
  create_at: form.createdAt.toISOString(),
})
```
