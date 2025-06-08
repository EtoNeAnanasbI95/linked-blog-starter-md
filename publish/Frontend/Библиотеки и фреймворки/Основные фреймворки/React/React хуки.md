## Хуки
### useCallBack
* Эта тема нужна чтобы меморизировать какую-нибудь функцию. Что это значит? Мемиризировать — ввести функцию в механизм реакта, чтобы ей можно было пользоваться из разных скоупов и пространства имён. 
* То есть если условно, вы написали хук, который возвращает какую-то функцию, то вам в вашем хуке нужно завернуть эту функцию в useCallback, дабы она могла быть вызвана извне вашего самописного хука
### useRef
* Хук который меморизирует какой-то класс. То есть если вам условно нужна какая-то глобальная переменная внутри вашего компонента, или вам понадобилось управлять атрибутами какого-то конкретного HTML тэга в вёрстке вашего компонента, то вы можете сделать себе const `divRef = useRef<HTMLDivElement>(null)`, прокинуть его в ваш `<div ref={divRef} />` и либо получать какие-то данные о вашем диве, либо наоборот как-то им управлять
* Он отличается от стейта тем, что useRef это в сути переменная с полем current, где current это ссылка на какие-то данные, и живёт она вне рендеров. То есть если стейт обновился, всегда нужен ререндер, а при ref нет
### useId
* ничего особенного, просто хук, который делает тебе уникальный айдишник, который можно куда-нибудь пристроить, например если ты делаешь какой-то input и хочешь сделать ему встроенный label, то тебе понадобится по айдишнику, чтобы связать твой label и input, и когда у тебя  дохуища таких элементов интерфейса, то чтобы не накосяпорить можно прокидывать айдишиники этим хуком
### useState
* Хук для управления состоянием какой-то единичной переменной. То есть мы объявляем какую-то переменную состояния через useState
```tsx
const example: React.FC = () => {
	const [counter, setCounter] = useState<number>(0)


	return (
		<div>
			<button onClick={() => setCounter(counter+1)}>increase</button>
			<p>{counter}</p>
		</div>
	)
}
```
* То есть мы сделали переменную, которая несёт в себе какое состояние, и прокинули её в вёрстку. Сделали. мы это, чтобы ввести переменную в механизм реакта, дабы он следил за изменением её состояния, и в случае изменения, обновлял контент в вёрстке
### useReducer
* Хук, который используется для управления сложным состоянием в компоненте, когда логика обновления состояния становится громоздкой для `useState`. `useReducer` принимает редюсер (функцию, которая определяет, как обновлять состояние) и начальное состояние, а возвращает текущее состояние и функцию `dispatch` для отправки действий.
* Подходит для случаев, когда состояние зависит от предыдущего значения или когда нужно обрабатывать разные типы действий (например, добавление, удаление, обновление).
```ts
// .../toso-reducer.ts

// Типы для состояния и действий
interface TodoState {
  todos: Array<{ id: string; text: string; completed: boolean }>;
  filter: 'all' | 'active' | 'completed';
}

type TodoAction =
  | { type: 'ADD_TODO'; payload: string }
  | { type: 'TOGGLE_TODO'; payload: string }
  | { type: 'SET_FILTER'; payload: 'all' | 'active' | 'completed' };

// Начальное состояние
export const initialState: TodoState = {
  todos: [],
  filter: 'all',
};

// Редюсер для обработки действий
export const todoReducer = (state: TodoState, action: TodoAction): TodoState => {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [
          ...state.todos,
          { id: crypto.randomUUID(), text: action.payload, completed: false },
        ],
      };
    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map((todo) =>
          todo.id === action.payload ? { ...todo, completed: !todo.completed } : todo
        ),
      };
    case 'SET_FILTER':
      return {
        ...state,
        filter: action.payload,
      };
    default:
      return state;
  }
};
```

```tsx
// .../todo-component.tsx

import React, { useReducer, FC } from 'react';

// Компонент, использующий useReducer
const TodoApp: FC = () => {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  const [input, setInput] = useState('');

  const handleAddTodo = () => {
    if (input.trim()) {
      dispatch({ type: 'ADD_TODO', payload: input });
      setInput('');
    }
  };

  const filteredTodos = state.todos.filter((todo) => {
    if (state.filter === 'active') return !todo.completed;
    if (state.filter === 'completed') return todo.completed;
    return true;
  });

  return (
    <div>
      <h1>Список задач</h1>
      <input
        type="text"
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="Добавить задачу"
      />
      <button onClick={handleAddTodo}>Добавить</button>
      <div>
        <button onClick={() => dispatch({ type: 'SET_FILTER', payload: 'all' })}>
          Все
        </button>
        <button onClick={() => dispatch({ type: 'SET_FILTER', payload: 'active' })}>
          Активные
        </button>
        <button onClick={() => dispatch({ type: 'SET_FILTER', payload: 'completed' })}>
          Завершённые
        </button>
      </div>
      <ul>
        {filteredTodos.map((todo) => (
          <li
            key={todo.id}
            onClick={() => dispatch({ type: 'TOGGLE_TODO', payload: todo.id })}
            style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}
          >
            {todo.text}
          </li>
        ))}
      </ul>
    </div>
  );
};
```
* ***Описание:***
	* В этом примере создаётся компонент TodoApp, который управляет списком задач с помощью useReducer.
	* Состояние (TodoState) включает массив задач (todos) и фильтр (filter) для отображения всех, активных или завершённых задач.
	* Редюсер (todoReducer) обрабатывает три типа действий: ADD_TODO (добавление задачи), TOGGLE_TODO (переключение статуса задачи) и SET_FILTER (изменение фильтра).
	* Пользователь может вводить текст в поле, добавлять задачи, переключать их статус (завершена/не завершена) и фильтровать список через кнопки.
	* Функция dispatch отправляет действия в редюсер, который обновляет состояние. Например, при добавлении задачи создаётся новая задача с уникальным ID через crypto.randomUUID().
* ***Почему это удобно:***
	* useReducer делает код более предсказуемым и структурированным, особенно когда состояние сложное или зависит от множества операций.
	* Редюсер изолирует логику обновления состояния, что упрощает тестирование и отладку.
	* Подходит для сценариев, где нужно обрабатывать разные типы действий, как в этом примере с добавлением, переключением и фильтрацией задач.
	* Легко расширять: можно добавить новые действия, просто обновив TodoAction и todoReducer.
### useMemo
* Короче это хук который позволяет  [[Frontend/TS & JS/TS & JS#Мемоизация]] какое-то значение. То есть
```tsx
const Example: React.FC = () => {
    const [number, setNumber] = useState<number>(0);

    const factorial = useMemo(() => {
        const calculateFactorial = (n: number): number => {
            return n <= 0 ? 1 : n * calculateFactorial(n - 1)
        };
        return calculateFactorial(number)
    }, [number])

    return (
        <div>
            <button onClick={() => setNumber(number + 1)}>Увеличить</button>
            <p>Число: {number}</p>
            <p>Факториал: {factorial}</p>
        </div>
    )
}

```
* То есть чё происходит — **Как работает** ⁠useMemo:
	1. **Первый рендер**:
		▪ Когда компонент впервые рендерится, ⁠useMemo выполняет функцию, переданную в него, и сохраняет её результат.
	2. **Изменение зависимостей**:
		▪ В следующий раз, когда компонент будет рендериться, ⁠useMemo проверяет зависимости (в нашем примере это переменная ⁠number).
		▪ Если ⁠number изменился (например, если ты нажмёшь кнопку для увеличения числа), ⁠useMemo снова выполнит функцию и обновит сохранённое значение.
		▪ Если ⁠number не изменился, ⁠useMemo вернёт предыдущее сохранённое значение без выполнения функции.
	3. **Каждый рендер**:
		▪ Таким образом, ⁠useMemo "срабатывает" каждый раз, когда компонент рендерится, но фактические вычисления происходят только при изменении указанных зависимостей.
### useEffect
* Хук ⁠useEffect выполняется после рендеринга компонента. Это значит, что он запускается после того, как все изменения были применены к DOM
	1. **Когда срабатывает**:
		▪ Хук ⁠useEffect выполняется после рендеринга компонента. Это значит, что он запускается после того, как все изменения были применены к DOM.
		▪ Основная задача — выполнять побочные эффекты (например, запросы к API, подписки, таймеры и т. д.).
	2. **Как работает**:
		▪ При первом рендере ⁠useEffect выполняет переданную функцию.
		▪ При последующих рендерах ⁠useEffect срабатывает после каждого обновления компонента.
		▪ Если ты передаёшь второй аргумент — массив зависимостей, ⁠useEffect выполнится только тогда, когда хотя бы одно из значений в этом массиве изменится.
		▪ Если передать пустой массив (т.е. ⁠[]), ⁠useEffect сработает только один раз при монтировании компонента.
	3. **Очистка**:
		▪ Если передать функции, возвращаемые из ⁠useEffect, этот код будет выполнен при размонтировании компонента или перед следующими вызовами эффекта.
```tsx
const Example: React.FC = () => {
	const [count, setCount] = useState(0)
	useEffect(() => {
		console.log('Компонент обновлён или монтирован')
		return () => {
			console.log('Компонент размонтирован или перед следующим рендером')
		}
	}, [count]) // Зависимость от count
	return (
        <div>
            <button onClick={() => setCount(count + 1)}>Увеличить</button>
            <p>Счёт: {count}</p>
        </div>
    )
}
```
* useEffect выполняется асинхронно после рендеринга и не блокирует отображение
### useLayoutEffect
* Грубо говоря брат скромник useEffect
1. **Когда срабатывает**:
	▪ Хук ⁠useLayoutEffect срабатывает синхронно после всех изменений в DOM, но перед тем, как браузер отрисует изменения
	▪ Это может быть полезно, когда нужно выполнить какие-то действия, которые должны произойти перед визуализацией обновлений на экране
2. **Как работает**:
	▪ При первом рендере ⁠useLayoutEffect выполняется после изменения DOM, но до отрисовки.
	▪ Также как и ⁠useEffect, он может принимать массив зависимостей. Если зависимости изменяются, он будет выполняться снова
	▪ Функция, возвращаемая из ⁠useLayoutEffect, также может использоваться для очистки
3. **Применение**:
	▪ Он лучше подходит для случаев, когда нужно работать непосредственно с размерами или положением DOM-элементов, или если отложенные обновления могут вызвать мерцание интерфейса
```tsx
const Example: React.FC = () => {
    const [width, setWidth] = useState(0)
    const divRef = useRef<HTMLDivElement>(null)

    useLayoutEffect(() => {
        if (divRef.current) {
            setWidth(divRef.current.getBoundingClientRect().width)
        }
    })

    return (
        <div ref={divRef}>
            <p>Ширина элемента: {width}px</p>
        </div>
    )
}
```
* useLayoutEffect выполняется синхронно до отрисовки и может блокировать интерфейс, если используется неправильно
### useTransition
* Хук который позволяет делать какие-то отложенные изменения в интерфейсе, потому что позволяет запустить исполнение определённой тяжёлой функции в менее приоритетном ассинхронном режиме в [[Runtime#Браузерный]] 
* 
```tsx
const Example: React.FC = () => {
    const [input, setInput] = useState('')
    const [list, setList] = useState<string[]>([])
    const [isPending, startTransition] = useTransition()

    const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
        const value = e.target.value;
        setInput(value);
        startTransition(() => {
            const newList = Array.from({ length: 10000 }, (_, i) => `${value} ${i}`)
            setList(newList)
        })
    }
    return (
        <div>
            <input type="text" value={input} onChange={handleChange} />
            {isPending && <p>Загрузка...</p>}
            <ul>
                {list.map((item, index) => (
                    <li key={index}>{item}</li>
                ))}
            </ul>
        </div>
    )
}
```
* ***Описание:**
	* В этом примере создаётся простой компонент, где пользователь может вводить текст в поле ввода.
	* При изменении текста в поле, срабатывает функция ⁠handleChange, которая обновляет значение ⁠input и начинает новую транзакцию, оборачивая обновление списка в ⁠startTransition.
	* Если происходит обновление состояния списка (например, при вводе большого количества данных), React отображает индикатор загрузки ("Загрузка...") до завершения этой транзакции. Это обеспечивает более плавный переход между состояниями.
* Хук ⁠useTransition не создает новый поток, а позволяет пометить определённые операции как менее важные в контексте рендеринга. Это означает, что когда вы используете ⁠startTransition, вы говорите React, что данное обновление не критично для отзывчивости пользовательского интерфейса.
* React может обработать изменения состояния внутри ⁠startTransition позже, когда у него будет время, сохраняя при этом высокую отзывчивость интерфейса.
### useContext
* Хук, который позволяет получать данные из React Context без необходимости передавать пропсы через всю иерархию компонентов. Контекст полезен для глобальных данных, таких как настройки темы, данные пользователя или другие общие состояния.
* `useContext` используется в связке с `createContext` и провайдером, который оборачивает компоненты, чтобы передать им данные. Внутри провайдера можно управлять состоянием через `useState` или `useReducer`.
```tsx
// .../theme-context.ts

import React, { createContext, useContext, useState, FC } from 'react';

// Создаём контекст
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// Провайдер с состоянием
const ThemeProvider: FC<{ children: React.ReactNode }> = ({ children }) => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const toggleTheme = () => {
    setTheme((prev) => (prev === 'light' ? 'dark' : 'light'));
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// Кастомный хук для удобного доступа к контексту
const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};
```

```tsx
// .../theme-toggle-button.tsx

// Пример компонента, использующего контекст
const ThemeToggle: FC = () => {
  const { theme, toggleTheme } = useTheme();

  return (
    <div style={{ background: theme === 'light' ? '#fff' : '#333', color: theme === 'light' ? '#000' : '#fff' }}>
      <p>Текущая тема: {theme}</p>
      <button onClick={toggleTheme}>Переключить тему</button>
    </div>
  );
};

```

```tsx
// .../App.tsx

// Использование в приложении
const App: FC = () => {
  return (
    <ThemeProvider>
      <div>
        <h1>Пример работы с useContext</h1>
        <ThemeToggle />
      </div>
    </ThemeProvider>
  );
};
```
* ***Описание:***
	* В этом примере создаётся контекст ThemeContext для управления темой приложения (светлая/тёмная).
	* Компонент ThemeProvider использует useState для хранения текущей темы (light или dark) и предоставляет функцию toggleTheme для её изменения. Провайдер оборачивает дочерние компоненты и передаёт им данные через value.
	* Кастомный хук useTheme упрощает доступ к контексту и добавляет проверку, чтобы убедиться, что хук используется внутри ThemeProvider.
	* Компонент ThemeToggle использует useTheme для получения текущей темы и функции переключения. При нажатии на кнопку тема меняется, а интерфейс обновляется в зависимости от значения theme.
	* Стили в компоненте ThemeToggle зависят от текущей темы, что демонстрирует, как контекст влияет на UI.
* ***Почему это удобно:***
	* useContext позволяет избежать "prop drilling" — передачи пропсов через множество промежуточных компонентов.
	* Состояние в провайдере (в данном случае theme) централизованно, и любое изменение автоматически обновляет все компоненты, которые используют этот контекст.
	* Кастомный хук (useTheme) делает код чище и предотвращает ошибки, если контекст не определён.
	* Подходит для глобальных данных, таких как аутентификация, настройки или локализация, которые нужно использовать в разных частях приложения.