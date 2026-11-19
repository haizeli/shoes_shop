# Установка и запуск ShoeStore

Приложение «Магазин обуви» (ShoeStore.exe) — десктопная программа для Windows. **База данных PostgreSQL в EXE не упакована**: она запускается отдельно в Docker.

## Требования

- Windows 10/11;
- Docker Desktop (Linux containers) и Docker Compose;
- свободный порт `5433` на компьютере.

## Состав комплекта

- `application/ShoeStore/` — приложение (`ShoeStore.exe` и папка `_internal`; EXE нельзя копировать отдельно);
- `deployment/` — `compose.yaml`, `.env.example`, `docker/db-check`, `database/shoes_shop.sql`;
- `config.ini.example` — образец настроек подключения;
- `README.md`, `INSTALL.md`.

## 1. Запуск базы данных

Откройте PowerShell в папке `deployment`:

```powershell
Copy-Item .env.example .env
```

Откройте `.env` и замените `POSTGRES_PASSWORD` на свой пароль (латинские буквы и цифры). Затем:

```powershell
docker compose up -d
docker compose ps
```

Сервис `db` должен получить статус `healthy`. Сервис `db-check` — одноразовая проверка, он завершается с кодом 0.

## 2. Импорт данных (только если база пустая)

```powershell
docker compose cp database/shoes_shop.sql db:/tmp/shoes_shop.sql
docker compose exec db sed -i '/transaction_timeout/d' /tmp/shoes_shop.sql
docker compose exec db psql -v ON_ERROR_STOP=1 -U postgres -d shoes_shop -f /tmp/shoes_shop.sql
docker compose run --rm --no-deps db-check
```

Ожидается строка `OK: база=shoes_shop; таблиц=7`. Строка с `sed` нужна, потому что скрипт выгружен из более новой версии PostgreSQL, чем в контейнере (14). Если база уже заполнена, **повторно импортировать нельзя**: будет ошибка `relation already exists`.

Если установлена только программа `docker-compose`, используйте её вместо `docker compose`. В Git Bash перед командами с путём `/tmp/...` добавляйте `MSYS_NO_PATHCONV=1`.

## 3. Настройка приложения

Скопируйте `config.ini.example` в папку `application\ShoeStore\` под именем `config.ini` (рядом с `ShoeStore.exe`) и укажите тот же пароль, что в `.env`:

```ini
[database]
host = localhost
port = 5433
name = shoes_shop
user = postgres
password = ВАШ_ПАРОЛЬ
```

Файл `config.ini` с настоящим паролем никому не передавайте.

## 4. Запуск

Запустите `application\ShoeStore\ShoeStore.exe`. Откроется окно входа. Вход выполняется под учебной учётной записью из таблицы `users`.

## 5. Остановка

```powershell
docker compose stop
```

Команду `docker compose down -v` не используйте: она удаляет том `db_data` вместе с базой данных.

## Типичные проблемы

| Симптом | Причина | Что делать |
|---|---|---|
| EXE сразу закрывается, в консоли `KeyError: 'database'` | нет `config.ini` рядом с EXE | создать файл по пункту 3 |
| «Не удалось подключиться к базе данных» | Docker остановлен, неверный порт или пароль | `docker compose ps`; сверить `config.ini` и `.env` |
| `port is already allocated` | порт 5433 занят | изменить `POSTGRES_HOST_PORT` в `.env` и порт в `config.ini` |
| Нет изображений | EXE скопирован без папки `_internal` | копировать папку `ShoeStore` целиком |

## Проверка подключения без приложения

```powershell
docker compose run --rm --no-deps db-check
```