# Удалить пользователя

|  |  |
|------|------|
| URL | /api/v1/users/{user_id} |
| Service Method | DELETE |
| Description | Удаляет пользователя по уникальному идентификатору |
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

### 2.2 Query params

Отсутствуют.

### 2.3 Path params

| Name | Type | Max length | Required | Description |
|------|------|------|------|------|
| user_id | STRING | 36 | Yes | Уникальный идентификатор пользователя |

### 2.4 Request body

Отсутствует (метод DELETE).

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
- **Authorization**: доступен пользователю с административными правами.
- **Session policy**: при удалении пользователя все активные refresh-сессии пользователя отзываются.
- **Rate limiting**: ограничение на количество запросов — не более 20 запросов в минуту с одного IP или токена.
- **Data Encryption**: передача данных только по HTTPS (TLS 1.2+).

## 5. How the service works

1. Клиент отправляет `DELETE /api/v1/users/{user_id}`.
2. Сервер валидирует Bearer access-токен, входные параметры и административные права.
3. Сервер отзывает все активные refresh-сессии пользователя в `auth_sessions`, заполняя `revoked_at` и `revoked_reason = user_deleted`.
4. Сервер удаляет пользователя или переводит его в неактивное состояние согласно политике хранения данных.
5. Сервер возвращает `204 No Content`.

### 5.1 Sequence diagram

Не требуется: метод не содержит сложных интеграционных сценариев.

### 5.2 Mapping

| Path param | DB column / value |
|------|------|
| user_id | `users.id`, `auth_sessions.user_id` |
| delete operation | `users.id` |
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
| `revoked_at` | `TIMESTAMPTZ` | nullable | Дата отзыва сессии |
| `revoked_reason` | `TEXT` | nullable | Причина отзыва сессии |

## 8. HTTP status codes

| HTTP status code | Description | Success | errorCode | errorMessage |
|------|------|------|------|------|
| 204 | Операция успешно выполнена | true | — | — |
| 400 | Некорректные параметры запроса | false | `bad_request` | "Описание ошибки валидации" |
| 401 | Не авторизован | false | `unauthorized` | "Unauthorized" |
| 403 | Доступ запрещён | false | `forbidden` | "Access denied" |
| 404 | Ресурс не найден | false | `not_found` | "Resource not found" |
| 429 | Превышен лимит запросов | false | `rate_limit_exceeded` | "Too many requests" |
| 500 | Внутренняя ошибка сервера | false | `internal_server_error` | "An internal server error occurred" |
| 503 | Сервер временно недоступен | false | `service_unavailable` | "Service is temporarily unavailable. Please try again later" |

## 9. Example

### 9.1 Response 204 No Content

**Request:**

```http
DELETE /api/v1/users/{user_id}
Authorization: Bearer <access_token>
```

**Response (204):**

```http
HTTP/1.1 204 No Content
```

### 9.2 Response 400 Bad Request

**Request:**

```http
DELETE /api/v1/users/invalid-user-id
Authorization: Bearer <access_token>
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
DELETE /api/v1/users/{user_id}
Authorization: Bearer <invalid_access_token>
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
DELETE /api/v1/users/{user_id}
Authorization: Bearer <access_token>
```

**Response (403):**

```json
{
  "success": false,
  "data": null,
  "errorMessage": "Access denied"
}
```

### 9.5 Response 404 Not Found

**Request:**

```http
DELETE /api/v1/users/99999999-9999-9999-9999-999999999999
Authorization: Bearer <access_token>
```

**Response (404):**

```json
{
  "success": false,
  "data": null,
  "errorMessage": "Resource not found"
}
```

### 9.6 Response 429 Too Many Requests

**Request:**

```http
DELETE /api/v1/users/{user_id}
Authorization: Bearer <access_token>
```

**Response (429):**

```json
{
  "success": false,
  "data": null,
  "errorMessage": "Too many requests"
}
```

### 9.7 Response 500 Internal Server Error

**Request:**

```http
DELETE /api/v1/users/{user_id}
Authorization: Bearer <access_token>
```

**Response (500):**

```json
{
  "success": false,
  "data": null,
  "errorMessage": "An internal server error occurred"
}
```

### 9.8 Response 503 Service Unavailable

**Request:**

```http
DELETE /api/v1/users/{user_id}
Authorization: Bearer <access_token>
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
