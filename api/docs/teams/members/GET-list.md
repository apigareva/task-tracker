# Получить список участников команды

|  |  |
|------|------|
| URL | /api/v1/teams/{team_id}/members |
| Service Method | GET |
| Description | Возвращает список участников команды |
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

### 2.2 Query params

| Name | Type | Max length | Required | Description |
|------|------|------|------|------|
| limit | NUMBER | — | No | Максимальное количество записей для загрузки |
| offset | NUMBER | — | No | Количество записей, которые нужно пропустить |

### 2.3 Path params

| Name | Type | Max length | Required | Description |
|------|------|------|------|------|
| team_id | STRING | 36 | Yes | Уникальный идентификатор команды |

### 2.4 Request body

Отсутствует (метод GET).

## 3. Response parameters

| Name | Type | Max length | Description |
|------|------|------|------|
| success | BOOLEAN | — | `true` — запрос выполнен успешно, `false` — ошибка |
| errorMessage | STRING | — | Сообщение об ошибке (при `success: false`) |
| data | ARRAY OF OBJECTS | — | Массив объектов |
| data[].id | STRING | 36 | Уникальный идентификатор |
| data[].member_id | STRING | 36 | Пользователь |
| data[].team_id | STRING | 36 | Команда, к которой относится |
| data[].role_id | STRING | 36 | Роль в команде |
| total | NUMBER | — | Общее количество записей с учётом фильтров, без пагинации |

## 4. Security requirements of method

- **Authentication**: обязательна — JWT Bearer access-токен в заголовке `Authorization`.
- **Authorization**: пользователь может получить список участников команды, к которой у него есть доступ.
- **Rate limiting**: ограничение на количество запросов — не более 100 запросов в минуту с одного IP или токена.
- **Data Encryption**: передача данных только по HTTPS (TLS 1.2+).

## 5. How the service works

1. Клиент отправляет `GET /api/v1/teams/{team_id}/members`.
2. Сервер валидирует входные параметры и права доступа.
3. Сервер выполняет операцию и формирует ответ.

### 5.1 Sequence diagram

Не требуется: метод не содержит сложных интеграционных сценариев.

### 5.2 Mapping

| Path param | DB column / value |
|------|------|
| team_id | `team_id` |

| Query param | DB column / value |
|------|------|
| limit | `limit` |
| offset | `offset` |

## 6. Logging requirements

- Логировать: `user_id`, временную метку запроса, параметры запроса, HTTP-статус ответа.
- Логировать ошибки аутентификации (401), авторизации (403) и внутренние ошибки сервера (500) с уровнем `ERROR`.

## 7. Data model used by method

### 7.1 Table `members`

| Field | Type | Constraint | Description |
|------|------|------|------|
| `id` | `UUID` | PK, not null | Уникальный идентификатор |
| `member_id` | `UUID` | FK users.id, not null | Пользователь |
| `team_id` | `UUID` | FK teams.id, not null | Команда, к которой относится |
| `role_id` | `UUID` | FK roles.id, not null | Роль в команде |

## 8. HTTP status codes

| HTTP status code | Description | Success | errorCode | errorMessage |
|------|------|------|------|------|
| 200 | Запрос успешно выполнен | true | — | — |
| 400 | Некорректные параметры запроса | false | `bad_request` | "Описание ошибки валидации" |
| 401 | Не авторизован | false | `unauthorized` | "Unauthorized" |
| 403 | Доступ запрещён | false | `forbidden` | "Access denied" |
| 404 | Ресурс не найден | false | `not_found` | "Resource not found" |
| 429 | Превышен лимит запросов | false | `rate_limit_exceeded` | "Too many requests" |
| 500 | Внутренняя ошибка сервера | false | `internal_server_error` | "An internal server error occurred" |
| 503 | Сервер временно недоступен | false | `service_unavailable` | "Service is temporarily unavailable. Please try again later" |

## 9. Example

### 9.1 Response 200 OK

**Request:**

```http
GET /api/v1/teams/{team_id}/members
Authorization: Bearer <access_token>
```

**Response (200):**

```json
{
  "success": true,
  "data": [
    {
      "id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
      "member_id": "550e8400-e29b-41d4-a716-446655440000",
      "team_id": "11111111-1111-1111-1111-111111111111",
      "role_id": "22222222-2222-2222-2222-222222222222"
    }
  ],
  "total": 1,
  "errorMessage": null
}
```

### 9.2 Response 400 Bad Request

**Request:**

```http
GET /api/v1/teams/invalid-team-id/members
Authorization: Bearer <access_token>
```

**Response (400):**

```json
{
  "success": false,
  "data": null,
  "total": null,
  "errorMessage": "Описание ошибки валидации"
}
```

### 9.3 Response 401 Unauthorized

**Request:**

```http
GET /api/v1/teams/{team_id}/members
Authorization: Bearer <invalid_access_token>
```

**Response (401):**

```json
{
  "success": false,
  "data": null,
  "total": null,
  "errorMessage": "Unauthorized"
}
```

### 9.4 Response 403 Forbidden

**Request:**

```http
GET /api/v1/teams/{team_id}/members
Authorization: Bearer <access_token>
```

**Response (403):**

```json
{
  "success": false,
  "data": null,
  "total": null,
  "errorMessage": "Access denied"
}
```

### 9.5 Response 404 Not Found

**Request:**

```http
GET /api/v1/teams/99999999-9999-9999-9999-999999999999/members
Authorization: Bearer <access_token>
```

**Response (404):**

```json
{
  "success": false,
  "data": null,
  "total": null,
  "errorMessage": "Resource not found"
}
```

### 9.6 Response 429 Too Many Requests

**Request:**

```http
GET /api/v1/teams/{team_id}/members
Authorization: Bearer <access_token>
```

**Response (429):**

```json
{
  "success": false,
  "data": null,
  "total": null,
  "errorMessage": "Too many requests"
}
```

### 9.7 Response 500 Internal Server Error

**Request:**

```http
GET /api/v1/teams/{team_id}/members
Authorization: Bearer <access_token>
```

**Response (500):**

```json
{
  "success": false,
  "data": null,
  "total": null,
  "errorMessage": "An internal server error occurred"
}
```

### 9.8 Response 503 Service Unavailable

**Request:**

```http
GET /api/v1/teams/{team_id}/members
Authorization: Bearer <access_token>
```

**Response (503):**

```json
{
  "success": false,
  "data": null,
  "total": null,
  "errorMessage": "Service is temporarily unavailable. Please try again later"
}
```

## 10. Links

| Descriptions | Link |
|------|------|
| Swagger |  |
| Postman |  |
| Storage/table | `members` |
