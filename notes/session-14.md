## Главное правило занятия

OpenAPI — это исполняемая граница договорённостей, а не каталог URL:

```text
требования и риски занятия 13
              |
              v
     OpenAPI: операции и схемы
       /          |          \
      v           v           v
разработка   Bruno-тесты   документация
      \           |           /
       +----------+----------+
                  |
                  v
        один проверяемый контракт
```

Карта восьми endpoint

|   № | Method и path                       | Operation ID        | Основной результат                | Ключевые требования                                    |
| --: | ----------------------------------- | ------------------- | --------------------------------- | ------------------------------------------------------ |
|   1 | `POST /auth/login`                  | `login`             | `200 AuthToken`                   | `REQ-AUTH-001`                                         |
|   2 | `POST /customers`                   | `createCustomer`    | `201 Customer`                    | `REQ-AUTH-001`                                         |
|   3 | `PATCH /customers/{customerId}/kyc` | `updateCustomerKyc` | `200 Customer`                    | `REQ-KYC-001`                                          |
|   4 | `POST /accounts`                    | `createAccount`     | `201 Account`                     | `REQ-AUTH-001`                                         |
|   5 | `GET /accounts/{accountId}`         | `getAccount`        | `200 Account`                     | `REQ-ACC-001`                                          |
|   6 | `GET /accounts/{accountId}/balance` | `getAccountBalance` | `200 Balance`                     | `REQ-ACC-001`                                          |
|   7 | `POST /transfers`                   | `createTransfer`    | `201` или idempotent replay `200` | `REQ-TRF-001..004`, `REQ-ACC-002`, `REQ-IDEM-001..002` |
|   8 | `GET /transfers/{transferId}`       | `getTransfer`       | `200 Transfer`                    | `REQ-AUTH-001`                                         |

## Requirement coverage review

- Review date: YYYY-MM-DD
- Reviewed files: `docs/requirements.md`, `docs/risk-matrix.md`, `openapi/fintech-api.yaml`
- Missing requirement IDs: none
- Eight operation IDs: confirmed
- Foreign account policy: masking `404`
- Idempotency: `201` first request, `200` identical replay, `409` changed payload
- Money representation: integer minor units
- Open questions: none

## Bruno import review

- Imported file: `openapi/fintech-api.yaml`
- Destination: temporary folder outside the repository
- Imported operations: 8
- Login without Bearer auth: confirmed
- Transfer Idempotency-Key: confirmed
- Main Bruno collection changed: no

```

```
