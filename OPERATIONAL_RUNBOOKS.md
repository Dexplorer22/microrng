# Операционные Runbooks и документация

**Проект:** Prize Competitions Platform  
**Масштаб:** Enterprise (100K+ пользователей)  
**Дата:** 2026-04-06

---

## 1. Runbook: Развёртывание нового сервиса

### Предусловия
- Доступ к GitHub репозиторию
- Доступ к AWS/GCP/Azure
- Доступ к Kubernetes кластеру
- Доступ к Docker Registry

### Шаги

#### 1.1 Создание структуры сервиса
```bash
# Клонировать репозиторий
git clone https://github.com/your-org/gaming-platform.git
cd gaming-platform

# Создать новый сервис из шаблона
mkdir -p services/new-service
cp -r services/template/* services/new-service/

# Обновить package.json
cd services/new-service
npm init -y
npm install express typescript jest @types/node

# Создать структуру папок
mkdir -p src/{controllers,services,repositories,models,middleware,routes,config,utils,types}
mkdir -p tests/{unit,integration}
mkdir -p migrations
```

#### 1.2 Реализация сервиса
```bash
# Создать основные файлы
touch src/app.ts
touch src/server.ts
touch Dockerfile
touch docker-compose.yml
touch .env.example

# Инициализировать TypeScript
npx tsc --init

# Создать ESLint конфиг
npx eslint --init
```

#### 1.3 Добавление в Docker Compose
```yaml
# docker-compose.yml
new-service:
  build:
    context: ./services/new-service
    dockerfile: Dockerfile
  ports:
    - "3005:3000"
  environment:
    - DATABASE_URL=postgresql://gaming:gaming123@postgres:5432/gaming_db
    - REDIS_URL=redis://redis:6379
  depends_on:
    - postgres
    - redis
  networks:
    - gaming-network
```

#### 1.4 Создание Kubernetes манифестов
```yaml
# infrastructure/kubernetes/new-service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: new-service
  namespace: gaming-platform
spec:
  replicas: 3
  selector:
    matchLabels:
      app: new-service
  template:
    metadata:
      labels:
        app: new-service
    spec:
      containers:
      - name: new-service
        image: ghcr.io/your-org/gaming-platform/new-service:latest
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: redis-url
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
```

#### 1.5 Добавление в CI/CD pipeline
```yaml
# .github/workflows/new-service-deploy.yml
name: Deploy New Service

on:
  push:
    paths:
      - 'services/new-service/**'
    branches: [main, dev]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build and push Docker image
        run: |
          docker build -t ghcr.io/your-org/gaming-platform/new-service:latest services/new-service/
          docker push ghcr.io/your-org/gaming-platform/new-service:latest
      - name: Deploy to Kubernetes
        run: |
          kubectl apply -f infrastructure/kubernetes/new-service-deployment.yaml
```

#### 1.6 Проверка развёртывания
```bash
# Проверить статус пода
kubectl get pods -n gaming-platform | grep new-service

# Проверить логи
kubectl logs -f deployment/new-service -n gaming-platform

# Проверить health endpoint
curl http://localhost:3000/health

# Проверить готовность
kubectl get deployment new-service -n gaming-platform
```

---

## 2. Runbook: Инцидент - высокая ошибка rate

### Симптомы
- Error rate > 5%
- Alerts в Slack/PagerDuty
- Пользователи жалуются на ошибки

### Диагностика

#### Шаг 1: Проверить метрики
```bash
# Проверить Prometheus
curl http://prometheus:9090/api/v1/query?query=rate(http_requests_total{status=~"5.."}[5m])

# Проверить Grafana dashboard
# Открыть: http://grafana:3000/d/gaming-platform

# Проверить какой сервис вызывает ошибки
kubectl top pods -n gaming-platform
```

#### Шаг 2: Проверить логи
```bash
# Получить логи последних 100 строк
kubectl logs -n gaming-platform deployment/problematic-service --tail=100

# Получить логи с фильтром по ошибкам
kubectl logs -n gaming-platform deployment/problematic-service | grep ERROR

# Получить логи в JSON формате
kubectl logs -n gaming-platform deployment/problematic-service -o json | jq '.log' | grep ERROR
```

#### Шаг 3: Проверить ресурсы
```bash
# Проверить использование CPU и памяти
kubectl top pods -n gaming-platform

# Проверить лимиты ресурсов
kubectl describe pod <pod-name> -n gaming-platform

# Проверить дисковое пространство
kubectl exec -it <pod-name> -n gaming-platform -- df -h
```

#### Шаг 4: Проверить зависимости
```bash
# Проверить статус базы данных
kubectl exec -it postgres-0 -n gaming-platform -- pg_isready

# Проверить статус Redis
kubectl exec -it redis-0 -n gaming-platform -- redis-cli ping

# Проверить статус RabbitMQ
kubectl exec -it rabbitmq-0 -n gaming-platform -- rabbitmq-diagnostics -q ping
```

### Решение

#### Вариант 1: Перезагрузить сервис
```bash
# Перезагрузить deployment
kubectl rollout restart deployment/problematic-service -n gaming-platform

# Проверить статус
kubectl rollout status deployment/problematic-service -n gaming-platform
```

#### Вариант 2: Масштабировать сервис
```bash
# Увеличить количество реплик
kubectl scale deployment problematic-service --replicas=5 -n gaming-platform

# Проверить статус
kubectl get pods -n gaming-platform | grep problematic-service
```

#### Вариант 3: Откатить последний деплой
```bash
# Получить историю деплоев
kubectl rollout history deployment/problematic-service -n gaming-platform

# Откатить на предыдущую версию
kubectl rollout undo deployment/problematic-service -n gaming-platform

# Проверить статус
kubectl rollout status deployment/problematic-service -n gaming-platform
```

#### Вариант 4: Перезагрузить зависимость
```bash
# Перезагрузить PostgreSQL
kubectl rollout restart statefulset/postgres -n gaming-platform

# Перезагрузить Redis
kubectl rollout restart statefulset/redis -n gaming-platform

# Перезагрузить RabbitMQ
kubectl rollout restart statefulset/rabbitmq -n gaming-platform
```

### Post-Incident

```bash
# Собрать логи для анализа
kubectl logs deployment/problematic-service -n gaming-platform > incident-logs.txt

# Создать incident report
# Включить: время начала, симптомы, диагностика, решение, время восстановления

# Обновить runbook на основе опыта
# Добавить новые checks если необходимо
```

---

## 3. Runbook: Обновление базы данных

### Предусловия
- Backup текущей БД
- Maintenance window запланирован
- Миграция протестирована на staging

### Шаги

#### 1. Создать backup
```bash
# Создать backup PostgreSQL
kubectl exec -it postgres-0 -n gaming-platform -- \
  pg_dump -U gaming gaming_db > backup-$(date +%Y%m%d-%H%M%S).sql

# Проверить размер backup
ls -lh backup-*.sql

# Загрузить backup в S3
aws s3 cp backup-*.sql s3://gaming-platform-backups/
```

#### 2. Запустить миграцию
```bash
# Применить миграцию
kubectl exec -it postgres-0 -n gaming-platform -- \
  psql -U gaming -d gaming_db -f /migrations/001_new_schema.sql

# Проверить результат
kubectl exec -it postgres-0 -n gaming-platform -- \
  psql -U gaming -d gaming_db -c "\dt"
```

#### 3. Проверить целостность
```bash
# Проверить количество записей
kubectl exec -it postgres-0 -n gaming-platform -- \
  psql -U gaming -d gaming_db -c "SELECT COUNT(*) FROM users;"

# Проверить индексы
kubectl exec -it postgres-0 -n gaming-platform -- \
  psql -U gaming -d gaming_db -c "\di"

# Проверить constraints
kubectl exec -it postgres-0 -n gaming-platform -- \
  psql -U gaming -d gaming_db -c "\d users"
```

#### 4. Откатить если необходимо
```bash
# Восстановить из backup
kubectl exec -it postgres-0 -n gaming-platform -- \
  psql -U gaming -d gaming_db < backup-*.sql

# Проверить восстановление
kubectl exec -it postgres-0 -n gaming-platform -- \
  psql -U gaming -d gaming_db -c "SELECT COUNT(*) FROM users;"
```

---

## 4. Runbook: Масштабирование под нагрузку

### Когда масштабировать
- CPU usage > 80%
- Memory usage > 85%
- Request latency p95 > 500ms
- Queue depth > 1000

### Шаги

#### 1. Горизонтальное масштабирование (Pods)
```bash
# Увеличить количество реплик
kubectl scale deployment game-service --replicas=10 -n gaming-platform

# Проверить статус
kubectl get pods -n gaming-platform | grep game-service

# Проверить load balancing
kubectl get svc game-service -n gaming-platform
```

#### 2. Вертикальное масштабирование (Resources)
```bash
# Обновить лимиты ресурсов
kubectl set resources deployment game-service \
  --limits=cpu=1000m,memory=1Gi \
  --requests=cpu=500m,memory=512Mi \
  -n gaming-platform

# Проверить обновление
kubectl describe deployment game-service -n gaming-platform
```

#### 3. Масштабирование БД
```bash
# Увеличить размер диска PostgreSQL
kubectl patch pvc postgres-data -p '{"spec":{"resources":{"requests":{"storage":"500Gi"}}}}' -n gaming-platform

# Добавить read replicas
kubectl apply -f infrastructure/kubernetes/postgres-replica.yaml

# Проверить репликацию
kubectl exec -it postgres-0 -n gaming-platform -- \
  psql -U gaming -d gaming_db -c "SELECT * FROM pg_stat_replication;"
```

#### 4. Масштабирование кэша
```bash
# Увеличить Redis memory
kubectl set env statefulset/redis REDIS_MEMORY=4gb -n gaming-platform

# Добавить Redis cluster nodes
kubectl scale statefulset redis --replicas=6 -n gaming-platform

# Проверить кластер
kubectl exec -it redis-0 -n gaming-platform -- redis-cli cluster info
```

#### 5. Мониторинг после масштабирования
```bash
# Проверить метрики
kubectl top pods -n gaming-platform

# Проверить latency
curl http://prometheus:9090/api/v1/query?query=histogram_quantile(0.95,http_request_duration_seconds)

# Проверить error rate
curl http://prometheus:9090/api/v1/query?query=rate(http_requests_total{status=~"5.."}[5m])
```

---

## 5. Runbook: Откат деплоя

### Когда откатывать
- Critical bugs в production
- Performance degradation
- Data corruption
- Security vulnerability

### Шаги

#### 1. Быстрый откат (Blue-Green)
```bash
# Переключить трафик на синюю версию
kubectl patch service game-service \
  -p '{"spec":{"selector":{"version":"blue"}}}' \
  -n gaming-platform

# Проверить трафик
kubectl get endpoints game-service -n gaming-platform

# Проверить health
curl http://api.gaming-platform.com/health
```

#### 2. Откат через Kubernetes
```bash
# Получить историю деплоев
kubectl rollout history deployment/game-service -n gaming-platform

# Откатить на предыдущую версию
kubectl rollout undo deployment/game-service -n gaming-platform

# Откатить на конкретную версию
kubectl rollout undo deployment/game-service --to-revision=5 -n gaming-platform

# Проверить статус
kubectl rollout status deployment/game-service -n gaming-platform
```

#### 3. Откат БД (если необходимо)
```bash
# Восстановить из backup
kubectl exec -it postgres-0 -n gaming-platform -- \
  psql -U gaming -d gaming_db < backup-pre-deploy.sql

# Проверить целостность
kubectl exec -it postgres-0 -n gaming-platform -- \
  psql -U gaming -d gaming_db -c "SELECT COUNT(*) FROM users;"
```

#### 4. Коммуникация
```bash
# Уведомить команду в Slack
curl -X POST https://hooks.slack.com/services/YOUR/WEBHOOK \
  -d '{"text":"Deployment rolled back due to critical bug"}'

# Создать incident в PagerDuty
curl -X POST https://events.pagerduty.com/v2/enqueue \
  -d '{"routing_key":"YOUR_KEY","event_action":"trigger"}'
```

---

## 6. Runbook: Обновление зависимостей

### Когда обновлять
- Security patches
- Bug fixes
- Performance improvements
- Quarterly updates

### Шаги

#### 1. Проверить обновления
```bash
# Проверить доступные обновления
npm outdated

# Проверить security vulnerabilities
npm audit

# Проверить Snyk
snyk test
```

#### 2. Обновить зависимости
```bash
# Обновить patch версии
npm update

# Обновить minor версии
npm upgrade

# Обновить specific пакет
npm install package-name@latest
```

#### 3. Тестирование
```bash
# Запустить unit tests
npm run test:unit

# Запустить integration tests
npm run test:integration

# Запустить E2E tests
npm run test:e2e

# Проверить coverage
npm run test:coverage
```

#### 4. Развёртывание
```bash
# Создать PR с обновлениями
git checkout -b deps/update-dependencies
git add package.json package-lock.json
git commit -m "chore(deps): update dependencies"
git push origin deps/update-dependencies

# Создать PR
gh pr create --title "Update dependencies" --body "Security and performance updates"

# После approval, merge и deploy
git merge deps/update-dependencies
git push origin main
```

---

## 7. Контрольный список для production

### Pre-Deployment
- [ ] Все тесты проходят (unit, integration, E2E)
- [ ] Code review одобрен (2+ approvals)
- [ ] Security scan пройден
- [ ] Performance tests в норме
- [ ] Database migrations протестированы
- [ ] Backup создан
- [ ] Rollback plan готов
- [ ] Команда уведомлена

### During Deployment
- [ ] Мониторинг активен
- [ ] Логи собираются
- [ ] Alerts настроены
- [ ] Smoke tests запущены
- [ ] Health checks проходят
- [ ] Трафик распределяется корректно

### Post-Deployment
- [ ] Error rate < 0.1%
- [ ] Latency p95 < 500ms
- [ ] Database replication в норме
- [ ] Cache hit rate > 80%
- [ ] No critical alerts
- [ ] User feedback positive

---

## 8. Контакты и эскалация

### Уровень 1: Team Lead
- Время ответа: 15 минут
- Решает: Обычные проблемы, масштабирование
- Контакт: Slack #engineering

### Уровень 2: Engineering Lead (cs-engineering-lead)
- Время ответа: 5 минут
- Решает: Critical issues, архитектурные проблемы
- Контакт: PagerDuty, Slack @engineering-lead

### Уровень 3: CTO (cs-cto-advisor)
- Время ответа: 2 минуты
- Решает: Major incidents, business impact
- Контакт: PagerDuty, Phone

---

## 9. Полезные команды

```bash
# Kubernetes
kubectl get pods -n gaming-platform
kubectl logs -f deployment/service-name -n gaming-platform
kubectl describe pod <pod-name> -n gaming-platform
kubectl exec -it <pod-name> -n gaming-platform -- /bin/bash

# Docker
docker ps
docker logs <container-id>
docker exec -it <container-id> /bin/bash

# Database
psql -U gaming -d gaming_db -c "SELECT * FROM users LIMIT 10;"
redis-cli ping
rabbitmq-diagnostics -q ping

# Monitoring
curl http://prometheus:9090/api/v1/query?query=up
curl http://grafana:3000/api/health
```

---

**Статус:** Ready for Operations  
**Последнее обновление:** 2026-04-06  
**Версия:** 1.0
