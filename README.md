<<<<<<< HEAD
# ShoesShop — магазин обуви
=======
# ShoesShop — система заказов обуви
>>>>>>> feature/stock-label

Учебный проект демонстрационного экзамена: десктопное приложение на PySide6
с базой данных PostgreSQL для работы с каталогом обуви и заказами.

## Возможности

- авторизация пользователей, разные страницы для менеджера и администратора;
- просмотр каталога товаров с ценой, скидкой и остатком на складе;
- добавление, редактирование и удаление товаров;
- создание, изменение и удаление заказов, смена статуса заказа.

## Требования

- Python 3.13
- PostgreSQL
- библиотеки из файла requirements.txt (PySide6, psycopg2-binary)

## Установка и запуск

1. Клонировать репозиторий: `git clone https://github.com/kmlessu/ShoesShop.git`
2. Создать окружение и установить зависимости:
   `python -m venv .venv`, `source .venv/bin/activate` (Windows: `.venv\Scripts\activate`),
   `pip install -r requirements.txt`
3. Создать базу данных `shoes_shop` и загрузить скрипт `database/shoes_shop.sql`.
4. Скопировать `config.ini.example` в `config.ini` и указать свои параметры подключения.
5. Запустить приложение: `python main.py`

## Структура репозитория

- `main.py` — точка входа
- `database.py` — работа с базой данных
- `ui/` — страницы и виджеты интерфейса
- `resources/` — изображения и логотип
- `database/` — скрипт базы данных
- `docs/` — ER-диаграмма
## ER-диаграмма базы данных

Диаграмма базы данных находится в файле `docs/er_diagram.png`.

Основные таблицы:
- `products` — товары (артикул, название, цена, скидка, количество и др.);
- `orders` — заказы (даты, статус, код получения, пункт выдачи, покупатель);
- `order_items` — состав заказов (товар, количество, цена);
- `users` — пользователи приложения;
- `roles` — роли пользователей;
- `warehouses` — пункты выдачи;
- `stock_levels` — остатки товаров по пунктам выдачи (товар, пункт выдачи, количество).
## Автор
Хазбиева Хава, группа ПКС-44


## Запуск PostgreSQL через Docker Compose

Приложение запускается в Windows, а PostgreSQL и вспомогательный сервис `db-check` работают в контейнерах.

### Требования

- Docker Desktop (Linux containers) и Docker Compose v2;
- Python и зависимости из `requirements.txt`.

### Подготовка

1. Скопировать `.env.example` в `.env` и задать свой локальный пароль (латинские буквы и цифры) в `POSTGRES_PASSWORD`.
2. Скопировать `config.ini.example` в `config.ini`: `host = localhost`, `port = 5433`, `name = shoes_shop`, `user = postgres`, пароль такой же, как в `.env`.

### Запуск

```bash
docker compose up --build -d
docker compose ps
```

Сервис `db` должен иметь статус `healthy`. Сервис `db-check` выполняет одноразовую проверку и завершается с кодом 0.

### Импорт базы данных

Команды выполняются из корня проекта (в Git Bash перед `docker` добавить `MSYS_NO_PATHCONV=1`):

```bash
docker compose cp database/shoes_shop.sql db:/tmp/shoes_shop.sql
docker compose exec db sed -i '/transaction_timeout/d' /tmp/shoes_shop.sql
docker compose exec db psql -v ON_ERROR_STOP=1 -U postgres -d shoes_shop -f /tmp/shoes_shop.sql
```

Строка с `sed` нужна, потому что скрипт выгружен из более новой версии PostgreSQL, чем в контейнере (14).

### Проверка

```bash
docker compose run --rm --no-deps db-check
```

Ожидается строка `OK: база=shoes_shop; таблиц=7`. Затем запустить приложение: `python main.py`.

### Адреса подключения

- приложение в Windows: `localhost:5433`;
- сервис `db-check` внутри Compose: `db:5432`.

### Остановка

```bash
docker compose stop
```

Команду `docker compose down -v` не использовать: она удаляет данные базы.