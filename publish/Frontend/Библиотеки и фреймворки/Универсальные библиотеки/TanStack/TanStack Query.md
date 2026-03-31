# TanStack Query

-   [TanStack Query](https://tanstack.com/query/latest) -- библиотека
    для управления серверным состоянием (данные с API)

> [!TIP] Референсный проект
> Пример можно посмотреть тут:
> https://github.com/TanStack/query/tree/main/examples/react
# Инициализация
-   Установка:

``` bash
pnpm add @tanstack/react-query
```

-   Базовая настройка провайдера:

``` tsx
// main.tsx / app.tsx

import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

const queryClient = new QueryClient()

<QueryClientProvider client={queryClient}>
  <App />
</QueryClientProvider>
```

# Получение данных (Query)
-  Все что нужно воспользоваться query, нужно дать запросу ключ кэша, чтоб он знал куда ему кэшировать данные, и какую-то некую query fn, которая возвращает какой-то промис
- Пример запроса:

``` tsx
// .../users/useUsers.ts

import { useQuery } from '@tanstack/react-query'

const fetchUsers = async () => {
  return await fetch('/api/users') 
}

export const useUsers = () => {
  return useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
  })
}
```
- И по результату запроса, нам вернутся данные data, и состояния: загрузки, ошибки и тд  
- Использование в компоненте:

``` tsx
// app.tsx

const App = () => {
  const { data, isLoading, error } = useUsers()

  if (isLoading) return <div>Loading...</div>
  if (error) return <div>Error</div>

  return (
    <div>
      {data.map((user) => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  )
}
```
# Изменение данных (Mutation)

``` tsx
// .../users/useCreateUser.ts

import { useMutation } from '@tanstack/react-query'

const createUser = async (user) => {
  const res = await fetch('/api/users', {
    method: 'POST',
    body: JSON.stringify(user),
  })
  return res.json()
}

export const useCreateUser = () => {
  return useMutation({
    mutationFn: createUser,
  })
}
```
-   Использование:

``` tsx
const { mutate } = useCreateUser()

mutate({ name: 'John' })
```

# Обновление данных после mutation

``` tsx
import { useQueryClient } from '@tanstack/react-query'

const queryClient = useQueryClient()

const mutation = useMutation({
  mutationFn: createUser,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['users'] })
  },
})
```
# Кеширование
-   Все данные кешируются по `queryKey`

``` tsx
useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
  staleTime: 1000 * 60, // данные считаются свежими 1 мин
})
```
# Параметризованные запросы

``` tsx
useQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
})
```
# Отключение авто-запроса
* Когда ты пишешь useQuery происходит следующее:
1. Компонент отрендерился
2. useQuery зарегистрировался
3. TanStack Query сам вызывает fetchUsers()
4. Ставит isLoading = true
5. Когда ответ пришёл → кладёт в data
``` tsx
useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
  enabled: false,
})
```
## Почему это называется “авто”
* Потому что тебе НЕ нужно делать:
```tsx
useEffect(() => {
  fetchUsers()
}, [])
```
* библиотека делает это за тебя
* “Автозапрос” — это не только первый вызвызов
  
  TanStack Query может сам перезапускать запрос, если:
	- компонент снова появился (mount)
	- окно стало активным (refetchOnWindowFocus)
	- сеть восстановилась
	- истёк staleTime

# Prefetch (предзагрузка)

``` tsx
import { useQueryClient } from '@tanstack/react-query'

const queryClient = useQueryClient()

queryClient.prefetchQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
})
```

# Работа вне компонентов
-   нужно использовать `queryClient`

``` ts
// queryClient.ts

import { QueryClient } from '@tanstack/react-query'

export const queryClient = new QueryClient()
```

``` ts
// где угодно

await queryClient.fetchQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
})
```
# DevTools

``` bash
pnpm add @tanstack/react-query-devtools
```

``` tsx
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'

<ReactQueryDevtools initialIsOpen={false} />
```
# Отличие от Zustand

-   Zustand → локальный state (UI, формы, флаги)
-   TanStack Query → серверные данные (API)

> ❗ Используются вместе, а не вместо друг друга
# Итог
TanStack Query решает: 
- кеширование API 
- синхронизацию данных 
- обновление данных (invalidate)
- состояния загрузки и ошибок
