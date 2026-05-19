# Изменить пароль

|  |  |
|------|------|
| URL | /api/v1/profile/password |
| Service Method | PATCH |
| Description | Изменяет пароль текущего пользователя |
| Return Type | JSON |
| Third-party service or system name | — |

## 1. List of changes

| Date | Responsible SA | JIRA | What was changed |
|------|------|------|------|
| 15.05.2026 | | | Первичное создание |

## 2. Input parameters

### 2.1 Header parameters

| Name | Type | Required | Description |
|------|------|------|------|
| Authorization | STRING | Yes | Bearer access-токен авторизации (`Bearer <access_token>`) |
| Content-Type | STRING | Yes | `application/json` |

### 2.2 Query params

Отсутствуют.

### 2.3 Path params

Отсутствуют.

### 2.4 Request body

| Name | Type | Max length | Required | Description |
|------|------|------|------|------|
| current_password | STRING | — | Yes | Текущий пароль |
| new_password | STRING | — | Yes | Новый пароль |

## 3. Response parameters

| Name | Type | Max length | Description |
|------|------|------|------|
| success | BOOLEAN | — | `true` — запрос выполнен успешно, `false` — ошибка |
| errorMessage | STRING | — | Сообщение об ошибке (при `success: false`) |
| data | OBJECT | — | `null` при успешной смене пароля |

## 4. Security requirements of method

- **Authentication**: обязательна — JWT Bearer access-токен в заголовке `Authorization`.
- **Authorization**: пользователь изменяет только собственный пароль.
- **Password protection**: пароль не логируется и не возвращается в ответе.
- **Session policy**: после успешной смены пароля все активные refresh-сессии пользователя отзываются; клиент должен выполнить повторный вход в систему.
- **Rate limiting**: ограничение на количество запросов — не более 30 запросов в минуту с одного IP или токена.
- **Data Encryption**: передача данных только по HTTPS (TLS 1.2+).

## 5. How the service works

1. Клиент отправляет `PATCH /api/v1/profile/password`.
2. Сервер валидирует Bearer access-токен и определяет пользователя по JWT claim `sub`.
3. Сервер проверяет `current_password` через `users.password_hash`.
4. Сервер хэширует `new_password`.
5. Сервер обновляет `users.password_hash`, `users.password_changed_at` и `users.updated_at`.
6. Сервер отзывает все активные refresh-сессии пользователя в `auth_sessions`, заполняя `revoked_at` и `revoked_reason = password_changed`.
7. Сервер возвращает успешный ответ. Новая сессия не создается; клиент должен повторно выполнить `POST /api/v1/auth/login`.

### 5.1 Sequence diagram

Не требуется: метод не содержит сложных интеграционных сценариев.

### 5.2 Mapping

| Request body | DB column / value |
|------|------|
| current_password | Проверяется с `users.password_hash` |
| new_password | Хэшируется и сохраняется в `users.password_hash` |
| password changed timestamp | `users.password_changed_at` |
| session revoke operation | `auth_sessions.revoked_at`, `auth_sessions.revoked_reason` |

## 6. Logging requirements

- Логировать: `user_id`, временную метку запроса, параметры запроса, HTTP-статус ответа.
- Логировать ошибки аутентификации (401), авторизации (403) и внутренние ошибки сервера (500) с уровнем `ERROR`.

## 7. Data model used by method

### 7.1 Table `users`

| Field | Type | Constraint | Description |
|------|------|------|------|
| `id` | `UUID` | PK, not null | Уникальный идентификатор |
| `email` | `TEXT` | UNIQUE, not null | Почта пользователя |
| `phone` | `VARCHAR(20)` | nullable | Телефон пользователя |
| `password_hash` | `TEXT` | not null | Хэш пароля пользователя |
| `first_name` | `VARCHAR(100)` | nullable | Имя пользователя |
| `last_name` | `VARCHAR(100)` | nullable | Фамилия пользователя |
| `status` | `VARCHAR(100)` | ENUM user_status, default ACTIVE | Статус пользователя |
| `created_at` | `TIMESTAMPTZ` | default now() | Дата создания |
| `updated_at` | `TIMESTAMPTZ` | default now() | Дата редактирования |
| `last_login_at` | `TIMESTAMPTZ` | nullable | Дата последнего входа в систему |
| `password_changed_at` | `TIMESTAMPTZ` | nullable | Дата последней смены пароля |
| `failed_login_attempts` | `INTEGER` | default 0 | Количество последовательных неуспешных попыток входа |
| `locked_until` | `TIMESTAMPTZ` | nullable | Время, до которого вход пользователя временно заблокирован |
| `email_verified_at` | `TIMESTAMPTZ` | nullable | Дата подтверждения email |

### 7.2 Table `auth_sessions`

| Field | Type | Constraint | Description |
|------|------|------|------|
| `id` | `UUID` | PK, not null | Уникальный идентификатор сессии |
| `user_id` | `UUID` | FK `users.id`, not null | Пользователь, которому принадлежит сессия |
| `refresh_token_hash` | `TEXT` | UNIQUE, not null | Хэш refresh-токена |
| `jwt_id` | `TEXT` | UNIQUE, not null | Идентификатор JWT (`jti`) |
| `expires_at` | `TIMESTAMPTZ` | not null | Дата истечения refresh-токена |
| `revoked_at` | `TIMESTAMPTZ` | nullable | Дата отзыва сессии |
| `revoked_reason` | `TEXT` | nullable | Причина отзыва сессии |

## 8. HTTP status codes

| HTTP status code | Description | Success | errorCode | errorMessage |
|------|------|------|------|------|
| 200 | Запрос успешно выполнен | true | — | — |
| 400 | Некорректные параметры запроса | false | `bad_request` | "Описание ошибки валидации" |
| 401 | Не авторизован | false | `unauthorized` | "Unauthorized" |
| 403 | Доступ запрещён | false | `forbidden` | "Current password is incorrect" |
| 429 | Превышен лимит запросов | false | `rate_limit_exceeded` | "Too many requests" |
| 500 | Внутренняя ошибка сервера | false | `internal_server_error` | "An internal server error occurred" |
| 503 | Сервер временно недоступен | false | `service_unavailable` | "Service is temporarily unavailable. Please try again later" |

## 9. Example

### 9.1 Response 200 OK

**Request:**

```http
PATCH /api/v1/profile/password
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "current_password": "<current_password>",
  "new_password": "<new_password>"
}
```

**Response (200):**

```json
{
  "success": true,
  "data": null,
  "errorMessage": null
}
```

### 9.2 Response 400 Bad Request

**Request:**

```http
PATCH /api/v1/profile/password
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "current_password": "",
  "new_password": "123"
}
```

**Response (400):**

```json
{
  "success": false,
  "data": null,
  "errorMessage": "Описание ошибки валидации"
}
```

### 9.3 Response 401 Unauthorized

**Request:**

```http
PATCH /api/v1/profile/password
Authorization: Bearer <invalid_access_token>
Content-Type: application/json

{
  "current_password": "<current_password>",
  "new_password": "<new_password>"
}
```

**Response (401):**

```json
{
  "success": false,
  "data": null,
  "errorMessage": "Unauthorized"
}
```

### 9.4 Response 403 Forbidden

**Request:**

```http
PATCH /api/v1/profile/password
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "current_password": "<current_password>",
  "new_password": "<new_password>"
}
```

**Response (403):**

```json
{
  "success": false,
  "data": null,
  "errorMessage": "Current password is incorrect"
}
```

### 9.5 Response 429 Too Many Requests

**Request:**

```http
PATCH /api/v1/profile/password
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "current_password": "<current_password>",
  "new_password": "<new_password>"
}
```

**Response (429):**

```json
{
  "success": false,
  "data": null,
  "errorMessage": "Too many requests"
}
```

### 9.6 Response 500 Internal Server Error

**Request:**

```http
PATCH /api/v1/profile/password
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "current_password": "<current_password>",
  "new_password": "<new_password>"
}
```

**Response (500):**

```json
{
  "success": false,
  "data": null,
  "errorMessage": "An internal server error occurred"
}
```

### 9.7 Response 503 Service Unavailable

**Request:**

```http
PATCH /api/v1/profile/password
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "current_password": "<current_password>",
  "new_password": "<new_password>"
}
```

**Response (503):**

```json
{
  "success": false,
  "data": null,
  "errorMessage": "Service is temporarily unavailable. Please try again later"
}
```

## 10. Links

| Descriptions | Link |
|------|------|
| Swagger |  |
| Postman |  |
| Storage/table | `users`, `auth_sessions` |
