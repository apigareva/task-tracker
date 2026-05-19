# Обновить проект

|  |  |
|------|------|
| URL | /api/v1/projects/{project_id} |
| Service Method | PATCH |
| Description | Обновляет данные проекта |
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

| Name | Type | Max length | Required | Description |
|------|------|------|------|------|
| project_id | STRING | 36 | Yes | Уникальный идентификатор проекта |

### 2.4 Request body

| Name | Type | Max length | Required | Description |
|------|------|------|------|------|
| title | STRING | 100 | No | Название проекта |
| description | STRING | — | No | Описание проекта |
| owner_id | STRING | 36 | No | UUID ответственного участника |

## 3. Response parameters

| Name | Type | Max length | Description |
|------|------|------|------|
| success | BOOLEAN | — | `true` — запрос выполнен успешно, `false` — ошибка |
| errorMessage | STRING | — | Сообщение об ошибке (при `success: false`) |
| data | OBJECT | — | Объект с данными |
| data.id | STRING | 36 | Уникальный идентификатор |
| data.title | STRING | 100 | Название |
| data.description | STRING | — | Описание |
| data.created_by | STRING | 36 | Кем создан |
| data.created_at | DATE/TIME | — | Дата создания |
| data.team_id | STRING | 36 | Команда, к которой относится |
| data.owner_id | STRING | 36 | Ответственный за проект |

## 4. Security requirements of method

- **Authentication**: обязательна — JWT Bearer access-токен в заголовке `Authorization`.
- **Authorization**: пользователь может обновить только проект, к которому у него есть доступ.
- **Rate limiting**: ограничение на количество запросов — не более 60 запросов в минуту с одного IP или токена.
- **Data Encryption**: передача данных только по HTTPS (TLS 1.2+).

## 5. How the service works

1. Клиент отправляет `PATCH /api/v1/projects/{project_id}`.
2. Сервер валидирует входные параметры и права доступа.
3. Сервер выполняет операцию и формирует ответ.

### 5.1 Sequence diagram

Не требуется: метод не содержит сложных интеграционных сценариев.

### 5.2 Mapping

| Path param | DB column / value |
|------|------|
| project_id | `project_id` |

| Request body | DB column / value |
|------|------|
| title | `projects.title` |
| description | `projects.description` |
| owner_id | `projects.owner_id` |

## 6. Logging requirements

- Логировать: `user_id`, временную метку запроса, параметры запроса, HTTP-статус ответа.
- Логировать ошибки аутентификации (401), авторизации (403) и внутренние ошибки сервера (500) с уровнем `ERROR`.

## 7. Data model used by method

### 7.1 Table `projects`

| Field | Type | Constraint | Description |
|------|------|------|------|
| `id` | `UUID` | PK, not null | Уникальный идентификатор |
| `title` | `VARCHAR(100)` | not null | Название |
| `description` | `TEXT` | nullable | Описание |
| `created_by` | `UUID` | FK users.id, not null | Кем создан |
| `created_at` | `TIMESTAMPTZ` | default now() | Дата создания |
| `team_id` | `UUID` | FK teams.id, not null | Команда, к которой относится |
| `owner_id` | `UUID` | FK members.id, not null | Ответственный за проект |

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
PATCH /api/v1/projects/{project_id}
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "title": "Task Tracker API",
  "description": "Разработка API для приложения трекинга задач",
  "owner_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8"
}
```

**Response (200):**

```json
{
  "success": true,
  "data": {
    "id": "a1b2c3d4-0000-0000-0000-000000000001",
    "title": "Task Tracker API",
    "description": "Разработка API для приложения трекинга задач",
    "created_by": "550e8400-e29b-41d4-a716-446655440000",
    "created_at": "2026-05-15T09:30:00Z",
    "team_id": "11111111-1111-1111-1111-111111111111",
    "owner_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8"
  },
  "errorMessage": null
}
```

### 9.2 Response 400 Bad Request

**Request:**

```http
PATCH /api/v1/projects/invalid-id
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
PATCH /api/v1/projects/{project_id}
Authorization: Bearer <invalid_access_token>
Content-Type: application/json

{
  "title": "Task Tracker API",
  "description": "Разработка API для приложения трекинга задач",
  "owner_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8"
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
PATCH /api/v1/projects/{project_id}
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "title": "Task Tracker API",
  "description": "Разработка API для приложения трекинга задач",
  "owner_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8"
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
PATCH /api/v1/projects/99999999-9999-9999-9999-999999999999
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "title": "Task Tracker API",
  "description": "Разработка API для приложения трекинга задач",
  "owner_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8"
}
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
PATCH /api/v1/projects/{project_id}
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "title": "Task Tracker API",
  "description": "Разработка API для приложения трекинга задач",
  "owner_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8"
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
PATCH /api/v1/projects/{project_id}
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "title": "Task Tracker API",
  "description": "Разработка API для приложения трекинга задач",
  "owner_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8"
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
PATCH /api/v1/projects/{project_id}
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "title": "Task Tracker API",
  "description": "Разработка API для приложения трекинга задач",
  "owner_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8"
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
| Storage/table | `projects` |
