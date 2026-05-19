# Обновить токены авторизации

|  |  |
|------|------|
| URL | /api/v1/auth/refresh |
| Service Method | POST |
| Description | Обновляет access token по refresh token |
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
| Content-Type | STRING | Yes | `application/json` |

### 2.2 Query params

Отсутствуют.

### 2.3 Path params

Отсутствуют.

### 2.4 Request body

| Name | Type | Max length | Required | Description |
|------|------|------|------|------|
| refresh_token | STRING | — | Yes | Refresh-токен |

## 3. Response parameters

| Name | Type | Max length | Description |
|------|------|------|------|
| success | BOOLEAN | — | `true` — запрос выполнен успешно, `false` — ошибка |
| errorMessage | STRING | — | Сообщение об ошибке (при `success: false`) |
| data | OBJECT | — | Объект с токенами и данными пользователя |
| data.access_token | STRING | — | Access JWT-токен |
| data.refresh_token | STRING | — | Refresh-токен |
| data.expires_in | NUMBER | — | Время жизни access-токена в секундах |
| data.user | OBJECT | — | Данные пользователя |
| data.user.id | STRING | 36 | UUID пользователя |
| data.user.email | STRING | 320 | Email пользователя |
| data.user.status | STRING | 100 | Статус пользователя |

## 4. Security requirements of method

- **Authentication**: не требуется для вызова метода.
- **Rate limiting**: ограничение на количество запросов — не более 30 запросов в минуту с одного IP или токена.
- **Data Encryption**: передача данных только по HTTPS (TLS 1.2+).

## 5. How the service works

1. Клиент отправляет `POST /api/v1/auth/refresh`.
2. Сервер валидирует формат `refresh_token` и вычисляет его хэш.
3. Сервер ищет активную запись в `auth_sessions` по `refresh_token_hash`.
4. Сервер проверяет, что сессия не отозвана, `auth_sessions.expires_at` не истек, а пользователь активен.
5. Сервер выполняет ротацию refresh-токена: отзывает текущую сессию и создает новую запись в `auth_sessions`.
6. Сервер возвращает новый access JWT, новый refresh-токен и данные пользователя.

### 5.1 Sequence diagram

Не требуется: метод не содержит сложных интеграционных сценариев.

### 5.2 Mapping

| Request body | DB column / value |
|------|------|
| refresh_token | Хэшируется и сравнивается с `auth_sessions.refresh_token_hash` |
| generated refresh_token | `auth_sessions.refresh_token_hash` новой сессии |
| generated JWT `jti` | `auth_sessions.jwt_id` новой сессии |
| previous session | `auth_sessions.revoked_at`, `auth_sessions.revoked_reason`, `auth_sessions.replaced_by_session_id` |

## 6. Logging requirements

- Логировать: `user_id`, временную метку запроса, параметры запроса, HTTP-статус ответа.
- Логировать ошибки аутентификации (401), авторизации (403) и внутренние ошибки сервера (500) с уровнем `ERROR`.

## 7. Data model used by method

### 7.1 Table name

| Table | Purpose |
|------|------|
| `auth_sessions` | Поиск активной сессии по хэшу refresh-токена, отзыв старой сессии и создание новой |
| `users` | Проверка статуса пользователя и возврат пользовательских данных |

## 8. HTTP status codes

| HTTP status code | Description | Success | errorCode | errorMessage |
|------|------|------|------|------|
| 200 | Запрос успешно выполнен | true | — | — |
| 400 | Некорректные параметры запроса | false | `bad_request` | "Описание ошибки валидации" |
| 401 | Не авторизован | false | `unauthorized` | "Invalid or expired refresh token" |
| 429 | Превышен лимит запросов | false | `rate_limit_exceeded` | "Too many requests" |
| 500 | Внутренняя ошибка сервера | false | `internal_server_error` | "An internal server error occurred" |
| 503 | Сервер временно недоступен | false | `service_unavailable` | "Service is temporarily unavailable. Please try again later" |

## 9. Example

### 9.1 Response 200 OK

**Request:**

```http
POST /api/v1/auth/refresh
Content-Type: application/json

{
  "refresh_token": "<refresh_token>"
}
```

**Response (200):**

```json
{
  "success": true,
  "data": {
    "access_token": "<access_token>",
    "refresh_token": "<refresh_token>",
    "expires_in": 3600,
    "user": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "email": "user@example.com",
      "status": "ACTIVE"
    }
  },
  "errorMessage": null
}
```

### 9.2 Response 400 Bad Request

**Request:**

```http
POST /api/v1/auth/refresh
Content-Type: application/json

{
  "refresh_token": ""
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
POST /api/v1/auth/refresh
Content-Type: application/json

{
  "refresh_token": "<refresh_token>"
}
```

**Response (401):**

```json
{
  "success": false,
  "data": null,
  "errorMessage": "Invalid or expired refresh token"
}
```

### 9.4 Response 429 Too Many Requests

**Request:**

```http
POST /api/v1/auth/refresh
Content-Type: application/json

{
  "refresh_token": "<refresh_token>"
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
POST /api/v1/auth/refresh
Content-Type: application/json

{
  "refresh_token": "<refresh_token>"
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
POST /api/v1/auth/refresh
Content-Type: application/json

{
  "refresh_token": "<refresh_token>"
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
| Storage/table | `auth_sessions`, `users` |
