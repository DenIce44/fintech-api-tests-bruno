| Test case ID  | Requirement        | Risk                     | Type              | Input        | Expected                                          |
| ------------- | ------------------ | ------------------------ | ----------------- | ------------ | ------------------------------------------------- |
| `POST-ID-001` | `REQ-POST-GET-001` | `R-POST-01`, `R-POST-05` | happy             | `postId=1`   | `200`, contract публикации, `id=1`, `userId=1`    |
| `POST-ID-002` | `REQ-POST-GET-001` | `R-POST-02`              | boundary          | `postId=100` | `200`, contract публикации, `id=100`, `userId=10` |
| `POST-ID-003` | `REQ-POST-GET-001` | `R-POST-02`, `R-POST-04` | boundary-negative | `postId=101` | `404`, поля публикации отсутствуют                |
| `POST-ID-004` | `REQ-POST-GET-001` | `R-POST-03`, `R-POST-04` | negative          | `postId=0`   | `404`, поля публикации отсутствуют                |
| `POST-ID-005` | `REQ-POST-GET-001` | `R-POST-03`, `R-POST-04` | negative          | `postId=abc` | `404`, поля публикации отсутствуют                |
