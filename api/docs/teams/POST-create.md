# Создать команду

|  |  |
|------|------|
| URL | /api/v1/teams |
| Service Method | POST |
| Description | Создает команду |
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
| title | STRING | 100 | Yes | Название команды |
| description | STRING | — | No | Описание команды |

## 3. Response parameters

| Name | Type | Max length | Description |
|------|------|------|------|
| success | BOOLEAN | — | `true` — запрос выполнен успешно, `false` — ошибка |
| errorMessage | STRING | — | Сообщение об ошибке (при `success: false`) |
| data | OBJECT | — | Объект с данными |
| data.id | STRING | 36 | Уникальный идентификатор |
| data.title | STRING | 100 | Название команды |
| data.description | STRING | — | Описание |
| data.created_by | STRING | 36 | Кем создана |
| data.created_at | DATE/TIME | — | Дата создания |

## 4. Security requirements of method

- **Authentication**: обязательна — JWT Bearer access-токен в заголовке `Authorization`.
- **Authorization**: пользователь может создать команду при наличии необходимых прав.
- **Rate limiting**: ограничение на количество запросов — не более 60 запросов в минуту с одного IP или токена.
- **Data Encryption**: передача данных только по HTTPS (TLS 1.2+).

## 5. How the service works

1. Клиент отправляет `POST /api/v1/teams`.
2. Сервер валидирует входные параметры и права доступа.
3. Сервер выполняет операцию и формирует ответ.

### 5.1 Sequence diagram

Не требуется: метод не содержит сложных интеграционных сценариев.

### 5.2 Mapping

| Request body | DB column / value |
|------|------|
| title | `teams.title` |
| description | `teams.description` |

## 6. Logging requirements

- Логировать: `user_id`, временную метку запроса, параметры запроса, HTTP-статус ответа.
- Логировать ошибки аутентификации (401), авторизации (403) и внутренние ошибки сервера (500) с уровнем `ERROR`.

## 7. Data model used by method

### 7.1 Table `teams`

| Field | Type | Constraint | Description |
|------|------|------|------|
| `id` | `UUID` | PK, not null | Уникальный идентификатор |
| `title` | `VARCHAR(100)` | not null | Название команды |
| `description` | `TEXT` | nullable | Описание |
| `created_by` | `UUID` | FK users.id, not null | Кем создана |
| `created_at` | `TIMESTAMPTZ` | default now() | Дата создания |

## 8. HTTP status codes

| HTTP status code | Description | Success | errorCode | errorMessage |
|------|------|------|------|------|
| 201 | Ресурс успешно создан | true | — | — |
| 400 | Некорректные параметры запроса | false | `bad_request` | "Описание ошибки валидации" |
| 401 | Не авторизован | false | `unauthorized` | "Unauthorized" |
| 403 | Доступ запрещён | false | `forbidden` | "Access denied" |
| 404 | Ресурс не найден | false | `not_found` | "Related entity not found" |
| 429 | Превышен лимит запросов | false | `rate_limit_exceeded` | "Too many requests" |
| 500 | Внутренняя ошибка сервера | false | `internal_server_error` | "An internal server error occurred" |
| 503 | Сервер временно недоступен | false | `service_unavailable` | "Service is temporarily unavailable. Please try again later" |

## 9. Example

### 9.1 Response 201 Created

**Request:**

```http
POST /api/v1/teams
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "title": "Backend Team",
  "description": "Команда backend-разработки"
}
```

**Response (201):**

```json
{
  "success": true,
  "data": {
    "id": "11111111-1111-1111-1111-111111111111",
    "title": "Backend Team",
    "description": "Команда backend-разработки",
    "created_by": "550e8400-e29b-41d4-a716-446655440000",
    "created_at": "2026-05-15T09:30:00Z"
  },
  "errorMessage": null
}
```

### 9.2 Response 400 Bad Request

**Request:**

```http
POST /api/v1/teams
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "title": ""
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
POST /api/v1/teams
Authorization: Bearer <invalid_access_token>
Content-Type: application/json

{
  "title": "Backend Team",
  "description": "Команда backend-разработки"
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
POST /api/v1/teams
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "title": "Backend Team",
  "description": "Команда backend-разработки"
}
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
POST /api/v1/teams
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "title": "Backend Team",
  "description": "Команда backend-разработки"
}
```

**Response (404):**

```json
{
  "success": false,
  "data": null,
  "errorMessage": "Related entity not found"
}
```

### 9.6 Response 429 Too Many Requests

**Request:**

```http
POST /api/v1/teams
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "title": "Backend Team",
  "description": "Команда backend-разработки"
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

### 9.7 Response 500 Internal Server Error

**Request:**

```http
POST /api/v1/teams
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "title": "Backend Team",
  "description": "Команда backend-разработки"
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

### 9.8 Response 503 Service Unavailable

**Request:**

```http
POST /api/v1/teams
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "title": "Backend Team",
  "description": "Команда backend-разработки"
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
| Storage/table | `teams` |
