# Zentry

Zentry - это небольшая социальная платформа на Django. Проект сочетает REST API, серверный веб-интерфейс, уведомления и личные сообщения в реальном времени через WebSocket.

## Возможности

- регистрация и аутентификация по email
- профили пользователей с био и аватаром
- лента постов с текстом и загрузкой изображений
- лайки, комментарии и подписки на пользователей
- уведомления о подписках, лайках и комментариях
- личные диалоги и история сообщений
- чат в реальном времени на Django Channels
- JWT-аутентификация для API и WebSocket

## Стек технологий

- Python 3.12
- Django 6
- Django REST Framework
- Simple JWT
- Django Channels + Daphne
- PostgreSQL или SQLite
- Redis для production-ready channel layer
- WhiteNoise для статических файлов

## Структура проекта

- `accounts/` - кастомная модель пользователя, регистрация, API профиля
- `posts/` - посты и лента
- `social/` - комментарии, лайки, подписки
- `notifications/` - модели уведомлений, API и вспомогательные сервисы
- `chat/` - диалоги, сообщения, WebSocket consumer
- `frontend/` - HTML-шаблоны, формы, браузерные views, статика
- `zentry/` - настройки проекта, ASGI/WSGI, корневые URL

## Локальный запуск

### 1. Установка зависимостей

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 2. Настройка переменных окружения

Для локальной разработки проект может работать и со значениями по умолчанию, но лучше явно задать:

```env
DEBUG=True
SECRET_KEY=change-me
ALLOWED_HOSTS=127.0.0.1,localhost
CSRF_TRUSTED_ORIGINS=http://127.0.0.1:8000,http://localhost:8000
DATABASE_URL=
REDIS_URL=
```

Примечания:

- если `DATABASE_URL` пустой, Django использует локальную SQLite (`db.sqlite3`)
- если `REDIS_URL` пустой, Channels использует in-memory transport
- in-memory transport подходит для локальной разработки, но не для production с несколькими инстансами

### 3. Применение миграций

```bash
python manage.py migrate
```

### 4. Создание администратора

```bash
python manage.py createsuperuser
```

### 5. Запуск приложения

Для полноценной ASGI-работы, включая WebSocket-чат:

```bash
daphne -b 127.0.0.1 -p 8000 zentry.asgi:application
```

Сайт будет доступен по адресу `http://127.0.0.1:8000/`.

## Аутентификация

API использует JWT:

- `POST /api/login/` - получить `access` и `refresh`
- `POST /api/token/refresh/` - обновить access token

Для защищенных API-запросов передавайте:

```http
Authorization: Bearer <access_token>
```

Браузерный чат тоже использует JWT для подключения по WebSocket.

## Основные маршруты

### Веб-интерфейс

- `/` - главная лента
- `/register/` - регистрация
- `/login/` - вход
- `/users/<id>/` - профиль пользователя
- `/notifications/` - уведомления
- `/conversations/` - список диалогов
- `/conversations/<id>/` - страница чата

### REST API

- `POST /api/register/`
- `GET|PATCH /api/users/<id>/`
- `GET|POST /api/posts/`
- `GET /api/posts/<id>/`
- `GET /api/feed/`
- `POST /api/comments/`
- `DELETE /api/comments/<id>/`
- `GET /api/posts/<post_id>/comments/`
- `POST /api/likes/`
- `POST /api/like/`
- `POST /api/follows/`
- `POST /api/follow/`
- `GET /api/users/<user_id>/followers/`
- `GET /api/users/<user_id>/following/`
- `GET /api/notifications/`
- `POST /api/notifications/<id>/read/`
- `POST /api/notifications/read-all/`
- `GET|POST /api/conversations/`
- `POST /api/messages/`
- `GET /api/messages/<conversation_id>/`
- `POST /api/conversations/<conversation_id>/read/`

### WebSocket

- `/ws/chat/<conversation_id>/`

Поддерживаемые события в чате:

- `message` - отправка сообщения
- `typing` - индикатор набора текста
- `read` - отметка сообщений как прочитанных

## Модель данных

Основные сущности:

- `User`
- `Post`
- `Comment`
- `Like`
- `Follow`
- `Notification`
- `Conversation`
- `Message`

## Статические и медиафайлы

- статические файлы собираются в `staticfiles/`
- загружаемые файлы сохраняются в `media/`
- в production локальное хранение медиа является временным, если не подключить постоянное хранилище

## Деплой на Render

В репозитории уже есть отдельные заметки по деплою в [README_RENDER.md](/home/aman/zentry/README_RENDER.md).

Кратко:

- build command: `./build.sh`
- start command: `daphne -b 0.0.0.0 -p $PORT zentry.asgi:application`
- обязательные переменные: `DEBUG`, `SECRET_KEY`, `ALLOWED_HOSTS`, `CSRF_TRUSTED_ORIGINS`, `DATABASE_URL`
- необязательная переменная для стабильного realtime-чата: `REDIS_URL`

## Полезные команды

```bash
python manage.py makemigrations
python manage.py migrate
python manage.py collectstatic --no-input
python manage.py test
```
