# Session 08 — multi-step response data chain

## Data flow

| Step | Request                | Value extracted from response | Runtime variable     | Used by                   |
| ---: | ---------------------- | ----------------------------- | -------------------- | ------------------------- |
|    1 | `01-find-user`         |                               | `chainUserId`        | step 2 query              |
|    2 | `02-get-user-posts`    |                               | `chainPostId`        | step 3 path               |
|    3 | `03-get-post-comments` |                               | `chainCommentId`     | step 4 path               |
|    3 | `03-get-post-comments` |                               | `chainCommentPostId` | step 4 relationship check |
|    3 | `03-get-post-comments` |                               | `chainCommentEmail`  | step 4 identity check     |

## State transitions

|    After step | Expected state  | Actual state | Result |
| ------------: | --------------- | ------------ | ------ |
| pre-request 1 | `lookup-user`   |              |        |
|             1 | `load-posts`    |              |        |
|             2 | `load-comments` |              |        |
|             3 | `load-comment`  |              |        |
|             4 | `complete`      |              |        |

## Run results

| Run                      | Expected                                    | Actual | Result |
| ------------------------ | ------------------------------------------- | ------ | ------ |
| Bruno Desktop folder run | 4 requests pass in order                    |        |        |
| CLI folder run           | 4 requests, exit code 0                     |        |        |
| CLI `--tags=chain`       | exactly 4 requests pass                     |        |        |
| Full regression          | previous suite plus 4 chain requests passes |        |        |

## Failure experiments

### Wrong order

- Request started out of order:
- State expected by the request:
- Actual state:
- Error message:
- Why no HTTP request should be sent:

### Corrupted intermediate value

- Variable changed: chainCommentEmail
- Temporary value: "Eliseo@gardner.biz"
- Check that detected the mismatch:
- Error message:
- How the valid state was restored:

## Reflection

- Why hard-coding the discovered IDs would invalidate the exercise:
- Why response validation must happen before `bru.setVar()`:
- Which final assertion proves the whole `user → post → comment` relationship:
- How this pattern can be reused for `login → customer → account → transfer`:
