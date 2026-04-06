# WheelV1 Contest Engine

Математическая модель колеса фортуны с гарантией `profit >= 0` на каждом спине.

## Архитектура

**Budget Envelope** — каждый спин увеличивает revenue. Приз выдаётся только если его стоимость не превышает оставшийся бюджет:

```
budget = revenue × (1 - min_margin) - total_payout
prize выдаётся только если prize.cost <= budget
→ profit >= 0 ВСЕГДА
```

Интегральный контроллер (I-controller) корректирует win-rate в реальном времени, чтобы фактическая маржа сходилась к `target_margin`.

---

## Админка и доступ по паролю

Админские эндпоинты защищены **HTTP Basic Auth** (логин и пароль от API). Публичные эндпоинты (спин, state для игроков, health, version) работают без авторизации.

**Переменные окружения:**

| Переменная | По умолчанию | Описание |
|------------|:---:|----------|
| `WHEELV1_ADMIN_USER` | `admin` | Логин для админки |
| `WHEELV1_ADMIN_PASSWORD` | *(пусто)* | Пароль. Если не задан — проверка отключена (все эндпоинты открыты) |

Чтобы открыть админку заказчику: задайте `WHEELV1_ADMIN_PASSWORD` в окружении (и при необходимости `WHEELV1_ADMIN_USER`), передайте ему логин и пароль. Доступ к админским операциям: Swagger `/docs`, прямой вызов API с заголовком `Authorization: Basic <base64(login:password)>`, либо любой фронт, который подставляет эти учётные данные.

**Защищённые эндпоинты:** список/создание/изменение/удаление колёс, призы, раунды, сброс, статистика, `GET /wheels/{id}/ui/admin_state`.  
**Без пароля:** `GET /wheels/{id}/ui/state`, `POST /wheels/{id}/ui/spin`, история игрока, `POST /wheels/{id}/spin`, `/health`, `/version`.

---

## Как задаётся «когда выпадает сектор» (ответ заказчику)

Логика верная: **билет (ticket)** — это идентификатор попытки с фронта; движок не подбирает билет под сектор, а возвращает результат спина (WIN/LOSE, приз, `target_angle`, `spin_id`). Сектор определяется по `target_angle`.

**Когда и сколько раз за N спинов выпадает сектор/приз** задаётся **весами (weights)** и ограничениями:

- У каждого приза есть `weight`: чем больше вес, тем чаще приз участвует в розыгрыше относительно других. Точная частота выпадения — вероятностная, не «ровно раз в N спинов».
- Дополнительно: `qty` (лимит выдачи приза), `tier_mode` (bronze/silver/gold), Budget Envelope (приз не выпадет, если его cost превышает текущий бюджет), `visual_only` (только на колесе, в RNG не участвует). Джекпот — отдельная механика (chance_n / probability).

Итого: «запрограммировать ровно на спине №K выпадет сектор X» движок не умеет; настройка идёт через веса и лимиты, итог — вероятностный.

---

## Разработка и окружение

Рекомендуется использовать **Conda**, чтобы версии зависимостей совпадали и тесты не ломались из‑за окружения (в т.ч. Starlette/httpx).

```bash
# Создать окружение
conda env create -f environment.yml

# Активировать
conda activate wheelv1-contest

# Запуск API
uvicorn main:app --host 0.0.0.0 --port 8000

# Тесты
pytest test_ui.py test_api.py -v
```

Обновить окружение после изменения `environment.yml`:  
`conda env update -f environment.yml --prune`

Без Conda: `python -m venv .venv`, `source .venv/bin/activate` (или `.venv\Scripts\activate` на Windows), затем `pip install -r requirements.txt` и `pip install pytest`.

---

## API Reference

Базовый URL: `http://karouniform:8000`

Все ответы в JSON. Все endpoints синхронные.

---

### 1. Создать колесо

```
POST /wheels
```

**Body:**
```json
{
  "name": "Summer Raffle",
  "config": {
    "spin_price": 2.00,
    "target_margin": 0.10,
    "min_margin": 0.0,
    "segments_total": 60,
    "loss_label": "NO WIN",
    "tiers": {
      "enabled": true,
      "bronze": { "win_rate": 0.35, "max_return_pct": 5.0 },
      "silver": { "win_rate": 0.40, "max_return_pct": 5.0 },
      "gold":   { "win_rate": 0.45, "max_return_pct": 5.0 },
      "silver_threshold": 30.0,
      "gold_threshold": 100.0,
      "webcredit_win_mult": 0.20
    },
    "jackpot": {
      "enabled": true,
      "contribution_rate": 0.50,
      "hit_mode": "chance_n",
      "hit_n": 500,
      "payout_mode": "full",
      "min_balance": 50.0,
      "visual_max": 1000.0
    },
    "seed": 42
  }
}
```

**Ответ:**
```json
{ "status": "ok", "wheel_id": 1 }
```

**curl:**
```bash
curl -X POST http://karouniform:8000/wheels \
  -H "Content-Type: application/json" \
  -d '{"name":"Test Wheel","config":{"spin_price":2.0,"target_margin":0.10,"seed":42}}'
```

---

### 2. Задать призы

```
POST /wheels/{wheel_id}/prizes
```

**Body:** массив призов.

```json
[
  { "id": "p1", "name": "Sticker",       "cost": 0.50, "weight": 5.0, "tier_mode": "all" },
  { "id": "p2", "name": "Keychain",      "cost": 1.50, "weight": 4.0, "tier_mode": "all" },
  { "id": "p3", "name": "T-Shirt",       "cost": 5.00, "weight": 2.0, "tier_mode": "all" },
  { "id": "p4", "name": "Headphones",    "cost": 15.0, "weight": 0.8, "tier_mode": "gold" },
  { "id": "p5", "name": "Tablet",        "cost": 50.0, "weight": 0.1, "tier_mode": "gold", "qty": 3 },
  { "id": "p_car", "name": "Sports Car", "cost": 25000, "weight": 0.001, "visual_only": true }
]
```

| Поле | Тип | По умолчанию | Описание |
|------|-----|:---:|----------|
| `id` | string | — | Уникальный ID приза |
| `name` | string | — | Название |
| `cost` | float | — | Себестоимость (£) |
| `weight` | float | 1.0 | Вес для RNG (больше = чаще) |
| `qty` | int\|null | null | Лимит выдачи (null = безлимит) |
| `tier_mode` | string | "all" | Кому доступен: `all`, `bronze`/`t1`, `silver`/`t2`, `gold`/`t3` |
| `is_active` | bool | true | Активен ли |
| `visual_only` | bool | false | Только на колесе, НЕ участвует в RNG |
| `sort_order` | int | 0 | Порядок сортировки |

**Ответ:**
```json
{ "status": "ok", "prizes_count": 6, "visual_only_count": 1, "rng_prizes_count": 5 }
```

**curl:**
```bash
curl -X POST http://karouniform:8000/wheels/1/prizes \
  -H "Content-Type: application/json" \
  -d '[{"id":"p1","name":"Sticker","cost":0.5,"weight":5},{"id":"p2","name":"T-Shirt","cost":5,"weight":2}]'
```

---

### 3. Спин

```
POST /wheels/{wheel_id}/spin
```

**Body:**
```json
{
  "player_id": "player_42",
  "player_name": "Alice",
  "web_credit": 0.0
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| `player_id` | string | Уникальный ID игрока |
| `player_name` | string\|null | Имя (опционально) |
| `web_credit` | float | Если > 0 → режим "web" (win_rate × 0.2, payout = cost/5) |

**Ответ:**
```json
{
  "spin_id": 1,
  "player_id": "player_42",
  "player_name": "Alice",
  "tier": "bronze",
  "outcome": "PRIZE",
  "prize": { "id": "p1", "name": "Sticker", "cost": 0.5 },
  "target_angle": 234.567,
  "jackpot_won": null,
  "effective_win_rate": 0.472,
  "round_id": 1,
  "spin_index": 1,
  "spend_total": 2.0,
  "jackpot_add": 0.75,
  "jackpot_balance": 0.75
}
```

| Поле ответа | Описание |
|-------------|----------|
| `outcome` | `PRIZE` — выигрыш, `LOSE` — проигрыш, `JACKPOT` — джекпот |
| `prize` | Объект приза (null если LOSE) |
| `target_angle` | Угол для анимации колеса (0-360) |
| `jackpot_won` | Сумма джекпота (только при JACKPOT) |
| `effective_win_rate` | Эффективный win-rate на момент спина |
| `round_id` | ID текущего раунда |
| `spin_index` | Номер спина этого игрока |
| `spend_total` | Суммарные траты игрока |
| `jackpot_add` | Сколько добавлено в джекпот этим спином |
| `jackpot_balance` | Баланс джекпота после спина |

**curl:**
```bash
curl -X POST http://karouniform:8000/wheels/1/spin \
  -H "Content-Type: application/json" \
  -d '{"player_id":"p1","player_name":"Alice"}'
```

---

### 4. Статистика колеса

```
GET /wheels/{wheel_id}/stats
```

**Ответ:**
```json
{
  "spins_count": 1000,
  "total_revenue": 2000.0,
  "total_payout": 1800.0,
  "profit": 200.0,
  "margin": 0.10,
  "jackpot_balance": 53.25,
  "players_count": 150,
  "prizes_awarded": { "p1": 320, "p2": 180 },
  "tier_distribution": { "bronze": 400, "silver": 300, "gold": 280, "web": 20 },
  "budget_remaining": 200.0,
  "win_count": 500,
  "loss_count": 500,
  "actual_win_rate": 0.5,
  "active_round_id": 1
}
```

**curl:**
```bash
curl http://karouniform:8000/wheels/1/stats
```

---

### 5. Детали колеса (config + segments + prizes)

```
GET /wheels/{wheel_id}
```

Возвращает **полную информацию** о колесе: конфигурацию, список призов, массив секторов для UI и текущую статистику (включая `jackpot_balance`).

**Ключевые поля ответа:**

| Поле | Описание |
|------|----------|
| `config.spin_price` | Цена спина |
| `config.loss_label` | Текст проигрышного сектора |
| `config.segments_total` | Кол-во секторов |
| `config.jackpot.visual_max` | Потолок шкалы джекпота |
| `stats.jackpot_balance` | Текущий баланс джекпота |
| `prizes` | Массив всех призов |
| `segments` | Готовый массив секторов для UI |

**Формат одного сегмента:**
```json
{
  "index": 0,
  "label": "Sticker",
  "type": "prize",
  "prize_id": "p1",
  "prize_cost": 0.5,
  "visual_only": false,
  "angle_from": 0.0,
  "angle_to": 6.0
}
```

Типы секторов: `prize` (выигрышный), `loss` (проигрыш), `prize` с `visual_only=true` (на колесе, но не выпадает).

**curl:**
```bash
curl http://localhost:8000/wheels/1
```

---

### 6. История спинов игрока

```
GET /wheels/{wheel_id}/player/{player_id}/spins?limit=50&offset=0
```

Полная история спинов игрока из БД. Отсортировано по убыванию (новые первые). Поддерживает пагинацию.

**Query params:**

| Параметр | По умолчанию | Описание |
|----------|:---:|----------|
| `limit` | 50 | Кол-во записей |
| `offset` | 0 | Смещение |

**Ответ:**
```json
{
  "player_id": "alice",
  "wheel_id": 1,
  "total": 20,
  "limit": 5,
  "offset": 0,
  "spins": [
    {
      "spin_id": 20,
      "round_id": 1,
      "spin_index": 20,
      "player_mode": "silver",
      "spend_total": 40.0,
      "outcome_type": "PRIZE",
      "prize_id": "p1",
      "prize_title": "Sticker",
      "prize_cost": 0.5,
      "jackpot_add": 0.75,
      "jackpot_balance": 12.0,
      "target_angle": 103.775,
      "effective_win_rate": 0.472,
      "created_at": "2026-02-15T..."
    }
  ]
}
```

**curl:**
```bash
# Последние 10 спинов
curl "http://localhost:8000/wheels/1/player/alice/spins?limit=10"

# Страница 2
curl "http://localhost:8000/wheels/1/player/alice/spins?limit=10&offset=10"
```

---

### 7. Теоретические EV

```
GET /wheels/{wheel_id}/ev
```

Возвращает теоретические ожидаемые значения для текущей конфигурации (призы `visual_only` исключены).

**curl:**
```bash
curl http://karouniform:8000/wheels/1/ev
```

---

### 8. Раунды

```
POST /wheels/{wheel_id}/round          — начать новый раунд
GET  /wheels/{wheel_id}/round/{id}/stats — статистика раунда
```

Если раунд не создан вручную, первый спин создаёт его автоматически.

**curl:**
```bash
curl -X POST http://localhost:8000/wheels/1/round
curl http://localhost:8000/wheels/1/round/1/stats
```

---

### 9. Информация об игроке

```
GET /wheels/{wheel_id}/player/{player_id}
```

**curl:**
```bash
curl http://localhost:8000/wheels/1/player/alice
```

---

### 10. Остальные endpoints

```
GET    /wheels                              — список всех колёс
GET    /wheels/{id}                         — детали (config + prizes + segments + stats)
PUT    /wheels/{id}                         — обновить имя/конфиг
DELETE /wheels/{id}                         — удалить колесо
GET    /wheels/{id}/prizes                  — получить призы
GET    /wheels/{id}/player/{pid}/spins      — история спинов игрока
POST   /wheels/{id}/reset                   — сбросить состояние
GET    /health                              — health-check
```

---

## Конфигурация

### Основные параметры (`config`)

| Параметр | Тип | По умолчанию | Описание |
|----------|-----|:---:|----------|
| `spin_price` | float | 2.0 | Цена одного спина (£) |
| `target_margin` | float | 0.10 | Целевая маржа (0.0 – 0.30) |
| `min_margin` | float | 0.0 | Жёсткий пол маржи (guardrail) |
| `segments_total` | int | 60 | Визуальные секторы колеса |
| `loss_label` | string | "NO WIN" | Текст проигрыша |
| `seed` | int\|null | null | Seed для RNG (для воспроизводимости) |

### Тиры (`config.tiers`)

| Параметр | По умолчанию | Описание |
|----------|:---:|----------|
| `enabled` | true | Включить тир-систему |
| `bronze.win_rate` | 0.35 | Win-rate Bronze |
| `silver.win_rate` | 0.40 | Win-rate Silver |
| `gold.win_rate` | 0.45 | Win-rate Gold |
| `silver_threshold` | 30.0 | £ потраченных для перехода Bronze → Silver |
| `gold_threshold` | 100.0 | £ потраченных для перехода Silver → Gold |
| `webcredit_win_mult` | 0.20 | Множитель win-rate для web-игроков |

**Логика определения тира:**
```
if web_credit > 0       → web   (win_rate = bronze × 0.20, payout = cost/5)
else if spent >= 100    → gold
else if spent >= 30     → silver
else                    → bronze
```

### Джекпот (`config.jackpot`)

| Параметр | По умолчанию | Описание |
|----------|:---:|----------|
| `enabled` | false | Включить джекпот |
| `contribution_rate` | 0.50 | Доля прибыли спина → джекпот |
| `hit_mode` | "chance_n" | `chance_n` (1/N) или `probability` (фикс. %) |
| `hit_n` | 500 | N для режима chance_n |
| `hit_pct` | 0.0005 | Вероятность для режима probability |
| `payout_mode` | "full" | `full` (весь баланс) или `percent` (часть) |
| `payout_pct` | 100.0 | % выплаты (для payout_mode=percent) |
| `min_balance` | 50.0 | Мин. баланс для возможности выпадения |

**Пополнение:**
```
jackpot_add = max(0, (spin_price - actual_payout) × contribution_rate)
```

---

## Полный пример workflow

```bash
# 1. Создать колесо
curl -X POST http://karouniform:8000/wheels \
  -H "Content-Type: application/json" \
  -d '{"name":"Contest #1","config":{"spin_price":2,"target_margin":0.10,"seed":42}}'
# → {"status":"ok","wheel_id":1}

# 2. Задать призы
curl -X POST http://karouniform:8000/wheels/1/prizes \
  -H "Content-Type: application/json" \
  -d '[
    {"id":"p1","name":"Sticker","cost":0.5,"weight":5},
    {"id":"p2","name":"Keychain","cost":1.5,"weight":4},
    {"id":"p3","name":"T-Shirt","cost":5,"weight":2},
    {"id":"p4","name":"Headphones","cost":15,"weight":0.8,"tier_mode":"gold"},
    {"id":"p5","name":"Tablet","cost":50,"weight":0.1,"tier_mode":"gold","qty":3}
  ]'

# 3. Начать раунд (опционально, создаётся автоматически)
curl -X POST http://karouniform:8000/wheels/1/round

# 4. Спины
curl -X POST http://karouniform:8000/wheels/1/spin \
  -H "Content-Type: application/json" \
  -d '{"player_id":"alice","player_name":"Alice"}'

# 5. Статистика
curl http://karouniform:8000/wheels/1/stats

# 6. EV
curl http://karouniform:8000/wheels/1/ev

# 7. Игрок
curl http://karouniform:8000/wheels/1/player/alice

# 8. Сброс (призы сохраняются)
curl -X POST http://karouniform:8000/wheels/1/reset
```

---

## Математические гарантии

| Гарантия | Механизм |
|----------|----------|
| **profit >= 0 всегда** | Budget Envelope: приз блокируется если cost > budget |
| **margin → target** | I-controller с anti-windup корректирует win-rate |
| **visual_only не выпадает** | Исключён из RNG-пула |
| **prize.qty соблюдается** | Счётчик выдачи per-prize |
| **tier caps** | max_return_pct ограничивает выигрыш на игрока |