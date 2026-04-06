# Структура проекта и Scaffolding

**Проект:** Prize Competitions Platform  
**Масштаб:** Enterprise (100K+ пользователей)  
**Дата:** 2026-04-06

---

## 1. Структура репозитория

```
gaming-platform/
├── services/                          # Микросервисы
│   ├── auth-service/
│   ├── user-service/
│   ├── game-service/
│   ├── payment-service/
│   ├── prize-competition-service/
│   ├── rng-service/
│   ├── lottery-service/
│   ├── instant-win-service/
│   ├── notification-service/
│   ├── analytics-service/
│   ├── audit-service/
│   └── config-service/
│
├── packages/                          # Shared packages
│   ├── common/                        # Common utilities
│   ├── types/                         # Shared TypeScript types
│   ├── logger/                        # Logging library
│   ├── database/                      # Database utilities
│   ├── cache/                         # Cache utilities
│   └── messaging/                     # Message queue utilities
│
├── frontend/                          # Web application
│   ├── web/                           # Next.js web app
│   ├── admin/                         # Admin dashboard
│   └── mobile/                        # React Native mobile app
│
├── infrastructure/                    # Infrastructure as Code
│   ├── docker/                        # Docker configurations
│   ├── kubernetes/                    # Kubernetes manifests
│   ├── terraform/                     # Terraform IaC
│   └── helm/                          # Helm charts
│
├── tests/                             # Test suites
│   ├── e2e/                           # End-to-end tests
│   ├── integration/                   # Integration tests
│   └── load/                          # Load tests
│
├── docs/                              # Documentation
│   ├── architecture/                  # Architecture docs
│   ├── api/                           # API documentation
│   ├── deployment/                    # Deployment guides
│   └── runbooks/                      # Operational runbooks
│
├── scripts/                           # Utility scripts
│   ├── setup.sh                       # Initial setup
│   ├── dev.sh                         # Development environment
│   ├── deploy.sh                      # Deployment script
│   └── migrate.sh                     # Database migrations
│
├── .github/                           # GitHub configuration
│   └── workflows/                     # CI/CD workflows
│
├── docker-compose.yml                 # Local development stack
├── .env.example                       # Environment variables template
├── .gitignore                         # Git ignore rules
├── README.md                          # Project README
├── CONTRIBUTING.md                    # Contribution guidelines
└── package.json                       # Root package.json (monorepo)
```

---

## 2. Структура микросервиса (шаблон)

```
service-name/
├── src/
│   ├── controllers/                   # Request handlers
│   │   └── example.controller.ts
│   ├── services/                      # Business logic
│   │   └── example.service.ts
│   ├── repositories/                  # Data access layer
│   │   └── example.repository.ts
│   ├── models/                        # Data models
│   │   └── example.model.ts
│   ├── middleware/                    # Express middleware
│   │   ├── auth.middleware.ts
│   │   ├── error.middleware.ts
│   │   └── logging.middleware.ts
│   ├── routes/                        # API routes
│   │   └── index.ts
│   ├── config/                        # Configuration
│   │   ├── database.ts
│   │   ├── redis.ts
│   │   └── env.ts
│   ├── utils/                         # Utility functions
│   │   ├── logger.ts
│   │   ├── errors.ts
│   │   └── validators.ts
│   ├── types/                         # TypeScript types
│   │   └── index.ts
│   └── app.ts                         # Express app setup
│
├── tests/
│   ├── unit/                          # Unit tests
│   ├── integration/                   # Integration tests
│   └── fixtures/                      # Test data
│
├── migrations/                        # Database migrations
│   └── 001_initial_schema.sql
│
├── Dockerfile                         # Docker image
├── docker-compose.yml                 # Local development
├── .env.example                       # Environment template
├── package.json                       # Dependencies
├── tsconfig.json                      # TypeScript config
├── jest.config.js                     # Jest config
├── .eslintrc.json                     # ESLint config
├── .prettierrc                        # Prettier config
└── README.md                          # Service documentation
```

---

## 3. Структура фронтенда

### 3.1 Web Application (Next.js)
```
frontend/web/
├── src/
│   ├── app/                           # Next.js app directory
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── auth/
│   │   ├── games/
│   │   ├── competitions/
│   │   └── account/
│   ├── components/                    # React components
│   │   ├── common/
│   │   ├── layout/
│   │   ├── games/
│   │   └── forms/
│   ├── hooks/                         # Custom React hooks
│   ├── services/                      # API clients
│   ├── store/                         # Redux store
│   ├── types/                         # TypeScript types
│   ├── utils/                         # Utility functions
│   └── styles/                        # Global styles
│
├── public/                            # Static assets
├── tests/                             # Test files
├── .env.example
├── next.config.js
├── tsconfig.json
├── jest.config.js
└── package.json
```

### 3.2 Admin Dashboard
```
frontend/admin/
├── src/
│   ├── pages/
│   │   ├── dashboard/
│   │   ├── users/
│   │   ├── games/
│   │   ├── analytics/
│   │   └── settings/
│   ├── components/
│   ├── hooks/
│   ├── services/
│   ├── store/
│   └── types/
├── public/
├── tests/
└── package.json
```

### 3.3 Mobile App (React Native)
```
frontend/mobile/
├── src/
│   ├── screens/
│   │   ├── AuthScreen/
│   │   ├── GameScreen/
│   │   ├── LeaderboardScreen/
│   │   └── AccountScreen/
│   ├── components/
│   ├── hooks/
│   ├── services/
│   ├── store/
│   ├── types/
│   └── navigation/
├── assets/
├── tests/
├── app.json
├── package.json
└── tsconfig.json
```

---

## 4. Структура инфраструктуры

### 4.1 Docker
```
infrastructure/docker/
├── Dockerfile.base                    # Base image
├── Dockerfile.service                 # Service template
├── docker-compose.yml                 # Local development
├── docker-compose.prod.yml            # Production
└── .dockerignore
```

### 4.2 Kubernetes
```
infrastructure/kubernetes/
├── namespaces/
│   └── gaming-platform.yaml
├── services/
│   ├── auth-service.yaml
│   ├── user-service.yaml
│   ├── game-service.yaml
│   └── ...
├── deployments/
│   ├── auth-deployment.yaml
│   ├── user-deployment.yaml
│   └── ...
├── configmaps/
│   └── app-config.yaml
├── secrets/
│   └── app-secrets.yaml
├── ingress/
│   └── ingress.yaml
└── monitoring/
    ├── prometheus.yaml
    └── grafana.yaml
```

### 4.3 Terraform
```
infrastructure/terraform/
├── main.tf                            # Main configuration
├── variables.tf                       # Variables
├── outputs.tf                         # Outputs
├── vpc.tf                             # VPC setup
├── rds.tf                             # RDS database
├── elasticache.tf                     # Redis
├── eks.tf                             # Kubernetes
├── iam.tf                             # IAM roles
├── monitoring.tf                      # Monitoring
└── environments/
    ├── dev/
    ├── staging/
    └── prod/
```

---

## 5. Shared Packages

### 5.1 Common Package
```
packages/common/
├── src/
│   ├── constants/
│   ├── enums/
│   ├── errors/
│   ├── utils/
│   └── index.ts
├── package.json
└── tsconfig.json
```

### 5.2 Types Package
```
packages/types/
├── src/
│   ├── auth/
│   ├── user/
│   ├── game/
│   ├── payment/
│   └── index.ts
├── package.json
└── tsconfig.json
```

### 5.3 Logger Package
```
packages/logger/
├── src/
│   ├── logger.ts
│   ├── transports/
│   └── index.ts
├── package.json
└── tsconfig.json
```

---

## 6. Файлы конфигурации

### 6.1 Root package.json (Monorepo)
```json
{
  "name": "gaming-platform",
  "version": "1.0.0",
  "private": true,
  "workspaces": [
    "services/*",
    "packages/*",
    "frontend/*"
  ],
  "scripts": {
    "dev": "npm run dev --workspaces",
    "build": "npm run build --workspaces",
    "test": "npm run test --workspaces",
    "lint": "npm run lint --workspaces",
    "format": "npm run format --workspaces"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "eslint": "^8.0.0",
    "prettier": "^3.0.0",
    "jest": "^29.0.0"
  }
}
```

### 6.2 docker-compose.yml (Local Development)
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: gaming
      POSTGRES_PASSWORD: gaming123
      POSTGRES_DB: gaming_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  rabbitmq:
    image: rabbitmq:3.12-management
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data

  kibana:
    image: docker.elastic.co/kibana/kibana:8
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

volumes:
  postgres_data:
  redis_data:
  rabbitmq_data:
  elasticsearch_data:
```

### 6.3 .env.example
```
# Database
DATABASE_URL=postgresql://gaming:gaming123@localhost:5432/gaming_db
REDIS_URL=redis://localhost:6379

# RabbitMQ
RABBITMQ_URL=amqp://guest:guest@localhost:5672

# Auth
JWT_SECRET=your-secret-key-here
JWT_EXPIRATION=15m
REFRESH_TOKEN_EXPIRATION=7d

# Payment
STRIPE_SECRET_KEY=sk_test_...
STRIPE_PUBLIC_KEY=pk_test_...

# AWS
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...

# Logging
LOG_LEVEL=info
LOG_FORMAT=json

# Environment
NODE_ENV=development
PORT=3000
```

---

## 7. CI/CD Pipeline (.github/workflows)

### 7.1 Build & Test
```yaml
name: Build & Test

on:
  push:
    branches: [main, dev]
  pull_request:
    branches: [main, dev]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run lint
      - run: npm run test
      - run: npm run build
```

### 7.2 Deploy to Staging
```yaml
name: Deploy to Staging

on:
  push:
    branches: [dev]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to staging
        run: |
          # Deployment script
          ./scripts/deploy.sh staging
```

### 7.3 Deploy to Production
```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to production
        run: |
          # Deployment script
          ./scripts/deploy.sh prod
```

---

## 8. Скрипты утилиты

### 8.1 setup.sh
```bash
#!/bin/bash
# Initial project setup

echo "Setting up gaming platform..."

# Install dependencies
npm install

# Setup environment
cp .env.example .env

# Start Docker services
docker-compose up -d

# Run migrations
npm run migrate

echo "Setup complete!"
```

### 8.2 dev.sh
```bash
#!/bin/bash
# Start development environment

echo "Starting development environment..."

# Start Docker services
docker-compose up -d

# Start all services
npm run dev

echo "Development environment started!"
```

### 8.3 deploy.sh
```bash
#!/bin/bash
# Deploy to environment

ENVIRONMENT=$1

if [ -z "$ENVIRONMENT" ]; then
  echo "Usage: ./deploy.sh [dev|staging|prod]"
  exit 1
fi

echo "Deploying to $ENVIRONMENT..."

# Build Docker images
docker build -t gaming-platform:latest .

# Push to registry
docker push gaming-platform:latest

# Deploy to Kubernetes
kubectl apply -f infrastructure/kubernetes/

echo "Deployment to $ENVIRONMENT complete!"
```

---

## 9. Инициализация проекта

### Шаг 1: Клонирование репозитория
```bash
git clone https://github.com/your-org/gaming-platform.git
cd gaming-platform
```

### Шаг 2: Установка зависимостей
```bash
npm install
```

### Шаг 3: Настройка окружения
```bash
cp .env.example .env
# Отредактировать .env с нужными значениями
```

### Шаг 4: Запуск локального окружения
```bash
docker-compose up -d
npm run migrate
npm run dev
```

### Шаг 5: Проверка здоровья
```bash
# Проверить API Gateway
curl http://localhost:3000/health

# Проверить базу данных
npm run db:check

# Проверить Redis
npm run redis:check
```

---

## 10. Соглашения по кодированию

### Naming Conventions
- **Services:** `{name}-service` (auth-service, user-service)
- **Controllers:** `{name}.controller.ts`
- **Services:** `{name}.service.ts`
- **Repositories:** `{name}.repository.ts`
- **Models:** `{name}.model.ts`
- **Types:** `{name}.types.ts`
- **Constants:** `CONSTANT_NAME`
- **Variables:** `camelCase`
- **Classes:** `PascalCase`
- **Functions:** `camelCase`

### File Organization
- One class/interface per file
- Group related files in folders
- Keep files under 300 lines
- Use index.ts for exports

### Git Workflow
- Branch naming: `feature/service-name`, `fix/issue-name`
- Commit messages: `feat(service): description`
- PR reviews: Minimum 2 approvals
- Merge strategy: Squash commits

---

## 11. Checklist для нового сервиса

- [ ] Создать папку в `services/`
- [ ] Инициализировать package.json
- [ ] Создать структуру папок (src, tests, migrations)
- [ ] Настроить TypeScript
- [ ] Настроить ESLint и Prettier
- [ ] Создать Dockerfile
- [ ] Написать README
- [ ] Добавить в docker-compose.yml
- [ ] Создать Kubernetes manifests
- [ ] Добавить в CI/CD pipeline
- [ ] Написать unit tests
- [ ] Написать integration tests
- [ ] Документировать API endpoints

---

**Статус:** Ready for Implementation  
**Последнее обновление:** 2026-04-06  
**Версия:** 1.0
