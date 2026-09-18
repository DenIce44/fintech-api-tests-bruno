# Занятие 2

## Из чего состоял мой POST-запрос

- Метод: post
- Путь: /post
- Query-параметры: lesson, mode
- Заголовки: Content-Type, X-Lesson
- Тело: "{
  "operation": "create-transfer-draft",
  "amountMinor": 1250,
  "currency": "USD",
  "approved": false,
  "tags": ["lesson-2", "practice"]
  }"

## Чем отличаются число и строка в JSON

- Число: 1
- Строка: "1"
- Как тест обнаружил разницу: ...
