# FULLSTACK ARCHITECT - МИКРОСЕРВИСНАЯ АРХИТЕКТУРА

**Дата создания:** 2026-04-06  
**Версия:** 1.0.0  
**Уровень:** Advanced  
**Фокус:** Микросервисы, масштабируемость, production-ready

---

## 📑 СОДЕРЖАНИЕ

1. [Основы микросервисной архитектуры](#основы-микросервисной-архитектуры)
2. [Паттерны проектирования](#паттерны-проектирования)
3. [Декомпозиция системы](#декомпозиция-системы)
4. [Коммуникация между сервисами](#коммуникация-между-сервисами)
5. [Data Management](#data-management)
6. [Resilience & Fault Tolerance](#resilience--fault-tolerance)
7. [Monitoring & Observability](#monitoring--observability)
8. [Deployment & Scaling](#deployment--scaling)
9. [Security в микросервисах](#security-в-микросервисах)
10. [Case Study: Wheel Fortune](#case-study-wheel-fortune)

---

## 🏗️ ОСНОВЫ МИКРОСЕРВИСНОЙ АРХИТЕКТУРЫ

### Определение

Микросервисная архитектура - это подход к разработке системы как набора небольших, независимых сервисов, которые:
- Работают в отдельных процессах
- Взаимодействуют через четко определенные API
- Могут быть развернуты независимо
- Организованы вокруг бизнес-возможностей

### Преимущества

✅ **Масштабируемость**
- Масштабировать отдельные сервисы независимо
- Оптимизировать ресурсы для каждого сервиса

✅ **Гибкость**
- Выбирать технологический стек для каждого сервиса
- Обновлять сервисы независимо

✅ **Надежность**
- Отказ одного сервиса не влияет на другие
- Легче изолировать проблемы

✅ **Скорость разработки**
- Разные команды работают на разных сервисах
- Быстрее развертывать изменения

### Недостатки

❌ **Сложность**
- Распределенная система сложнее для отладки
- Нужна хорошая инфраструктура

❌ **Сетевая задержка**
- Коммуникация между сервисами медленнее
- Нужна обработка сетевых ошибок

❌ **Консистентность данных**
- Сложнее обеспечить ACID транзакции
- Нужна eventual consistency

---

## 🎯 ПАТТЕРНЫ ПРОЕКТИРОВАНИЯ

### 1. Service Per Team (Конвей's Law)

```
Организационная структура → Архитектура системы

Team A (Auth)          → Auth Service
Team B (Payments)      → Payment Service
Team C (Analytics)     → Analytics Service
```

**Преимущества:**
- Четкая ответственность
- Минимальные зависимости между командами
- Быстрое развитие

### 2. Domain-Driven Design (DDD)

```
Bounded Contexts:

┌─────────────────────┐
│   User Context      │
│  - Registration     │
│  - Profile          │
│  - Preferences      │
└─────────────────────┘

┌─────────────────────┐
│  Order Context      │
│  - Order Creation   │
│  - Order Status     │
│  - Fulfillment      │
└─────────────────────┘

┌─────────────────────┐
│ Payment Context     │
│  - Payment Process  │
│  - Refunds          │
│  - Reconciliation   │
└─────────────────────┘
```

### 3. API Gateway Pattern

```
┌──────────────────────────────────────┐
│         Client Applications          │
└──────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────┐
│         API Gateway                  │
│  - Routing                           │
│  - Authentication                    │
│  - Rate Limiting                     │
│  - Request/Response Transform        │
└──────────────────────────────────────┘
        │        │        │
        ▼        ▼        ▼
    ┌────┐  ┌────┐  ┌────┐
    │ S1 │  │ S2 │  │ S3 │
    └────┘  └────┘  └────┘
```

### 4. Service Discovery

```
┌─────────────────────────────────┐
│   Service Registry              │
│  - Service A: 192.168.1.10:8001 │
│  - Service B: 192.168.1.11:8002 │
│  - Service C: 192.168.1.12:8003 │
└─────────────────────────────────┘
        ▲           ▲
        │           │
    Register    Discover
        │           │
    ┌───┴───┐   ┌───┴───┐
    │ Svc A │   │Client │
    └───────┘   └───────┘
```

### 5. Circuit Breaker Pattern

```
States:
┌─────────┐
│ CLOSED  │ ──(failure threshold)──> ┌─────────┐
│ Normal  │                          │  OPEN   │
│ traffic │ <──(timeout)──────────── │ Reject  │
└─────────┘                          │ requests│
    ▲                                └─────────┘
    │                                    │
    └────(success)─────────────────────┌─────────┐
                                       │ HALF-   │
                                       │ OPEN    │
                                       │ Test    │
                                       └─────────┘
```

---

## 🔄 ДЕКОМПОЗИЦИЯ СИСТЕМЫ

### Шаг 1: Идентифицировать Bounded Contexts

```
Wheel Fortune Микросервисы:

1. Auth Service
   - User registration
   - Login/Logout
   - Token management
   - Permissions

2. Game Service
   - Spin execution
   - Prize allocation
   - RNG generation
   - Game state

3. Wallet Service
   - Balance management
   - Transaction history
   - Payouts
   - Refunds

4. Analytics Service
   - Statistics collection
   - Reporting
   - Metrics
   - Dashboards

5. Notification Service
   - Email notifications
   - SMS alerts
   - Push notifications
   - Webhooks
```

### Шаг 2: Определить API контракты

```
Auth Service API:
POST   /auth/login
POST   /auth/register
POST   /auth/refresh
GET    /auth/verify

Game Service API:
POST   /games/{id}/spin
GET    /games/{id}/state
GET    /games/{id}/history
POST   /games/{id}/reset

Wallet Service API:
GET    /wallet/{user_id}
POST   /wallet/{user_id}/payout
GET    /wallet/{user_id}/transactions
POST   /wallet/{user_id}/refund

Analytics Service API:
GET    /analytics/stats
GET    /analytics/reports
GET    /analytics/metrics
POST   /analytics/events

Notification Service API:
POST   /notifications/email
POST   /notifications/sms
POST   /notifications/push
GET    /notifications/history
```

### Шаг 3: Определить зависимости

```
Dependency Graph:

Client
  ├─> API Gateway
  │    ├─> Auth Service
  │    ├─> Game Service
  │    │    ├─> Wallet Service
  │    │    └─> Analytics Service
  │    ├─> Wallet Service
  │    ├─> Analytics Service
  │    └─> Notification Service
  │         ├─> Auth Service (verify user)
  │         └─> Wallet Service (get balance)
```

---

## 📡 КОММУНИКАЦИЯ МЕЖДУ СЕРВИСАМИ

### 1. Synchronous Communication (REST/gRPC)

**REST:**
```
Service A → HTTP Request → Service B
           ← HTTP Response ←
```

**Преимущества:**
- Простая реализация
- Легко отладить
- Стандартный протокол

**Недостатки:**
- Блокирующие вызовы
- Сетевые задержки
- Сложнее обрабатывать ошибки

**Пример:**
```python
# Service A вызывает Service B
response = requests.get('http://service-b:8002/api/data')
data = response.json()
```

### 2. Asynchronous Communication (Message Queue)

**Architecture:**
```
Service A → Message Queue → Service B
           (RabbitMQ/Kafka)
```

**Преимущества:**
- Неблокирующие вызовы
- Лучше масштабируется
- Легче обрабатывать нагрузку

**Недостатки:**
- Сложнее отладить
- Eventual consistency
- Нужна инфраструктура

**Пример:**
```python
# Service A публикует событие
producer.send('order.created', {
    'order_id': 123,
    'user_id': 456,
    'amount': 100
})

# Service B подписывается на событие
@consumer.on('order.created')
def handle_order_created(event):
    process_order(event)
```

### 3. Event-Driven Architecture

```
┌──────────────┐
│ Order Service│
└──────┬───────┘
       │ publishes
       ▼
   order.created
       │
       ├─> Payment Service (process payment)
       ├─> Notification Service (send email)
       ├─> Analytics Service (track event)
       └─> Inventory Service (update stock)
```

---

## 💾 DATA MANAGEMENT

### 1. Database Per Service

```
Auth Service
  └─> PostgreSQL (users, tokens)

Game Service
  └─> MongoDB (game states, spins)

Wallet Service
  └─> PostgreSQL (transactions, balance)

Analytics Service
  └─> ClickHouse (time-series data)
```

**Преимущества:**
- Выбирать БД для каждого сервиса
- Масштабировать независимо
- Изолировать данные

**Недостатки:**
- Сложнее обеспечить консистентность
- Нужна синхронизация данных
- Сложнее делать JOIN'ы

### 2. Saga Pattern (Distributed Transactions)

```
Сценарий: Создание заказа

1. Order Service: Create order (PENDING)
2. Payment Service: Process payment
   ├─ Success → Order Service: Update to PAID
   └─ Failure → Order Service: Compensate (CANCELLED)
3. Inventory Service: Reserve items
   ├─ Success → Order Service: Update to RESERVED
   └─ Failure → Payment Service: Refund (Compensate)
4. Notification Service: Send confirmation
```

### 3. Event Sourcing

```
Events Log:
1. order.created (2026-04-06 10:00:00)
2. payment.processed (2026-04-06 10:01:00)
3. order.confirmed (2026-04-06 10:02:00)
4. order.shipped (2026-04-06 10:05:00)

Current State (reconstructed from events):
{
  order_id: 123,
  status: "shipped",
  total: 100,
  created_at: "2026-04-06 10:00:00"
}
```

---

## 🛡️ RESILIENCE & FAULT TOLERANCE

### 1. Retry Logic

```python
def call_service_with_retry(url, max_retries=3):
    for attempt in range(max_retries):
        try:
            response = requests.get(url, timeout=5)
            return response
        except requests.RequestException as e:
            if attempt < max_retries - 1:
                wait_time = 2 ** attempt  # Exponential backoff
                time.sleep(wait_time)
            else:
                raise
```

### 2. Timeout Management

```python
# Set timeouts to prevent hanging
response = requests.get(
    'http://service-b:8002/api/data',
    timeout=5  # 5 seconds
)
```

### 3. Bulkhead Pattern

```
┌─────────────────────────────────┐
│      Service A                  │
├─────────────┬───────────────────┤
│ Thread Pool │ Thread Pool       │
│ for Service │ for Service       │
│ B (10 thr)  │ C (10 thr)        │
└─────────────┴───────────────────┘

Изолирует ресурсы для разных сервисов
```

### 4. Health Checks

```python
@app.get("/health")
def health_check():
    return {
        "status": "healthy",
        "timestamp": datetime.now(),
        "dependencies": {
            "database": check_db(),
            "cache": check_cache(),
            "external_api": check_external_api()
        }
    }
```

---

## 📊 MONITORING & OBSERVABILITY

### 1. Logging

```python
import logging

logger = logging.getLogger(__name__)

logger.info("User logged in", extra={
    "user_id": 123,
    "timestamp": datetime.now(),
    "service": "auth-service"
})

logger.error("Payment failed", extra={
    "order_id": 456,
    "error": str(e),
    "service": "payment-service"
})
```

### 2. Metrics

```python
from prometheus_client import Counter, Histogram

# Counter
requests_total = Counter(
    'requests_total',
    'Total requests',
    ['method', 'endpoint']
)

# Histogram
request_duration = Histogram(
    'request_duration_seconds',
    'Request duration',
    ['endpoint']
)

@app.get("/api/data")
def get_data():
    requests_total.labels(method='GET', endpoint='/api/data').inc()
    with request_duration.labels(endpoint='/api/data').time():
        return {"data": "..."}
```

### 3. Distributed Tracing

```python
from opentelemetry import trace

tracer = trace.get_tracer(__name__)

@app.get("/api/order/{order_id}")
def get_order(order_id):
    with tracer.start_as_current_span("get_order") as span:
        span.set_attribute("order_id", order_id)
        
        # Call other services
        with tracer.start_as_current_span("call_payment_service"):
            payment_info = call_payment_service(order_id)
        
        with tracer.start_as_current_span("call_wallet_service"):
            wallet_info = call_wallet_service(order_id)
        
        return {
            "order_id": order_id,
            "payment": payment_info,
            "wallet": wallet_info
        }
```

---

## 🚀 DEPLOYMENT & SCALING

### 1. Container Orchestration (Kubernetes)

```yaml
# Auth Service Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: auth-service
  template:
    metadata:
      labels:
        app: auth-service
    spec:
      containers:
      - name: auth-service
        image: auth-service:1.0.0
        ports:
        - containerPort: 8001
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8001
          initialDelaySeconds: 30
          periodSeconds: 10
```

### 2. Auto-Scaling

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: auth-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: auth-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### 3. Service Mesh (Istio)

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: auth-service
spec:
  hosts:
  - auth-service
  http:
  - match:
    - uri:
        prefix: "/api/v1"
    route:
    - destination:
        host: auth-service
        port:
          number: 8001
        subset: v1
      weight: 90
    - destination:
        host: auth-service
        port:
          number: 8001
        subset: v2
      weight: 10
```

---

## 🔒 SECURITY В МИКРОСЕРВИСАХ

### 1. Service-to-Service Authentication

```python
# Using mTLS (mutual TLS)
import ssl

ssl_context = ssl.create_default_context()
ssl_context.load_cert_chain(
    certfile='service.crt',
    keyfile='service.key',
    password=b'password'
)
ssl_context.load_verify_locations('ca.crt')

response = requests.get(
    'https://service-b:8002/api/data',
    cert=('service.crt', 'service.key'),
    verify='ca.crt'
)
```

### 2. API Gateway Authentication

```python
from fastapi import Depends, HTTPException
from fastapi.security import HTTPBearer

security = HTTPBearer()

async def verify_token(credentials = Depends(security)):
    token = credentials.credentials
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        return payload
    except jwt.InvalidTokenError:
        raise HTTPException(status_code=401, detail="Invalid token")

@app.get("/api/protected")
async def protected_route(user = Depends(verify_token)):
    return {"user": user}
```

### 3. Rate Limiting

```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@app.get("/api/data")
@limiter.limit("100/minute")
async def get_data(request: Request):
    return {"data": "..."}
```

---

## 📚 CASE STUDY: WHEEL FORTUNE

### Текущая архитектура (Монолит)

```
┌─────────────────────────────────┐
│      Wheel Fortune              │
│  (Single FastAPI Application)   │
├─────────────────────────────────┤
│ - Auth                          │
│ - Game Logic                    │
│ - Wallet Management             │
│ - Analytics                     │
│ - Notifications                 │
└─────────────────────────────────┘
         │
         ▼
    SQLite DB
```

### Микросервисная архитектура (Рекомендуемая)

```
┌──────────────────────────────────────────────────────┐
│              API Gateway (Kong/Nginx)                │
│  - Routing, Auth, Rate Limiting, Load Balancing     │
└──────────────────────────────────────────────────────┘
    │         │         │         │         │
    ▼         ▼         ▼         ▼         ▼
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│ Auth   │ │ Game   │ │Wallet  │ │Analytics│ │Notif   │
│Service │ │Service │ │Service │ │Service  │ │Service │
└────────┘ └────────┘ └────────┘ └────────┘ └────────┘
    │         │         │         │         │
    ▼         ▼         ▼         ▼         ▼
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│ PG DB  │ │ Mongo  │ │ PG DB  │ │ClickH  │ │ PG DB  │
└────────┘ └────────┘ └────────┘ └────────┘ └────────┘

Message Queue (RabbitMQ/Kafka):
- order.created
- payment.processed
- spin.completed
- user.registered
```

### Миграционный путь

**Фаза 1: Подготовка (1-2 недели)**
- Выделить Auth Service
- Создать API Gateway
- Настроить Service Discovery

**Фаза 2: Миграция Game Logic (2-3 недели)**
- Выделить Game Service
- Настроить коммуникацию с Auth Service
- Миграция данных

**Фаза 3: Миграция Wallet (2-3 недели)**
- Выделить Wallet Service
- Настроить Event-Driven коммуникацию
- Реализовать Saga Pattern

**Фаза 4: Analytics & Notifications (1-2 недели)**
- Выделить Analytics Service
- Выделить Notification Service
- Настроить Event Sourcing

**Фаза 5: Оптимизация (Ongoing)**
- Мониторинг и логирование
- Масштабирование
- Оптимизация производительности

---

## 🎓 BEST PRACTICES

### 1. API Versioning

```
/api/v1/users
/api/v2/users (with new fields)
```

### 2. Backward Compatibility

```python
# Old clients still work
@app.get("/api/v1/users/{user_id}")
def get_user_v1(user_id: int):
    user = get_user(user_id)
    return {
        "id": user.id,
        "name": user.name
    }

# New clients get more data
@app.get("/api/v2/users/{user_id}")
def get_user_v2(user_id: int):
    user = get_user(user_id)
    return {
        "id": user.id,
        "name": user.name,
        "email": user.email,
        "created_at": user.created_at
    }
```

### 3. Documentation

```python
@app.get("/api/users/{user_id}")
async def get_user(user_id: int):
    """
    Get user by ID
    
    Args:
        user_id: The user's unique identifier
    
    Returns:
        User object with id, name, email
    
    Raises:
        HTTPException: 404 if user not found
    """
    return get_user_from_db(user_id)
```

### 4. Testing Strategy

```
Unit Tests (80%)
├─ Service logic
├─ Data validation
└─ Error handling

Integration Tests (15%)
├─ Service-to-service communication
├─ Database operations
└─ Message queue

E2E Tests (5%)
├─ Full user workflows
└─ Critical paths
```

---

## 📋 CHECKLIST ДЛЯ МИКРОСЕРВИСНОЙ АРХИТЕКТУРЫ

- [ ] Определены Bounded Contexts
- [ ] Спроектированы API контракты
- [ ] Выбрана стратегия коммуникации (sync/async)
- [ ] Спроектирована стратегия данных (DB per service)
- [ ] Реализована обработка ошибок (retry, circuit breaker)
- [ ] Настроено логирование и мониторинг
- [ ] Реализована аутентификация между сервисами
- [ ] Спроектирована стратегия развертывания
- [ ] Настроено auto-scaling
- [ ] Реализована система оповещений
- [ ] Написаны тесты (unit, integration, e2e)
- [ ] Создана документация

---

**Конец документации**
