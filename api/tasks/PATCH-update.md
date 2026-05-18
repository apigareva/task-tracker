# Обновить задачу

| | |
|------|------------------------|
| URL | /api/v1/tasks/{task_id} |
| Service Method | PATCH | 
| Description | Обновляет данные задачи |
| Return Type| JSON |
| Third-party service or system name| — |

## 1. List of changes

| Date | Responsible SA | JIRA | What was changed |
|------|---------------|------|-----------------|
| 15.05.2026 | | | Первичное создание |

## 2. Input parameters

### 2.1 Header parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| Authorization | String | Yes | Bearer-токен авторизации (`Bearer <token>`) |
| Content-Type | String | Yes | `application/json` |

### 2.2 Query params

Отсутствуют.

### 2.3 Path params

| Name | Type | Max length | Required | Description |
|------|------|------------|----------|-------------|
| task_id | STRING | 36 | Yes | Уникальный идентификатор задачи |

### 2.4 Request body

| Name | Type | Max length | Required | Description |
|------|------|------------|----------|-------------|
| title | STRING | 100 | No | Название задачи |
| description | STRING | — | No | Описание задачи |
| priority | STRING | — | No | Приоритет задачи (`task_priority` ENUM) |
| state | STRING | — | No | Статус задачи (`task_state` ENUM) |
| due_date | DATE | — | No | Дата завершения задачи |
| assigned_to | STRING | 36 | No | UUID исполнителя задачи (`members.id`) |
| project_id | STRING | 36 | No | UUID проекта (`projects.id`) |

## 3. Response parameters

| Name | Type | Max length | Description |
|------|------|------------|-------------|
| success | BOOLEAN | — | `true` — запрос выполнен успешно, `false` — ошибка |
| errorMessage | STRING | — | Сообщение об ошибке (при `success: false`) |
| data | OBJECT | — | Объект с данными задачи |
| data.id | STRING | 36 | Уникальный идентификатор задачи (UUID) |
| data.title | STRING | 100 | Название задачи |
| data.description | STRING | — | Описание задачи |
| data.priority | STRING | — | Приоритет задачи (`task_priority` ENUM) |
| data.state | STRING | — | Статус задачи (`task_state` ENUM) |
| data.due_date | DATE | — | Дата завершения задачи |
| data.created_by | STRING | 36 | UUID создателя задачи (`users.id`) |
| data.created_at | DATE/TIME | — | Дата и время создания задачи |
| data.assigned_to | STRING | 36 | UUID исполнителя задачи (`members.id`) |
| data.project_id | STRING | 36 | UUID проекта (`projects.id`) |

## 4. Security requirements of method

- **Authentication**: обязательна — JWT Bearer-токен в заголовке `Authorization`.
- **Authorization**: пользователь может обновить данные задачи, к которой у него есть доступ.
- **Rate limiting**: ограничение на количество запросов — не более 100 запросов в минуту с одного токена.
- **Data Encryption**: передача данных только по HTTPS (TLS 1.2+).

## 5. How the service works

1. Клиент отправляет `PATCH /api/v1/tasks/{task_id}` с Bearer-токеном в заголовке и данными в теле запроса для обновления задачи.
2. Сервер валидирует токен и определяет идентификатор пользователя.
3. Осуществляется поиск задачи по уникальному идентификатору `task_id`.
4. Сервер проверяет имеет ли доступ пользователь к данным этой задачи: пользователь может обновить задачу, если является создателем задачи, исполнителем или участником проекта (projects.members), к которому привязана задача.
5. Сервер обновляет только те данные задачи, которые указаны в теле запроса и если у пользователя есть доступ к редактированию.
6. Возвращаются обновленные данные задачи.

### 5.1 Sequence diagram

### 5.2 Mapping

| Path param | DB column (таблица `tasks`) |
|-------------|------------------------------|
| task_id | `tasks.id` |

| Request body | DB column (таблица `tasks`) |
|-------------|------------------------------|
| title | `tasks.title` |
| description | `tasks.description` |
| priority | `tasks.priority` |
| state | `tasks.state` |
| due_date | `tasks.due_date` |
| assigned_to | `tasks.assigned_to` |
| project_id | `tasks.project_id` |

## 6. Logging requirements

- Логировать: `user_id`, `task_id`, временную метку запроса, тело запроса, HTTP-статус ответа.
- Логировать ошибки аутентификации (401) и внутренние ошибки сервера (500) с уровнем `ERROR`.

## 7. Data model used by method

### 7.1 Table `tasks`

| Field | Type | Constraint | Description |
|-------|------|------------|-------------|
| `id` | `UUID` | PK, not null | Уникальный идентификатор |
| `title` | `VARCHAR(100)` | not null | Название задачи |
| `description` | `TEXT` | nullable | Описание задачи |
| `priority` | `TEXT` | not null, ENUM `task_priority` | Приоритет |
| `state` | `TEXT` | not null, ENUM `task_state` | Статус |
| `due_date` | `DATE` | not null | Дата завершения |
| `created_by` | `UUID` | FK `users.id`, not null | Кем создана |
| `created_at` | `TIMESTAMPTZ` | default `now()` | Дата создания |
| `assigned_to` | `UUID` | FK `members.id`, not null | Исполнитель |
| `project_id` | `UUID` | FK `projects.id`, not null | Проект |

## 8. HTTP status codes

| HTTP status code | Description | Success | errorCode | errorMessage |
|-----------------|-------------|---------|-----------|--------------|
| 200 | Данные задачи успешно обновлены | true | — | — |
| 400 | Некорректные параметры запроса | false | `bad_request` | Описание ошибки валидации |
| 401 | Не авторизован | false | `unauthorized` | "Unauthorized" |
| 403 | Доступ запрещён | false | `forbidden` | "Access denied" |
| 404 | Задача не найдена | false | `not_found` | "Task with id {task_id} not found" |
| 429 | Превышен лимит запросов | false | `rate_limit_exceeded` | "Too many requests" |
| 500 | Внутренняя ошибка сервера | false | `internal_server_error` | "An internal server error occurred" |
| 503 | Сервер временно недоступен | false | `service_unavailable` | "Service is temporarily unavailable. Please try again later" |

## 9. Example

### 9.1 Response 200 OK

**Request:**

```http
PATCH /api/v1/tasks/123e4567-e89b-12d3-a456-426614174000
Authorization: Bearer <your_token_here>
Content-Type: application/json

{
  "title": "Разработать API для управления задачами",
  "state": "IN_PROGRESS",
  "priority": "HIGH"
}
```

**Response (200):**

```json
{
  "success": true,
  "data": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "title": "Разработать API для управления задачами",
    "description": "Реализовать CRUD операции для задач с учётом прав доступа",
    "priority": "HIGH",
    "state": "IN_PROGRESS",
    "due_date": "2026-05-30",
    "created_by": "550e8400-e29b-41d4-a716-446655440000",
    "created_at": "2026-05-15T09:30:00Z",
    "assigned_to": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
    "project_id": "a1b2c3d4-0000-0000-0000-000000000001"
  },
  "errorMessage": null
}
```

### 9.2 Response 400 Bad Request

**Request:**

```http
PATCH /api/v1/tasks/123e4567-e89b-12d3-a456-426614174000
Authorization: Bearer <your_token_here>
Content-Type: application/json

{
  "priority": "CRITICAL"
}
```

**Response (400):**

```json
{
  "success": false,
  "data": null,
  "errorMessage": "Invalid priority value"
}
```

### 9.3 Response 401 Unauthorized

**Request:**

```http
PATCH /api/v1/tasks/123e4567-e89b-12d3-a456-426614174000
Authorization: Bearer <invalid_token>
Content-Type: application/json

{
  "title": "Разработать API для управления задачами"
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
PATCH /api/v1/tasks/123e4567-e89b-12d3-a456-426614174000
Authorization: Bearer <your_token_here>
Content-Type: application/json

{
  "assigned_to": "7ba7b810-9dad-11d1-80b4-00c04fd430c8"
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
PATCH /api/v1/tasks/99999999-9999-9999-9999-999999999999
Authorization: Bearer <your_token_here>
Content-Type: application/json

{
  "title": "Новый заголовок"
}
```

**Response (404):**

```json
{
  "success": false,
  "data": null,
  "errorMessage": "Task with id 99999999-9999-9999-9999-999999999999 not found"
}
```

### 9.6 Response 429 Too Many Requests

**Request:**

```http
PATCH /api/v1/tasks/123e4567-e89b-12d3-a456-426614174000
Authorization: Bearer <your_token_here>
Content-Type: application/json

{
  "description": "Добавить валидацию входных данных и юнит-тесты"
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
PATCH /api/v1/tasks/123e4567-e89b-12d3-a456-426614174000
Authorization: Bearer <your_token_here>
Content-Type: application/json

{
  "description": "Добавить валидацию входных данных и юнит-тесты"
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
PATCH /api/v1/tasks/123e4567-e89b-12d3-a456-426614174000
Authorization: Bearer <your_token_here>
Content-Type: application/json

{
  "description": "Добавить валидацию входных данных и юнит-тесты"
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
|--------------|------|
| Swagger | |
| Postman | |
| Storage/table | `tasks` |
