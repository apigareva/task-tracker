# Удалить задачу

| | |
|------|------------------------|
| URL | /api/v1/tasks/{task_id} |
| Service Method | DELETE | 
| Description | Удалить задачу по уникальному идентификатору |
| Return Type | No Content |
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

### 2.2 Query params

Отсутствуют.

### 2.3 Path params

| Name | Type | Max length | Required | Description |
|------|------|------------|----------|-------------|
| task_id | STRING | 36 | Yes | Уникальный идентификатор задачи |

### 2.4 Request body

Отсутствует (метод DELETE).

## 3. Response parameters

Для успешного ответа `204 No Content` тело ответа отсутствует.

Для ошибок возвращается JSON:

| Name | Type | Max length | Description |
|------|------|------------|-------------|
| success | BOOLEAN | — | `false` — ошибка |
| errorMessage | STRING | — | Сообщение об ошибке |
| data | OBJECT | — | `null` |

## 4. Security requirements of method

- **Authentication**: обязательна — JWT Bearer-токен в заголовке `Authorization`.
- **Authorization**: пользователь может удалить задачу, к которой у него есть доступ.
- **Rate limiting**: ограничение на количество запросов — не более 100 запросов в минуту с одного токена.
- **Data Encryption**: передача данных только по HTTPS (TLS 1.2+).

## 5. How the service works

1. Клиент отправляет `DELETE /api/v1/tasks/{task_id}` с Bearer-токеном в заголовке.
2. Сервер валидирует токен и определяет идентификатор пользователя.
3. Сервер проверяет, имеет ли пользователь доступ на удаление задачи.
4. Сервер удаляет задачу.
5. Возвращается ответ `204 No Content` без тела.

### 5.1 Sequence diagram

### 5.2 Mapping

| Path param | DB column (таблица `tasks`) |
|-------------|------------------------------|
| task_id | `tasks.id` |

## 6. Logging requirements

- Логировать: `user_id`, `task_id`, временную метку запроса, HTTP-статус ответа.
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
| 204 | Задача успешно удалена | true | — | — |
| 400 | Некорректные параметры запроса | false | `bad_request` | Описание ошибки валидации |
| 401 | Не авторизован | false | `unauthorized` | "Unauthorized" |
| 403 | Доступ запрещён | false | `forbidden` | "Access denied" |
| 404 | Задача не найдена | false | `not_found` | "Task with id {task_id} not found" |
| 429 | Превышен лимит запросов | false | `rate_limit_exceeded` | "Too many requests" |
| 500 | Внутренняя ошибка сервера | false | `internal_server_error` | "An internal server error occurred" |
| 503 | Сервер временно недоступен | false | `service_unavailable` | "Service is temporarily unavailable. Please try again later" |

## 9. Example

### 9.1 Response 204 No Content

**Request:**

```http
DELETE /api/v1/tasks/123e4567-e89b-12d3-a456-426614174000
Authorization: Bearer <your_token_here>
```

**Response (204):**

```http
HTTP/1.1 204 No Content
```

### 9.2 Response 400 Bad Request

**Request:**

```http
DELETE /api/v1/tasks/invalid-task-id
Authorization: Bearer <your_token_here>
```

**Response (400):**

```json
{
  "success": false,
  "data": null,
  "errorMessage": "Invalid task_id format"
}
```

### 9.3 Response 401 Unauthorized

**Request:**

```http
DELETE /api/v1/tasks/123e4567-e89b-12d3-a456-426614174000
Authorization: Bearer <invalid_token>
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
DELETE /api/v1/tasks/123e4567-e89b-12d3-a456-426614174000
Authorization: Bearer <your_token_here>
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
DELETE /api/v1/tasks/99999999-9999-9999-9999-999999999999
Authorization: Bearer <your_token_here>
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
DELETE /api/v1/tasks/123e4567-e89b-12d3-a456-426614174000
Authorization: Bearer <your_token_here>
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
DELETE /api/v1/tasks/123e4567-e89b-12d3-a456-426614174000
Authorization: Bearer <your_token_here>
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
DELETE /api/v1/tasks/123e4567-e89b-12d3-a456-426614174000
Authorization: Bearer <your_token_here>
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

