# Введение
* **React** — библиотека JavaScript для создания пользовательских интерфейсов, основана на компонентах, написание которых происходит в крутом jsx формате (html прям в js файлики).
**Почему библиотека**: Не полноценный фреймворк, предоставляет только инструменты для UI (рендеринг, управление состоянием), без встроенных решений для маршрутизации, HTTP-запросов и т.д.
# Инициализация
- **Vite**: pnpm create vite — быстрый, легковесный, с горячей перезагрузкой.
- **Create React App (CRA)**: pnpm create react-app — классический, но устаревает, тяжёлый.
- **Next.js**: npx create-next-app — для SSR/SSG, с маршрутизацией.
- **Remix**: npx create-remix — упор на SSR и производительность.
# Работа
## Strickt mode
* Просто двойной рендер во время dev mode, который позволяет проверить не происходит ли работа с side effect, то есть не меняем ли мы какие-то переменные которые валяются вне скоупа нашего компонента
## Сборка
* SPA как и все другие сайты для браузера представляют собой просто html документ, к которому прикрутили какой-то скрипт js. Этот скрипт занимается рендерингом страницы. Файлик скрипта он один, и растягивается на сотни тысяч строк js кода, но когда мы пишем проект он разбиваем его на компоненты и тучу разных файликов, которые лежат в разных папках. Все эти файлики кода реакта потом надо как-то превратить в один ахуенно здоровый скрипт, этим занимаются сборщики, есть куча разных:
	* [[Сборщики#Vite]]
	* [[Сборщики#Webpack]]
	* [[Сборщики#Turbopack]]
	* [[Сборщики#esbuild]]
	* [[Сборщики#Rollup]]
	* [[Сборщики#Parcel]]
## **Где работает по умолчанию**:
- **CSR (Client-Side Rendering)**: React рендерит UI в браузере, используя DOM. Vite/CRA — CSR по умолчанию.
- **SSR (Server-Side Rendering)**: Рендеринг на сервере, отправка готового HTML. Требует фреймворков (Next.js, Remix).
# Особенности при разработке
## Обработка ивентов
* При обработке ивентов нажатия и тд, вместо обычных html надо использовать их же, но из библиотеки реакт, потому что мы работаем не с DOM а с VirtualDOM