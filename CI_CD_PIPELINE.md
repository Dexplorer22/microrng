# CI/CD Pipeline для системы микросервисов

**Проект:** Prize Competitions Platform  
**Масштаб:** Enterprise (100K+ пользователей)  
**Дата:** 2026-04-06

---

## 1. Обзор CI/CD Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│                    Git Push / PR                             │
└────────────────────┬────────────────────────────────────────┘
                     │
        ┌────────────▼────────────┐
        │   GitHub Actions        │
        │   Trigger Workflow      │
        └────────────┬────────────┘
                     │
    ┌────────────────┼────────────────┐
    │                │                │
┌───▼──┐      ┌──────▼──┐      ┌─────▼──┐
│Build │      │  Test   │      │ Lint   │
│      │      │         │      │        │
└───┬──┘      └──────┬──┘      └─────┬──┘
    │                │               │
    └────────────────┼───────────────┘
                     │
        ┌────────────▼────────────┐
        │  Code Quality Check     │
        │  (SonarQube)            │
        └────────────┬────────────┘
                     │
        ┌────────────▼────────────┐
        │  Security Scan          │
        │  (Snyk, OWASP)          │
        └────────────┬────────────┘
                     │
        ┌────────────▼────────────┐
        │  Build Docker Images    │
        │  Push to Registry       │
        └────────────┬────────────┘
                     │
    ┌────────────────┼────────────────┐
    │                │                │
┌───▼──────┐  ┌──────▼──────┐  ┌─────▼──────┐
│ Deploy   │  │   Deploy    │  │  Deploy    │
│ Staging  │  │  Canary     │  │ Production │
│          │  │             │  │            │
└───┬──────┘  └──────┬──────┘  └─────┬──────┘
    │                │               │
    └────────────────┼───────────────┘
                     │
        ┌────────────▼────────────┐
        │  Smoke Tests            │
        │  Health Checks          │
        └────────────┬────────────┘
                     │
        ┌────────────▼────────────┐
        │  Monitoring & Alerts    │
        └────────────────────────┘
```

---

## 2. GitHub Actions Workflows

### 2.1 Build & Test Workflow (.github/workflows/build-test.yml)

```yaml
name: Build & Test

on:
  push:
    branches: [main, dev]
  pull_request:
    branches: [main, dev]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [20.x]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Lint code
        run: npm run lint
      
      - name: Run unit tests
        run: npm run test:unit
      
      - name: Run integration tests
        run: npm run test:integration
      
      - name: Build project
        run: npm run build
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json
          flags: unittests
          name: codecov-umbrella

  test-coverage:
    runs-on: ubuntu-latest
    needs: build
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 20.x
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Check coverage threshold
        run: npm run test:coverage -- --coverage-threshold=80
```

---

### 2.2 Code Quality Workflow (.github/workflows/code-quality.yml)

```yaml
name: Code Quality

on:
  push:
    branches: [main, dev]
  pull_request:
    branches: [main, dev]

jobs:
  sonarqube:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 20.x
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests with coverage
        run: npm run test:coverage
      
      - name: SonarQube Scan
        uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        with:
          args: >
            -Dsonar.projectKey=gaming-platform
            -Dsonar.organization=your-org
            -Dsonar.sources=src
            -Dsonar.tests=tests
            -Dsonar.coverage.exclusions=**/*.test.ts,**/node_modules/**
            -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info

  eslint:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 20.x
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run ESLint
        run: npm run lint -- --format json --output-file eslint-report.json
        continue-on-error: true
      
      - name: Upload ESLint report
        uses: actions/upload-artifact@v3
        with:
          name: eslint-report
          path: eslint-report.json
```

---

### 2.3 Security Scan Workflow (.github/workflows/security.yml)

```yaml
name: Security Scan

on:
  push:
    branches: [main, dev]
  pull_request:
    branches: [main, dev]
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM

jobs:
  snyk:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 20.x
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run Snyk scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

  owasp-dependency-check:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Run OWASP Dependency Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: 'gaming-platform'
          path: '.'
          format: 'JSON'
          args: >
            --enableExperimental
            --enableRetired
      
      - name: Upload OWASP report
        uses: actions/upload-artifact@v3
        with:
          name: owasp-report
          path: reports/

  container-scan:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Build Docker image
        run: docker build -t gaming-platform:scan .
      
      - name: Run Trivy scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'gaming-platform:scan'
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
```

---

### 2.4 Docker Build & Push Workflow (.github/workflows/docker-build.yml)

```yaml
name: Docker Build & Push

on:
  push:
    branches: [main, dev]
    tags: ['v*']
  pull_request:
    branches: [main, dev]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Log in to Container Registry
        if: github.event_name != 'pull_request'
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

---

### 2.5 Deploy to Staging Workflow (.github/workflows/deploy-staging.yml)

```yaml
name: Deploy to Staging

on:
  push:
    branches: [dev]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: staging
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Update kubeconfig
        run: |
          aws eks update-kubeconfig \
            --name gaming-platform-staging \
            --region us-east-1
      
      - name: Deploy to Kubernetes
        run: |
          kubectl apply -f infrastructure/kubernetes/staging/
          kubectl rollout status deployment -n gaming-platform
      
      - name: Run smoke tests
        run: |
          npm run test:smoke -- --environment=staging
      
      - name: Health check
        run: |
          curl -f https://staging-api.gaming-platform.com/health || exit 1
      
      - name: Notify Slack
        if: always()
        uses: slackapi/slack-github-action@v1
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK }}
          payload: |
            {
              "text": "Staging deployment ${{ job.status }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Staging Deployment* - ${{ job.status }}\n${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
                  }
                }
              ]
            }
```

---

### 2.6 Deploy to Production Workflow (.github/workflows/deploy-production.yml)

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]
    tags: ['v*']

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Update kubeconfig
        run: |
          aws eks update-kubeconfig \
            --name gaming-platform-prod \
            --region us-east-1
      
      - name: Deploy with Blue-Green strategy
        run: |
          # Deploy to green environment
          kubectl apply -f infrastructure/kubernetes/production/green/
          
          # Wait for rollout
          kubectl rollout status deployment -n gaming-platform
          
          # Switch traffic to green
          kubectl patch service gaming-platform-api \
            -p '{"spec":{"selector":{"version":"green"}}}'
      
      - name: Run smoke tests
        run: |
          npm run test:smoke -- --environment=production
      
      - name: Health check
        run: |
          curl -f https://api.gaming-platform.com/health || exit 1
      
      - name: Monitor metrics
        run: |
          # Check error rate, latency, etc.
          npm run test:monitoring -- --environment=production
      
      - name: Rollback on failure
        if: failure()
        run: |
          # Switch traffic back to blue
          kubectl patch service gaming-platform-api \
            -p '{"spec":{"selector":{"version":"blue"}}}'
      
      - name: Notify team
        if: always()
        uses: slackapi/slack-github-action@v1
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK }}
          payload: |
            {
              "text": "Production deployment ${{ job.status }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Production Deployment* - ${{ job.status }}\n${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
                  }
                }
              ]
            }
```

---

## 3. Pipeline Stages

### Stage 1: Trigger (0-1 min)
- Git push или PR создан
- GitHub Actions запускается
- Checkout кода

### Stage 2: Build (2-5 min)
- Установка зависимостей
- Компиляция TypeScript
- Сборка артефактов

### Stage 3: Test (5-10 min)
- Unit tests (>80% coverage)
- Integration tests
- E2E tests (critical flows)

### Stage 4: Quality (3-5 min)
- ESLint
- Prettier check
- SonarQube analysis
- Code coverage validation

### Stage 5: Security (5-10 min)
- Snyk dependency scan
- OWASP Dependency Check
- Container image scan (Trivy)
- SAST analysis

### Stage 6: Build Docker (3-5 min)
- Build Docker image
- Push to registry (GHCR)
- Tag with version/commit

### Stage 7: Deploy Staging (5-10 min)
- Deploy to staging environment
- Run smoke tests
- Health checks

### Stage 8: Deploy Production (10-15 min)
- Blue-green deployment
- Smoke tests
- Monitoring checks
- Rollback if needed

---

## 4. Environment Configuration

### Development (.env.dev)
```
NODE_ENV=development
LOG_LEVEL=debug
DATABASE_URL=postgresql://dev:dev@localhost:5432/gaming_dev
REDIS_URL=redis://localhost:6379
```

### Staging (.env.staging)
```
NODE_ENV=staging
LOG_LEVEL=info
DATABASE_URL=postgresql://staging:***@staging-db.rds.amazonaws.com:5432/gaming_staging
REDIS_URL=redis://staging-redis.elasticache.amazonaws.com:6379
```

### Production (.env.prod)
```
NODE_ENV=production
LOG_LEVEL=warn
DATABASE_URL=postgresql://prod:***@prod-db.rds.amazonaws.com:5432/gaming_prod
REDIS_URL=redis://prod-redis.elasticache.amazonaws.com:6379
```

---

## 5. Deployment Strategies

### Blue-Green Deployment
```
1. Deploy new version to "green" environment
2. Run smoke tests on green
3. Switch traffic from blue to green
4. Keep blue as rollback point
5. After 24h, decommission blue
```

### Canary Deployment
```
1. Deploy to 5% of traffic
2. Monitor metrics for 30 minutes
3. If healthy, increase to 25%
4. Monitor for 30 minutes
5. If healthy, increase to 100%
6. Rollback if issues detected
```

### Rolling Deployment
```
1. Update 1 pod at a time
2. Wait for health check
3. Move to next pod
4. Continue until all updated
5. Automatic rollback on failure
```

---

## 6. Monitoring & Alerts

### Key Metrics
- Build time: <10 minutes
- Test coverage: >80%
- Deployment success rate: >99%
- MTTR (Mean Time To Recovery): <5 minutes

### Alerts
- Build failure
- Test failure
- Security vulnerability found
- Deployment failure
- High error rate in production
- High latency in production

---

## 7. Rollback Procedure

### Automatic Rollback
```bash
# If health checks fail
kubectl rollout undo deployment/gaming-platform-api

# If error rate > 5%
kubectl rollout undo deployment/gaming-platform-api

# If latency p95 > 500ms
kubectl rollout undo deployment/gaming-platform-api
```

### Manual Rollback
```bash
# Get previous revision
kubectl rollout history deployment/gaming-platform-api

# Rollback to specific revision
kubectl rollout undo deployment/gaming-platform-api --to-revision=5
```

---

## 8. Performance Targets

| Metric | Target | Current |
|--------|--------|---------|
| Build time | <10 min | - |
| Test time | <10 min | - |
| Quality check | <5 min | - |
| Security scan | <10 min | - |
| Docker build | <5 min | - |
| Deploy staging | <10 min | - |
| Deploy production | <15 min | - |
| Total pipeline | <60 min | - |

---

## 9. Troubleshooting

### Build Failure
```bash
# Check logs
gh run view <run-id> --log

# Re-run failed job
gh run rerun <run-id>
```

### Test Failure
```bash
# Run tests locally
npm run test:debug

# Check coverage
npm run test:coverage
```

### Deployment Failure
```bash
# Check Kubernetes events
kubectl describe deployment gaming-platform-api

# Check pod logs
kubectl logs -f deployment/gaming-platform-api

# Rollback
kubectl rollout undo deployment/gaming-platform-api
```

---

**Статус:** Ready for Implementation  
**Последнее обновление:** 2026-04-06  
**Версия:** 1.0
