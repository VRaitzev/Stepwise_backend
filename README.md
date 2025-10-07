# Stepwise Backend - FastAPI Development

[![FastAPI](https://img.shields.io/badge/FastAPI-0.104+-009688)](https://fastapi.tiangolo.com)
[![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-2.0+-d71f00)](https://sqlalchemy.org)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15+-336791)](https://postgresql.org)
[![JWT](https://img.shields.io/badge/JWT-Auth-orange)](https://jwt.io)

Высокопроизводительный FastAPI бэкенд для платформы саморазвития Stepwise — асинхронная архитектура с полной CRUD функциональностью, безопасной JWT-аутентификацией и чистым модульным кодом.

---

## Быстрый старт

### Установка и запуск

    # Клонирование репозитория
    git clone https://github.com/yourusername/stepwise-backend.git
    cd stepwise-backend

    # Создание виртуального окружения
    python -m venv venv
    source venv/bin/activate  # Linux/Mac
    # venv\Scripts\activate   # Windows

    # Установка зависимостей
    poetry install  # или pip install -r requirements.txt

    # Запуск development сервера
    uvicorn app.main:app --reload

API доступно по адресу:
http://localhost:8000

Документация:
- Swagger UI → http://localhost:8000/docs
- ReDoc → http://localhost:8000/redoc

---

## Архитектура проекта

    project-root/
    ├── app/                      # Основное приложение
    │   ├── __pycache__/          # Скомпилированные Python файлы
    │   ├── auth/                 # Аутентификация и авторизация (JWT, bcrypt)
    │   ├── core/                 # Базовые настройки (config, database, utils)
    │   ├── models/               # SQLAlchemy модели базы данных
    │   ├── routers/              # API endpoints (роутеры FastAPI)
    │   ├── tests/                # Тесты и вспомогательные скрипты
    │   │   ├── __init__.py
    │   │   ├── create_tables_async.py
    │   │   └── default_data.py
    │   ├── main.py               # Точка входа в приложение
    │   └── default_data.py       # Скрипт начальных данных
    ├── migrations/               # Alembic миграции базы данных
    ├── .gitignore                # Исключения для Git
    ├── Dockerfile                # Конфигурация контейнера
    ├── README.md                 # Документация проекта
    ├── alembic.ini               # Настройки Alembic
    ├── poetry.lock               # Lock-файл зависимостей Poetry
    ├── pyproject.toml            # Конфигурация зависимостей Poetry
    └── wait-for-it.sh            # Скрипт ожидания готовности БД (Docker Compose)

Описание ключевых папок:
- app/auth: логика аутентификации, генерация и верификация JWT, хэширование паролей.
- app/core: конфигурация приложения, подключение к базе, общие утилиты.
- app/models: определения моделей SQLAlchemy (Pydantic схемы рядом при необходимости).
- app/routers: разделение по функциональным модулям API (users, physicalPlans, mentalPlans, resources, otherTasks).
- app/tests: тесты и вспомогательные скрипты для локальной и CI проверки.
- migrations: скрипты Alembic для управления схемой БД.

---

## Технологический стек

Core:
- FastAPI — асинхронный фреймворк
- Pydantic — валидация и схемы
- Uvicorn — ASGI сервер

Database & ORM:
- SQLAlchemy 2.0 (async)
- PostgreSQL
- Alembic для миграций

Authentication & Security:
- JWT для аутентификации
- BCrypt для хэширования паролей
- CORS middleware

Development:
- Python 3.11+
- Type hints
- Docker
- Poetry или pip

---

## API Endpoints

Аутентификация (префикс /users/):
    POST /users/              # Регистрация нового пользователя
    POST /users/signIn        # Вход в систему (возврат JWT)

Фитнес планы (префикс /physicalPlans/):
    POST /physicalPlans/      # Создание фитнес-плана
    PATCH /physicalPlans/{id} # Обновление плана
    GET /users/{id}/physical-plan  # Получение плана пользователя

Ментальные планы (префикс /mentalPlans/):
    POST /mentalPlans/        # Создание образовательного плана
    PATCH /mentalPlans/{id}   # Обновление плана
    GET /users/{id}/mental-plan    # Получение плана пользователя

Ресурсы (префикс /resources/):
    POST /resources/          # Создание ресурса (книга/видео/курс)
    POST /resources/bulk/     # Массовое создание ресурсов
    PATCH /resources/{id}     # Обновление ресурса и прогресса
    DELETE /resources/{id}    # Удаление ресурса

Задачи (префикс /outherTasks/):
    GET /outherTasks/         # Получение всех задач
    POST /outherTasks/        # Создание задачи
    PATCH /outherTasks/{id}   # Обновление задачи
    DELETE /outherTasks/{id}  # Удаление задачи

---

## Модели базы данных (кратко)

    # User — Пользователи
    class User(Base):
        id: int
        login: str
        password: str

    # PhysicalPlan — Фитнес планы
    class PhysicalPlan(Base):
        user_id: int
        goal: str
        age: int
        gender: str
        weight: float
        height: float
        bmi: float
        progress: int

    # MentalPlan — Образовательные планы
    class MentalPlan(Base):
        user_id: int
        name: str
        goal: str

    # Resource — Ресурсы (книги/видео/курсы)
    class Resource(Base):
        type: str
        name: str
        description: str
        volume: int

    # OuterTask — Дополнительные задачи
    class OuterTask(Base):
        user_id: int
        title: str
        status: bool

Связи:
- User 1:1 PhysicalPlan
- User 1:1 MentalPlan
- MentalPlan 1:M MentalPlanResource
- PhysicalPlan 1:M Workout
- Workout M:M Exercise

---

## Аутентификация и безопасность

JWT:
    # Создание токена
    access_token = create_access_token(username)

    # Защита эндпоинтов через Depends
    @router.get("/protected")
    async def protected_route(user: str = Depends(authenticate)):
        return {"user": user}

Хэширование паролей:
    # Создание хэша
    hashed_password = hash_password.create_hash(plain_password)

    # Проверка пароля
    is_valid = hash_password.verify_hash(plain_password, hashed_password)

CORS:
    app.add_middleware(
        CORSMiddleware,
        allow_origins=["*"],  # В production указать конкретные домены
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )

---

## Асинхронные операции и оптимизация запросов

    # Асинхронная сессия SQLAlchemy
    async with db.begin():
        db.add(new_object)
        await db.commit()

    # Оптимизированный запрос с joinedload
    query = await db.execute(
        select(User)
        .options(joinedload(User.physical_plan))
        .where(User.id == user_id)
    )

Рекомендации:
- Использовать connection pooling
- Применять eager loading для связанных сущностей
- Кэшировать частые запросы (Redis) при необходимости

---

## Конфигурация и переменные окружения

    DATABASE_URL=postgresql+asyncpg://user:pass@localhost:5432/stepwise
    JWT_SECRET_KEY=your-super-secret-key
    JWT_ALGORITHM=HS256
    ACCESS_TOKEN_EXPIRE_MINUTES=30

---

## Docker и деплой

Dockerfile (пример):

    FROM python:3.11-slim

    WORKDIR /app
    COPY pyproject.toml poetry.lock ./
    RUN pip install poetry && poetry install --no-root

    COPY . .
    CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]

Миграции:
    # Создание миграций
    alembic revision --autogenerate -m "init"

    # Применение миграций
    alembic upgrade head

---

## Производительность и мониторинг

- Асинхронные операции (async/await)
- Connection pooling
- Eager loading для избежания N+1
- Pydantic валидация на уровне схем
- Возможная интеграция Redis, Prometheus, Grafana

---

## Планы развития

- Redis для кэширования
- WebSocket для real-time уведомлений
- Background tasks для тяжелых операций
- Rate limiting и защита от DDoS
- Мониторинг и метрики

---

Stepwise Backend — мощный и масштабируемый API.
