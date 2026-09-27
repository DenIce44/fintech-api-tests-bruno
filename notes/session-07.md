# Session 07 — OpenAPI import and comparison

## Import result

- Bruno Desktop version:
- Bruno CLI version:
- Imported operations:
- Folder grouping selected:
- How the OpenAPI tag was represented after import:
- Did parameter examples import correctly:

## Contract comparison

- What OpenAPI described well:
- What the importer generated:
- What still required manual tests:
- One mismatch or limitation found:

## Run results

| Check                          | Expected                  | Actual | Result |
| ------------------------------ | ------------------------- | ------ | ------ |
| Imported `GET /posts/{postId}` | 200 and valid `Post`      |        |        |
| Imported `GET /posts?userId=1` | 200 and non-empty array   |        |        |
| CLI import                     | `.bru` collection created |        |        |
| Main regression                | 11 requests pass          |        |        |

## Reflection

- Why import is not the same as test coverage:
- Which part of an OpenAPI operation I can now read without help:

```

```
