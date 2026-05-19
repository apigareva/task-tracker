# Загрузить файл задачи

|  |  |
|------|------|
| URL | /api/v1/tasks/{task_id}/files |
| Service Method | POST |
| Description | Загружает файл и прикрепляет его к задаче |
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
| Content-Type | STRING | Yes | `multipart/form-data` |

### 2.2 Query params

Отсутствуют.

### 2.3 Path params

| Name | Type | Max length | Required | Description |
|------|------|------|------|------|
| task_id | STRING | 36 | Yes | Уникальный идентификатор задачи |

### 2.4 Request body

| Name | Type | Max length | Required | Description |
|------|------|------|------|------|
| file | FILE | — | Yes | Файл для загрузки |

## 3. Response parameters

| Name | Type | Max length | Description |
|------|------|------|------|
| success | BOOLEAN | — | `true` — запрос выполнен успешно, `false` — ошибка |
| errorMessage | STRING | — | Сообщение об ошибке (при `success: false`) |
| data | OBJECT | — | Объект с данными |
| data.id | STRING | 36 | Уникальный идентификатор |
| data.original_name | STRING | 255 | Оригинальное название |
| data.size_bytes | BIGINT | — | Размер файла |
| data.mime_type | STRING | 100 | Тип файла |
| data.uploaded_by | STRING | 36 | Кем загружен |
| data.uploaded_at | DATE/TIME | — | Дата загрузки |
| data.task_id | STRING | 36 | Задача, к которой относится |

## 4. Security requirements of method

- **Authentication**: обязательна — JWT Bearer access-токен в заголовке `Authorization`.
- **Authorization**: пользователь может загрузить файл к задаче, к которой у него есть доступ.
- **Rate limiting**: ограничение на количество запросов — не более 60 запросов в минуту с одного IP или токена.
- **Data Encryption**: передача данных только по HTTPS (TLS 1.2+).

## 5. How the service works

1. Клиент отправляет `POST /api/v1/tasks/{task_id}/files`.
2. Сервер валидирует входные параметры и права доступа.
3. Сервер выполняет операцию и формирует ответ.

### 5.1 Sequence diagram

Не требуется: метод не содержит сложных интеграционных сценариев.

### 5.2 Mapping

| Path param | DB column / value |
|------|------|
| task_id | `task_id` |

| Request body | DB column / value |
|------|------|
| file | `files.file` |

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
| 201 | Ресурс успешно создан | true | — | — |
| 400 | Некорректные параметры запроса | false | `bad_request` | "Описание ошибки валидации" |
| 401 | Не авторизован | false | `unauthorized` | "Unauthorized" |
| 403 | Доступ запрещён | false | `forbidden` | "Access denied" |
| 404 | Ресурс не найден | false | `not_found` | "Task not found" |
| 429 | Превышен лимит запросов | false | `rate_limit_exceeded` | "Too many requests" |
| 500 | Внутренняя ошибка сервера | false | `internal_server_error` | "An internal server error occurred" |
| 503 | Сервер временно недоступен | false | `service_unavailable` | "Service is temporarily unavailable. Please try again later" |

## 9. Example

### 9.1 Response 201 Created

**Request:**

```http
POST /api/v1/tasks/{task_id}/files
Authorization: Bearer <access_token>
Content-Type: multipart/form-data
```

**Response (201):**

```json
{
  "success": true,
  "data": {
    "id": "44444444-4444-4444-4444-444444444444",
    "original_name": "requirements.pdf",
    "storage_path": "/tasks/123e4567/requirements.pdf",
    "size_bytes": 204800,
    "mime_type": "application/pdf",
    "uploaded_by": "550e8400-e29b-41d4-a716-446655440000",
    "uploaded_at": "2026-05-15T09:30:00Z",
    "task_id": "123e4567-e89b-12d3-a456-426614174000"
  },
  "errorMessage": null
}
```

### 9.2 Response 400 Bad Request

**Request:**

```http
POST /api/v1/tasks/{task_id}/files
Authorization: Bearer <access_token>
Content-Type: multipart/form-data
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
POST /api/v1/tasks/{task_id}/files
Authorization: Bearer <invalid_access_token>
Content-Type: multipart/form-data
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
POST /api/v1/tasks/{task_id}/files
Authorization: Bearer <access_token>
Content-Type: multipart/form-data
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
POST /api/v1/tasks/{task_id}/files
Authorization: Bearer <access_token>
Content-Type: multipart/form-data
```

**Response (404):**

```json
{
  "success": false,
  "data": null,
  "errorMessage": "Task not found"
}
```

### 9.6 Response 429 Too Many Requests

**Request:**

```http
POST /api/v1/tasks/{task_id}/files
Authorization: Bearer <access_token>
Content-Type: multipart/form-data
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
POST /api/v1/tasks/{task_id}/files
Authorization: Bearer <access_token>
Content-Type: multipart/form-data
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
POST /api/v1/tasks/{task_id}/files
Authorization: Bearer <access_token>
Content-Type: multipart/form-data
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
