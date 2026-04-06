# 🚀 QUICK START GUIDE

## Запуск всей системы (Frontend + Backend)

### Windows
```bash
start-full.bat
```

### Linux/Mac
```bash
bash start-full.sh
```

## ✅ Что будет запущено

- **Frontend** на http://localhost:3000
- **Backend API** на http://localhost:8000
- **Admin Panel** на http://localhost:8000/admin.html
- **SQLite Database** в `server/data/wheelv1.db`

## 📍 Доступные сервисы

| Сервис | URL | Описание |
|--------|-----|---------|
| Frontend | http://localhost:3000 | Тестовая панель управления |
| Backend API | http://localhost:8000 | REST API endpoints |
| Admin Panel | http://localhost:8000/admin.html | Админ интерфейс |
| Health Check | http://localhost:8000/health | Проверка статуса |

## 🔐 Учетные данные

```
Admin Username: admin
Admin Password: admin123
API Key: test-api-key-12345
```

## 📊 Тестирование

### 1. Проверить что всё работает
Откройте в браузере: http://localhost:3000

Вы должны увидеть:
- ✓ Backend is online (зеленый статус)
- Форму для тестирования API

### 2. Выполнить спин
1. Введите Wheel ID: `1`
2. Введите Player ID: `player_123`
3. Введите Player Name: `Test Player`
4. Нажмите кнопку "Perform Spin"

Вы должны получить ответ с:
- `ticket_number` (реальное число, не 00001)
- `ticket_code` (отформатированный код)
- Информацию о выигрыше/проигрыше

### 3. Проверить cookies
1. Откройте DevTools (F12)
2. Перейдите на вкладку Application → Cookies
3. Проверьте что cookies имеют `SameSite=None`

## 🛠️ Полезные команды

```bash
# Просмотреть все логи
docker-compose logs -f

# Просмотреть логи backend
docker-compose logs -f backend

# Просмотреть логи frontend
docker-compose logs -f frontend

# Остановить контейнеры
docker-compose down

# Перезагрузить контейнеры
docker-compose restart

# Очистить всё (включая БД)
docker-compose down -v

# Пересобрать образы
docker-compose build --no-cache
docker-compose up -d
```

## 📚 Документация

- **README.md** - Основная документация
- **TESTING.md** - Сценарии тестирования
- **SUMMARY.md** - Полная информация о проекте
- **CHECKLIST.md** - Чек-лист исправлений

## 🔍 Проверка исправлений

### Cookies (SameSite=None)
✅ Исправлено в:
- wheel.js (4 места)
- wheel_v2.js (4 места)
- wallet.php (1 место)

### RNG Ticket Numbers
✅ Исправлено в:
- wheel.js (добавлены 3 строки)
- wheel_v2.js (добавлены 3 строки)

## 🐛 Troubleshooting

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

# Убить процесс (Linux/Mac)
kill -9 <PID>
```

### Пересобрать всё с нуля
```bash
docker-compose down -v
docker-compose build --no-cache
docker-compose up -d
```

## 📞 Контакты

Для вопросов обратитесь к разработчику.

---

**Готово к использованию! 🎉**

Все баги исправлены и система готова к тестированию.
