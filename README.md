# Stepwise Backend - FastAPI Development

[![FastAPI](https://img.shields.io/badge/FastAPI-0.104+-009688)](https://fastapi.tiangolo.com)
[![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-2.0+-d71f00)](https://sqlalchemy.org)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15+-336791)](https://postgresql.org)
[![JWT](https://img.shields.io/badge/JWT-Auth-orange)](https://jwt.io)

Stepwise Backend — асинхронный FastAPI-сервис с CRUD, JWT-аутентификацией и модульной структурой.

---

## Быстрый старт

1. Клонирование репозитория и переход в папку:
    
    git clone https://github.com/yourusername/stepwise-backend.git
    cd stepwise-backend

2. Создание и активация виртуального окружения (пример для Linux/macOS):

    python -m venv venv
    source venv/bin/activate

3. Установка зависимостей:

    poetry install    # либо: pip install -r requirements.txt

4. Запуск development-сервера:

    uvicorn app.main:app --reload

API будет доступно по адресу: http://localhost:8000  
Документация:  
- Swagger UI: http://localhost:8000/docs  
- ReDoc: http://localhost:8000/redoc

---

## Архитектура проекта

    project-root/
    ├── app/                      # Основное приложение
    │   ├── __pycache__/          # Скомпилированные Python файлы
    │   ├── auth/                 # Аутентификация и авторизация (JWT, bcrypt)
    │   ├── core/                 # Конфигурация (database, config)
    │   ├── models/               # SQLAlchemy модели
    │   ├── routers/              # Роутеры FastAPI (endpoints)
    │   ├── tests/                # Тесты и скрипты подготовки данных
    │   ├── main.py               # Точка входа приложения
    │   └── default_data.py       # Скрипт для заполнения начальными данными
    ├── migrations/               # Alembic миграции
    ├── Dockerfile                # Docker-конфигурация
    ├── pyproject.toml / requirements.txt
    ├── alembic.ini
    └── README.md

Краткое назначение папок:
- `app/auth` — логика регистрации/логина, создание и верификация JWT, хэширование паролей.  
- `app/core` — подключение к БД, общие настройки и утилиты.  
- `app/models` — описания сущностей базы данных (SQLAlchemy).  
- `app/routers` — разбиение API по функциональным областям (users, physicalPlans, mentalPlans, resources, otherTasks).  
- `migrations` — Alembic для версионирования схемы БД.

---

## Модели и связи (актуально для этой версии)

Ниже — аккуратно оформленное описание моделей и их полей/связей в проекте. Это не код, а документация для README — можно прямо использовать как reference.

### User
- `id` — integer, PK  
- `login` — string, уникальный, обязательный  
- `password` — string, обязательный (хранить как хэш)  
- `created_at` — timestamp, по умолчанию `now`

Связи:
- `physical_plans` — one-to-many к `PhysicalPlan`  
- `mental_plans` — one-to-many к `MentalPlan`  
- `tasks` — one-to-many к `OuterTask`

---

### PhysicalPlan
- `id` — integer, PK  
- `user_id` — FK → `users.id`  
- `goal` — string, обязательное поле  
- `age` — integer  
- `gender` — enum (`male`, `female`)  
- `weight` — float  
- `height` — float  
- `bmi` — float, опционально  
- `progress` — float, по умолчанию 0.0  
- `day` — integer, текущий день/этап плана

Связи:
- `user` — many-to-one к `User`  
- `workouts` — one-to-many к `Workout`

---

### Workout
- `id` — integer, PK  
- `physical_plan_id` — FK → `physical_plans.id`  
- `day_of_week` — integer (номер дня)

Связи:
- `physical_plan` — many-to-one к `PhysicalPlan`  
- `exercises` — one-to-many к `WorkoutExercise`

---

### Exercise
- `id` — integer, PK  
- `name` — string  
- `calories_burned` — float  
- `description` — text, опционально  
- `start_reps` — integer  
- `end_reps` — integer  
- `step` — integer

---

### WorkoutExercise
- `id` — integer, PK  
- `workout_id` — FK → `workouts.id`  
- `exercise_id` — FK → `exercises.id`

Связи:
- `workout` — many-to-one к `Workout`  
- `exercise` — many-to-one к `Exercise`

---

### MentalPlan
- `id` — integer, PK  
- `user_id` — FK → `users.id`  
- `name` — string, обязательное  
- `goal` — string, обязательное  
- `progress` — integer, по умолчанию 30

Связи:
- `user` — many-to-one к `User`  
- `resources` — one-to-many к `MentalPlanResource`

---

### Resource
- `id` — integer, PK  
- `type` — enum (`book`, `course`, `video`)  
- `name` — string, обязательное  
- `description` — string, обязательное  
- `volume` — float (объём, длительность или количество страниц и т.д.)

---

### MentalPlanResource
- `id` — integer, PK  
- `mental_plan_id` — FK → `mental_plans.id`  
- `resource_id` — FK → `resources.id`  
- `progress` — float, по умолчанию 0.0

Связи:
- `mental_plan` — many-to-one к `MentalPlan`  
- `resource` — many-to-one к `Resource`

---

### OuterTask
- `id` — integer, PK, autoincrement  
- `user_id` — FK → `users.id`  
- `title` — string, обязательное  
- `status` — boolean, по умолчанию `false`

Связи:
- `user` — many-to-one к `User`

---

## Схема связей (кратко)
- `User` 1:N `PhysicalPlan`  
- `User` 1:N `MentalPlan`  
- `User` 1:N `OuterTask`  
- `PhysicalPlan` 1:N `Workout`  
- `Workout` 1:N `WorkoutExercise`  
- `Exercise` 1:N `WorkoutExercise`  
- `MentalPlan` 1:N `MentalPlanResource`  
- `Resource` 1:N `MentalPlanResource`

---

## API (основные эндпоинты)

Аутентификация (`/users/`):
- `POST /users/` — регистрация  
- `POST /users/signIn` — логин, возврат JWT

Фитнес-планы (`/physicalPlans/`):
- `POST /physicalPlans/` — создать план  
- `PATCH /physicalPlans/{id}` — обновить план  
- `GET /users/{id}/physical-plan` — получить план пользователя

Ментальные планы (`/mentalPlans/`):
- `POST /mentalPlans/` — создать план  
- `PATCH /mentalPlans/{id}` — обновить план  
- `GET /users/{id}/mental-plan` — получить план пользователя

Ресурсы (`/resources/`):
- `POST /resources/` — создать ресурс  
- `POST /resources/bulk/` — массовое создание  
- `PATCH /resources/{id}` — обновление ресурса / прогресса  
- `DELETE /resources/{id}` — удалить ресурс

Задачи (`/outherTasks/`):
- `GET /outherTasks/`  
- `POST /outherTasks/`  
- `PATCH /outherTasks/{id}`  
- `DELETE /outherTasks/{id}`

---

## Аутентификация и безопасность

- JWT для авторизации; токен создаётся на логине и используется в заголовке `Authorization: Bearer <token>`.  
- Пароли хранятся в виде хэшей (bcrypt).  
- Endpoints защищаются через зависимость (Depends) и валидацию токена.  
- CORS: в development допускаются все источники, в production указывать конкретные домены.

---

## Асинхронность и оптимизация

Рекомендации по БД и ORM:
- Использовать асинхронную работу SQLAlchemy 2.0 + asyncpg для PostgreSQL.  
- Применять connection pooling и eager-loading (joinedload) для избежания N+1.  
- Кэшировать тяжёлые/часто используемые запросы (Redis) при необходимости.  
- Писать оптимизированные запросы, минимизируя количество round-trip к БД.

---

## Конфигурация (пример `.env`)

    DATABASE_URL=postgresql+asyncpg://user:pass@localhost:5432/stepwise
    JWT_SECRET_KEY=your-super-secret-key
    JWT_ALGORITHM=HS256
    ACCESS_TOKEN_EXPIRE_MINUTES=30

---

## Docker и деплой

Пример `Dockerfile` (минимальный образ):

    FROM python:3.11-slim

    WORKDIR /app
    COPY pyproject.toml poetry.lock ./
    RUN pip install poetry && poetry install --no-root

    COPY . .
    CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]

Миграции Alembic:
- Создание: `alembic revision --autogenerate -m "init"`  
- Применение: `alembic upgrade head`

---

## Мониторинг и производительность

- Асинхронные обработчики (async/await).  
- Connection pooling для PostgreSQL.  
- Eager loading и оптимизированные JOIN'ы.  
- Pydantic-схемы для быстрой и корректной валидации.  
- План интеграции Redis, Prometheus и Grafana.

---

## Roadmap / планы развития

- Redis для кэширования и ускорения ответов.  
- WebSocket / SSE для real-time уведомлений.  
- Background tasks (Celery/BackgroundTasks) для тяжёлых операций.  
- Rate limiting (защита от злоупотреблений).  
- Миграция на Type Hints / stricter typing и добавление тестов покрытия.
