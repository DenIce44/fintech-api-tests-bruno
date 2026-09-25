# Session 06 — tags and selective runs

## Run results

| Filter                        | Expected requests | Actual requests | Result |
| ----------------------------- | ----------------: | --------------: | ------ |
| smoke                         |                 5 |                 |        |
| regression                    |                11 |                 |        |
| negative                      |                 1 |                 |        |
| regression excluding negative |                10 |                 |        |
| smoke OR negative             |                 6 |                 |        |

## Dependency check

- Why requests 10 and 11 share the smoke tag: потому что запрос 11 должен выполняться после запроса 10
- What would fail if request 10 were excluded: нет

## Response-time check

- Observed response time:
- Why the 10-second threshold is not a production SLA:

## Reflection

- What became clearer:
- Which filtering mistake I can now diagnose:
