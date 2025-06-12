* [Zustand](https://zustand-demo.pmnd.rs/) -- Минималистичный, не глобальный менеджер состояния
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