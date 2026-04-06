# 🚀 DEPLOYMENT GUIDE

## 📦 Что готово к деплою

Все исправленные файлы находятся в папке:
```
C:\tmp\prod\prodactions\
```

## 🎯 Для деплоя на WordPress

### Шаг 1: Скопировать плагин
Скопируйте папку `prodactions/` в директорию плагинов WordPress:
```
/wp-content/plugins/prodactions/
```

### Шаг 2: Активировать плагин
1. Перейдите в WordPress Admin
2. Plugins → Installed Plugins
3. Найдите "TM Wheel Engine"
4. Нажмите "Activate"

### Шаг 3: Настроить плагин
1. Перейдите в Settings → TM Wheel
2. Укажите Engine Base URL
3. Сохраните настройки

## 🐳 Для деплоя в Docker

### Шаг 1: Подготовить сервер
```bash
# Установить Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Установить Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### Шаг 2: Скопировать файлы
```bash
# Скопировать весь проект на сервер
scp -r C:\tmp\prod user@server:/home/user/wheel-fortune
```

### Шаг 3: Запустить контейнеры
```bash
cd /home/user/wheel-fortune
docker-compose up -d
```

### Шаг 4: Проверить статус
```bash
docker-compose ps
docker-compose logs -f
```

## 📋 Файлы для деплоя

### Обязательные файлы
- ✅ `prodactions/` - Весь плагин WordPress
- ✅ `server/` - Backend API
- ✅ `docker-compose.yml` - Конфигурация контейнеров
- ✅ `nginx.conf` - Конфигурация веб-сервера
- ✅ `.env` - Переменные окружения

### Опциональные файлы
- 📄 `README.md` - Документация
- 📄 `TESTING.md` - Сценарии тестирования
- 📄 `QUICKSTART.md` - Быстрый старт
- 📄 `STATUS.md` - Статус проекта

## 🔧 Конфигурация

### .env файл
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

### Для production
Измените значения в `.env`:
```env
WHEELV1_ADMIN_PASSWORD=<strong-password>
API_KEY=<secure-api-key>
BACKEND_URL=https://api.yourdomain.com
FRONTEND_URL=https://yourdomain.com
```

## 🔐 Security Checklist

- [ ] Изменить пароль admin
- [ ] Изменить API Key
- [ ] Включить HTTPS
- [ ] Настроить firewall
- [ ] Настроить backup БД
- [ ] Настроить логирование
- [ ] Настроить мониторинг

## 📊 Мониторинг

### Проверить логи
```bash
docker-compose logs -f backend
docker-compose logs -f frontend
```

### Проверить статус
```bash
curl http://localhost:8000/health
```

### Проверить БД
```bash
sqlite3 server/data/wheelv1.db ".tables"
```

## 🆘 Troubleshooting

### Backend не запускается
```bash
docker-compose logs backend
docker-compose restart backend
```

### Port занят
```bash
# Linux/Mac
lsof -i :8000
kill -9 <PID>

# Windows
netstat -ano | findstr :8000
taskkill /PID <PID> /F
```

### Очистить всё
```bash
docker-compose down -v
docker-compose build --no-cache
docker-compose up -d
```

## 📞 Support

Для вопросов обратитесь к разработчику.

---

**Готово к деплою! 🎉**

Все баги исправлены и система готова к использованию в production.

*Generated: 2026-04-06 18:09:58 UTC*
