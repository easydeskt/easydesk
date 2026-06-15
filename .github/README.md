# EasyDesk — развёртывание

Docker Compose конфигурация для запуска EasyDesk на production-сервере. Образы сервисов загружаются из GitHub Container Registry.

## Сервисы

| Сервис | Образ | Назначение |
|---|---|---|
| `postgres` | `postgres:17-alpine` | База данных |
| `backend` | `ghcr.io/easydeskt/backend` | Ktor-приложение: REST API, каналы, боты |
| `mini-app` | `ghcr.io/easydeskt/mini-app` | Telegram Mini App (React SPA) |
| `caddy` | `caddy:2-alpine` | Обратный прокси, TLS-терминация |

Все сервисы изолированы в общей Docker-сети. Внешний трафик принимают только порты 80 и 443.

## Развёртывание

### 1. Клонирование репозитория

```bash
git clone https://github.com/easydeskt/easydesk.git
cd easydesk
```

### 2. Настройка окружения

```bash
cp .env.example .env
```

Заполните переменные в `.env` согласно разделу [Переменные окружения](#переменные-окружения).

### 3. Запуск

```bash
docker compose up -d
```

Перед запуском домен должен быть направлен на IP-адрес сервера. TLS-сертификат Caddy получит от Let's Encrypt при первом старте.

### Обновление образов

```bash
docker compose pull && docker compose up -d
```

## Переменные окружения

Скопируйте `.env.example` в `.env` и укажите актуальные значения.

### Инфраструктура

| Переменная | Описание | Пример |
|---|---|---|
| `DOMAIN` | Домен для выпуска TLS-сертификата | `easydesk.example.com` |
| `BACKEND_VERSION` | Тег образа backend | `latest`, `1.2.0` |
| `MINI_APP_VERSION` | Тег образа mini-app | `latest`, `1.2.0` |

### Приложение

| Переменная | Описание |
|---|---|
| `WORKSPACE_NAME` | Отображаемое название рабочего пространства |

### База данных

| Переменная | Описание | По умолчанию |
|---|---|---|
| `DATABASE_USERNAME` | Пользователь PostgreSQL | — |
| `DATABASE_PASSWORD` | Пароль PostgreSQL | — |
| `DATABASE_MAX_POOL_SIZE` | Максимальный размер пула соединений | `10` |
| `DATABASE_MIN_IDLE` | Минимальное число простаивающих соединений | `2` |

URL подключения к базе данных (`jdbc:postgresql://postgres:5432/easydesk`) задан в `docker-compose.yml` и не требует настройки.

### Канал: Email

Канал необязателен. Если Email не используется, переменные можно не задавать.

| Переменная | Описание |
|---|---|
| `CHANNEL_EMAIL_IMAP_HOST` | Адрес IMAP-сервера |
| `CHANNEL_EMAIL_IMAP_USERNAME` | Имя пользователя IMAP |
| `CHANNEL_EMAIL_IMAP_PASSWORD` | Пароль IMAP |
| `CHANNEL_EMAIL_SMTP_HOST` | Адрес SMTP-сервера |
| `CHANNEL_EMAIL_SMTP_USERNAME` | Имя пользователя SMTP |
| `CHANNEL_EMAIL_SMTP_PASSWORD` | Пароль SMTP |

### Канал: Telegram

| Переменная | Описание |
|---|---|
| `CHANNEL_TELEGRAM_BOT_TOKEN` | Токен бота для получения сообщений от клиентов |
| `CHANNEL_TELEGRAM_WEBHOOK_SECRET` | Секрет для проверки подлинности webhook-запросов |

### Канал: ВКонтакте

Канал необязателен. Если ВКонтакте не используется, переменные можно не задавать.

| Переменная | Описание |
|---|---|
| `CHANNEL_VK_COMMUNITY_TOKEN` | Токен сообщества VK |
| `CHANNEL_VK_CALLBACK_SECRET` | Секрет для проверки подлинности Callback API |
| `CHANNEL_VK_CONFIRMATION_CODE` | Код подтверждения Callback API |

### Супервизор: Telegram

Бот создаёт топик в Telegram-супергруппе для каждого тикета и ведёт переписку с клиентами от имени поддержки.

| Переменная | Описание |
|---|---|
| `TELEGRAM_SUPERVISOR_BOT_TOKEN` | Токен бота-оператора |
| `TELEGRAM_SUPERVISOR_GROUP_ID` | ID супергруппы с форум-топиками |
| `TELEGRAM_SUPERVISOR_SUPERADMIN_ID` | Telegram ID первого администратора |

## Маршрутизация

```
HTTPS → Caddy
         ├── /api/*   → backend:8080
         └── /*       → mini-app:80  (SPA, fallback на index.html)
```

Схема базы данных мигрируется автоматически при каждом запуске backend через Flyway.
