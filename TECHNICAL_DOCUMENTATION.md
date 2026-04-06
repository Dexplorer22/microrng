# WHEEL FORTUNE - ПОЛНАЯ ТЕХНИЧЕСКАЯ ДОКУМЕНТАЦИЯ

**Дата создания:** 2026-04-06  
**Версия:** 1.0.0  
**Статус:** ✅ Production Ready  
**Язык:** Русский/English

---

## 📑 СОДЕРЖАНИЕ

1. [Обзор проекта](#обзор-проекта)
2. [Архитектура системы](#архитектура-системы)
3. [Компоненты и функционал](#компоненты-и-функционал)
4. [Технический стек](#технический-стек)
5. [API Документация](#api-документация)
6. [Конфигурация](#конфигурация)
7. [Развертывание](#развертывание)
8. [Мониторинг и логирование](#мониторинг-и-логирование)
9. [Безопасность](#безопасность)
10. [Тестирование](#тестирование)
11. [Troubleshooting](#troubleshooting)

---

## 🎯 ОБЗОР ПРОЕКТА

### Описание
Wheel Fortune - это полнофункциональная система колеса фортуны с поддержкой:
- Множественных колес (multi-wheel)
- Системы тиров (Bronze/Silver/Gold)
- Джекпота с накоплением
- Генерации билетов (RNG)
- Бюджетного конверта (Budget Envelope)
- SQLite персистентности

### Основные характеристики
- **Конкурс:** Contest Wheel - 100k Tickets
- **Цена спина:** £1.00
- **Целевая маржа:** 30%
- **Секторов на колесе:** 24
- **Система тиров:** 3 уровня (Bronze/Silver/Gold)
- **Джекпот:** Включен (1 из 500)

### Ключевые метрики (на 10,000 спинов)
- Доход: £10,000.00
- Выплаты: £7,014.85
- Прибыль: £2,985.15
- Маржа: 29.85%
- Win Rate: 87.87%

---

## 🏗️ АРХИТЕКТУРА СИСТЕМЫ

### Общая схема

```
┌─────────────────────────────────────────────────────────────┐
│                     WHEEL FORTUNE                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐         ┌──────────────┐                │
│  │   Frontend   │         │   Backend    │                │
│  │  (Nginx)     │◄───────►│  (FastAPI)   │                │
│  │  Port 3000   │         │  Port 8000   │                │
│  └──────────────┘         └──────────────┘                │
│         │                         │                        │
│         │                         ▼                        │
│         │                  ┌──────────────┐               │
│         │                  │  Engine      │               │
│         │                  │  (RNG/Logic) │               │
│         │                  └──────────────┘               │
│         │                         │                        │
│         │                         ▼                        │
│         │                  ┌──────────────┐               │
│         └─────────────────►│  SQLite DB   │               │
│                            │  wheelv1.db  │               │
│                            └──────────────┘               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Слои архитектуры

**1. Presentation Layer (Frontend)**
- HTML/CSS/JavaScript
- Test Dashboard
- Real-time UI updates
- CORS-enabled communication

**2. API Layer (Backend)**
- FastAPI REST API
- JWT Authentication
- RBAC (Role-Based Access Control)
- Request validation

**3. Business Logic Layer**
- RNG Engine
- Budget Envelope Algorithm
- Tier System
- Jackpot Management

**4. Data Layer**
- SQLite Database
- Transaction logging
- Audit trails
- Persistence

---

## 🔧 КОМПОНЕНТЫ И ФУНКЦИОНАЛ

### Frontend (http://localhost:3000)

**Технология:** HTML5 + JavaScript + Nginx

**Функционал:**
- Test Dashboard для тестирования API
- Health Check endpoint
- Spin execution
- Real-time statistics
- Prize distribution display

**Файлы:**
- `prodactions/tm-wheel-theme-sfx-restored-muted/index.html` - Test Dashboard
- `prodactions/tm-wheel-theme-sfx-restored-muted/assets/js/wheel.js` - Main wheel logic
- `prodactions/tm-wheel-theme-sfx-restored-muted/assets/js/wheel_v2.js` - Alternative wheel logic

**Исправления:**
- ✅ Cookies: SameSite=None; Secure
- ✅ RNG Tickets: Правильная генерация
- ✅ CORS: Полностью настроен

### Backend (http://localhost:8000)

**Технология:** Python 3.12 + FastAPI + SQLite

**Основные компоненты:**

#### 1. Authentication (auth.py)
```python
- JWT Token generation
- Password hashing (bcrypt)
- Role-based permissions
- User management
```

**Эндпоинты:**
- POST /auth/login - Получить JWT токен
- GET /auth/me - Информация о текущем пользователе
- GET /auth/permissions - Список доступных прав

#### 2. Wheel Management (main.py)
```python
- Create/Read/Update/Delete wheels
- Prize configuration
- Spin execution
- Statistics collection
```

**Эндпоинты:**
- GET /wheels - Список всех колес
- POST /wheels - Создать новое колесо
- GET /wheels/{wheel_id} - Информация о колесе
- PUT /wheels/{wheel_id} - Обновить колесо
- DELETE /wheels/{wheel_id} - Удалить колесо

#### 3. Game Engine (engine.py)
```python
- RNG generation (1-999999)
- Spin logic
- Prize allocation
- Ticket generation
- Budget envelope algorithm
```

**Ключевые функции:**
- `spin()` - Выполнить спин
- `_gen_ticket_number()` - Генерировать номер билета
- `_allocate_prize()` - Выбрать приз
- `_apply_budget_envelope()` - Применить бюджетный конверт

#### 4. Database (database.py)
```python
- SQLite ORM
- Transaction management
- Audit logging
- Data persistence
```

**Таблицы:**
- wheels - Конфигурация колес
- spins - История спинов
- players - Информация об игроках
- audit_logs - Логи действий

#### 5. Models (models.py)
```python
- Pydantic data models
- Request/Response schemas
- Validation rules
```

**Основные модели:**
- WheelCreate/WheelResponse
- SpinRequest/SpinResult
- PrizeConfig
- PlayerInfo
- UISpinResponse

### REST API Эндпоинты

#### Управление колесами
```
GET    /wheels                    - Список всех колес
POST   /wheels                    - Создать колесо
GET    /wheels/{wheel_id}         - Получить колесо
PUT    /wheels/{wheel_id}         - Обновить колесо
DELETE /wheels/{wheel_id}         - Удалить колесо
```

#### Управление призами
```
GET    /wheels/{wheel_id}/prizes  - Список призов
POST   /wheels/{wheel_id}/prizes  - Установить призы
```

#### Игровые операции
```
POST   /wheels/{wheel_id}/ui/spin           - Выполнить спин
GET    /wheels/{wheel_id}/ui/state          - Получить состояние
GET    /wheels/{wheel_id}/ui/player/{id}/history - История игрока
```

#### Статистика
```
GET    /wheels/{wheel_id}/stats             - Статистика колеса
GET    /wheels/{wheel_id}/ev                - EV анализ
GET    /wheels/{wheel_id}/simulation        - Симуляция
GET    /wheels/{wheel_id}/player/{id}       - Информация об игроке
```

#### Аутентификация
```
POST   /auth/login                - Получить токен
GET    /auth/me                   - Текущий пользователь
GET    /auth/permissions          - Доступные права
```

#### Управление
```
GET    /health                    - Health check
POST   /wheels/{wheel_id}/reset   - Сбросить колесо
GET    /version                   - Версия API
```

---

## 💻 ТЕХНИЧЕСКИЙ СТЕК

### Frontend
- **HTML5** - Структура
- **CSS3** - Стилизация (градиенты, анимации)
- **JavaScript (ES6+)** - Логика
- **Nginx** - Web server
- **Docker** - Контейнеризация

### Backend
- **Python 3.12** - Язык программирования
- **FastAPI** - Web framework
- **Pydantic** - Data validation
- **SQLite** - Database
- **PyJWT** - JWT tokens
- **bcrypt** - Password hashing
- **Uvicorn** - ASGI server
- **Docker** - Контейнеризация

### DevOps
- **Docker Compose** - Оркестрация контейнеров
- **Docker** - Контейнеризация
- **Nginx** - Reverse proxy

### Инструменты разработки
- **Git** - Version control
- **PowerShell/Bash** - Скрипты
- **curl/Invoke-WebRequest** - API тестирование

---

## 🔐 КОНФИГУРАЦИЯ

### Переменные окружения (.env)

```env
# Backend Configuration
WHEELV1_DB_PATH=/app/data/wheelv1.db
WHEELV1_ADMIN_USER=admin
WHEELV1_ADMIN_PASSWORD=admin123

# Frontend Configuration
BACKEND_URL=http://localhost:8000
FRONTEND_URL=http://localhost:3000

# API Configuration
API_KEY=test-api-key-12345
```

### Конфигурация колеса

```json
{
  "name": "Contest Wheel - 100k Tickets",
  "config": {
    "spin_price": 1,
    "target_margin": 0.3,
    "min_margin": 0.1,
    "segments_total": 24,
    "loss_label": "Try Again",
    "tiers_enabled": true,
    "tier_configs": [
      {
        "name": "bronze",
        "min_spend": 0,
        "max_spend": 30,
        "win_rate": 0.75,
        "max_payout": 5
      },
      {
        "name": "silver",
        "min_spend": 30,
        "max_spend": 100,
        "win_rate": 0.8,
        "max_payout": 5
      },
      {
        "name": "gold",
        "min_spend": 100,
        "max_spend": 999999,
        "win_rate": 0.95,
        "max_payout": 5
      }
    ],
    "jackpot_enabled": true,
    "jackpot_config": {
      "contribution_rate": 0.5,
      "chance_mode": "chance_n",
      "chance_n": 500,
      "payout_mode": "full_balance",
      "min_balance": 50,
      "visual_max": 1000
    },
    "ticket_allocation_mode": "spins_per_ticket",
    "ticket_allocation_value": 1000,
    "ticket_digits_min": 1,
    "ticket_digits_max": 5,
    "ticket_max": 100000
  }
}
```

### Конфигурация призов

```json
[
  {"id": "prize_100", "name": "100", "cost": 100, "weight": 100, "visual_only": false},
  {"id": "prize_20", "name": "20", "cost": 20, "weight": 500, "visual_only": false},
  {"id": "prize_5", "name": "5", "cost": 5, "weight": 1000, "visual_only": false},
  {"id": "prize_1", "name": "1", "cost": 1, "weight": 5000, "visual_only": false},
  {"id": "prize_050", "name": "0.50", "cost": 0.5, "weight": 10000, "visual_only": false},
  {"id": "prize_010", "name": "0.10", "cost": 0.1, "weight": 10000, "visual_only": false},
  {"id": "prize_005", "name": "0.05", "cost": 0.05, "weight": 50000, "visual_only": false},
  {"id": "prize_3000", "name": "3000", "cost": 3000, "weight": 50, "visual_only": true},
  {"id": "prize_2000", "name": "2000", "cost": 2000, "weight": 75, "visual_only": true},
  {"id": "prize_1000", "name": "1000", "cost": 1000, "weight": 1, "visual_only": true}
]
```

---

## 🚀 РАЗВЕРТЫВАНИЕ

### Требования
- Docker & Docker Compose
- Python 3.12+
- Node.js (опционально)
- 2GB RAM минимум
- 1GB свободного места

### Быстрый старт

**Windows:**
```bash
start-full.bat
```

**Linux/Mac:**
```bash
bash start-full.sh
```

### Ручное развертывание

```bash
# 1. Распаковать архив
unzip money_weel_060426.zip

# 2. Перейти в директорию
cd prodactions

# 3. Запустить Docker Compose
docker-compose up -d

# 4. Проверить статус
docker-compose ps

# 5. Просмотреть логи
docker-compose logs -f
```

### Доступные сервисы после запуска

| Сервис | URL | Описание |
|--------|-----|---------|
| Frontend | http://localhost:3000 | Test Dashboard |
| Backend API | http://localhost:8000 | REST API |
| Admin Panel | http://localhost:8000/admin.html | Admin interface |
| Health Check | http://localhost:8000/health | Status check |

### Учетные данные

```
Username: admin
Password: admin123
API Key: test-api-key-12345
```

---

## 📊 МОНИТОРИНГ И ЛОГИРОВАНИЕ

### Логи контейнеров

```bash
# Все логи
docker-compose logs -f

# Только backend
docker-compose logs -f backend

# Только frontend
docker-compose logs -f frontend

# Последние 100 строк
docker-compose logs --tail=100
```

### Метрики системы

**Health Check:**
```bash
curl http://localhost:8000/health
```

**Ответ:**
```json
{
  "status": "healthy",
  "wheels_count": 1,
  "total_spins": 10000
}
```

### Статистика колеса

```bash
curl -H "Authorization: Bearer <token>" \
  http://localhost:8000/wheels/1/stats
```

**Ответ содержит:**
- spins_count - Всего спинов
- total_revenue - Доход
- total_payout - Выплаты
- profit - Прибыль
- margin - Маржа
- prizes_awarded - Распределение призов
- tier_distribution - Распределение по тирам

---

## 🔒 БЕЗОПАСНОСТЬ

### Аутентификация

**JWT Tokens:**
- Срок действия: 24 часа
- Алгоритм: HS256
- Хранение: Bearer token в заголовке Authorization

**Пример:**
```bash
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..." \
  http://localhost:8000/wheels
```

### Авторизация (RBAC)

**Роли:**
- owner - Полный доступ
- admin - Управление колесами и призами
- user - Только чтение

**Права:**
- wheels:read, wheels:write, wheels:delete
- prizes:read, prizes:write
- stats:read
- game:manage
- users:read, users:write
- audit:read

### Cookies

**Конфигурация:**
```javascript
document.cookie = 'tmwl_wallet_email=value; path=/; SameSite=None; Secure';
document.cookie = 'tmwl_player_email=value; path=/; SameSite=None; Secure';
document.cookie = 'tmwl_wallet_order=value; path=/; SameSite=None; Secure';
```

**Параметры:**
- SameSite=None - Разрешить cross-site
- Secure - Только HTTPS
- HttpOnly - Недоступно для JavaScript (на сервере)

### CORS

**Конфигурация:**
```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### API Key

**Использование:**
```bash
curl -H "X-API-Key: test-api-key-12345" \
  http://localhost:8000/wheels/1/ui/spin
```

---

## 🧪 ТЕСТИРОВАНИЕ

### Unit Tests

```bash
# Запустить все тесты
pytest tests/

# С покрытием
pytest --cov=app tests/

# Конкретный тест
pytest tests/test_engine.py::test_spin
```

### Integration Tests

```bash
# Тестирование API
pytest tests/test_api.py

# Тестирование БД
pytest tests/test_database.py
```

### Load Testing

```bash
# 10,000 спинов
for i in {1..10000}; do
  curl -X POST http://localhost:8000/wheels/1/ui/spin \
    -H "X-API-Key: test-api-key-12345" \
    -H "Content-Type: application/json" \
    -d '{"player_id":"test_$i","player_name":"Test $i","web_credit":0,"is_free":false}'
done
```

### Manual Testing

**Test Dashboard:**
1. Открыть http://localhost:3000
2. Нажать "Health Check"
3. Нажать "List Wheels"
4. Нажать "Perform Spin"
5. Проверить результаты

---

## 🔧 TROUBLESHOOTING

### Проблема: Backend не запускается

**Решение:**
```bash
# Проверить логи
docker-compose logs backend

# Пересобрать образ
docker-compose build --no-cache backend

# Перезагрузить
docker-compose restart backend
```

### Проблема: Frontend не видит Backend

**Решение:**
```bash
# Проверить CORS
curl -H "Origin: http://localhost:3000" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: Content-Type" \
  -X OPTIONS http://localhost:8000/health

# Проверить nginx конфиг
docker-compose exec frontend cat /etc/nginx/nginx.conf
```

### Проблема: Port уже занят

**Решение:**
```bash
# Найти процесс на порту 8000
lsof -i :8000

# Найти процесс на порту 3000
lsof -i :3000

# Убить процесс
kill -9 <PID>
```

### Проблема: Cookies не сохраняются

**Решение:**
1. Проверить SameSite=None; Secure
2. Проверить HTTPS (для production)
3. Проверить domain в cookies
4. Очистить браузер кэш

### Проблема: Маржа не соответствует целевой

**Решение:**
1. Проверить конфигурацию призов
2. Проверить веса (weight) призов
3. Проверить бюджетный конверт
4. Запустить симуляцию

---

## 📈 ПРОИЗВОДИТЕЛЬНОСТЬ

### Оптимизация

**Backend:**
- Connection pooling для БД
- Кэширование результатов
- Асинхронные операции
- Индексы на часто используемых полях

**Frontend:**
- Минификация CSS/JS
- Кэширование статических файлов
- Lazy loading изображений
- Compression (gzip)

### Масштабирование

**Горизонтальное:**
- Несколько инстансов backend
- Load balancer (nginx)
- Shared database

**Вертикальное:**
- Увеличить RAM
- Увеличить CPU cores
- Оптимизировать запросы

---

## 📝 ЛИЦЕНЗИЯ И АВТОРСТВО

**Проект:** Wheel Fortune  
**Версия:** 1.0.0  
**Дата:** 2026-04-06  
**Статус:** Production Ready  

---

## 📞 ПОДДЕРЖКА

**Документация:** /README.md  
**Тестирование:** /TESTING.md  
**Развертывание:** /DEPLOYMENT.md  
**Быстрый старт:** /QUICKSTART.md  

---

**Конец документации**
