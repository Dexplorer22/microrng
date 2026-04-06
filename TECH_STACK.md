# Tech Stack для системы микросервисов азартных игр

**Проект:** Prize Competitions Platform  
**Масштаб:** Enterprise (100K+ пользователей)  
**Дата:** 2026-04-06

---

## 1. Backend Runtime & Framework

### Primary: Node.js + Express.js
**Почему:**
- ✅ Асинхронная обработка (идеально для I/O операций)
- ✅ Быстрое развитие
- ✅ Большое сообщество
- ✅ Хорошая поддержка микросервисов
- ✅ Легко масштабируется

**Версия:** Node.js 20 LTS

**Альтернативы:**
- Go (Gin/Echo) - для критичных по производительности сервисов (RNG, Payment)
- Python (FastAPI) - для Analytics Service

**Рекомендация:** Node.js для большинства, Go для RNG и Payment Services

---

## 2. API & Communication

### REST API
- **Framework:** Express.js
- **Validation:** Joi / Zod
- **Documentation:** OpenAPI/Swagger
- **Rate Limiting:** express-rate-limit

### Message Queue
- **Primary:** RabbitMQ (reliability, AMQP protocol)
- **Alternative:** Apache Kafka (для high-volume analytics)
- **Node.js Client:** amqplib (RabbitMQ), kafkajs (Kafka)

### gRPC (для inter-service communication)
- **Use case:** High-performance service-to-service calls
- **Services:** RNG Service, Payment Service
- **Node.js:** @grpc/grpc-js

---

## 3. Database

### Primary Database: PostgreSQL 15
**Почему:**
- ✅ ACID compliance (критично для платежей)
- ✅ JSON support (гибкость)
- ✅ Full-text search
- ✅ Excellent reliability
- ✅ Mature ecosystem

**Node.js Driver:** pg (node-postgres)

**ORM:** Prisma или TypeORM
- **Prisma:** Рекомендуется (лучше DX, миграции)
- **TypeORM:** Альтернатива (если нужен TypeScript)

### Cache Layer: Redis 7
**Use cases:**
- Session storage
- Real-time leaderboards
- Rate limiting
- Cache layer
- Pub/Sub для notifications

**Node.js Client:** ioredis (рекомендуется)

### Analytics Database: ClickHouse
**Use case:** Time-series data, analytics queries
**Node.js Client:** @clickhouse/client

### Search: Elasticsearch 8
**Use case:** Logs, audit trails, full-text search
**Node.js Client:** @elastic/elasticsearch

---

## 4. Authentication & Security

### JWT (JSON Web Tokens)
- **Library:** jsonwebtoken
- **Refresh tokens:** Stored in Redis
- **Expiration:** Access token 15min, Refresh token 7 days

### OAuth2 / OpenID Connect
- **Library:** passport.js
- **Providers:** Google, Apple, Facebook

### Password Hashing
- **Library:** bcrypt
- **Cost factor:** 12

### Secrets Management
- **Development:** .env files (dotenv)
- **Production:** HashiCorp Vault or AWS Secrets Manager

### API Security
- **CORS:** cors middleware
- **Helmet:** helmet.js (security headers)
- **Input Validation:** Joi/Zod
- **SQL Injection Prevention:** Parameterized queries (Prisma/TypeORM)

---

## 5. Frontend Stack

### Web Application
- **Framework:** React 18 + Next.js 14
- **State Management:** Redux Toolkit or Zustand
- **UI Library:** Material-UI (MUI) or Tailwind CSS
- **HTTP Client:** axios or fetch API
- **Real-time:** Socket.io client

### Mobile Application
- **Framework:** React Native or Flutter
- **State Management:** Redux or Riverpod (Flutter)
- **HTTP Client:** axios or Dio (Flutter)

### Admin Dashboard
- **Framework:** React + Next.js
- **Charts:** Recharts or Chart.js
- **Tables:** React Table (TanStack Table)

---

## 6. DevOps & Infrastructure

### Containerization
- **Docker:** Docker 24+
- **Docker Compose:** для local development
- **Registry:** Docker Hub или AWS ECR

### Orchestration
- **Kubernetes:** EKS (AWS), GKE (Google), AKS (Azure)
- **Helm:** для package management
- **Service Mesh:** Istio (optional, для advanced scenarios)

### CI/CD Pipeline
- **GitHub Actions** (если GitHub)
- **GitLab CI** (если GitLab)
- **Jenkins** (self-hosted alternative)

**Pipeline stages:**
1. Build & Test
2. Code Quality (SonarQube)
3. Security Scan (Snyk, OWASP)
4. Build Docker image
5. Push to registry
6. Deploy to staging
7. Integration tests
8. Deploy to production

### Infrastructure as Code
- **Terraform:** для AWS/GCP/Azure resources
- **CloudFormation:** если только AWS

### Monitoring & Logging

**Metrics:**
- **Prometheus:** metrics collection
- **Grafana:** visualization

**Logging:**
- **ELK Stack:** Elasticsearch + Logstash + Kibana
- **Alternative:** Datadog, New Relic, Splunk

**Distributed Tracing:**
- **Jaeger:** для trace analysis
- **Alternative:** Zipkin

**Uptime Monitoring:**
- **Uptime Kuma** или **Pingdom**

---

## 7. Testing

### Unit Testing
- **Framework:** Jest
- **Coverage:** >80%

### Integration Testing
- **Framework:** Jest + Supertest
- **Database:** Test containers (testcontainers-node)

### E2E Testing
- **Framework:** Playwright или Cypress
- **Coverage:** Critical user flows

### Load Testing
- **Tool:** k6 или Apache JMeter
- **Target:** 1000+ concurrent users

### Security Testing
- **OWASP ZAP:** Automated security scanning
- **Snyk:** Dependency vulnerability scanning

---

## 8. Development Tools

### Version Control
- **Git:** GitHub, GitLab, или Gitea
- **Branching:** Git Flow

### Code Quality
- **Linter:** ESLint
- **Formatter:** Prettier
- **Type Checking:** TypeScript

### Package Management
- **npm:** v10+
- **Alternative:** yarn, pnpm

### Local Development
- **Docker Compose:** для local services (PostgreSQL, Redis, RabbitMQ)
- **Nodemon:** для auto-reload
- **Postman/Insomnia:** для API testing

### Documentation
- **API Docs:** Swagger/OpenAPI
- **Architecture:** Miro, Lucidchart
- **Wiki:** Confluence или GitHub Wiki

---

## 9. Recommended Tech Stack Summary

| Layer | Technology | Version | Notes |
|-------|-----------|---------|-------|
| **Runtime** | Node.js | 20 LTS | Primary backend |
| **Framework** | Express.js | 4.x | REST APIs |
| **Language** | TypeScript | 5.x | Type safety |
| **Database** | PostgreSQL | 15 | Primary DB |
| **Cache** | Redis | 7 | Session, cache |
| **Message Queue** | RabbitMQ | 3.12 | Async processing |
| **Search** | Elasticsearch | 8 | Logs, search |
| **Analytics** | ClickHouse | 23.x | Time-series |
| **ORM** | Prisma | 5.x | Database access |
| **Auth** | JWT + OAuth2 | - | Authentication |
| **Frontend** | React + Next.js | 18/14 | Web app |
| **Mobile** | React Native | 0.73 | Mobile app |
| **Container** | Docker | 24 | Containerization |
| **Orchestration** | Kubernetes | 1.28 | Container mgmt |
| **CI/CD** | GitHub Actions | - | Automation |
| **Monitoring** | Prometheus + Grafana | - | Metrics |
| **Logging** | ELK Stack | 8 | Log aggregation |
| **Testing** | Jest + Playwright | - | Test automation |

---

## 10. Development Environment Setup

### Prerequisites
```bash
# Required
- Node.js 20 LTS
- Docker & Docker Compose
- Git
- PostgreSQL 15 (or via Docker)
- Redis 7 (or via Docker)
- RabbitMQ 3.12 (or via Docker)

# Optional
- Kubernetes (minikube or Docker Desktop)
- Terraform
- AWS CLI / gcloud CLI
```

### Local Development Stack (Docker Compose)
```yaml
services:
  postgres:
    image: postgres:15
    ports: ["5432:5432"]
  
  redis:
    image: redis:7
    ports: ["6379:6379"]
  
  rabbitmq:
    image: rabbitmq:3.12-management
    ports: ["5672:5672", "15672:15672"]
  
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8
    ports: ["9200:9200"]
  
  kibana:
    image: docker.elastic.co/kibana/kibana:8
    ports: ["5601:5601"]
```

---

## 11. Performance Targets

| Metric | Target | Notes |
|--------|--------|-------|
| API Response Time (p95) | <200ms | Excluding external APIs |
| RNG Generation | <50ms | Cryptographic operations |
| Payment Processing | <5s | Including gateway calls |
| Database Query | <100ms | p95 |
| Cache Hit Rate | >80% | Redis cache |
| Uptime | 99.99% | 4 nines |
| Deployment Time | <5min | Blue-green deployment |

---

## 12. Security Checklist

- [ ] TLS 1.3 for all communications
- [ ] Data encryption at rest
- [ ] API authentication (OAuth2/JWT)
- [ ] Rate limiting on all endpoints
- [ ] Input validation & sanitization
- [ ] SQL injection prevention
- [ ] XSS protection
- [ ] CSRF protection
- [ ] CORS properly configured
- [ ] Secrets not in code
- [ ] Regular security audits
- [ ] Compliance: GDPR, PCI-DSS, local gaming regulations

---

## 13. Cost Optimization

### Development
- Use free tier services (GitHub, Docker Hub)
- Local development with Docker Compose

### Staging
- Smaller instance sizes
- Shared database
- Scheduled auto-shutdown

### Production
- Reserved instances (AWS)
- Auto-scaling groups
- CDN for static assets
- Database read replicas

---

## 14. Migration Path

### Phase 1: MVP (Weeks 1-6)
- Node.js + Express
- PostgreSQL + Redis
- Basic Docker setup
- GitHub Actions CI/CD

### Phase 2: Scale (Weeks 7-14)
- Add Kubernetes
- Implement RabbitMQ
- Add Elasticsearch
- Implement monitoring

### Phase 3: Optimize (Weeks 15+)
- Add ClickHouse for analytics
- Implement gRPC for critical services
- Add service mesh (Istio)
- Performance tuning

---

**Статус:** Recommended  
**Последнее обновление:** 2026-04-06  
**Версия:** 1.0
