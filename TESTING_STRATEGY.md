# Тестовая стратегия для системы микросервисов

**Проект:** Prize Competitions Platform  
**Масштаб:** Enterprise (100K+ пользователей)  
**Дата:** 2026-04-06

---

## 1. Пирамида тестирования

```
                    ▲
                   /|\
                  / | \
                 /  |  \
                / E2E  \
               /_______ \
              /         \
             /  Integration\
            /_______________\
           /                 \
          /      Unit Tests    \
         /_____________________ \
```

### Распределение тестов
- **Unit Tests:** 70% (быстрые, изолированные)
- **Integration Tests:** 20% (взаимодействие компонентов)
- **E2E Tests:** 10% (критичные user flows)

---

## 2. Unit Testing

### 2.1 Фреймворк и инструменты
- **Framework:** Jest
- **Coverage:** >80%
- **Mocking:** jest.mock(), sinon
- **Assertions:** expect()

### 2.2 Структура unit тестов

```typescript
// src/services/user.service.test.ts
import { UserService } from './user.service';
import { UserRepository } from '../repositories/user.repository';

describe('UserService', () => {
  let userService: UserService;
  let userRepository: jest.Mocked<UserRepository>;

  beforeEach(() => {
    userRepository = {
      findById: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
    } as any;
    userService = new UserService(userRepository);
  });

  describe('getUserById', () => {
    it('should return user when found', async () => {
      const userId = 'uuid-123';
      const mockUser = { id: userId, email: 'user@example.com' };
      
      userRepository.findById.mockResolvedValue(mockUser);
      
      const result = await userService.getUserById(userId);
      
      expect(result).toEqual(mockUser);
      expect(userRepository.findById).toHaveBeenCalledWith(userId);
    });

    it('should throw error when user not found', async () => {
      userRepository.findById.mockResolvedValue(null);
      
      await expect(userService.getUserById('invalid-id'))
        .rejects
        .toThrow('User not found');
    });
  });

  describe('createUser', () => {
    it('should create user with valid data', async () => {
      const userData = {
        email: 'newuser@example.com',
        password: 'SecurePass123!',
        firstName: 'John',
        lastName: 'Doe'
      };
      
      userRepository.create.mockResolvedValue({ id: 'uuid-123', ...userData });
      
      const result = await userService.createUser(userData);
      
      expect(result.id).toBeDefined();
      expect(userRepository.create).toHaveBeenCalledWith(userData);
    });

    it('should validate email format', async () => {
      const invalidData = {
        email: 'invalid-email',
        password: 'SecurePass123!',
        firstName: 'John',
        lastName: 'Doe'
      };
      
      await expect(userService.createUser(invalidData))
        .rejects
        .toThrow('Invalid email format');
    });
  });
});
```

### 2.3 Coverage targets

| Component | Target | Notes |
|-----------|--------|-------|
| Services | >90% | Business logic |
| Controllers | >80% | Request handling |
| Repositories | >85% | Data access |
| Utils | >95% | Helper functions |
| Overall | >80% | Project average |

---

## 3. Integration Testing

### 3.1 Фреймворк и инструменты
- **Framework:** Jest + Supertest
- **Database:** Test containers (PostgreSQL)
- **Cache:** Redis test instance
- **Message Queue:** RabbitMQ test instance

### 3.2 Структура integration тестов

```typescript
// tests/integration/auth.integration.test.ts
import request from 'supertest';
import { app } from '../../src/app';
import { UserRepository } from '../../src/repositories/user.repository';
import { AuthService } from '../../src/services/auth.service';

describe('Auth Integration Tests', () => {
  let userRepository: UserRepository;
  let authService: AuthService;

  beforeAll(async () => {
    // Setup test database
    await setupTestDatabase();
    userRepository = new UserRepository();
    authService = new AuthService(userRepository);
  });

  afterAll(async () => {
    await teardownTestDatabase();
  });

  describe('POST /auth/register', () => {
    it('should register new user successfully', async () => {
      const response = await request(app)
        .post('/auth/register')
        .send({
          email: 'newuser@example.com',
          password: 'SecurePass123!',
          firstName: 'John',
          lastName: 'Doe',
          dateOfBirth: '1990-01-15',
          country: 'US'
        });

      expect(response.status).toBe(201);
      expect(response.body.success).toBe(true);
      expect(response.body.data.accessToken).toBeDefined();
      expect(response.body.data.refreshToken).toBeDefined();
    });

    it('should reject duplicate email', async () => {
      // First registration
      await request(app)
        .post('/auth/register')
        .send({
          email: 'duplicate@example.com',
          password: 'SecurePass123!',
          firstName: 'John',
          lastName: 'Doe',
          dateOfBirth: '1990-01-15',
          country: 'US'
        });

      // Second registration with same email
      const response = await request(app)
        .post('/auth/register')
        .send({
          email: 'duplicate@example.com',
          password: 'DifferentPass123!',
          firstName: 'Jane',
          lastName: 'Doe',
          dateOfBirth: '1992-03-20',
          country: 'US'
        });

      expect(response.status).toBe(409);
      expect(response.body.success).toBe(false);
      expect(response.body.error.code).toBe('EMAIL_ALREADY_EXISTS');
    });
  });

  describe('POST /auth/login', () => {
    beforeEach(async () => {
      // Create test user
      await request(app)
        .post('/auth/register')
        .send({
          email: 'testuser@example.com',
          password: 'SecurePass123!',
          firstName: 'Test',
          lastName: 'User',
          dateOfBirth: '1990-01-15',
          country: 'US'
        });
    });

    it('should login with valid credentials', async () => {
      const response = await request(app)
        .post('/auth/login')
        .send({
          email: 'testuser@example.com',
          password: 'SecurePass123!'
        });

      expect(response.status).toBe(200);
      expect(response.body.success).toBe(true);
      expect(response.body.data.accessToken).toBeDefined();
    });

    it('should reject invalid password', async () => {
      const response = await request(app)
        .post('/auth/login')
        .send({
          email: 'testuser@example.com',
          password: 'WrongPassword123!'
        });

      expect(response.status).toBe(401);
      expect(response.body.success).toBe(false);
    });
  });

  describe('Cross-service integration', () => {
    it('should create user and update balance', async () => {
      // Register user
      const registerResponse = await request(app)
        .post('/auth/register')
        .send({
          email: 'balance-test@example.com',
          password: 'SecurePass123!',
          firstName: 'Balance',
          lastName: 'Test',
          dateOfBirth: '1990-01-15',
          country: 'US'
        });

      const userId = registerResponse.body.data.userId;
      const token = registerResponse.body.data.accessToken;

      // Deposit funds
      const depositResponse = await request(app)
        .post('/payments/deposit')
        .set('Authorization', `Bearer ${token}`)
        .send({
          amount: 100.00,
          currency: 'USD',
          paymentMethod: 'credit_card'
        });

      expect(depositResponse.status).toBe(201);

      // Check balance
      const balanceResponse = await request(app)
        .get(`/users/${userId}/balance`)
        .set('Authorization', `Bearer ${token}`);

      expect(balanceResponse.status).toBe(200);
      expect(balanceResponse.body.data.balance).toBe(100.00);
    });
  });
});
```

### 3.3 Test database setup

```typescript
// tests/setup/database.ts
import { Pool } from 'pg';
import { GenericContainer } from 'testcontainers';

let postgresContainer: any;
let pool: Pool;

export async function setupTestDatabase() {
  postgresContainer = await new GenericContainer('postgres:15')
    .withEnvironment('POSTGRES_USER', 'test')
    .withEnvironment('POSTGRES_PASSWORD', 'test')
    .withEnvironment('POSTGRES_DB', 'gaming_test')
    .withExposedPorts(5432)
    .start();

  const host = postgresContainer.getHost();
  const port = postgresContainer.getMappedPort(5432);

  pool = new Pool({
    user: 'test',
    password: 'test',
    host,
    port,
    database: 'gaming_test'
  });

  // Run migrations
  await runMigrations(pool);
}

export async function teardownTestDatabase() {
  await pool.end();
  await postgresContainer.stop();
}

async function runMigrations(pool: Pool) {
  // Run all migration files
  const migrations = [
    require('./migrations/001_initial_schema.sql'),
    require('./migrations/002_add_indexes.sql')
  ];

  for (const migration of migrations) {
    await pool.query(migration);
  }
}
```

---

## 4. End-to-End Testing

### 4.1 Фреймворк и инструменты
- **Framework:** Playwright
- **Browser:** Chromium, Firefox, WebKit
- **Headless:** true (CI/CD)
- **Timeout:** 30 seconds per test

### 4.2 Структура E2E тестов

```typescript
// tests/e2e/user-flow.e2e.test.ts
import { test, expect } from '@playwright/test';

test.describe('User Game Flow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('http://localhost:3000');
  });

  test('should complete full game flow', async ({ page }) => {
    // 1. Register
    await page.click('text=Sign Up');
    await page.fill('input[name="email"]', 'e2e-user@example.com');
    await page.fill('input[name="password"]', 'SecurePass123!');
    await page.fill('input[name="firstName"]', 'E2E');
    await page.fill('input[name="lastName"]', 'User');
    await page.click('button:has-text("Register")');

    // Wait for redirect to dashboard
    await page.waitForURL('**/dashboard');
    expect(page.url()).toContain('/dashboard');

    // 2. Deposit funds
    await page.click('text=Deposit');
    await page.fill('input[name="amount"]', '100');
    await page.click('button:has-text("Deposit")');

    // Wait for success message
    await page.waitForSelector('text=Deposit successful');

    // 3. Play game
    await page.click('text=Play Now');
    await page.waitForSelector('[data-testid="game-board"]');

    // 4. Check balance
    const balance = await page.textContent('[data-testid="balance"]');
    expect(balance).toContain('$');

    // 5. Logout
    await page.click('[data-testid="user-menu"]');
    await page.click('text=Logout');

    // Verify redirect to login
    await page.waitForURL('**/login');
  });

  test('should handle payment flow', async ({ page }) => {
    // Login
    await page.click('text=Sign In');
    await page.fill('input[name="email"]', 'existing-user@example.com');
    await page.fill('input[name="password"]', 'SecurePass123!');
    await page.click('button:has-text("Login")');

    // Navigate to payments
    await page.click('text=Account');
    await page.click('text=Payments');

    // Deposit
    await page.click('button:has-text("Add Funds")');
    await page.fill('input[name="amount"]', '50');
    await page.selectOption('select[name="method"]', 'credit_card');
    await page.click('button:has-text("Continue")');

    // Handle payment modal
    await page.frameLocator('iframe[name="payment"]').locator('input[name="cardNumber"]').fill('4242424242424242');
    await page.frameLocator('iframe[name="payment"]').locator('input[name="expiry"]').fill('12/25');
    await page.frameLocator('iframe[name="payment"]').locator('input[name="cvc"]').fill('123');
    await page.frameLocator('iframe[name="payment"]').locator('button:has-text("Pay")').click();

    // Verify success
    await page.waitForSelector('text=Payment successful');
  });

  test('should display leaderboard', async ({ page }) => {
    await page.click('text=Leaderboard');
    
    // Wait for leaderboard to load
    await page.waitForSelector('[data-testid="leaderboard-table"]');

    // Verify table structure
    const rows = await page.locator('[data-testid="leaderboard-row"]').count();
    expect(rows).toBeGreaterThan(0);

    // Verify columns
    await expect(page.locator('text=Rank')).toBeVisible();
    await expect(page.locator('text=Player')).toBeVisible();
    await expect(page.locator('text=Winnings')).toBeVisible();
  });
});
```

### 4.3 Critical user flows

| Flow | Steps | Priority |
|------|-------|----------|
| Registration | Sign up → Verify email → Login | P0 |
| Deposit | Add payment method → Deposit funds → Verify balance | P0 |
| Play Game | Start game → Make moves → Claim prize | P0 |
| Withdrawal | Request withdrawal → Verify bank account → Confirm | P1 |
| Leaderboard | View rankings → Filter by period → Share | P2 |

---

## 5. Performance Testing

### 5.1 Инструменты
- **Tool:** k6
- **Load:** 1000 concurrent users
- **Duration:** 5 minutes
- **Ramp-up:** 1 minute

### 5.2 Performance test script

```javascript
// tests/performance/load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '1m', target: 100 },   // Ramp up
    { duration: '3m', target: 1000 },  // Stay at peak
    { duration: '1m', target: 0 },     // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500', 'p(99)<1000'],
    http_req_failed: ['rate<0.1'],
  },
};

export default function () {
  // Test login
  const loginRes = http.post('http://localhost:3000/auth/login', {
    email: `user${__VU}@example.com`,
    password: 'SecurePass123!'
  });

  check(loginRes, {
    'login status is 200': (r) => r.status === 200,
    'login time < 500ms': (r) => r.timings.duration < 500,
  });

  const token = loginRes.json('data.accessToken');

  // Test get balance
  const balanceRes = http.get('http://localhost:3000/users/me/balance', {
    headers: { Authorization: `Bearer ${token}` }
  });

  check(balanceRes, {
    'balance status is 200': (r) => r.status === 200,
    'balance time < 200ms': (r) => r.timings.duration < 200,
  });

  // Test play game
  const gameRes = http.post('http://localhost:3000/games/start', {
    gameType: 'instant_win',
    betAmount: 10
  }, {
    headers: { Authorization: `Bearer ${token}` }
  });

  check(gameRes, {
    'game start status is 201': (r) => r.status === 201,
    'game start time < 1000ms': (r) => r.timings.duration < 1000,
  });

  sleep(1);
}
```

### 5.3 Performance targets

| Endpoint | P95 | P99 | Notes |
|----------|-----|-----|-------|
| /auth/login | <300ms | <500ms | Auth critical |
| /users/me/balance | <100ms | <200ms | Cached |
| /games/start | <500ms | <1000ms | RNG involved |
| /games/{id}/result | <200ms | <500ms | Read-heavy |
| /payments/deposit | <2000ms | <5000ms | External API |

---

## 6. Security Testing

### 6.1 OWASP Top 10 Coverage

| Vulnerability | Test | Tool |
|---|---|---|
| Injection | SQL injection, NoSQL injection | OWASP ZAP |
| Broken Auth | Token expiration, MFA bypass | Manual + Automated |
| Sensitive Data | Encryption, PII exposure | Snyk, Manual |
| XML External Entities | XXE attacks | OWASP ZAP |
| Broken Access Control | RBAC bypass, Privilege escalation | Manual |
| Security Misconfiguration | Headers, CORS, SSL | OWASP ZAP |
| XSS | Stored, Reflected, DOM XSS | OWASP ZAP |
| Insecure Deserialization | Object injection | Manual |
| Using Components with Known Vulns | Dependency scan | Snyk |
| Insufficient Logging | Audit trail verification | Manual |

### 6.2 Security test examples

```bash
# SQL Injection test
curl -X POST http://localhost:3000/auth/login \
  -d "email=admin' OR '1'='1&password=anything"

# XSS test
curl -X POST http://localhost:3000/users/me \
  -d "firstName=<script>alert('XSS')</script>"

# CSRF test
# Verify CSRF token validation

# Rate limiting test
for i in {1..100}; do
  curl http://localhost:3000/auth/login
done
```

---

## 7. Test Execution Schedule

### Daily (CI/CD)
- Unit tests: All
- Integration tests: All
- E2E tests: Critical flows only
- Security scan: Dependency check

### Weekly
- Full E2E test suite
- Performance tests
- Security audit
- Code coverage review

### Monthly
- Load testing (1000+ users)
- Chaos engineering
- Penetration testing
- Compliance audit

---

## 8. Test Metrics & Reporting

### Key Metrics
- **Code Coverage:** >80%
- **Test Pass Rate:** >99%
- **Test Execution Time:** <30 minutes
- **Bug Escape Rate:** <5%
- **MTTR:** <5 minutes

### Reporting
- Daily: Test results in Slack
- Weekly: Coverage report
- Monthly: Quality metrics dashboard

---

## 9. Test Data Management

### Test Data Strategy
- Use factories for consistent data
- Isolate test data per test
- Clean up after each test
- Use realistic data volumes

### Example factory

```typescript
// tests/factories/user.factory.ts
import { faker } from '@faker-js/faker';

export function createUserData(overrides = {}) {
  return {
    email: faker.internet.email(),
    password: 'SecurePass123!',
    firstName: faker.name.firstName(),
    lastName: faker.name.lastName(),
    dateOfBirth: faker.date.birthdate({ min: 18, max: 65, mode: 'age' }),
    country: faker.address.countryCode(),
    ...overrides
  };
}
```

---

## 10. Continuous Improvement

### Quarterly Review
- Analyze test failures
- Identify gaps in coverage
- Update test strategy
- Optimize test performance

### Metrics to track
- Test execution time trend
- Coverage trend
- Bug escape rate
- Test maintenance effort

---

**Статус:** Ready for Implementation  
**Последнее обновление:** 2026-04-06  
**Версия:** 1.0
