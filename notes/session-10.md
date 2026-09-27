# Session 10 — Negative authentication scenarios

> Не вставляйте в этот файл access token, refresh token, пароль, JWT payload, заголовок `Authorization`, `Set-Cookie` или их фрагменты. Фиксируйте только тип credential, статус, безопасное сообщение и результат проверки.

## Configuration

| Check | Expected | Actual | Result |
|---|---|---|---|
| `authBaseUrl` | `https://dummyjson.com` | | |
| `authUsername` | configured as secret | | |
| `authPassword` | configured as secret | | |
| Secret values in `local.bru` | absent | | |
| Automatic cookie storage | disabled | | |
| Existing `dummyjson.com` cookies | removed | | |
| JWT-like strings in tracked files | absent | | |

## Decision table

| Scenario | Credential | Expected status | Actual status | Safe message present | Sensitive fields absent | Result |
|---|---|---:|---:|---|---|---|
| Valid-token control | fresh access token | `200` | | n/a | n/a | |
| Invalid credentials | correct username, wrong password | `400` | | | | |
| Missing token | none | `401` | | | | |
| Malformed token | safe invalid literal | `401` | | | | |
| Forbidden contract lab | none; status stub | `403` | | | | |
| Expired token | previously valid access token | `401` | | | | |

## Positive control

| Check | Expected | Actual | Result |
|---|---|---|---|
| Control login status | `200` | | |
| Access token field | present, not recorded | | |
| User ID | positive integer | | |
| Username matches login input | yes | | |
| `/auth/me` status with fresh token | `200` | | |
| `/auth/me` ID matches login | yes | | |
| `/auth/me` username matches login | yes | | |

## Error observations

Записывайте только безопасный текст поля ошибки. Если ответ содержит секрет, не копируйте его; отметьте нарушение отдельно.

| Scenario | Actual safe message | Response is JSON | Profile absent | Tokens absent | Password absent |
|---|---|---|---|---|---|
| Invalid credentials | | | | | |
| Missing token | | | | | |
| Malformed token | | | | | |
| Expired token | | | | | |

## Expiry timeline

| Event | Expected | Actual | Result |
|---|---|---|---|
| Short-lived login | access token issued for 1 minute | | |
| Immediate `/auth/me` | `200` | | |
| Same runtime variable reused | yes | | |
| Approximate wait | at least 65 seconds | | |
| `/auth/me` after wait | `401` | | |
| Error message present | yes | | |
| User profile absent | yes | | |
| `expiryState` after experiment | `complete` | | |

## `403` scope check

| Question | Answer |
|---|---|
| Which authenticated user was used? | |
| Which permission was denied? | |
| Which owned resource was protected? | |
| Does `/http/403/...` prove role enforcement? | |
| What future fintech scenario will provide a real `403` test? | |

Required conclusion:

- What the lab proves:
- What the lab does not prove:

## Run results

| Run | Expected | Actual | Result |
|---|---|---|---|
| Bruno Desktop `03-auth-negative` | requests 1–5 pass in order | | |
| Desktop `auth-contract-lab` | one `403` contract check passes | | |
| CLI `--tags=auth-negative` | exactly 5 requests; exit code 0 | | |
| Full regression | previous suite plus deterministic auth negatives passes | | |
| Expiry manual lab | `200` before and `401` after | | |

## Security review

| Check | Expected | Actual | Result |
|---|---|---|---|
| Tokens copied manually | no | | |
| Tokens printed to console | no | | |
| Tokens written to notes | no | | |
| Correct password placed in request file | no | | |
| Missing-token request authenticated by cookie | no | | |
| CLI used `--disable-cookies` | yes | | |
| Error response exposed profile data | no | | |
| Error response exposed session data | no | | |
| JWT pattern found in tracked files | no matches | | |

## Findings

### Unexpected status

- Scenario:
- Expected status:
- Actual status:
- Was the positive control successful:
- Were cookies disabled:
- Was the correct Auth type used:
- Conclusion:

### Unexpected response contract

- Scenario:
- Missing or excessive field:
- Security impact:
- Proposed expected contract:

### External service limitation

- Observation:
- Why it is not treated as the fintech API requirement:
- How the future local API should behave:

## Reflection

- Why a positive control is required before interpreting a negative result:
- Why `401` and `403` are not interchangeable:
- Why the invalid-login test expects `400` specifically for DummyJSON:
- Why the full error text is not the only regression oracle:
- Why the expiry lab is not part of the deterministic regression tag:
- How the same runtime token was proven valid before it expired:
- Why the `403` status stub is not an authorization test:
- What roles, owners and resources are needed for a real fintech `403` scenario:
