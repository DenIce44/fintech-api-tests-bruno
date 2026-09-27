# Session 09 — Bearer authentication chain

> Не вставляйте в этот файл access token, refresh token, пароль или их фрагменты. Фиксируйте только наличие данных, статусы, связи и результат проверок.

## Configuration

| Check                               | Expected                | Actual | Result |
| ----------------------------------- | ----------------------- | ------ | ------ |
| `authBaseUrl`                       | `https://dummyjson.com` |        |        |
| `authUsername`                      | configured as secret    |        |        |
| `authPassword`                      | configured as secret    |        |        |
| Secret values in `local.bru`        | absent                  |        |        |
| Automatic cookie storage in Desktop | disabled                |        |        |
| Existing `dummyjson.com` cookies    | removed                 |        |        |
| JWT-like strings in tracked files   | absent                  |        |        |

## Authentication flow

| Step | Request                             | Credential used                 | Safe value recorded           | Expected state after step  | Actual state | Result |
| ---: | ----------------------------------- | ------------------------------- | ----------------------------- | -------------------------- | ------------ | ------ |
|    1 | `01-login`                          | username + password             | token fields present; user ID | `verify-session`           |              |        |
|    2 | `02-get-current-user`               | access token                    | matching user ID and username | `load-products`            |              |        |
|    3 | `03-get-protected-products`         | same access token variable      | product count                 | `refresh-token`            |              |        |
|    4 | `04-refresh-token`                  | refresh token                   | new token fields present      | `verify-refreshed-session` |              |        |
|    5 | `05-get-current-user-after-refresh` | refreshed access token variable | matching user ID and username | `complete`                 |              |        |

## Identity checks

| Check                                           | Expected    | Actual | Result |
| ----------------------------------------------- | ----------- | ------ | ------ |
| Login user ID equals first `/auth/me` user ID   | yes         |        |        |
| Login username equals first `/auth/me` username | yes         |        |        |
| User ID remains the same after refresh          | yes         |        |        |
| Username remains the same after refresh         | yes         |        |        |
| `authTokenVersion` after login                  | `login`     |        |        |
| `authTokenVersion` after refresh                | `refreshed` |        |        |

## Negative experiment

- Request executed without Bearer Auth:
- Expected status: `401`
- Actual status:
- Expected message: `Access Token is required`
- Actual message:
- Was the original Bearer Auth restored:
- Did the complete chain pass again after restoration:

## Run results

| Run                      | Expected                              | Actual | Result |
| ------------------------ | ------------------------------------- | ------ | ------ |
| Bruno Desktop folder run | 5 requests pass in order              |        |        |
| CLI folder run           | 5 requests, exit code 0               |        |        |
| CLI `--tags=auth-chain`  | exactly 5 requests pass               |        |        |
| Full regression          | previous suite plus auth chain passes |        |        |

## Security review

| Check                                   | Expected   | Actual | Result |
| --------------------------------------- | ---------- | ------ | ------ |
| Tokens copied manually between requests | no         |        |        |
| Tokens printed with `console.log()`     | no         |        |        |
| Tokens written to notes                 | no         |        |        |
| Cookies able to replace Bearer Auth     | no         |        |        |
| CLI run used `--disable-cookies`        | yes        |        |        |
| JWT pattern found by `rg`               | no matches |        |        |
| Real credentials used                   | no         |        |        |

## Failure diagnostics

### Wrong order

- Request started out of order:
- Expected state:
- Actual state:
- Error message:
- Recovery step:

### Missing runtime token

- Request attempted:
- Missing variable:
- Error message:
- Why the HTTP request should not be sent:

## Reflection

- Why manually pasting a token would invalidate the exercise:
- Why a `200` response alone does not prove the correct user was authenticated:
- Why both access and refresh tokens are replaced after refresh:
- Why token values must not appear in logs or reports:
- How this pattern can be reused for `login → account → transfer`:
