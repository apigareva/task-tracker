# Обновить свой профиль

|  |  |
|------|------|
| URL | /api/v1/profile |
| Service Method | PUT |
| Description | Обновляет профиль текущего пользователя |
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
| email | STRING | 320 | Yes | Почта пользователя |
| phone | STRING | 20 | No | Телефон пользователя |
| first_name | STRING | 100 | No | Имя пользователя |
| last_name | STRING | 100 | No | Фамилия пользователя |

## 3. Response parameters

| Name | Type | Max length | Description |
|------|------|------|------|
| success | BOOLEAN | — | `true` — запрос выполнен успешно, `false` — ошибка |
| errorMessage | STRING | — | Сообщение об ошибке (при `success: false`) |
| data | OBJECT | — | Объект с данными |
| data.id | STRING | 36 | Уникальный идентификатор |
| data.email | STRING | — | Почта пользователя |
| data.phone | STRING | 20 | Телефон пользователя |
| data.first_name | STRING | 100 | Имя пользователя |
| data.last_name | STRING | 100 | Фамилия пользователя |
| data.status | STRING | 100 | Статус пользователя |
| data.created_at | DATE/TIME | — | Дата создания |
| data.updated_at | DATE/TIME | — | Дата редактирования |
| data.last_login_at | DATE/TIME | — | Дата последнего входа в систему |

## 4. Security requirements of method

- **Authentication**: обязательна — JWT Bearer access-токен в заголовке `Authorization`.
- **Authorization**: пользователь обновляет только собственный профиль.
- **Rate limiting**: ограничение на количество запросов — не более 30 запросов в минуту с одного IP или токена.
- **Data Encryption**: передача данных только по HTTPS (TLS 1.2+).

## 5. How the service works

1. Клиент отправляет `PUT /api/v1/profile`.
2. Сервер валидирует Bearer access-токен и определяет пользователя по JWT claim `sub`.
3. Сервер обновляет профиль в `users` по `users.id = sub`.
4. Сервер формирует ответ без `password_hash` и security-полей.

### 5.1 Sequence diagram

Не требуется: метод не содержит сложных интеграционных сценариев.

### 5.2 Mapping

| Request body | DB column / value |
|------|------|
| JWT claim sub | `users.id` |
| email | `users.email` |
| phone | `users.phone` |
| first_name | `users.first_name` |
| last_name | `users.last_name` |

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

## 8. HTTP status codes

| HTTP status code | Description | Success | errorCode | errorMessage |
|------|------|------|------|------|
| 200 | Запрос успешно выполнен | true | — | — |
| 400 | Некорректные параметры запроса | false | `bad_request` | "Описание ошибки валидации" |
| 401 | Не авторизован | false | `unauthorized` | "Unauthorized" |
| 429 | Превышен лимит запросов | false | `rate_limit_exceeded` | "Too many requests" |
| 500 | Внутренняя ошибка сервера | false | `internal_server_error` | "An internal server error occurred" |
| 503 | Сервер временно недоступен | false | `service_unavailable` | "Service is temporarily unavailable. Please try again later" |

## 9. Example

### 9.1 Response 200 OK

**Request:**

```http
PUT /api/v1/profile
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "email": "user@example.com",
  "phone": "+79990000000",
  "first_name": "Иван",
  "last_name": "Иванов"
}
```

**Response (200):**

```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "user@example.com",
    "phone": "+79990000000",
    "first_name": "Иван",
    "last_name": "Иванов",
    "status": "ACTIVE",
    "created_at": "2026-05-15T09:30:00Z",
    "updated_at": "2026-05-15T09:30:00Z",
    "last_login_at": "2026-05-18T10:00:00Z"
  },
  "errorMessage": null
}
```

### 9.2 Response 400 Bad Request

**Request:**

```http
PUT /api/v1/profile
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "email": "invalid-email"
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
PUT /api/v1/profile
Authorization: Bearer <invalid_access_token>
Content-Type: application/json

{
  "email": "user@example.com",
  "phone": "+79990000000",
  "first_name": "Иван",
  "last_name": "Иванов"
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

### 9.4 Response 429 Too Many Requests

**Request:**

```http
PUT /api/v1/profile
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "email": "user@example.com",
  "phone": "+79990000000",
  "first_name": "Иван",
  "last_name": "Иванов"
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

### 9.5 Response 500 Internal Server Error

**Request:**

```http
PUT /api/v1/profile
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "email": "user@example.com",
  "phone": "+79990000000",
  "first_name": "Иван",
  "last_name": "Иванов"
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

### 9.6 Response 503 Service Unavailable

**Request:**

```http
PUT /api/v1/profile
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "email": "user@example.com",
  "phone": "+79990000000",
  "first_name": "Иван",
  "last_name": "Иванов"
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
| Storage/table | `users` |
