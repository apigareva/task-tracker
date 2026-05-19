# Выйти из системы

|  |  |
|------|------|
| URL | /api/v1/auth/logout |
| Service Method | POST |
| Description | Завершает текущую пользовательскую сессию |
| Return Type | No Content |
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
| refresh_token | STRING | — | No | Refresh-токен, который нужно отозвать |

## 3. Response parameters

Для успешного ответа `204 No Content` тело ответа отсутствует.

Для ошибок возвращается JSON:

| Name | Type | Max length | Description |
|------|------|------|------|
| success | BOOLEAN | — | `false` — ошибка |
| data | OBJECT | — | `null` |
| errorMessage | STRING | — | Сообщение об ошибке |

## 4. Security requirements of method

- **Authentication**: обязательна — JWT Bearer access-токен в заголовке `Authorization`.
- **Rate limiting**: ограничение на количество запросов — не более 30 запросов в минуту с одного IP или токена.
- **Data Encryption**: передача данных только по HTTPS (TLS 1.2+).

## 5. How the service works

1. Клиент отправляет `POST /api/v1/auth/logout`.
2. Сервер валидирует Bearer access JWT и определяет пользователя по `sub`.
3. Если передан `refresh_token`, сервер вычисляет его хэш и находит соответствующую запись в `auth_sessions`.
4. Сервер отзывает текущую сессию, заполняя `auth_sessions.revoked_at` и `auth_sessions.revoked_reason`.
5. Сервер возвращает `204 No Content`.

### 5.1 Sequence diagram

Не требуется: метод не содержит сложных интеграционных сценариев.

### 5.2 Mapping

| Request body | DB column / value |
|------|------|
| Authorization Bearer access_token | JWT claims `sub`, `jti` |
| refresh_token | Хэшируется и сравнивается с `auth_sessions.refresh_token_hash` |
| logout operation | `auth_sessions.revoked_at`, `auth_sessions.revoked_reason` |

## 6. Logging requirements

- Логировать: `user_id`, временную метку запроса, параметры запроса, HTTP-статус ответа.
- Логировать ошибки аутентификации (401), авторизации (403) и внутренние ошибки сервера (500) с уровнем `ERROR`.

## 7. Data model used by method

### 7.1 Table name

| Table | Purpose |
|------|------|
| `auth_sessions` | Отзыв пользовательской сессии по refresh-токену или JWT `jti` |
| `users` | Проверка существования и статуса пользователя из Bearer JWT |

## 8. HTTP status codes

| HTTP status code | Description | Success | errorCode | errorMessage |
|------|------|------|------|------|
| 204 | Операция успешно выполнена | true | — | — |
| 400 | Некорректные параметры запроса | false | `bad_request` | "Описание ошибки валидации" |
| 401 | Не авторизован | false | `unauthorized` | "Unauthorized" |
| 429 | Превышен лимит запросов | false | `rate_limit_exceeded` | "Too many requests" |
| 500 | Внутренняя ошибка сервера | false | `internal_server_error` | "An internal server error occurred" |
| 503 | Сервер временно недоступен | false | `service_unavailable` | "Service is temporarily unavailable. Please try again later" |

## 9. Example

### 9.1 Response 204 No Content

**Request:**

```http
POST /api/v1/auth/logout
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "refresh_token": "<refresh_token>"
}
```

**Response (204):**

```http
HTTP/1.1 204 No Content
```

### 9.2 Response 400 Bad Request

**Request:**

```http
POST /api/v1/auth/logout
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "refresh_token": 123
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
POST /api/v1/auth/logout
Authorization: Bearer <invalid_access_token>
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
  "errorMessage": "Unauthorized"
}
```

### 9.4 Response 429 Too Many Requests

**Request:**

```http
POST /api/v1/auth/logout
Authorization: Bearer <access_token>
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
POST /api/v1/auth/logout
Authorization: Bearer <access_token>
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
POST /api/v1/auth/logout
Authorization: Bearer <access_token>
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
