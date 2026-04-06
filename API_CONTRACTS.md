# API Контракты между микросервисами

**Проект:** Prize Competitions Platform  
**Масштаб:** Enterprise (100K+ пользователей)  
**Дата:** 2026-04-06

---

## 1. API Gateway

### Base URL
```
http://api.gaming-platform.com/v1
```

### Authentication
Все запросы должны содержать JWT токен в заголовке:
```
Authorization: Bearer <jwt_token>
```

### Response Format
```json
{
  "success": true,
  "data": {},
  "error": null,
  "timestamp": "2026-04-06T20:20:43.929Z"
}
```

### Error Response
```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "ERROR_CODE",
    "message": "Error description",
    "details": {}
  },
  "timestamp": "2026-04-06T20:20:43.929Z"
}
```

---

## 2. Auth Service

### POST /auth/register
Регистрация нового пользователя

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123!",
  "firstName": "John",
  "lastName": "Doe",
  "dateOfBirth": "1990-01-15",
  "country": "US"
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "userId": "uuid",
    "email": "user@example.com",
    "accessToken": "jwt_token",
    "refreshToken": "refresh_token",
    "expiresIn": 900
  }
}
```

---

### POST /auth/login
Вход в систему

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123!"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "userId": "uuid",
    "accessToken": "jwt_token",
    "refreshToken": "refresh_token",
    "expiresIn": 900
  }
}
```

---

### POST /auth/refresh-token
Обновление access token

**Request:**
```json
{
  "refreshToken": "refresh_token"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "accessToken": "new_jwt_token",
    "expiresIn": 900
  }
}
```

---

### POST /auth/logout
Выход из системы

**Response (200):**
```json
{
  "success": true,
  "data": null
}
```

---

### POST /auth/verify
Проверка токена

**Response (200):**
```json
{
  "success": true,
  "data": {
    "valid": true,
    "userId": "uuid",
    "email": "user@example.com"
  }
}
```

---

## 3. User Service

### GET /users/{userId}
Получить профиль пользователя

**Response (200):**
```json
{
  "success": true,
  "data": {
    "userId": "uuid",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "dateOfBirth": "1990-01-15",
    "country": "US",
    "kycStatus": "verified",
    "balance": 1500.50,
    "createdAt": "2026-01-01T00:00:00Z",
    "updatedAt": "2026-04-06T20:20:43.929Z"
  }
}
```

---

### PUT /users/{userId}
Обновить профиль пользователя

**Request:**
```json
{
  "firstName": "John",
  "lastName": "Doe",
  "preferences": {
    "emailNotifications": true,
    "pushNotifications": false
  }
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "userId": "uuid",
    "firstName": "John",
    "lastName": "Doe",
    "preferences": {
      "emailNotifications": true,
      "pushNotifications": false
    }
  }
}
```

---

### GET /users/{userId}/balance
Получить баланс пользователя

**Response (200):**
```json
{
  "success": true,
  "data": {
    "userId": "uuid",
    "balance": 1500.50,
    "currency": "USD",
    "lastUpdated": "2026-04-06T20:20:43.929Z"
  }
}
```

---

### GET /users/{userId}/transactions
Получить историю транзакций

**Query Parameters:**
- `limit`: 20 (default)
- `offset`: 0 (default)
- `type`: deposit|withdrawal|game_win|game_loss

**Response (200):**
```json
{
  "success": true,
  "data": {
    "transactions": [
      {
        "transactionId": "uuid",
        "type": "deposit",
        "amount": 100.00,
        "currency": "USD",
        "status": "completed",
        "createdAt": "2026-04-06T20:20:43.929Z"
      }
    ],
    "total": 150,
    "limit": 20,
    "offset": 0
  }
}
```

---

## 4. Game Service

### POST /games/start
Начать новую игру

**Request:**
```json
{
  "gameType": "prize_competition",
  "competitionId": "uuid",
  "betAmount": 10.00
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "gameId": "uuid",
    "userId": "uuid",
    "gameType": "prize_competition",
    "status": "in_progress",
    "betAmount": 10.00,
    "startedAt": "2026-04-06T20:20:43.929Z"
  }
}
```

---

### GET /games/{gameId}
Получить информацию об игре

**Response (200):**
```json
{
  "success": true,
  "data": {
    "gameId": "uuid",
    "userId": "uuid",
    "gameType": "prize_competition",
    "status": "completed",
    "betAmount": 10.00,
    "winAmount": 50.00,
    "result": {
      "outcome": "win",
      "details": {}
    },
    "startedAt": "2026-04-06T20:20:43.929Z",
    "completedAt": "2026-04-06T20:21:43.929Z"
  }
}
```

---

### POST /games/{gameId}/play
Выполнить действие в игре

**Request:**
```json
{
  "action": "spin",
  "parameters": {}
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "gameId": "uuid",
    "status": "completed",
    "result": {
      "outcome": "win",
      "winAmount": 50.00
    }
  }
}
```

---

### GET /games/{gameId}/result
Получить результат игры

**Response (200):**
```json
{
  "success": true,
  "data": {
    "gameId": "uuid",
    "outcome": "win",
    "winAmount": 50.00,
    "details": {}
  }
}
```

---

### GET /games/leaderboard
Получить таблицу лидеров

**Query Parameters:**
- `limit`: 100 (default)
- `period`: daily|weekly|monthly|all

**Response (200):**
```json
{
  "success": true,
  "data": {
    "leaderboard": [
      {
        "rank": 1,
        "userId": "uuid",
        "userName": "Player1",
        "totalWinnings": 5000.00,
        "gamesPlayed": 150
      }
    ],
    "period": "weekly",
    "generatedAt": "2026-04-06T20:20:43.929Z"
  }
}
```

---

## 5. Payment Service

### POST /payments/deposit
Пополнить счёт

**Request:**
```json
{
  "userId": "uuid",
  "amount": 100.00,
  "currency": "USD",
  "paymentMethod": "credit_card",
  "paymentMethodId": "pm_xxx"
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "transactionId": "uuid",
    "userId": "uuid",
    "amount": 100.00,
    "currency": "USD",
    "status": "pending",
    "createdAt": "2026-04-06T20:20:43.929Z"
  }
}
```

---

### POST /payments/withdraw
Вывести средства

**Request:**
```json
{
  "userId": "uuid",
  "amount": 50.00,
  "currency": "USD",
  "bankAccount": "xxx"
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "transactionId": "uuid",
    "userId": "uuid",
    "amount": 50.00,
    "currency": "USD",
    "status": "pending",
    "createdAt": "2026-04-06T20:20:43.929Z"
  }
}
```

---

### GET /payments/{transactionId}
Получить статус платежа

**Response (200):**
```json
{
  "success": true,
  "data": {
    "transactionId": "uuid",
    "userId": "uuid",
    "amount": 100.00,
    "currency": "USD",
    "status": "completed",
    "type": "deposit",
    "createdAt": "2026-04-06T20:20:43.929Z",
    "completedAt": "2026-04-06T20:21:43.929Z"
  }
}
```

---

### POST /payments/{transactionId}/refund
Вернуть платёж

**Request:**
```json
{
  "reason": "user_request"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "transactionId": "uuid",
    "refundId": "uuid",
    "status": "refunded",
    "amount": 100.00
  }
}
```

---

## 6. Prize Competition Service

### POST /competitions
Создать конкурс

**Request:**
```json
{
  "name": "Spring Prize Competition",
  "description": "Win amazing prizes",
  "startDate": "2026-04-10T00:00:00Z",
  "endDate": "2026-04-30T23:59:59Z",
  "entryFee": 10.00,
  "maxParticipants": 1000,
  "prizes": [
    {
      "rank": 1,
      "amount": 5000.00
    }
  ]
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "competitionId": "uuid",
    "name": "Spring Prize Competition",
    "status": "active",
    "participantCount": 0,
    "createdAt": "2026-04-06T20:20:43.929Z"
  }
}
```

---

### GET /competitions/{competitionId}
Получить информацию о конкурсе

**Response (200):**
```json
{
  "success": true,
  "data": {
    "competitionId": "uuid",
    "name": "Spring Prize Competition",
    "description": "Win amazing prizes",
    "status": "active",
    "startDate": "2026-04-10T00:00:00Z",
    "endDate": "2026-04-30T23:59:59Z",
    "entryFee": 10.00,
    "participantCount": 250,
    "maxParticipants": 1000,
    "prizes": []
  }
}
```

---

### POST /competitions/{competitionId}/join
Присоединиться к конкурсу

**Request:**
```json
{
  "userId": "uuid"
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "participationId": "uuid",
    "competitionId": "uuid",
    "userId": "uuid",
    "joinedAt": "2026-04-06T20:20:43.929Z"
  }
}
```

---

### GET /competitions/{competitionId}/results
Получить результаты конкурса

**Response (200):**
```json
{
  "success": true,
  "data": {
    "competitionId": "uuid",
    "status": "completed",
    "results": [
      {
        "rank": 1,
        "userId": "uuid",
        "userName": "Winner1",
        "prizeAmount": 5000.00
      }
    ]
  }
}
```

---

### POST /competitions/{competitionId}/claim-prize
Получить приз

**Request:**
```json
{
  "userId": "uuid"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "claimId": "uuid",
    "competitionId": "uuid",
    "userId": "uuid",
    "prizeAmount": 5000.00,
    "status": "claimed",
    "claimedAt": "2026-04-06T20:20:43.929Z"
  }
}
```

---

## 7. RNG Service

### POST /rng/generate
Генерировать случайное число

**Request:**
```json
{
  "min": 1,
  "max": 100,
  "count": 1
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "rngId": "uuid",
    "numbers": [42],
    "seed": "seed_hash",
    "timestamp": "2026-04-06T20:20:43.929Z"
  }
}
```

---

### GET /rng/{rngId}/audit
Получить audit trail для RNG

**Response (200):**
```json
{
  "success": true,
  "data": {
    "rngId": "uuid",
    "seed": "seed_hash",
    "algorithm": "ChaCha20",
    "timestamp": "2026-04-06T20:20:43.929Z",
    "verified": true
  }
}
```

---

### POST /rng/verify
Проверить RNG результат

**Request:**
```json
{
  "rngId": "uuid",
  "seed": "seed_hash"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "valid": true,
    "rngId": "uuid"
  }
}
```

---

## 8. Lottery Service

### POST /lotteries
Создать лотерею

**Request:**
```json
{
  "name": "Weekly Lottery",
  "ticketPrice": 5.00,
  "drawDate": "2026-04-13T20:00:00Z",
  "maxTickets": 10000,
  "prizes": [
    {
      "position": 1,
      "amount": 50000.00
    }
  ]
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "lotteryId": "uuid",
    "name": "Weekly Lottery",
    "status": "active",
    "ticketsSold": 0,
    "createdAt": "2026-04-06T20:20:43.929Z"
  }
}
```

---

### POST /lotteries/{lotteryId}/buy-ticket
Купить билет лотереи

**Request:**
```json
{
  "userId": "uuid",
  "quantity": 5
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "purchaseId": "uuid",
    "lotteryId": "uuid",
    "userId": "uuid",
    "ticketCount": 5,
    "totalCost": 25.00,
    "tickets": ["ticket_1", "ticket_2"],
    "purchasedAt": "2026-04-06T20:20:43.929Z"
  }
}
```

---

### POST /lotteries/{lotteryId}/draw
Провести розыгрыш

**Response (200):**
```json
{
  "success": true,
  "data": {
    "drawId": "uuid",
    "lotteryId": "uuid",
    "winners": [
      {
        "position": 1,
        "ticketId": "ticket_1",
        "userId": "uuid",
        "prizeAmount": 50000.00
      }
    ],
    "drawnAt": "2026-04-06T20:20:43.929Z"
  }
}
```

---

### GET /lotteries/{lotteryId}/winners
Получить список победителей

**Response (200):**
```json
{
  "success": true,
  "data": {
    "lotteryId": "uuid",
    "winners": [
      {
        "position": 1,
        "userId": "uuid",
        "userName": "Winner1",
        "prizeAmount": 50000.00
      }
    ]
  }
}
```

---

## 9. Instant Win Service

### POST /instant-win/play
Играть в мгновенную игру

**Request:**
```json
{
  "userId": "uuid",
  "gameType": "scratch_card",
  "betAmount": 5.00
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "gameId": "uuid",
    "userId": "uuid",
    "gameType": "scratch_card",
    "betAmount": 5.00,
    "status": "in_progress",
    "startedAt": "2026-04-06T20:20:43.929Z"
  }
}
```

---

### GET /instant-win/{gameId}/result
Получить результат игры

**Response (200):**
```json
{
  "success": true,
  "data": {
    "gameId": "uuid",
    "outcome": "win",
    "winAmount": 25.00,
    "multiplier": 5,
    "completedAt": "2026-04-06T20:20:43.929Z"
  }
}
```

---

### POST /instant-win/{gameId}/claim
Получить выигрыш

**Response (200):**
```json
{
  "success": true,
  "data": {
    "claimId": "uuid",
    "gameId": "uuid",
    "winAmount": 25.00,
    "status": "claimed",
    "claimedAt": "2026-04-06T20:20:43.929Z"
  }
}
```

---

## 10. Notification Service

### POST /notifications/send
Отправить уведомление

**Request:**
```json
{
  "userId": "uuid",
  "type": "email",
  "template": "game_win",
  "data": {
    "winAmount": 100.00,
    "gameType": "lottery"
  }
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "notificationId": "uuid",
    "userId": "uuid",
    "type": "email",
    "status": "sent",
    "sentAt": "2026-04-06T20:20:43.929Z"
  }
}
```

---

## 11. Analytics Service

### GET /analytics/dashboard
Получить аналитический dashboard

**Query Parameters:**
- `period`: daily|weekly|monthly
- `startDate`: ISO 8601
- `endDate`: ISO 8601

**Response (200):**
```json
{
  "success": true,
  "data": {
    "period": "daily",
    "metrics": {
      "activeUsers": 5000,
      "totalRevenue": 50000.00,
      "totalWinnings": 45000.00,
      "gamesPlayed": 25000,
      "conversionRate": 0.15
    },
    "generatedAt": "2026-04-06T20:20:43.929Z"
  }
}
```

---

## 12. Audit Service

### GET /audit/logs
Получить audit logs

**Query Parameters:**
- `userId`: uuid (optional)
- `action`: string (optional)
- `limit`: 100 (default)
- `offset`: 0 (default)

**Response (200):**
```json
{
  "success": true,
  "data": {
    "logs": [
      {
        "logId": "uuid",
        "userId": "uuid",
        "action": "game_started",
        "details": {},
        "timestamp": "2026-04-06T20:20:43.929Z"
      }
    ],
    "total": 1000,
    "limit": 100,
    "offset": 0
  }
}
```

---

## 13. Config Service

### GET /config/feature-flags
Получить feature flags

**Response (200):**
```json
{
  "success": true,
  "data": {
    "flags": {
      "new_game_enabled": true,
      "maintenance_mode": false,
      "instant_win_multiplier": 5
    }
  }
}
```

---

## 14. Error Codes

| Code | HTTP | Description |
|------|------|-------------|
| INVALID_REQUEST | 400 | Invalid request parameters |
| UNAUTHORIZED | 401 | Missing or invalid authentication |
| FORBIDDEN | 403 | Insufficient permissions |
| NOT_FOUND | 404 | Resource not found |
| CONFLICT | 409 | Resource already exists |
| RATE_LIMITED | 429 | Too many requests |
| INTERNAL_ERROR | 500 | Internal server error |
| SERVICE_UNAVAILABLE | 503 | Service temporarily unavailable |

---

## 15. Rate Limiting

- **Default:** 100 requests per minute per user
- **Auth endpoints:** 10 requests per minute per IP
- **Payment endpoints:** 5 requests per minute per user
- **RNG endpoints:** 1000 requests per minute per service

---

**Статус:** Ready for Implementation  
**Последнее обновление:** 2026-04-06  
**Версия:** 1.0
