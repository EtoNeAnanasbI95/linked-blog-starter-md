* 109 серия курса React + Redux от purple school
# Создание
```bash
pnpm create next-app --example with redux

# или докачать пакеты в уже сущетсвующий проект

pnpm add react-redux @reduxjs/toolkit
```
# Работа с Redux
* Короче эта тема, это глобальное хранилище состояния. Потому где-нибудь внутри проекта просто создайте глобальный стор состояния
```ts
// .../store/store.ts

import { configureStore } from '@reduxjs/toolkit'

export const store = configureStore({
	reducer: {}
})
```
* Внутри стора есть вот такие функции:
	* **dispatch** -- как в useReducer, вызов функции какого изменения состояния
	* **getState** -- получение состояния
	* **replaceReducer** -- если надо заменить какой-то редюсер
	* **subscribe** -- и подписка на изменение состояния
* Далее надо просто прокинуть типизацию
```ts
export type RootState = ReturnType<typeof store.getState>
export type AppDispatch = typeof store.dispatch
```
* Далее можно подключить сделанный стор, в наше приложение:
```tsx
<Provider store={store}>
 ...
</Provider>
```
* После этого можно сделать первый слайс состояния, в котором уже можно будет прописать ему редюсер
```ts
// .../user-slice.ts

import { createSlice } from '@reduxjs/toolkit'

export interface UserState { // интерфейс для состояния
	jwt: string | null;
}
const initialState: UserState = { // константа начального состояния
	jwt: null
｝；

export const userSlice = createSlice({
	name: 'user' // имя слайса для redux toolkit dev tools
	initialState,
	reducers: {
		addJwt: (state, action: PayloadAction<string>) => {state.jwt = action.payload}
	}
})

export userSlice.reducer
export const userActions = userSlice.actions
```
* В файлике стора добавим слайс
```ts
export const store = configureStore({
	reducer: {
		user: userSlice 
	}
})
```

* Всё, теперь можно использовать его в компонентах чтоб изменять состояние через редьюсеры
```tsx
import { FC } from 'react'

const AuthComponent: FC = () => {
	const dispatch = useDispatch<AppDispatch>()

	const onLogin = () => {
		dispatch(userActions.addJwt('jwt'))
	}
	return(
		<button onClick={() => onLogin()}>login</button>
	)
}
```
* И получать состояния через selectorы
```tsx
import { FC } from 'react'
import { useSelector } from 'react-redux'

const BebebeComponent: FC = () => {
	const jwt = useSelector((s: RootState) => s.user.jwt)
	return (
		<div>
			<p>Your jwt is: <span>{jwt}</span></p>
		</div>
	)
}
```
## asyncThunk
* В примере выше, я написал базовый пример того, как в целом можно стучаться в глобальное хранилище редакса
* НО так же внутри редакса есть крутая штука, называется asyncThunk, она создана как надстройка надо функциями из редюсера состояния, и позволяет запускать из ассинхнонно
### Работа с асинхронными запросами через createAsyncThunk

* Для работы с асинхронными операциями (например, сетевыми запросами) в Redux Toolkit используется `createAsyncThunk`. Это удобно, потому что `createAsyncThunk` автоматически создаёт три состояния жизненного цикла запроса: `pending` (начало), `fulfilled` (успех) и `rejected` (ошибка). Это позволяет легко отслеживать весь процесс выполнения запроса и обновлять состояние в зависимости от его статуса.

* Создаём асинхронный thunk для сетевого запроса, например, для авторизации:

```ts
// .../user-slice.ts
import { createAsyncThunk } from '@reduxjs/toolkit'

export const loginUser = createAsyncThunk(
  'user/login', // имя action для dev tools
  async (credentials: { email: string; password: string }, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        body: JSON.stringify(credentials),
      });
      const data = await response.json()
      if (!response.ok) {
        throw new Error(data.message || 'Login failed')
      }
      return data.jwt // возвращаем данные для fulfilled
    } catch (error) {
      return rejectWithValue(error.message) // возвращаем ошибку для rejected
    }
  }
)
```
- Для обработки состояний pending, fulfilled и rejected в слайсе используется extraReducers. Они позволяют обрабатывать действия, созданные вне редюсеров, например, асинхронные действия от createAsyncThunk. В extraReducers используется билдер для удобной обработки кейсов.
- Добавляем extraReducers в слайс для обработки состояний асинхронного запроса:
```ts
// .../user-slice.ts

export interface UserState {
  jwt: string | null;
  loading: boolean; // для отслеживания состояния загрузки
  error: string | null; // для хранения ошибки
}

const initialState: UserState = {
  jwt: null,
  loading: false,
  error: null,
};

export const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    addJwt: (state, action: PayloadAction<string>) => {
      state.jwt = action.payload;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(loginUser.pending, (state) => {
        state.loading = true; // запрос начался
        state.error = null; // сбрасываем ошибку
      })
      .addCase(loginUser.fulfilled, (state, action: PayloadAction<string>) => {
        state.loading = false; // запрос завершён
        state.jwt = action.payload; // сохраняем полученный JWT
      })
      .addCase(loginUser.rejected, (state, action) => {
        state.loading = false; // запрос завершён с ошибкой
        state.error = action.payload as string; // сохраняем ошибку
      });
  },
});

export const userSlice.reducer;
export const userActions = userSlice.actions;
```
* Теперь можно использовать loginUser в компонентах через dispatch:
```ts
import { FC } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { loginUser } from './user-slice';

const AuthComponent: FC = () => {
  const dispatch = useDispatch<AppDispatch>();
  const { loading, error } = useSelector((s: RootState) => s.user);

  const onLogin = () => {
    dispatch(loginUser({ email: 'user@example.com', password: 'password' }));
  };

  return (
    <div>
      <button onClick={onLogin} disabled={loading}>
        {loading ? 'Logging in...' : 'Login'}
      </button>
      {error && <p>Error: {error}</p>}
    </div>
  );
};
```