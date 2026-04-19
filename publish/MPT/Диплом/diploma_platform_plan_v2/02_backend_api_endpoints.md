# Backend API — обновлённый план эндпоинтов (Django + DRF, JWT)
Этот API-план -- часть [[MPT/Диплом/data|диплома]], вырос из [[01_functional_checklist|functional checklist]], завязан на [[04_db_entities_relationships|db relationships]] и относится к общему [[Backend]].

Обновления по твоему чеклисту:
- Добавлены эндпоинты для **работы с файлами** (создание/редактирование/удаление).
- Пополнение баланса зафиксировано как **ЮKassa test** (без “mock” как основной вариант).
- Ролевая модель уточнена:
  - **Admin** работает через **Django Admin** (не через самописную админку).
  - **Manager** работает через **самописную панель** и имеет эндпоинты для скидок/промокодов/статистики.

---

## 0) Общие принципы

- Префикс: `/api/v1`
- JWT: `Authorization: Bearer <access>`
- Роли:
  - customer: доступ только к своим данным
  - manager: доступ к управлению каталогом/пулом серверов/скидками/промокодами/отчётами
  - admin: **Django Admin** (API можно не давать, кроме служебных endpoint’ов при необходимости)

---

## 1) Auth / Users (customer + manager)

### 1.1 Регистрация и подтверждение
- `POST /auth/register` → `{ email, password }`
- `POST /auth/verify-email` → `{ token }`

### 1.2 JWT
- `POST /auth/login` → `{ email, password }` → `{ access, refresh, user }`
- `POST /auth/refresh` → `{ refresh }` → `{ access }`
- `POST /auth/logout` → `{ refresh }` *(если blacklist refresh)*

### 1.3 Password reset
- `POST /auth/password/reset-request` → `{ email }`
- `POST /auth/password/reset-confirm` → `{ token, new_password }`

### 1.4 Профиль
- `GET /me`
- `PATCH /me` → контакты/реквизиты
- `POST /me/change-password` → `{ old_password, new_password }`

---

## 2) Catalog (Plans) — витрина

### 2.1 Список и карточка (доступно без auth — по желанию)
- `GET /plans`
  - query: `location, ram_gte, ram_lte, price_gte, price_lte, disk_type`
  - result: список + `available_count`
- `GET /plans/{plan_id}`
  - details + `available_count`

---

## 3) Discounts & Promo codes (для customer и manager)

### 3.1 Проверка промокода при покупке (customer)
- `POST /promocodes/validate`
  - body: `{ code, plan_id, period_type, period_count }`
  - result: `{ is_valid, discount_type, discount_value, final_price }`

### 3.2 Управление скидками (manager)
**Скидка** = правило для тарифа/периода/диапазона дат.
- `GET /manager/discounts`
- `POST /manager/discounts`
- `PATCH /manager/discounts/{discount_id}`
- `DELETE /manager/discounts/{discount_id}` *(soft delete допустим)*

### 3.3 Управление промокодами (manager)
- `GET /manager/promocodes`
- `POST /manager/promocodes`
- `PATCH /manager/promocodes/{promo_id}`
- `DELETE /manager/promocodes/{promo_id}`

---

## 4) Balance / YooKassa test

### 4.1 Баланс и операции
- `GET /balance` → `{ balance }`
- `GET /balance/transactions`
  - query: `type, date_from, date_to, page`

### 4.2 Создать пополнение (ЮKassa test)
- `POST /balance/topups`
  - body: `{ amount }`
  - result (вариант А — redirect): `{ topup_id, confirmation_url }`
  - result (вариант Б — если фронт сам строит оплату): `{ topup_id, payment_payload }`

### 4.3 Вебхук ЮKassa (test)
- `POST /payments/webhook/yookassa`
  - YooKassa присылает событие → backend:
    - проверяет подпись/валидность (как минимум idempotency)
    - меняет `topup.status`
    - если succeeded → создаёт `wallet_transaction(type=topup)` и увеличивает баланс

> Если на защите не хотят реальный webhook, можно сделать тестовую кнопку “проверить статус”:
- `POST /balance/topups/{topup_id}/sync`
  - backend запрашивает YooKassa API (test) и синхронизирует статус

---

## 5) Rentals (покупка за баланс) + операции

### 5.1 Создать аренду (списание баланса)
- `POST /rentals`
  - body: `{ plan_id, period_type, period_count, promo_code? }`
  - checks:
    - email verified
    - plan active
    - available server exists
    - рассчитывает цену (скидка/промокод)
    - balance >= final_price
  - txn:
    - списание баланса (wallet_transaction charge)
    - создание rentals(status=active)
    - allocation server → `servers.status=rented`
  - result: `{ rental_id, status, start_at, end_at }`

### 5.2 Мои аренды
- `GET /rentals`
  - query: `status, page`
- `GET /rentals/{rental_id}`

### 5.3 Доступы (секреты)
- `GET /rentals/{rental_id}/credentials`
  - result: `{ ip, ssh_port, login, password?, ssh_key? }`
  - side-effect: `secret_access_logs(event=view_credentials)`
  - access: owner + manager? (обычно **нет**, лучше только owner + admin via django)

### 5.4 Операции (как запрос операции)
- `POST /rentals/{rental_id}/actions/reboot`
- `POST /rentals/{rental_id}/actions/reset-password`
  - side-effect: rotate credentials + log `secret_access_logs(event=reset_password)`
- `POST /rentals/{rental_id}/actions/inject-ssh-key`
  - body: `{ ssh_public_key }`
  - log `secret_access_logs(event=inject_ssh_key)`

### 5.5 Метрики для графиков (Chart.js)
- `GET /rentals/{rental_id}/metrics`
  - query: `from, to, step`
  - result: timeseries

---

## 6) Files (CRUD) — “редактирование/создание/удаление файлов”

Рекомендованная реализация для диплома:
- хранение файлов в **Supabase Storage** (bucket `rental-files`)
- backend выдаёт **signed URL** (чтобы не открывать bucket публично)
- “редактирование” текста — либо через перезапись объекта, либо через хранение текстовых файлов в БД.

### 6.1 Список файлов по аренде
- `GET /rentals/{rental_id}/files`
  - result: `[{ file_id, path, size, updated_at }]`

### 6.2 Создать/загрузить файл
- `POST /rentals/{rental_id}/files`
  - body: `multipart/form-data` (`file`, `path?`)
  - result: `{ file_id, path }`

### 6.3 Скачать/получить ссылку
- `GET /rentals/{rental_id}/files/{file_id}/download`
  - result: `{ signed_url, expires_at }`

### 6.4 Редактировать файл (текстовый)
- `PUT /rentals/{rental_id}/files/{file_id}`
  - body: `{ content }` *(для text-файлов)*
  - result: `200`

### 6.5 Удалить файл
- `DELETE /rentals/{rental_id}/files/{file_id}`

> Важно: доступ только владельцу аренды. Каждое действие можно писать в `audit_logs`.

---

## 7) Manager Panel API (самописная менеджерка)

### 7.1 Plans CRUD
- `GET /manager/plans`
- `POST /manager/plans`
- `PATCH /manager/plans/{plan_id}`
- `DELETE /manager/plans/{plan_id}` *(soft delete)*

### 7.2 Servers pool
- `GET /manager/servers` (filters: status/location)
- `POST /manager/servers`
- `PATCH /manager/servers/{server_id}`
- `POST /manager/servers/{server_id}/status` → `{ status }`
- `GET /manager/servers/{server_id}/rentals-history`

### 7.3 Rentals view (без секретов)
- `GET /manager/rentals`
- `GET /manager/rentals/{rental_id}`
- `POST /manager/rentals/{rental_id}/terminate` *(если разрешаешь менеджеру; иначе только admin в django)*

### 7.4 Статистика (для менеджера)
- `GET /manager/stats/occupancy`
- `GET /manager/stats/sales/monthly?from&to`
- `GET /manager/stats/sales/by-location?from&to`

### 7.5 Экспорт XLSX
- `GET /manager/stats/occupancy.xlsx`
- `GET /manager/stats/sales/monthly.xlsx`
- `GET /manager/stats/sales/by-location.xlsx`

---

## 8) Audit (manager view / admin via django)

- `GET /manager/audit/logs` *(если хочешь дать менеджеру только чтение)*
  - query: `action, entity_type, date_from, date_to`
- `GET /manager/audit/secret-access` *(скорее только admin)*
