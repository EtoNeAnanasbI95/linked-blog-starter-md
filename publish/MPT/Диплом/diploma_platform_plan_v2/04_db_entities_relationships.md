# БД (Supabase/PostgreSQL) — обновлённые сущности и связи
Этот документ является частью [[MPT/Диплом/data|диплома]], напрямую связан с [[02_backend_api_endpoints|Backend API endpoints]], опирается на [[Instruments/Databases/❌ Databases|Databases]] и местами рифмуется с [[MPT/ТРПО/IDEF1X|IDEF1X]].

Изменения:
- Уточнены роли: **admin** — только Django Admin (но роль в users всё равно хранится).
- Добавлены **скидки** и **промокоды**.
- Добавлены сущности под **файлы** (Supabase Storage + метаданные в БД).
- Пополнение — через **YooKassa test** (topups + webhook).

---

## 1) Users

### users
- `id (uuid, PK)`
- `email (unique)`
- `password_hash`
- `is_email_verified (bool)`
- `role (enum: customer|manager|admin)`
- `created_at`

### user_profiles
- `user_id (PK/FK -> users.id)`
- `full_name`
- `phone`
- `company_name`
- `inn`
- `address`
- `updated_at`

---

## 2) Catalog

### plans
- `id (uuid, PK)`
- `name`
- `cpu_cores`
- `ram_gb`
- `disk_gb`
- `disk_type (ssd|hdd)`
- `traffic_tb`
- `location`
- `os_template`
- `price_per_hour`
- `price_per_day`
- `price_per_month`
- `is_active`
- `created_at`

---

## 3) Discounts & Promocodes

### discounts
Скидка как правило (например “-10% на план X в феврале”).
- `id (uuid, PK)`
- `plan_id (FK -> plans.id)`
- `discount_type (percent|fixed)`
- `discount_value (numeric)`
- `valid_from`
- `valid_to`
- `is_active`
- `created_at`

### promocodes
Промокод, который может применяться к плану/всем планам.
- `id (uuid, PK)`
- `code (unique)`
- `discount_type (percent|fixed)`
- `discount_value (numeric)`
- `plan_id (nullable FK -> plans.id)` *(null = на все планы)*
- `usage_limit (nullable int)`
- `used_count (int default 0)`
- `valid_from`
- `valid_to`
- `is_active`
- `created_at`

---

## 4) Servers pool

### servers
- `id (uuid, PK)`
- `hostname`
- `ip`
- `ssh_port`
- `location`
- `os`
- `cpu_cores`
- `ram_gb`
- `disk_gb`
- `disk_type`
- `traffic_tb`
- `status (available|rented|maintenance|disabled)`
- `comment`
- `created_at`

### server_credentials
- `id (uuid, PK)`
- `server_id (FK -> servers.id)`
- `login`
- `password_encrypted`
- `ssh_key_encrypted (nullable)`
- `created_at`
- `rotated_at`

---

## 5) Wallet / Ledger

### wallets
- `user_id (PK/FK -> users.id)`
- `balance (numeric)`
- `updated_at`

### wallet_transactions
- `id (uuid, PK)`
- `user_id (FK -> users.id)`
- `type (topup|charge|refund|adjustment)`
- `amount (numeric)`
- `currency (RUB)`
- `related_rental_id (nullable FK -> rentals.id)`
- `related_topup_id (nullable FK -> topups.id)`
- `created_at`
- `meta (jsonb)`

---

## 6) Topups (YooKassa test)

### topups
- `id (uuid, PK)`
- `user_id (FK -> users.id)`
- `amount`
- `provider (enum: yookassa_test)`
- `status (pending|succeeded|canceled|failed)`
- `external_payment_id` *(from YooKassa)*
- `created_at`
- `confirmed_at`

---

## 7) Rentals

### rentals
- `id (uuid, PK)`
- `user_id (FK -> users.id)`
- `plan_id (FK -> plans.id)`
- `server_id (FK -> servers.id, nullable until allocation)`
- `promo_id (nullable FK -> promocodes.id)`
- `discount_id (nullable FK -> discounts.id)` *(если фиксируешь применённую скидку)*
- `status (pending|active|expired|canceled)`
- `start_at`
- `end_at`
- `period_type (hour|day|month)`
- `period_count`
- `price_total`
- `auto_renew (bool)`
- `created_at`

**Ключевое ограничение:** один сервер — одна активная аренда (partial unique index по server_id where status='active').

---

## 8) Metrics

### server_metrics
- `id (uuid, PK)`
- `rental_id (FK -> rentals.id)`
- `ts`
- `cpu_percent`
- `ram_percent`
- `net_in_kbps`
- `net_out_kbps`

---

## 9) Files (Supabase Storage + метаданные)

### rental_files
Метаданные файла, сам файл лежит в Supabase Storage.
- `id (uuid, PK)`
- `rental_id (FK -> rentals.id)`
- `path (text)` *(например: /home/user/nginx.conf)*
- `storage_key (text)` *(путь в bucket, например: rentals/{rental_id}/nginx.conf)*
- `content_type`
- `size_bytes`
- `is_text (bool)` *(для разрешения PUT content)*
- `created_at`
- `updated_at`

---

## 10) Audit

### audit_logs
- `id (uuid, PK)`
- `actor_user_id (FK -> users.id)`
- `action`
- `entity_type`
- `entity_id`
- `ip`
- `created_at`
- `meta (jsonb)`

### secret_access_logs
- `id (uuid, PK)`
- `actor_user_id (FK -> users.id)`
- `rental_id (FK -> rentals.id)`
- `server_id (FK -> servers.id)`
- `event (view_credentials|reset_password|inject_ssh_key|rotate_credentials)`
- `created_at`
- `meta (jsonb)`

---

## ER (кратко)
- users 1—1 wallets
- users 1—N wallet_transactions
- users 1—N topups
- users 1—N rentals
- plans 1—N rentals
- plans 1—N discounts
- promocodes N—1 (optional) plans
- servers 1—N rentals
- servers 1—N server_credentials
- rentals 1—N server_metrics
- rentals 1—N rental_files
- users 1—N audit_logs
- users 1—N secret_access_logs
