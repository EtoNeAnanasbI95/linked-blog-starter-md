```table-of-contents
```

# Семейство библиотек TanStack

[TanStack](https://tanstack.com/) — коллекция open-source библиотек для упрощения фронтенд-разработки, включая управление состоянием, асинхронными данными, таблицами и маршрутизацией. Они поддерживают React, Vue, Solid, Svelte, Angular, Qwik, Lit и идеально интегрируются с Next.js благодаря типобезопасности и поддержке SSR/SSG.

## Основные библиотеки

### 1. TanStack Query

- **Назначение**: Управление асинхронным серверным состоянием (запросы, кэширование, синхронизация).
- **Особенности**:
    - Автоматическое кэширование и фоновое обновление.
    - Оптимистичные обновления и мутации.
    - Поддержка SSR/SSG в Next.js.
    - Типобезопасность с TypeScript.
- **Пример**:
    
    ```typescript
    import { QueryClient, QueryClientProvider, useQuery } from '@tanstack/react-query';
    import { dehydrate } from '@tanstack/react-query';
    
    const queryClient = new QueryClient();
    
    export async function getServerSideProps() {
      await queryClient.prefetchQuery(['posts'], () =>
        fetch('https://api.example.com/posts').then((res) => res.json())
      );
      return { props: { dehydratedState: dehydrate(queryClient) } };
    }
    
    function PostsPage() {
      const { data } = useQuery(['posts'], () =>
        fetch('https://api.example.com/posts').then((res) => res.json())
      );
      return <div>{data?.map((post) => <p key={post.id}>{post.title}</p>)}</div>;
    }
    
    export default function App({ dehydratedState }) {
      return (
        <QueryClientProvider client={queryClient} dehydratedState={dehydratedState}>
          <PostsPage />
        </QueryClientProvider>
      );
    }
    ```
    

### 2. [[TanStack Tables]]

- **Назначение**: Создание таблиц и датагридов с полной свободой в стилизации.
- **Особенности**:
    - Headless UI: только логика (сортировка, фильтрация, пагинация).
    - Виртуализация для больших данных.
    - Поддержка Server Components в Next.js.
- **Пример**:
    
    ```typescript
    import { useReactTable, getCoreRowModel } from '@tanstack/react-table';
    
    export default function Table({ data }) {
      const columns = [
        { accessorKey: 'id', header: 'ID' },
        { accessorKey: 'name', header: 'Name' },
      ];
      const table = useReactTable({
        data,
        columns,
        getCoreRowModel: getCoreRowModel(),
      });
    
      return (
        <table>
          <thead>
            {table.getHeaderGroups().map((headerGroup) => (
              <tr key={headerGroup.id}>
                {headerGroup.headers.map((header) => (
                  <th key={header.id}>{header.column.columnDef.header}</th>
                ))}
              </tr>
            ))}
          </thead>
          <tbody>
            {table.getRowModel().rows.map((row) => (
              <tr key={row.id}>
                {row.getVisibleCells().map((cell) => (
                  <td key={cell.id}>{cell.getValue()}</td>
                ))}
              </tr>
            ))}
          </tbody>
        </table>
      );
    }
    ```
    

### 3. TanStack Router

- **Назначение**: Типобезопасная маршрутизация.
- **Особенности**:
    - Типобезопасные маршруты с автодополнением.
    - Поддержка вложенных маршрутов и предварительной загрузки.
    - Интеграция с Next.js App Router.
- **Пример**:
    
    ```typescript
    import { createRouter, RouterProvider } from '@tanstack/react-router';
    
    const router = createRouter({
      routeTree: {
        path: '/',
        component: () => <div>Home</div>,
        children: [
          { path: '/about', component: () => <div>About</div> },
        ],
      },
    });
    
    export default function App() {
      return <RouterProvider router={router} />;
    }
    ```
    

### 4. TanStack Start

- **Назначение**: Full-stack React-фреймворк.
- **Особенности**:
    - SSR, потоковая передача, API-маршруты.
    - Интеграция с TanStack Router и Query.
- **Применение**: Альтернатива Next.js с большей модульностью.

### 5. TanStack Form

- **Назначение**: Управление формами с типобезопасностью.
- **Особенности**:
    - Headless UI для форм.
    - Поддержка Server Actions в Next.js.
- **Пример**:
    
    ```typescript
    import { useForm } from '@tanstack/react-form';
    
    export default function Form() {
      const form = useForm({
        defaultValues: { name: '' },
        onSubmit: async ({ value }) => {
          console.log(value);
        },
      });
    
      return (
        <form.Provider>
          <form.Field name="name">
            {(field) => (
              <input
                value={field.state.value}
                onChange={(e) => field.handleChange(e.target.value)}
              />
            )}
          </form.Field>
          <button type="submit">Submit</button>
        </form.Provider>
      );
    }
    ```
    

### 6. TanStack Virtual

- **Назначение**: Виртуализация списков и таблиц.
- **Особенности**: Лёгкая и типобезопасная виртуализация для больших данных.

### 7. Другие библиотеки

- **TanStack Ranger**: Управление диапазонами (слайдеры).
- **TanStack Loaders**: Дебансинг, троттлинг, управление очередями.
- И прочие...

## Преимущества

- **Типобезопасность**: Полная поддержка TypeScript.
- **Headless UI**: Гибкость в дизайне и разметке.
- **Кросс-фреймворковая поддержка**: React, Vue, Solid, Svelte и др.
- **Интеграция с Next.js**: Поддержка SSR, SSG, ISR, Server Components.
- **Активное сообщество**: Регулярные обновления (например, Query v5).[](https://x.com/TkDodo/status/1714262102305632643)

## Итог

TanStack — универсальный набор библиотек для упрощения фронтенд-разработки. Их типобезопасность и интеграция с Next.js делают их идеальными для масштабируемых приложений. Подробности: [официальный сайт TanStack](https://tanstack.com/).