# Wheel Fortune - Полная документация

## 📋 Что было сделано

### 1. Исправлены баги с Cookies (SameSite)
**Проблема:** Cookies отклонялись в cross-site контексте (iframe)
**Решение:** Изменено `SameSite=Lax` на `SameSite=None; Secure`

**Файлы исправлены:**
- `prodactions/tm-wheel-theme-sfx-restored-muted/assets/js/wheel.js` (4 места)
- `prodactions/tm-wheel-theme-sfx-restored-muted/assets/js/wheel_v2.js` (4 места)
- `prodactions/tm-wheel-theme-sfx-restored-muted/templates/wallet.php` (1 место)

### 2. Исправлены баги с RNG Ticket Numbers
**Проблема:** Билеты отображались как "00001" вместо реального номера
**Решение:** Добавлены отсутствующие элементы для отслеживания RNG анимации

**Файлы исправлены:**
- `prodactions/tm-wheel-theme-sfx-restored-muted/assets/js/wheel.js`
  - Добавлена ссылка на `rngBtnEl` (строка 123)
  - Добавлен класс `tmwl-rng-running` при начале анимации (после строки 1291)
  - Удален класс `tmwl-rng-running` при завершении (после строки 1312)

- `prodactions/tm-wheel-theme-sfx-restored-muted/assets/js/wheel_v2.js`
  - Добавлена ссылка на `rngBtnEl` (строка 115)
  - Добавлен класс `tmwl-rng-running` при начале анимации (после строки 1345)
  - Удален класс `tmwl-rng-running` при завершении (после строки 1366)

### 3. Создана Docker инфраструктура
**Компоненты:**
- Backend (FastAPI) на порту 8000
- Frontend (Nginx) на порту 3000
- SQLite база данных

**Файлы созданы:**
- `server/Dockerfile` - Docker образ для backend
- `docker-compose.yml` - Оркестрация контейнеров
- `nginx.conf` - Конфигурация Nginx
- `.env` - Переменные окружения
- `start.sh` - Скрипт запуска для Linux/Mac
- `start.bat` - Скрипт запуска для Windows

## 🚀 Быстрый старт

### Windows
```bash
start.bat
```

### Linux/Mac
```bash
bash start.sh
```

## 📍 Доступные URL

| Сервис | URL | Описание |
|--------|-----|---------|
| Frontend | http://localhost:3000 | Основной интерфейс колеса |
| Backend API | http://localhost:8000 | REST API |
| Admin Panel | http://localhost:8000/admin.html | Админ панель |
| Health Check | http://localhost:8000/health | Проверка статуса |

## 🔐 Учетные данные

```
Username: admin
Password: admin123
API Key: test-api-key-12345
```

## 📊 Структура проекта

```
C:\tmp\prod\
├── prodactions/                          # WordPress плагин
│   ├── tmwl-engine.php                  # Главный файл плагина
│   ├── inc/                             # PHP бэкенд
│   │   ├── rest-api.php                 # REST API эндпоинты
│   │   ├── wallet.php                   # Кошелек игрока
│   │   ├── engine.php                   # Интеграция с engine
│   │   ├── helpers.php                  # Вспомогательные функции
│   │   ├── accounting.php               # Учет спинов
│   │   ├── email.php                    # Email уведомления
│   │   └── woocommerce.php              # WooCommerce интеграция
│   ├── admin/                           # Админ панель
│   └── tm-wheel-theme-sfx-restored-muted/  # Фронтенд тема
│       ├── assets/
│       │   ├── js/
│       │   │   ├── wheel.js             # ✅ ИСПРАВЛЕН
│       │   │   ├── wheel_v2.js          # ✅ ИСПРАВЛЕН
│       │   │   ├── tmwl-ui.js
│       │   │   ├── tmwl-sfx.js
│       │   │   └── admin.js
│       │   ├── css/
│       │   └── sfx/
│       └── templates/
│           ├── wheel-live.php
│           ├── wallet.php               # ✅ ИСПРАВЛЕН
│           └── admin.php
├── server/                              # FastAPI backend
│   ├── Dockerfile
│   ├── main.py
│   ├── models.py
│   ├── engine.py
│   ├── database.py
│   ├── auth.py
│   └── requirements.txt
├── docker-compose.yml                   # Docker Compose конфиг
├── nginx.conf                           # Nginx конфиг
├── .env                                 # Переменные окружения
├── start.sh                             # Скрипт запуска (Linux/Mac)
├── start.bat                            # Скрипт запуска (Windows)
├── README.md                            # Основная документация
├── TESTING.md                           # Инструкции по тестированию
└── .gitignore                           # Git ignore файл
```

## 🔧 API Endpoints

### Создать колесо
```bash
POST /wheels
```

### Добавить призы
```bash
POST /wheels/{wheel_id}/prizes
```

### Выполнить спин
```bash
POST /wheels/{wheel_id}/ui/spin
```

### Получить статистику
```bash
GET /wheels/{wheel_id}/stats
```

### История спинов игрока
```bash
GET /wheels/{wheel_id}/ui/player/{player_id}/history
```

## ✅ Проверка исправлений

### 1. Cookies
- DevTools → Application → Cookies
- Проверить `SameSite=None` для `tmwl_wallet_email`, `tmwl_player_email`, `tmwl_wallet_order`

### 2. RNG Ticket Numbers
- Выполнить спин
- Проверить что `ticket_number` содержит реальное число
- Проверить что в истории отображается правильный номер

### 3. RNG Animation
- DevTools → Console
- Выполнить спин
- Проверить отсутствие ошибок
- Проверить корректность анимации

## 📝 Логи

```bash
# Все логи
docker-compose logs -f

# Только backend
docker-compose logs -f backend

# Только frontend
docker-compose logs -f frontend
```

## 🛑 Остановка

```bash
# Остановить контейнеры
docker-compose down

# Остановить и удалить данные
docker-compose down -v
```

## 📞 Поддержка

Для вопросов и проблем обратитесь к разработчику.

---

**Дата создания:** 2026-04-06
**Версия:** 1.0.0
**Статус:** ✅ Готово к тестированию
