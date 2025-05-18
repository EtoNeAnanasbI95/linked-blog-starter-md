# Предисловие
> [!TIP] Важно
> Я не люблю фронтенд, я обучился ему случайно с течением серии обстоятельств
> 
> Я работал фронтендром с 9 апреля 2025 года, и попал я туда вообще случайно. Изначально я Golang разработчик, и наибольший пласт знаний имею именно в backend разработке
# Road map

- Язык и основы
    - Язык ([[Frontend/TS & JS/TS & JS]])
    - Среда выполнения
        - Где исполняется код: браузер (DOM, BOM, Web APIs), Node.js.
        - Event Loop: микрозадачи, макрозадачи, рендеринг.
        - Основы работы браузера: рендеринг, reflow/repaint, critical rendering path.
- Инструменты и инфраструктура
    - [[Пакетные менеджеры]]
    - [[Сборщики]] и бандлеры
        - Webpack, Vite: конфигурация, оптимизация бандлов.
        - [[Модульные системы]]: CommonJS, ES Modules.
    - Инструменты для разработки
        - Линтеры (ESLint, Prettier).
        - Тестирование: Jest, Vitest.
- Архитектура и паттерны
    - Архитектурные подходы
        - Монолитный фронтенд.
        - Микрофронтенды: подходы, инструменты (Module Federation, single-spa).
        - Feature-Sliced Design (FSD).
        - Чистая архитектура для фронтенда.
        - Эволюционный дизайн (incremental refactoring).
- Библиотеки и фреймворки
    - Основные фреймворки
        - React: [[Runtime]], компоненты, [[React хуки]], контекст, роутинг (React Router).
	        - Подсемейства React: [[Next.js]] (SSR/SSG), Remix, Gatsby, Preact.
        - Vue: композиционный API, директивы, Vuex/Pinia.
	        - Подсемейства Vue: Nuxt.js. 
	- Универсальные библиотеки: Framer Motion, Headless UI, [[TanStack]].
    - Управление состоянием
        - Локальный стейт (useState, useReducer).
        - Глобальный стейт: Redux, MobX, Zustand.
        - Серверный dsстейт: React Query, SWR.
    - UI-библиотеки и компоненты
        - Material-UI, Ant Design, Chakra UI.
        - Headless-компоненты: Radix UI, Ariakit.
	* Препроцессоры стилей:
		* Tailwind CSS
		* Sass
		* Css
- **Тренды и экспериментальные технологии**
    - WebAssembly: использование с JavaScript.
    - PWA: Service Workers, manifest, offline-режим.
    - Новые API: WebGPU, WebRTC, Intersection Observer.