# Session 05 — variables and request chaining

## Variable map

| Variable           | Scope       | Created by             | Used by                   | Lifetime    |
| ------------------ | ----------- | ---------------------- | ------------------------- | ----------- |
| baseUrl            | environment | local environment      | requests 10–11            | saved       |
| referencePrefix    | request     | request 10 Vars        | request 10 pre-script     | saved       |
| transferRef        | runtime     | request 10 pre-script  | request 10 body and tests | current run |
| createdTransferRef | runtime     | request 10 post-script | request 11                | current run |

## Script order

1. Pre-request script creates transferRef.
2. Request 10 sends it.
3. Post-response script reads the echoed value.
4. Runtime variable createdTransferRef stores the value.
5. Request 11 uses and verifies it.

## Failure experiment

- What happened when request 11 ran first:
- Which assertion failed:
- Why this is the expected diagnostic:

## CLI result

- Date:
- Passed requests:
- Failed requests:
