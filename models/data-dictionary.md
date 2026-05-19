# Архитектура базы данных — Task Tracker Application

## Рекомендуемый стек хранения

- **Основная БД**: PostgreSQL.

### Общие соглашения

| Назначение | Тип PostgreSQL |
|------------|----------------|
| Первичный ключ, внешние ключи | `UUID` |
| Произвольный текст | `TEXT` |
| Текст ограниченной длины | `VARCHAR(N)` |
| Дата/время | `TIMESTAMPTZ` |
| Дата без времени | `DATE` |
| Флаг | `BOOLEAN` |
| IP-адрес | `INET` |

## Сущности и таблицы

## 1) Пользователи `users`

| Поле | Тип | Обязательность | Первичный/Внешний ключ | Описание |
|------|-----|----------------|------------------------|----------|
| `id` | `UUID` | not null | PK, default `gen_random_uuid()` | Уникальный идентификатор |
| `email` | `TEXT` | `UNIQUE`, not null | | Почта пользователя |
| `phone` | `VARCHAR(20)` | nullable | |Телефон пользователя |
| `password_hash` | `TEXT` | not null | | Хэш пароля пользователя|
| `first_name` | `VARCHAR(100)` | nullable | | Имя пользователя |
| `last_name` | `VARCHAR(100)` | nullable | | Фамилия пользователя |
| `status` (`ACTIVE`, `BLOCKED`) | `VARCHAR(100)` | not null, default `ACTIVE` | `user_status` ENUM | Статус пользователя |
| `created_at` | `TIMESTAMPTZ` | default `now()` | | Дата создания |
| `updated_at` | `TIMESTAMPTZ` | default `now()` | | Дата редактирования | 
| `last_login_at` | `TIMESTAMPTZ` | nullable | | Дата последнего входа в систему|
| `password_changed_at` | `TIMESTAMPTZ` | nullable | | Дата последней смены пароля; используется для инвалидирования старых токенов |
| `failed_login_attempts` | `INTEGER` | not null, default `0` | | Количество последовательных неуспешных попыток входа |
| `locked_until` | `TIMESTAMPTZ` | nullable | | Время, до которого вход пользователя временно заблокирован |
| `email_verified_at` | `TIMESTAMPTZ` | nullable | | Дата подтверждения email |

## 2) Сессии пользователя `auth_sessions`

| Поле | Тип | Обязательность | Первичный/Внешний ключ | Описание |
|------|-----|----------------|------------------------|----------|
| `id` | `UUID` | not null | PK, default `gen_random_uuid()` | Уникальный идентификатор |
| `user_id` | `UUID` | not null | FK `users.id` | Уникальный идентификатор пользователя |
| `refresh_token_hash` | `TEXT` | `UNIQUE`, not null | | Хэш refresh-токена; исходный refresh-токен в БД не хранится |
| `jwt_id` | `TEXT` | `UNIQUE`, not null | | Идентификатор JWT (`jti`) для связи access/refresh токенов с сессией |
| `user_agent` | `TEXT` | nullable | | User-Agent клиента, создавшего сессию |
| `ip_address` | `INET` | nullable | | IP-адрес клиента, создавшего сессию |
| `created_at` | `TIMESTAMPTZ` | default `now()` | | Дата создания сессии |
| `last_used_at` | `TIMESTAMPTZ` | nullable | | Дата последнего использования refresh-токена |
| `expires_at` | `TIMESTAMPTZ` | not null | | Дата истечения срока действия refresh-токена |
| `revoked_at` | `TIMESTAMPTZ` | nullable | | Дата отзыва сессии |
| `revoked_reason` | `TEXT` | nullable | | Причина отзыва сессии: logout, rotation, password_changed, security |
| `replaced_by_session_id` | `UUID` | nullable | FK `auth_sessions.id` | Новая сессия, созданная при ротации refresh-токена |

## 3) Команды `teams`

| Поле | Тип | Обязательность | Первичный/Внешний ключ | Описание |
|------|-----|----------------|------------------------|----------|
| `id` | `UUID` | not null | PK, default `gen_random_uuid()` | Уникальный идентификатор |
| `title` | `VARCHAR(100)` | not null | | Название команды |
| `description` | `TEXT` | nullable | | Описание |
| `created_by` | `UUID` | not null | FK `users.id`| Кем создана |
| `created_at` | `TIMESTAMPTZ` |  default `now()` | | Дата создания |

## 4) Участники команды `members`

| Поле | Тип | Обязательность | Первичный/Внешний ключ | Описание |
|------|-----|----------------|------------------------|----------|
| `id` | `UUID` | not null | PK, default `gen_random_uuid()` | Уникальный идентификатор |
| `member_id` | `UUID` | not null | FK `users.id`| Пользователь |
| `team_id` | `UUID` | not null | FK `teams.id`| Команда, к которой относится |
| `role_id` | `UUID` | not null | FK `roles.id`| Предоставленная роль в рамках команды |

## 5) Роли в команде `roles`

| Поле | Тип | Обязательность | Первичный/Внешний ключ | Описание |
|------|-----|----------------|------------------------|----------|
| `id` | `UUID` | not null | PK, default `gen_random_uuid()` | Уникальный идентификатор |
| `title` | `VARCHAR(50)` | not null | | Название |
| `description` | `TEXT` | nullable | | Описание |

## 6) Проекты `projects`

| Поле | Тип | Обязательность | Первичный/Внешний ключ | Описание |
|------|-----|----------------|------------------------|----------|
| `id` | `UUID` | not null | PK, default `gen_random_uuid()` | Уникальный идентификатор |
| `title` | `VARCHAR(100)` | not null | | Название |
| `description` | `TEXT` | nullable | | Описание |
| `created_by` | `UUID` | not null | FK `users.id`| Кем создан |
| `created_at` | `TIMESTAMPTZ` |  default `now()` | | Дата создания |
| `team_id` | `UUID` | not null | FK `teams.id`| Команда, к которой относится |
| `owner_id` | `UUID` | not null | FK `members.id`| Ответственный за проект |

## 7) Задачи `tasks`

| Поле | Тип | Обязательность | Первичный/Внешний ключ | Описание |
|------|-----|----------------|------------------------|----------|
| `id` | `UUID` | not null | PK, default `gen_random_uuid()` | Уникальный идентификатор |
| `title` | `VARCHAR(100)` | not null | | Название |
| `description` | `TEXT` | nullable | | Описание |
| `priority` | `TEXT` | not null | `task_priority` ENUM | Приоритет |
| `state` | `TEXT` | not null | `task_state` ENUM | Статус |
| `due_date` | `DATE` | not null | | Дата завершения |
| `created_by` | `UUID` | not null | FK `users.id`| Кем создан |
| `created_at` | `TIMESTAMPTZ` |  default `now()` | | Дата создания |
| `assigned_to` | `UUID` | not null | FK `members.id`| Исполнитель |
| `project_id` | `UUID` | not null | FK `projects.id`| Проект, к которому относится |

## 8) Комментарии к задачам `comments`

| Поле | Тип | Обязательность | Первичный/Внешний ключ | Описание |
|------|-----|----------------|------------------------|----------|
| `id` | `UUID` | not null | PK, default `gen_random_uuid()` | Уникальный идентификатор |
| `text` | `TEXT` | not null | | Текст |
| `created_by` | `UUID` | not null | FK `users.id` | Кем создан |
| `created_at` | `TIMESTAMPTZ` |  default `now()` | | Дата создания |
| `task_id` | `UUID` | not null | FK `tasks.id` | Задача, к которой относится |

## 9) Файлы к задачам `files`

| Поле | Тип | Обязательность | Первичный/Внешний ключ | Описание |
|------|-----|----------------|------------------------|----------|
| `id` | `UUID` | not null | PK, default `gen_random_uuid()` | Уникальный идентификатор |
| `original_name` | `VARCHAR(255)` | not null | | Оригинальное название |
| `storage_path` | `VARCHAR(500)` | `UNIQUE`, not null | | Путь в файловое хранилище |
| `size_bytes ` | `BIGINT` | not null | | Размер файла |
| `mime_type` | `VARCHAR(100)` | not null | | Тип файла |
| `uploaded_by` | `UUID` | not null | FK `users.id` | Кем загружен |
| `uploaded_at` | `TIMESTAMPTZ` | default `now()`  | | Дата загрузки |
| `task_id` | `TEXT` | not null | FK `tasks.id` | Задача, к которой относится |

## 10) Уведомления

## Ключевые связи (ER-уровень)

- `users` 0:N `projects`
- `users` 0:N `tasks`
- `users` 0:N `members`
- `users` 0:N `auth_sessions`
- `auth_sessions` 0:1 `auth_sessions`
- `teams` 0:N `members`
- `teams` 0:N `projects`
- `projects` 0:N `tasks`
- `members` 0:N `tasks`
- `members` 0:N `projects`
- `members` N:1 `roles`
- `tasks` 0:N `comments`
- `tasks` 0:N `files`
