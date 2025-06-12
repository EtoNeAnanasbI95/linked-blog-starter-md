* [Zustand](https://zustand-demo.pmnd.rs/) -- Минималистичный, не глобальный менеджер состояния

> [!TIP] Референсный проект
> Вот референсный проект, на котором можно посмореть не плохой пример кода: https://github.com/AlariCode/4-mini-zustand

# Инициализация
* Установка:
```bash
pnpm add zustand
```
* Инициализация будет на примере проекта "счётчик":

```tsx
// .../counter/counterStore.ts

import { create, StateCreator } from 'zustand'

type CounterState = {
	counter: number
}

type CounterActions = {
	increment: () = void
	decrement: () = void
}

const counterSlice: StateCreator<CounterState & CounterActions> = (set, get) => ({
	counter: 0 // default value of slice
	increment: () => {
		const { counter } = get()
		set((state) => ...state, counter: state.counter + 1 })) // or
		// set({ counter: counter-- })
	}
	decrement: () => {
		const { counter } = get()
		set((state) => ...state, counter: state.counter - 1 })) // or
		// set({ counter: counter++ })
	}
})

export const useCounterStore = create<CounterState & CounterActions>(counterSlice)
```
* Теперь можно просто взять и воспользоваться хуком в компоненте
```tsx
// app.tsx

import { FC } from 'react'

const App: FC = () => {
	const { counter, decrement, increment } = useCounterStore()
	return(
		<div>
			<button onClick={increment}>+</button>
			<span>{counter}</span>
			<button onClick={decrement}>-</button>
		</div>
	)
}

export default App
```
# Манипуляции с состоянием вне компонентов
 * Вне компонентов и хуков мне не можем использовать хук useCounterStore, но допустим нам надо каким-то образом изменить счётчик в какой-нибудь функции. Допустим я напишу в папке helpers файлик addTen.ts, но сначала поправлю counterStore

```ts
// .../counter/counterStore.ts

import { create, StateCreator } from 'zustand'

type CounterState = {
	counter: number
}

type CounterActions = {
	increment: () = void
	decrement: () = void
	incrementByAmount: (value: number) => void
}

const counterSlice: StateCreator<CounterState & CounterActions> = (set, get) => ({
	counter: 0 // default value of slice
	increment: () => {
		const { counter } = get()
		set((state) => ...state, counter: state.counter + 1 })) // or
		// set({ counter: counter-- })
	}
	decrement: () => {
		const { counter } = get()
		set((state) => ...state, counter: state.counter - 1 })) // or
		// set({ counter: counter++ })
	}
	incrementByAmount: (value: number) => {
	    const { counter } = get();
	    set({ counter: counter + value })
	},
})

export const useCounterStore = create<CounterState & CounterActions>(counterSlice)

export const incrementByAmount = (value: number) => {
	useCounterStore.getState().incrementByAmount(value)
} // функция которую можно вызвать откуда угодно

export const getCounter = () => useCounterStore.getState().counter // гетер с актуальным сосотоянием
```

```ts
// .../helpers/addTen.ts

import { getCounter, incrementByAmount } from ".../counter/counterStore";

export const addTen = () => {
  const counter = getCounter();
  console.log(counter);
  if (counter >= 0) {
    incrementByAmount(10);
  } else {
    incrementByAmount(-10);
  }
}
```
* И теперь я могу использовать этот addTen, например у себя в компоненте
```tsx
// app.tsx

import { FC } from 'react'
import { addTen } from "./helpers/addTen";


const App: FC = () => {
	const { counter, decrement, increment } = useCounterStore()

	return(
		<div>
			<button onClick={increment}>+</button>
			<span>{counter}</span>
			<button onClick={decrement}>-</button>
			<button onClick={addTen}>+10</button>
		</div>
	)
}

export default App
```
# Добавление Redux DevTools для отладки

- Zustand поддерживает интеграцию с Redux DevTools через middleware devtools, что позволяет отслеживать изменения состояния в браузерном расширении Redux DevTools.
- Для этого нужно установить пакет zustand/middleware и обновить counterStore.ts.
- Установка (если ещё не установлен zustand/middleware):
- Обновление counterStore.ts с использованием devtools:
```ts
// .../counter/counterStore.ts

import { create, StateCreator } from 'zustand'
import { devtools } from 'zustand/middleware'

type CounterState = {
	counter: number
}

type CounterActions = {
	increment: () => void
	decrement: () => void
	incrementByAmount: (value: number) => void
}

const counterSlice: StateCreator<CounterState & CounterActions, [["zustand/devtools", never]]> = (set, get) => ({
	counter: 0,
	increment: () => {
		const { counter } = get()
		set((state) => ({ ...state, counter: state.counter + 1 }), `increment counter to ${state.counter + 1}`)
	},
	decrement: () => {
		const { counter } = get()
		set((state) => ({ ...state, counter: state.counter - 1 }), `decrement counter to ${state.counter - 1}`)
	},
	incrementByAmount: (value: number) => {
		const { counter } = get()
		set({ counter: counter + value }, `increment counter on ${value}`)
	},
})

export const useCounterStore = create<CounterState & CounterActions>()(
	devtools(counterSlice, { name: 'CounterStore' }) // Добавляем devtools с именем стора
)

export const incrementByAmount = (value: number) => {
	useCounterStore.getState().incrementByAmount(value)
}

export const getCounter = () => useCounterStore.getState().counter
```
- **Объяснение**:
    - Импортируем devtools из zustand/middleware.
    - Оборачиваем counterSlice в devtools, указывая имя стора (CounterStore) для отображения в Redux DevTools.
    - Теперь все изменения состояния (например, increment, decrement, incrementByAmount) будут видны в Redux DevTools.
    - Убедитесь, что расширение Redux DevTools установлено в браузере.
    - Имя стора (CounterStore) помогает идентифицировать стор в DevTools, особенно если используется несколько сторов.
- **Примечание**: Никаких изменений в компонентах или других частях кода не требуется — Redux DevTools автоматически подхватит изменения состояния.