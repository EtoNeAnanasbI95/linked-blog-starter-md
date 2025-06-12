# Особенности работы со стейт-менеджерами в Next.js

Next.js, как фреймворк с поддержкой серверного рендеринга (SSR), статической генерации (SSG) и React Server Components (RSC), накладывает определённые ограничения и требования на использование стейт-менеджеров, таких как Redux, Zustand, Jotai или MobX. В отличие от классических SPA (Single Page Applications), где состояние обычно хранится глобально на клиенте, в Next.js необходимо учитывать особенности серверного окружения и гидратации. В этом разделе описаны ключевые аспекты работы со стейт-менеджерами и рекомендации по их правильной настройке.

## Проблемы глобального состояния на сервере

При использовании SSR в Next.js сервер обрабатывает запросы от разных пользователей одновременно. Если стейт-менеджер инициализируется как глобальный объект на уровне модуля (например, в виде синглтона), это может привести к **утечке данных** между запросами. Один пользователь может случайно получить доступ к состоянию, созданному для другого.

**Пример проблемы (неправильный подход):**

```ts
// ❌ Неправильно: глобальный стор на уровне модуля
import { create } from 'zustand';

export const useStore = create((set) => ({
  counter: 0,
  increment: () => set((state) => ({ counter: state.counter + 1 })),
}));
```

В этом случае `useStore` будет общим для всех запросов на сервере, что приведёт к непредсказуемому поведению.

**Решение**: Создавайте стор для каждого запроса отдельно, используя **фабричную функцию** и React Context, чтобы изолировать состояние.

## Использование React Context для изоляции состояния

Для корректной работы стейт-менеджера в Next.js рекомендуется использовать React Context для предоставления изолированного стора каждому запросу. Это особенно важно при SSR, чтобы серверный и клиентский рендеринг имели одинаковое состояние и не возникали ошибки гидратации.

**Пример с Zustand и Context:**

```ts
// store/counterStore.ts
import { createContext, useContext } from 'react';
import { createStore, useStore } from 'zustand';

// Типы для стора
type CounterState = {
  counter: number;
  increment: () => void;
  decrement: () => void;
};

// Фабричная функция для создания стора
const createCounterStore = () =>
  createStore<CounterState>((set) => ({
    counter: 0,
    increment: () => set((state) => ({ counter: state.counter + 1 })),
    decrement: () => set((state) => ({ counter: state.counter - 1 })),
  }));

// Создаём контекст
const CounterStoreContext = createContext<ReturnType<typeof createCounterStore> | null>(null);

// Провайдер для стора
export const CounterStoreProvider = ({ children }: { children: React.ReactNode }) => {
  const store = createCounterStore();
  return (
    <CounterStoreContext.Provider value={store}>
      {children}
    </CounterStoreContext.Provider>
  );
};

// Хук для доступа к стору
export const useCounterStore = <T,>(selector: (state: CounterState) => T): T => {
  const store = useContext(CounterStoreContext);
  if (!store) {
    throw new Error('CounterStoreContext is not provided');
  }
  return useStore(store, selector);
};
```

**Использование в компоненте:**

```tsx
// app/page.tsx
'use client'; // Обязательно для клиентских компонентов

import { CounterStoreProvider, useCounterStore } from '../store/counterStore';

export default function Page() {
  return (
    <CounterStoreProvider>
      <Counter />
    </CounterStoreProvider>
  );
}

function Counter() {
  const counter = useCounterStore((state) => state.counter);
  const increment = useCounterStore((state) => state.increment);
  const decrement = useCounterStore((state) => state.decrement);

  return (
    <div>
      <button onClick={increment}>+</button>
      <span>{counter}</span>
      <button onClick={decrement}>-</button>
    </div>
  );
}
```

**Объяснение**:

- Фабричная функция `createCounterStore` создаёт новый экземпляр стора для каждого запроса.
- `CounterStoreContext` изолирует стор, предоставляя его через провайдер.
- Директива `'use client'` указывает, что компонент является клиентским, так как стейт-менеджеры работают только на клиенте.
- Такой подход предотвращает утечки данных и обеспечивает корректную гидратацию.

## Работа с React Server Components (RSC)

С появлением App Router и React Server Components в Next.js 13+ стейт-менеджеры требуют дополнительного внимания. Серверные компоненты **не могут напрямую взаимодействовать** с клиентским состоянием, так как они рендерятся на сервере и не имеют доступа к хукам или клиентским библиотекам.

**Рекомендации**:

1. Инициализируйте стор только в **клиентских компонентах** (помеченных `'use client'`).
2. Используйте серверные компоненты для получения данных (например, через `fetch`) и передавайте их в клиентские компоненты через props.
3. Размещайте провайдер стейт-менеджера в клиентском компоненте, который оборачивает дочерние компоненты.

**Пример интеграции с RSC:**

```tsx
// app/page.tsx
// Серверный компонент
async function fetchInitialCount() {
  // Эмуляция получения данных
  return { initialCount: 42 };
}

export default async function Page() {
  const data = await fetchInitialCount();
  return (
    <CounterStoreProvider initialCount={data.initialCount}>
      <Counter />
    </CounterStoreProvider>
  );
}

// store/counterStore.ts
import { createContext, useContext } from 'react';
import { createStore, useStore } from 'zustand';

type CounterState = {
  counter: number;
  increment: () => void;
  decrement: () => void;
};

const createCounterStore = (initialCount: number) =>
  createStore<CounterState>((set) => ({
    counter: initialCount,
    increment: () => set((state) => ({ counter: state.counter + 1 })),
    decrement: () => set((state) => ({ counter: state.counter - 1 })),
  }));

const CounterStoreContext = createContext<ReturnType<typeof createCounterStore> | null>(null);

export const CounterStoreProvider = ({
  children,
  initialCount,
}: {
  children: React.ReactNode;
  initialCount: number;
}) => {
  const store = createCounterStore(initialCount);
  return (
    <CounterStoreContext.Provider value={store}>
      {children}
    </CounterStoreContext.Provider>
  );
};

export const useCounterStore = <T,>(selector: (state: CounterState) => T): T => {
  const store = useContext(CounterStoreContext);
  if (!store) {
    throw new Error('CounterStoreContext is not provided');
  }
  return useStore(store, selector);
};

// components/Counter.tsx
'use client';

import { useCounterStore } from '../store/counterStore';

export function Counter() {
  const counter = useCounterStore((state) => state.counter);
  const increment = useCounterStore((state) => state.increment);
  const decrement = useCounterStore((state) => state.decrement);

  return (
    <div>
      <button onClick={increment}>+</button>
      <span>{counter}</span>
      <button onClick={decrement}>-</button>
    </div>
  );
}
```

**Объяснение**:

- Серверный компонент (`Page`) запрашивает начальные данные (например, `initialCount`) и передаёт их в `CounterStoreProvider`.
- `CounterStoreProvider` инициализирует стор с переданными данными.
- Клиентский компонент (`Counter`) использует хук `useCounterStore` для доступа к состоянию.

## Совместимость с Redux DevTools

Многие стейт-менеджеры, такие как Zustand или Redux, поддерживают интеграцию с Redux DevTools для удобной отладки. В Next.js это работает аналогично, но важно инициализировать middleware только на клиенте, чтобы избежать ошибок на сервере.

**Пример с Zustand и Redux DevTools:**

```ts
// store/counterStore.ts
import { createContext, useContext } from 'react';
import { createStore, useStore } from 'zustand';
import { devtools } from 'zustand/middleware';

type CounterState = {
  counter: number;
  increment: () => void;
  decrement: () => void;
};

const createCounterStore = () =>
  createStore<CounterState>()(
    devtools(
      (set) => ({
        counter: 0,
        increment: () => set((state) => ({ counter: state.counter + 1 })),
        decrement: () => set((state) => ({ counter: state.counter - 1 })),
      }),
      { name: 'CounterStore', enabled: process.env.NODE_ENV === 'development' }
    )
  );

const CounterStoreContext = createContext<ReturnType<typeof createCounterStore> | null>(null);

export const CounterStoreProvider = ({ children }: { children: React.ReactNode }) => {
  const store = createCounterStore();
  return (
    <CounterStoreContext.Provider value={store}>
      {children}
    </CounterStoreContext.Provider>
  );
};

export const useCounterStore = <T,>(selector: (state: CounterState) => T): T => {
  const store = useContext(CounterStoreContext);
  if (!store) {
    throw new Error('CounterStoreContext is not provided');
  }
  return useStore(store, selector);
};
```

**Объяснение**:

- Middleware `devtools` подключается с опцией `{ enabled: process.env.NODE_ENV === 'development' }`, чтобы работать только в режиме разработки.
- Параметр `name` задаёт имя стора в Redux DevTools для удобной идентификации.

## Рекомендации по выбору стейт-менеджера

- **Zustand**: Лёгкий, минималистичный, хорошо подходит для небольших и средних приложений. Простая интеграция с Context для SSR.
- **Redux Toolkit**: Подходит для сложных приложений с большим количеством асинхронной логики. Требует больше настроек для SSR (например, через `getServerSideProps` или `getInitialProps`).
- **Jotai**: Отлично подходит для атомарного управления состоянием, особенно в связке с RSC, так как позволяет разбивать состояние на мелкие части.
- **MobX**: Удобен для приложений с реактивной моделью данных, но требует осторожности при SSR из-за возможных сложностей с гидратацией.

## Полезные советы

- **Избегайте хуков в серверных компонентах**: Хуки стейт-менеджеров (например, `useSelector`, `useStore`) должны использоваться только в клиентских компонентах.
- **Оптимизируйте гидратацию**: Убедитесь, что начальное состояние, переданное с сервера, совпадает с клиентским, чтобы избежать ошибок гидратации.
- **Используйте `getServerSideProps` или `getStaticProps` для начальных данных**: Передавайте начальное состояние в стор через props.
- **Тестируйте в SSR-режиме**: Проверяйте, как ваше приложение ведёт себя при серверном рендеринге, чтобы выявить потенциальные проблемы с состоянием.

## Ресурсы

- [Официальная документация Next.js по работе с данными](https://nextjs.org/docs/app/building-your-application/data-fetching)
- [Zustand: документация](https://zustand-demo.pmnd.rs/)
- [Redux Toolkit: настройка с Next.js](https://redux-toolkit.js.org/usage/nextjs)