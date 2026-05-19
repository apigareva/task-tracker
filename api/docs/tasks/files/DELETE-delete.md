# Удалить файл задачи

|  |  |
|------|------|
| URL | /api/v1/tasks/{task_id}/files/{file_id} |
| Service Method | DELETE |
| Description | Удаляет файл задачи |
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
| task_id | STRING | 36 | Yes | Уникальный идентификатор задачи |
| file_id | STRING | 36 | Yes | Уникальный идентификатор файла |

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
- **Authorization**: пользователь может удалить файл задачи, к которой у него есть доступ.
- **Rate limiting**: ограничение на количество запросов — не более 20 запросов в минуту с одного IP или токена.
- **Data Encryption**: передача данных только по HTTPS (TLS 1.2+).

## 5. How the service works

1. Клиент отправляет `DELETE /api/v1/tasks/{task_id}/files/{file_id}`.
2. Сервер валидирует входные параметры и права доступа.
3. Сервер выполняет операцию и формирует ответ.

### 5.1 Sequence diagram

Не требуется: метод не содержит сложных интеграционных сценариев.

### 5.2 Mapping

| Path param | DB column / value |
|------|------|
| task_id | `task_id` |
| file_id | `file_id` |

## 6. Logging requirements

- Логировать: `user_id`, временную метку запроса, параметры запроса, HTTP-статус ответа.
- Логировать ошибки аутентификации (401), авторизации (403) и внутренние ошибки сервера (500) с уровнем `ERROR`.

## 7. Data model used by method

### 7.1 Table `files`

| Field | Type | Constraint | Description |
|------|------|------|------|
| `id` | `UUID` | PK, not null | Уникальный идентификатор |
| `original_name` | `VARCHAR(255)` | not null | Оригинальное название |
| `storage_path` | `VARCHAR(500)` | UNIQUE, not null | Путь в файловое хранилище |
| `size_bytes` | `BIGINT` | not null | Размер файла |
| `mime_type` | `VARCHAR(100)` | not null | Тип файла |
| `uploaded_by` | `UUID` | FK users.id, not null | Кем загружен |
| `uploaded_at` | `TIMESTAMPTZ` | default now() | Дата загрузки |
| `task_id` | `UUID` | FK tasks.id, not null | Задача, к которой относится |

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
DELETE /api/v1/tasks/{task_id}/files/{file_id}
Authorization: Bearer <access_token>
```

**Response (204):**

```http
HTTP/1.1 204 No Content
```

### 9.2 Response 400 Bad Request

**Request:**

```http
DELETE /api/v1/tasks/invalid-task-id/files/invalid-file-id
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
DELETE /api/v1/tasks/{task_id}/files/{file_id}
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
DELETE /api/v1/tasks/{task_id}/files/{file_id}
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
DELETE /api/v1/tasks/123e4567-e89b-12d3-a456-426614174000/files/99999999-9999-9999-9999-999999999999
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
DELETE /api/v1/tasks/{task_id}/files/{file_id}
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
DELETE /api/v1/tasks/{task_id}/files/{file_id}
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
DELETE /api/v1/tasks/{task_id}/files/{file_id}
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
| Storage/table | `files` |
