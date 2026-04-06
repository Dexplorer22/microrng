# Тестирование Wheel Fortune

## Быстрый старт

### Windows
```bash
start.bat
```

### Linux/Mac
```bash
bash start.sh
```

## После запуска

### 1. Проверить что всё работает

```bash
# Проверить backend
curl http://localhost:8000/health

# Проверить frontend
curl http://localhost:3000
```

### 2. Открыть в браузере

- **Frontend**: http://localhost:3000
- **Admin Panel**: http://localhost:8000/admin.html
  - Username: `admin`
  - Password: `admin123`

## Тестовые сценарии

### Сценарий 1: Создать колесо

```bash
curl -X POST http://localhost:8000/wheels \
  -H "Content-Type: application/json" \
  -H "X-API-Key: test-api-key-12345" \
  -d '{
    "name": "Test Wheel",
    "config": {
      "spin_price": 1.0,
      "segments_total": 60,
      "min_margin": 0.25,
      "loss_label": "NO WIN"
    }
  }'
```

Ответ:
```json
{
  "id": 1,
  "name": "Test Wheel",
  "is_active": true,
  "created_at": "2026-04-06T18:07:56.813Z",
  "spins_count": 0,
  "profit": 0.0
}
```

### Сценарий 2: Добавить призы

```bash
curl -X POST http://localhost:8000/wheels/1/prizes \
  -H "Content-Type: application/json" \
  -H "X-API-Key: test-api-key-12345" \
  -d '{
    "prizes": [
      {
        "id": "prize_1",
        "name": "£10",
        "cost": 10.0,
        "qty_total": 100
      },
      {
        "id": "prize_2",
        "name": "£50",
        "cost": 50.0,
        "qty_total": 20
      },
      {
        "id": "prize_3",
        "name": "£100",
        "cost": 100.0,
        "qty_total": 5
      }
    ]
  }'
```

### Сценарий 3: Выполнить спин

```bash
curl -X POST http://localhost:8000/wheels/1/ui/spin \
  -H "Content-Type: application/json" \
  -H "X-API-Key: test-api-key-12345" \
  -d '{
    "player_id": "player_123",
    "player_name": "John Doe",
    "web_credit": 0,
    "is_free": false
  }'
```

Ответ:
```json
{
  "ok": true,
  "spin": {
    "spin_id": 1,
    "outcome_type": "win",
    "winner_label": "£10",
    "prize_id": "prize_1",
    "prize_cost": 10.0,
    "target_angle": 45.5,
    "segment_index": 5,
    "jackpot_won": null,
    "ticket_number": 12345,
    "ticket_code": "012345"
  },
  "jackpot": {
    "enabled": true,
    "balance": 0.0,
    "visual_max": 1000,
    "min_balance": 50
  },
  "player": {
    "id": "player_123",
    "name": "John Doe",
    "tier": "bronze",
    "web_credit": 0,
    "spins_total": 1,
    "spins_left": 999999,
    "total_spent": 1.0,
    "total_won": 10.0
  },
  "history": [
    {
      "spin_id": 1,
      "spin_index": 1,
      "outcome_type": "win",
      "winner_label": "£10",
      "prize_id": "prize_1",
      "prize_cost": 10.0,
      "target_angle": 45.5,
      "segment_index": 5,
      "jackpot_won": null,
      "ticket_number": 12345,
      "ticket_code": "012345",
      "created_at": "2026-04-06T18:07:56.813Z"
    }
  ]
}
```

### Сценарий 4: Получить статистику

```bash
curl http://localhost:8000/wheels/1/stats \
  -H "X-API-Key: test-api-key-12345"
```

## Проверка исправлений

### 1. Cookies (SameSite=None)
- Откройте DevTools (F12)
- Перейдите на вкладку Application → Cookies
- Проверьте что cookies `tmwl_wallet_email`, `tmwl_player_email`, `tmwl_wallet_order` имеют `SameSite=None`

### 2. RNG Ticket Numbers
- Выполните спин
- Проверьте что `ticket_number` в ответе API содержит реальное число (не 00001)
- Проверьте что в истории спинов отображается правильный номер билета

### 3. RNG Animation
- Откройте DevTools (F12)
- Перейдите на вкладку Console
- Выполните спин
- Проверьте что нет ошибок в консоли
- Проверьте что RNG анимация работает корректно

## Логи

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

## Остановка

```bash
docker-compose down
```

## Очистка данных

```bash
docker-compose down -v
```

## Troubleshooting

### Backend не запускается
```bash
docker-compose logs backend
```

### Port уже занят
```bash
# Найти процесс на порту 8000
lsof -i :8000

# Найти процесс на порту 3000
lsof -i :3000

# Убить процесс
kill -9 <PID>
```

### Пересобрать образы
```bash
docker-compose build --no-cache
docker-compose up -d
```

## Контакты

Для вопросов и проблем обратитесь к разработчику.
