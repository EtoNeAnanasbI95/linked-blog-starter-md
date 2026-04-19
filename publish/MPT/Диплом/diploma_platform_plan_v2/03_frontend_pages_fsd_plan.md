# План фронтенда (2 приложения) — страницы, роутинг, содержимое, FSD
Этот план фронта -- часть [[MPT/Диплом/data|диплома]], вырос из [[01_functional_checklist|functional checklist]] и напрямую опирается на [[Frontend]] и [[TanStack]].

Стек (зафиксировано):
- React
- TanStack Router
- TanStack Query
- Axios
- Chart.js
- Архитектура: Feature-Sliced Design (FSD)

Ниже — **два отдельных SPA**:
1) **Клиентка (Customer App)**
2) **Менеджерка (Manager App)**  
Админ (admin) работает через **Django Admin** (не через SPA).

---

## A) Customer App — страницы и что на них

### A.1 Навигация (верхний уровень)
- Публичные: каталог, карточка тарифа, авторизация
- Приватные (требуют JWT): кабинет, баланс, аренды, файлы, профиль, документы

---

### A.2 Роутинг (TanStack Router)

#### Public routes
- `/` — Каталог тарифов
- `/plans/$planId` — Карточка тарифа
- `/auth/login` — Вход
- `/auth/register` — Регистрация
- `/auth/verify-email` — Подтверждение email (token из query)
- `/auth/reset` — Запрос сброса пароля
- `/auth/reset/confirm` — Установка нового пароля (token из query)

#### Protected routes
- `/dashboard` — Обзор (кабинет)
- `/balance` — Баланс + пополнение YooKassa test + история операций
- `/rentals` — Мои аренды/серверы (список)
- `/rentals/$rentalId` — Детали аренды/сервера
- `/rentals/$rentalId/files` — Файлы аренды (FS)
- `/profile` — Профиль + реквизиты + смена пароля
- `/documents` — Документы (счета/акты/чеки) *(если в чеклисте есть)*

---

### A.3 Подробно по страницам (что отображается и какие запросы)

#### 1) `/` — Каталог тарифов
**UI:**
- список карточек тарифов: CPU/RAM/Disk/Traffic/Location/OS/Price
- бейдж наличия: “в наличии / осталось N”
- фильтры (слева/вверху): location, RAM>=, price<=, disk_type, OS
- кнопка “Арендовать” / “Открыть”

**API:**
- `GET /plans` (filters) → список + available_count

---

#### 2) `/plans/$planId` — Карточка тарифа
**UI:**
- детали тарифа
- выбор периода: hour/day/month + period_count
- поле промокода
- блок расчёта цены: base_price → discount → final_price
- кнопка “Арендовать” (покупка за баланс)
- подсказка “если не хватает — пополните баланс”

**API:**
- `GET /plans/{id}`
- `POST /promocodes/validate` (при вводе промо)
- `GET /balance` (проверка хватает ли)
- `POST /rentals` (создать аренду, списание баланса)

---

#### 3) `/auth/login` — Вход
**UI:** email/password, submit, ссылки register/reset  
**API:** `POST /auth/login`

---

#### 4) `/auth/register` — Регистрация
**UI:** email/password, submit, текст про подтверждение почты  
**API:** `POST /auth/register`

---

#### 5) `/auth/verify-email` — Подтверждение email
**UI:** состояние “проверяем… / успех / ошибка”  
**API:** `POST /auth/verify-email`

---

#### 6) `/auth/reset` — Запрос сброса пароля
**UI:** поле email, submit  
**API:** `POST /auth/password/reset-request`

---

#### 7) `/auth/reset/confirm` — Подтверждение сброса
**UI:** новый пароль + повтор, submit  
**API:** `POST /auth/password/reset-confirm`

---

#### 8) `/dashboard` — Обзор
**UI (виджеты):**
- карточка “Баланс”
- “Активные аренды” (top N)
- “Скоро истекают” (если есть)
- быстрые действия: “Пополнить баланс”, “Перейти к арендам”

**API:**
- `GET /balance`
- `GET /rentals?status=active&limit=...`

---

#### 9) `/balance` — Баланс / пополнение
**UI:**
- текущий баланс
- форма пополнения (amount)
- кнопка “Перейти к оплате” (YooKassa test)
- таблица операций (topup/charge)

**API:**
- `GET /balance`
- `GET /balance/transactions`
- `POST /balance/topups` → confirmation_url
- *(опционально)* `POST /balance/topups/{id}/sync`

---

#### 10) `/rentals` — Мои аренды/серверы
**UI:**
- таблица: тариф, локация, статус, start/end, цена
- фильтр по статусу
- переход в детали

**API:**
- `GET /rentals`

---

#### 11) `/rentals/$rentalId` — Детали аренды/сервера
**UI:**
- блок аренды: план, даты, статус, авто-продление (если есть)
- блок “Доступы”:
  - кнопка “Показать” → модалка с IP/login/password/ssh_key + copy
- блок “Операции”:
  - reboot
  - reset password
  - inject ssh key *(если есть)*
- блок “Мониторинг” (Chart.js):
  - графики CPU/RAM/NET (timeseries)

**API:**
- `GET /rentals/{id}`
- `GET /rentals/{id}/credentials`
- `POST /rentals/{id}/actions/reboot`
- `POST /rentals/{id}/actions/reset-password`
- `POST /rentals/{id}/actions/inject-ssh-key`
- `GET /rentals/{id}/metrics`

---

#### 12) `/rentals/$rentalId/files` — Файлы аренды (FS)
**UI:**
- список файлов (таблица: имя/путь/размер/дата)
- upload (multipart)
- действия: скачать, удалить
- (для text) редактор: textarea/monaco (опционально) + “Сохранить”

**API:**
- `GET /rentals/{id}/files`
- `POST /rentals/{id}/files` (upload)
- `GET /rentals/{id}/files/{fileId}/download` (signed_url)
- `PUT /rentals/{id}/files/{fileId}` (content) *(если is_text)*
- `DELETE /rentals/{id}/files/{fileId}`

---

#### 13) `/profile` — Профиль
**UI:**
- контакты/реквизиты (форма)
- смена пароля

**API:**
- `GET /me`
- `PATCH /me`
- `POST /me/change-password`

---

#### 14) `/documents` — Документы *(если есть в твоём чеклисте)*
**UI:** список документов + ссылка скачать  
**API:** например `GET /documents` / `GET /documents/{id}` *(если добавишь на backend)*

---

## B) Manager App — страницы и что на них

Менеджерка — самописная панель для:
- управление тарифами
- управление пулом серверов
- скидки/промокоды
- статистика + выгрузка XLSX

---

### B.1 Роутинг (TanStack Router)

#### Auth (можно общий `/auth/login`)
- `/auth/login`

#### Protected manager routes
- `/manager` — Дашборд
- `/manager/plans` — Тарифы (CRUD)
- `/manager/servers` — Серверы (пул) (CRUD + статусы)
- `/manager/discounts` — Скидки (CRUD)
- `/manager/promocodes` — Промокоды (CRUD)
- `/manager/rentals` — Аренды (просмотр)
- `/manager/reports` — Отчёты/статистика + графики + XLSX
- `/manager/audit` — AuditLog (read-only) *(если даёшь менеджеру)*

---

### B.2 Подробно по страницам (UI + API)

#### 1) `/manager` — Дашборд
**UI:**
- KPI cards: available / rented / maintenance
- график sales monthly (Chart.js)
- таблица “последние аренды”

**API:**
- `GET /manager/stats/occupancy`
- `GET /manager/stats/sales/monthly?from&to`
- `GET /manager/rentals?limit=...`

---

#### 2) `/manager/plans` — Тарифы
**UI:**
- таблица тарифов
- модалка create/edit
- action: activate/deactivate

**API:**
- `GET /manager/plans`
- `POST /manager/plans`
- `PATCH /manager/plans/{id}`
- `DELETE /manager/plans/{id}`

---

#### 3) `/manager/servers` — Серверы (пул)
**UI:**
- таблица: IP, location, status, конфиг, comment
- фильтры status/location
- модалка create/edit
- кнопки смены статуса

**API:**
- `GET /manager/servers`
- `POST /manager/servers`
- `PATCH /manager/servers/{id}`
- `POST /manager/servers/{id}/status`

---

#### 4) `/manager/discounts` — Скидки
**UI:**
- таблица скидок (plan, %, даты, active)
- create/edit/delete

**API:**
- `GET /manager/discounts`
- `POST /manager/discounts`
- `PATCH /manager/discounts/{id}`
- `DELETE /manager/discounts/{id}`

---

#### 5) `/manager/promocodes` — Промокоды
**UI:**
- таблица промокодов (code, value, plan, used/limit, dates)
- create/edit/delete

**API:**
- `GET /manager/promocodes`
- `POST /manager/promocodes`
- `PATCH /manager/promocodes/{id}`
- `DELETE /manager/promocodes/{id}`

---

#### 6) `/manager/rentals` — Аренды (просмотр)
**UI:**
- таблица аренды: user/email, plan, server ip, status, dates, price
- фильтры: статус, период, локация

**API:**
- `GET /manager/rentals`
- `GET /manager/rentals/{id}`

---

#### 7) `/manager/reports` — Статистика + XLSX
**UI:**
- вкладки:
  - occupancy
  - sales monthly
  - sales by location
- графики Chart.js
- кнопки “Скачать XLSX”

**API:**
- `GET /manager/stats/occupancy`
- `GET /manager/stats/sales/monthly?from&to`
- `GET /manager/stats/sales/by-location?from&to`
- `GET /manager/stats/occupancy.xlsx`
- `GET /manager/stats/sales/monthly.xlsx`
- `GET /manager/stats/sales/by-location.xlsx`

---

#### 8) `/manager/audit` — AuditLog *(если нужно менеджеру)*
**UI:** таблица audit, фильтры  
**API:** `GET /manager/audit/logs`

---

## C) Общие требования (для обоих приложений)

### C.1 Auth/JWT
- Access token подставляется в `Authorization` через Axios interceptor.
- На 401 → попытка refresh → повтор оригинального запроса.
- Refresh:
  - вариант 1 (лучше): refresh в HttpOnly cookie
  - вариант 2 (проще): refresh в localStorage (описать как компромисс)

### C.2 TanStack Query (Data layer)
- Отдельный `queryClient` на приложение.
- Кешируем:
  - customer: plans, balance, rentals, files, metrics
  - manager: plans, servers, discounts, promocodes, rentals, stats
- После мутаций: `invalidateQueries` (например `rentals`, `balance`, `plans`).

### C.3 Ошибки и UX состояния
- Единый Error boundary + toaster (shared/ui).
- Состояния:
  - loading (skeleton)
  - empty state
  - error state (с retry)

### C.4 Безопасность (минимум)
- Не хранить пароль сервера на клиенте; получать только через `/credentials`.
- Логи “просмотр доступов” и “reset password” на backend.
- Для файлов: скачивание только через signed_url с коротким TTL.

---

## D) FSD — рекомендуемая структура (оба приложения)

### D.1 Общее правило
- `entities` — данные домена (types + api + query hooks)
- `features` — пользовательские действия (forms/mutations)
- `widgets` — композиции UI блоков
- `pages` — страницы маршрутов
- `shared` — общие либы, UI-kit, утилиты
- `app` — провайдеры, router, queryClient, layout

---

### D.2 Customer App — скелет папок
- `app/`
  - `providers/` (QueryClientProvider, AuthProvider)
  - `router/` (route tree)
  - `layouts/` (PublicLayout, PrivateLayout)
- `pages/`
  - `catalog/`
  - `plan/`
  - `auth/`
  - `dashboard/`
  - `balance/`
  - `rentals/`
  - `rental-details/`
  - `rental-files/`
  - `profile/`
  - `documents/`
- `widgets/`
  - `plansGrid/`
  - `filtersBar/`
  - `balanceCard/`
  - `rentalsTable/`
  - `credentialsModal/`
  - `metricsCharts/`
  - `filesTable/`
- `features/`
  - `auth-login/`
  - `auth-register/`
  - `auth-reset/`
  - `apply-promocode/`
  - `rent-server/`
  - `topup-balance/`
  - `view-credentials/`
  - `server-actions/`
  - `files-upload/`
  - `files-edit-text/`
  - `files-delete/`
- `entities/`
  - `user/`
  - `plan/`
  - `rental/`
  - `wallet/`
  - `file/`
  - `metric/`
  - `promocode/`
- `shared/`
  - `api/axios.ts`
  - `api/authRefresh.ts`
  - `config/`
  - `lib/format/`
  - `ui/`

---

### D.3 Manager App — скелет папок
- `app/`
  - `providers/`
  - `router/`
  - `layouts/` (ManagerLayout)
- `pages/`
  - `auth/`
  - `dashboard/`
  - `plans/`
  - `servers/`
  - `discounts/`
  - `promocodes/`
  - `rentals/`
  - `reports/`
  - `audit/`
- `widgets/`
  - `kpiCards/`
  - `salesChart/`
  - `serversTable/`
  - `plansTable/`
  - `discountsTable/`
  - `promosTable/`
  - `reportsTabs/`
- `features/`
  - `manager-login/`
  - `plan-crud/`
  - `server-crud/`
  - `server-status/`
  - `discount-crud/`
  - `promocode-crud/`
  - `report-export/`
- `entities/`
  - `plan/`
  - `server/`
  - `discount/`
  - `promocode/`
  - `rental/`
  - `stats/`
  - `audit/`
- `shared/`
  - `api/axios.ts`
  - `ui/`
  - `lib/`

---

## E) Router tree (примерная схема)

Customer:
- root (PublicLayout)
  - index `/`
  - `/plans/$planId`
  - `/auth/*`
- root (PrivateLayout)
  - `/dashboard`
  - `/balance`
  - `/rentals`
  - `/rentals/$rentalId`
  - `/rentals/$rentalId/files`
  - `/profile`
  - `/documents`

Manager:
- root (PublicLayout)
  - `/auth/login`
- root (ManagerLayout)
  - `/manager` (index)
  - `/manager/plans`
  - `/manager/servers`
  - `/manager/discounts`
  - `/manager/promocodes`
  - `/manager/rentals`
  - `/manager/reports`
  - `/manager/audit`
