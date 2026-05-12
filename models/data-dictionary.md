# Архитектура базы данных — Task Tracker Application

## Рекомендуемый стек хранения

- **Основная БД**: PostgreSQL.
- **Файлы**: S3-совместимое хранилище (MinIO/AWS S3), в БД хранится метадата.

## Ключевые домены данных

## Сущности и таблицы

## 1) Пользователи

- `users`
  - `id` (PK)
  - `email`
  - `phone`
  - `password_hash`
  - `first_name`
  - `last_name`
  - `status` (`ACTIVE`, `BLOCKED`)
  - `created_at`
  - `updated_at`
  - `last_login_at`

## 2) Команды

- `teams`
  - `id` (PK)
  - `title`
  - `description` 
  - `creator_id` (FK)
  - `created_at`

## 3) Участники команды

- `members`
  - `id` (PK)
  - `user_id` (FK)
  - `team_id` (FK)
  - `role_id` (FK)

## 4) Роли в команде

- `roles`
  - `id` (PK)
  - `title`
  - `description`

## 5) Проекты

- `projects`
  - `id` (PK)
  - `title`
  - `description`  
  - `created_at`
  - `creator_id` (FK)
  - `team_id` (FK) 
  - `owner_id` (FK)

## 6) Задачи

- `tasks`
  - `id` (PK)
  - `title`
  - `description` 
  - `priority`
  - `state`
  - `due_date`
  - `created_at`
  - `creator_id` (FK)
  - `assigned_id` (FK)
  - `project_id` (FK)

## 7) Комментарии к задачам

- `comments`
  - `id` (PK)
  - `text`
  - `created_at`
  - `creator_id` (FK)
  - `task_id` (FK)

## 8) Файлы к задачам

- `files`
  - `id` (PK)
  - `data`
  - `created_at`
  - `creator_id` (FK)
  - `task_id` (FK)  
