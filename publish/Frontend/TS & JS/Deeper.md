
# Readonly
Readonly позволяет делать переменные константами (неизменяемыми) 
- Readonly в структурах
```tsx
type Color = 'red' | 'green' | 'blue';

function paint(color: Color) {
	console.log(color)
}

const values = {
	color: 'green'
}

paint(values.color); //Ошибка: так как поле color изменяемое TS не распознает в нем Color

const values2 = {
	color: 'green'
} as const //Тут мы явно указываем что все поля readonly

paint(values2.color); //Ошибки не будет
```
- Readonly в типах
```tsx
interface User {
	readonly id: number; //Такое поле TS не даст нам поменять
	name: string;
}
```

# Сужение типов
Иногда нужно определить с каким типом мы сейчас работаем, что правильно его обработать, тут как раз и помогает сужение типов.
- Проверка типа
```tsx
function fn(arg: number | string | null) {
	if (typeof arg === 'string') {
		//Выполняем действия связанные с string
	} else if (typeof arg === 'number') {
		//Выполняем действия связанные с number
	}
	
	return arg; //Здесь значение точно null и TS это знает 
}
```
- Нахождение поля в модели
```tsx
type User = {
	firstname: string;
	lastname: string;
}

type Person = {
	firstname: string;
	lastname: string;
	age: number;
}

function fn(arg: User | Person) {
	if('age' in arg) {
		//Здесь мы получаем горантировано Person
	}
}
```
- Нахождение по классам
```tsx
class Bmw {}
class Audi {}

function fn(arg: Bmw | Audi) {
	if(arg instanceof Bmw) { //Работате только с классами, ни с type, ни с interface такое не сработает
		//Здесь TS знает, что arg это класс Bmw
	}
}
```
- Type Guard - пользовательский механизм для сужения типов
```tsx
interface Car {
	maxSpeed: number;
	width: number;
}

interface Person {
	age: number;
	name: string;
}

function isCar(value: unknown): value is Car {
	return 'maxSpeed' is value && 'width' is valuse;
}

function isPerson(value: unknown): value is Person {
	return 'age' is value && 'name' is valuse;
}

function fn(data: Car | Person) {
	if(isCar(data)) {
		//TS тут будет значть, что это машина
	}
	
	if(isPerson(data)) {
		//Аналогично с машиной, только персона
	}
}
```
# Преобразование типов
- Неявное
```tsx
let a = 10
let b = a + '123' //Тут a станет string и js их просто склеинт
```
- Явное
```tsx
interface Person {
	age: number;
	name: number;
}

const obj = {
	age: 25,
	name: '123'
} as Person
//Тут obj стал Person, но этот способ не безопасный

interface Person2 {
	age: number;
	name: string;
	password: string;
}

const obj2 = {
	age: 25,
	name: '123'
} as Person2
//Тут obj2 стал Person2, не смотря на то, что ему не хватает полей
```
as лучше не использовать в продакшене (можно получить пизды от лида)
Где допустимо использовать as:
- В конфигах
- В тестах
- При работе с html элементами (иногда)
- В утилитарных функциях
```tsx
function JSONParse<T>(data: string): T {
	return JSON.parse(data) as T; //Это утилитарная функция и тут так можно
}

function keys<T>(data: T extends object): Array<keyof T> {
	return Object.keys(data) as Array<keyof T>; //Заместо массива строк получаем массив литералов
}
```
- Когда ТС не может вывести тип
```tsx
const parsedJson = JSON.parse('{age: 25}') //Тут допустимо делать as так, как иначе будет any

async function fn() {
	const data = await fetch('')
	cont parsedData = await data.json() //Тут тоже можно, данные все равно не безопасные и прийти может что угодно
}
```

# Mapped Types
Mapped types - это действия над существующими типами (сделать поле или поля необязательными или readonly и т.д.)
- Изменение типов
```tsx
interface User {
	name: string;
	age: number;
	friends: Array<string>;
}

type ReadonlyType<T> = {
	readonly [Key in keyof T]: T[Key]; //Здесь мы получем все ключи из T и записываем в Key,
									   //а после уже достаём типы данных из T по ключам из Key
}

type NewUser = ReadonlyType<User>;

//По аналогии можно сделать поля необязательными или null
type OptionalType<T> = {
	[Key in keyof T]?: T[Key] | null; 
}

type OptionalUser = OptionalType<User>;

//Также можно отменять readonly и опциональность
type EditType<T> = {
	-readonly [Key in keyof T]-?: T[Key]; 
}

type EditUser = EditType<NewUser>;
```
- Создание аналогов типов
```tsx
type AnalogMap<T> = {
	[K in number]: T;
}

const map: AnalogMap<string> = {
	5: '123',
	6: '321'
}
```
- Исключение поля
```tsx
interface User {
	name: string;
	age: number;
	friends: Array<string>;
	type: string;
}

interface Car {
	name: string;
	type: string;
}

type WithoutType<T> = {
		[K in keyof T as Exclude<K, 'type'>]: T[K]; //Преобразуем T в тип без поля type при помощи
													//утиитарного типа Exclude и вытягиваем его ключи
}

type WithoutUser = WithoutType<User> //На выходе получим тип без поля type
type WithoutCar = WithoutType<Car> //Тоже самое
```
- Изменение названия полей
```ts
 interface User {
	name: string;
	age: number;
	friends: Array<string>;
	type: string;
}

type GetMethods<T> = {
	[K in keyof T as `get${Capitalize<string & K>}`]: T[K]; //Здесь первая буква каждого поля переводится
															//в верхний регистр и спереди прибавляется get
}

type GetUser = GetMethods<User>; //getName, getAge и т.д.
```
# Utility Types
Utility Types - это уже готовые mapped types, которые разработчики любезно за нас написали

Останавливаться тут нет смысла, если нужно вот ссылка на доку:  [https://www.typescriptlang.org/docs/handbook/utility-types.html#lowercasestringtype](https://www.typescriptlang.org/docs/handbook/utility-types.html#lowercasestringtype)
# Assets
- asserts condition (не изменяет тип)
- asserts — это ключевое слово в TypeScript, которое используется в assertion functions (функциях утверждения). Оно говорит компилятору: "Я проверил значение, и теперь ты можешь доверять, что оно соответствует определённому типу или не null/undefined". Это способ уточнить тип переменной в коде, чтобы TypeScript не ругался, без фактического изменения значения
```tsx
function assertNotNull(value: unknown): asserts value {
	if (value === null || value === undefined) {
		throw new Error("Value is null or undefined")
	}
}

assertNotNull(null) //Вызовет ошибку 
assertNotNull(123) //Всё ок

let value: string | null = "hello";
assertNotNull(value);
// TypeScript теперь знает, что value — string (не null)
console.log(value.toUpperCase()); // OK
```
- asserts value is Type (изменяет тип)
```tsx
interface User {
	name: string;
	age: number;
}

function assertIsUser(data: any): asserts data is User {
	if(typeof data !== 'object' || data === null) {
		throw new Error('Object expected')
	}

	if(typeof data.name === 'string') {
		throw new Error('Property name must be a string')
	}

	if(typeof data.age === 'number') {
		throw new Error('Property age must be a number')
	}
}

function prepareUser(obj: User) {
	console.log(obj)
}

const obj = {
	name: 'name'
}

prepareUser(obj) //Тут будет ошибка из-за отсутствия поля age

assertIsUser(obj) 

prepareUser(obj) //Тут не будет ошибку потому, что выше мы проверили 
				 //тип и при не соответствии пробросилось бы исключение
```
## !!!Важно
Существует 2 object: object и Object и они имеют ключевое различие:
- Object - это общий тип, то есть любой тип данных мы туда можем записать, кроме null и undefined и он не защитит от передачи не объекта
- object - это объекты (ссылочные типы данных)
```tsx
const obj: Object = {} //Ок
const obj: Object = 1 //Ок
const obj: Object = "" //Ок
const obj: Object = () => {} //Ок
const obj: Object = null //Ошибка
const obj: Object = undefined //Ошибка

const obj: object = {} //Ок
const obj: object = new Date() //Ок
const obj: object = 1 //Ошибка
const obj: object = "" //Ошибка
const obj: object = () => {} //Ок
const obj: object = null //Ошибка
const obj: object = undefined //Ошибка
```
# typeof/keyof
- typeof
Делится сразу на 2 вида: предоставляемый JS и предоставляемы TS. Различие заключается в том, что первый работает в runtime и только с примитивными типами, второй же нужен для типов
```tsx
//Runtime (JS)

typeof 777
typeof "123"
typeof true
typeof {}
typeof []
typeof null
typeof undefined
typeof function
typeof Symbol("s")

//Для типов (TS)
const obj = {
	name: "name",
	age: 25
}

type Person = typeof obj; //Получаем модель, как будто писали её руками (не работате во время исполнения кода)
```
- Ещё про typeof в TS
```tsx
let color = 'red'

type RedColor = typeof color 
const green: RedColor = 'green' //Ошибка типа, ожидается только red

function getData(user: Person): string {
	return '123'
}

type GetDataFn = typeof getData; //Получаем стрелочную функцию (user: Person) => string
type GetDataReturnValue = ReturnType<typeof getData> //Получаем возвращаемое значение из функции (string)
type GetDataParams = Parameters<typeof getData> //Получаем значение которое принимает функция

```
- keyof - существует только в рамках TS и нужен, чтобы получать ключи из объектов
```tsx
const obj = {
	name: "name",
	age: 25
}

function getByKey<T, K extends keyof T>(obj: T, key: K): T[K] {
	return obj[key];
}

getByKey(obj, 'name') //TS сразу подскаже, что 2 аргумент name или age и что-то другое туда передать не выйдет
```
# Optional и non-null assertion
- optional - это довольно известный оператор и используется во многих языках. Нужен он для того, чтобы избежать исключения при обращении к пустому полю
```tsx
interface Person {
	name: string;
	address?: {
		street: string;
	};
	getAge?: () => number;
	arr?: string[]
} 

const person: Person = {
	name: 'name',
}

person.address?.street // Поскольку поле address не инициализировано, 
					   // то мы получим просто undefined, а не ошибку
user.getAge?.() // Тоже самое, но для функции
user.arr?.[0] // Тоже самое, но для массива
```
- non-null assertion - некоторая заглушка, которая позволяет обращаться к необязательным полям и не получать ворнинг от TS, но это не спасет от ошибки при исполнении кода
```tsx
interface Person {
	name: string;
	address?: {
		street: string;
	};
	getAge?: () => number;
	arr?: string[]
} 

const person: Person = {
	name: 'name',
}

person.address!.street //IDE ошибку не будет подсвечивать
user.getAge!.() //IDE ошибку не будет подсвечивать
user.arr!.[0] //IDE ошибку не будет подсвечивать
```
## !!! non-null assertion не желательно использовать в проде
# Enum
- Обычный Enum
```tsx
enum Color {
	RED = 'red',
	GREEN = 'green',
	BLUE = 'blue'
}

function setColor(color: Color) {}

setColor(Color.RED)
```
- Константный Enum - отличается от обычного в итоговом скомпилированном коде
```tsx
const enum Color {
	RED = 'red',
	GREEN = 'green',
	BLUE = 'blue'
}

function setColor(color: Color) {}

setColor(Color.RED)
```
- Const как замена Enum
```tsx
const Color {
	RED = 'red',
	GREEN = 'green',
	BLUE = 'blue'
} as const
type Color = typeof Color[keyof Color] //Здесь мы сначала получаем все ключи 
																			 //(RED, GREEN, BLUE), а после по ключам достаем
																			 //значения и делаем из них тип (код читается с права налево)

//Можно создать утилитарный тип для таких случаев
type ValueOf<T> = T[keyof T]
type Color = ValueOf<typeof Color>

function setColor(color: Color) {}

setColor(Color.RED)
```

![image.png](image%201.png)
# Условные типы и infer
- Условные типы - это по сути тернарные операторы используемые в типах
```tsx
type IsString<T> = T extends string ? true : false;

type True = IsString<string>;
type False = IsString<number>;
```
- infer
```tsx
function fn(arg1: string, arg2: number): string {
	return '123'
}

type MyParameters<T> = T extends (...args: infer R) => any ? R : never; //Дословно это читается как, если это 
																		//функция то запиши аргументы в R и верни его
																		//иначе верни never(такого типа не существует)

type FnArg = MyParameters<typeof fn> // [string, number]

type MyReturnType<T> = T extends (...args: any) => infer R ? R : never; //Дословно это читается как, если это функция
																		//то запиши возвращаемый тип в R и верни его
																		//иначе верни never(такого типа не существует)

type FnReturnType = MyReturnType<typeof fn> //string

type GetArrayItem<T extends any[]> = T extends (infer R)[] ? R : never;

const array: string[] = []
type ArrayItem = GetArrayItem<typeof array> //string
```