# Что такое TLS

TLS (Transport Layer Security) — это **криптографический протокол**, который защищает соединения между клиентом и сервером.
## Основные задачи TLS
1. **Шифрование** — никто не может просматривать трафик (например, пароли, токены).
2. **Аутентификация** — клиент и сервер проверяют друг друга через сертификаты.
3. **Целостность** — данные не могут быть подменены в пути.
## Как это выглядит для Docker TCP
- Docker может слушать порт 2376 с TLS:
```bash
dockerd --tlsverify \
  --tlscacert=/certs/ca.pem \
  --tlscert=/certs/server-cert.pem \
  --tlskey=/certs/server-key.pem \
  -H=0.0.0.0:2376
```
- Клиент Docker использует свои сертификаты:
```bash
docker --tlsverify \
  --tlscacert=ca.pem \
  --tlscert=cert.pem \
  --tlskey=key.pem -H=tcp://manager-node:2376 info
```
- В итоге:
    - TCP открыт, но соединение зашифровано
    - Клиент проверяет сервер
    - Никто не может “подключиться просто так” без своего сертификата
## ⚡ Итог
- SSH → проще, безопасно, не нужен TLS.
- TLS → полезно, если хочешь TCP-сокет, масштабирование, но нужно управлять сертификатами.
# Настройка
## Основные типы файлов TLS
Для Docker TLS тебе нужны:

| Файл              | Назначение                                                                                  |
| ----------------- | ------------------------------------------------------------------------------------------- |
| `ca.pem`          | Сертификат Центра Сертификации (CA), который подписывает серверные и клиентские сертификаты |
| `server-cert.pem` | Сертификат сервера, подписанный CA                                                          |
| `server-key.pem`  | Приватный ключ сервера                                                                      |
| `client-cert.pem` | Сертификат клиента (GitLab Runner или другой Docker клиент)                                 |
| `client-key.pem`  | Приватный ключ клиента                                                                      |
## Генерация самоподписанных сертификатов
### Шаг 1. Создаём CA
```bash
openssl genrsa -out ca-key.pem 4096
openssl req -x509 -new -nodes -key ca-key.pem -sha256 -days 3650 -out ca.pem -subj "/CN=MyDockerCA"
```
### Шаг 2. Сертификат сервера
```bash
openssl genrsa -out server-key.pem 4096
openssl req -new -key server-key.pem -out server.csr -subj "/CN=manager-node"
openssl x509 -req -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -days 365 -sha256
```
### Шаг 3. Сертификат клиента

```bash
openssl genrsa -out client-key.pem 4096
openssl req -new -key client-key.pem -out client.csr -subj "/CN=gitlab-runner"
openssl x509 -req -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out client-cert.pem -days 365 -sha256
```
## Настройка Docker daemon на менеджере
```bash
dockerd --tlsverify \
  --tlscacert=/path/to/ca.pem \
  --tlscert=/path/to/server-cert.pem \
  --tlskey=/path/to/server-key.pem \
  -H=0.0.0.0:2376
```
- `--tlsverify` — заставляет Docker проверять клиентские сертификаты.
- `-H=0.0.0.0:2376` — слушать TCP порт.
## Настройка Docker-клиента (Runner)

```bash
docker --tlsverify \
  --tlscacert=/path/to/ca.pem \
  --tlscert=/path/to/client-cert.pem \
  --tlskey=/path/to/client-key.pem \
  -H=tcp://manager-node:2376 info
```
✅ Теперь:
- Только клиент с правильным сертификатом может подключиться.
- Все данные между клиентом и Docker daemon шифруются.
### ⚡ Итог
- TLS = безопасный TCP-доступ к Docker.
- Сами файлы — просто ключи и сертификаты.
- Для продакшна желательно использовать **сертификаты от корпоративного CA** или Let's Encrypt, чтобы не ломать автоматизацию.
