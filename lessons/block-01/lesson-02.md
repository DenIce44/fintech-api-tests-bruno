# Блок 1. Занятие 2 — POST-запрос, JSON-тело, параметры и заголовки

**Продолжительность:** 90 минут  
**Формат:** 20% объяснение, 80% практика  
**Результат занятия:** в Bruno-коллекции есть проверенный `POST`-запрос с query-параметрами, пользовательским заголовком и JSON-телом; запрос успешно запускается в Bruno Desktop и Bruno CLI.

## Что вы научитесь делать

К концу занятия вы сможете:

- объяснить разницу между HTTP-методом, URL, query-параметрами, заголовками и телом запроса;
- отправить JSON в `POST`-запросе;
- правильно указать `Content-Type: application/json`;
- различать заголовки ответа и заголовки исходного запроса, которые вернул echo-сервис;
- проверять строку, число, boolean, массив и вложенный объект в JSON;
- находить ошибку, вызванную различием между числом и строкой;
- запустить два запроса коллекции из Bruno CLI;
- сохранить результат отдельным Git-коммитом.

## Что понадобится

- проект `fintech-api-tests-bruno` после занятия 1;
- Bruno Desktop;
- локально установленный Bruno CLI;
- environment `local` с переменной `baseUrl`, равной `https://postman-echo.com`;
- терминал и Git.

> Как и на занятии 1, используется публичный Postman Echo API. Он возвращает детали полученного запроса и не создаёт реальный перевод, счёт или другой финансовый объект. Не отправляйте реальные персональные или платёжные данные.

## План занятия

| Время | Этап | Результат |
|---:|---|---|
| 0–10 мин | Повторение HTTP и знакомство с `POST` | Понимаете назначение метода, параметров, заголовков и тела |
| 10–25 мин | Создание `POST`-запроса | Запрос сохранён в коллекции и возвращает `200` |
| 25–38 мин | Query-параметры и заголовки | Сервер получил параметры и пользовательский заголовок |
| 38–53 мин | JSON-тело | Отправлены данные пяти основных JSON-типов |
| 53–68 мин | Assertions и JavaScript-тесты | Проверены статус, параметры, заголовок и тело |
| 68–76 мин | Намеренно падающий тест | Найдена ошибка типа `number` против `string` |
| 76–83 мин | Запуск коллекции через CLI | Оба учебных запроса проходят из терминала |
| 83–90 мин | Итоги и коммит | Заполнены заметки и создан коммит |

## 1. Разберите устройство `POST`-запроса — 10 минут

На занятии будет отправлен запрос:

```http
POST https://postman-echo.com/post?lesson=2&mode=practice
Content-Type: application/json
X-Lesson: 2

{
  "operation": "create-transfer-draft",
  "amountMinor": 1250,
  "currency": "USD",
  "approved": false,
  "tags": ["lesson-2", "practice"]
}
```

Разберите его по частям:

1. `POST` — метод. Обычно он передаёт данные серверу для обработки или создания ресурса.
2. `/post` — путь к ресурсу на учебном сервере.
3. `lesson=2` и `mode=practice` — query-параметры. Они находятся в URL и передаются как строки.
4. `Content-Type: application/json` сообщает серверу, что тело запроса записано в JSON.
5. `X-Lesson: 2` — пользовательский заголовок. Префикс `X-` используется здесь только как понятное учебное имя.
6. JSON-тело содержит данные разных типов: строку, число, boolean и массив строк.

Echo-сервис не выполняет операцию `create-transfer-draft`. Он возвращает детали запроса в ответе, чтобы можно было проверить, что именно получил сервер.

Обратите внимание на поле `amountMinor`. Значение `1250` означает 1250 минимальных денежных единиц, например 12,50 основной единицы валюты. В учебном финтех-проекте это безопаснее, чем начинать вычисления с неточным числом с плавающей точкой `12.50`.

## 2. Подготовьте проект — 5 минут

Откройте проект после занятия 1 и убедитесь, что:

- коллекция `fintech-api` открывается в Bruno;
- выбран environment `local`;
- запрос `00-learning-basics/01-echo-get` сохранён и проходит;
- в environment есть `baseUrl = https://postman-echo.com`.

Если первый запрос не проходит, сначала исправьте его по инструкции занятия 1. Новый запрос будет использовать ту же коллекцию и тот же environment.

## 3. Создайте `POST`-запрос — 10 минут

1. В папке `00-learning-basics` создайте новый HTTP-запрос.
2. Назовите его `02-echo-post`.
3. Выберите метод `POST`.
4. В поле URL введите `{{baseUrl}}/post`.
5. Сохраните запрос.

Пока не добавляйте тело. Отправьте запрос один раз и изучите ответ.

Ожидаемый результат:

- статус — `200 OK`;
- тело ответа — JSON;
- в ответе есть поля `args`, `data`, `files`, `form`, `headers`, `json` и `url`;
- некоторые поля пока пустые или равны `null`, потому что запрос ещё не содержит данных.

> Не путайте поведение учебного echo-маршрута с контрактом реального API. Реальный `POST`, создающий ресурс, часто возвращает `201 Created`, но `/post` у Postman Echo отвечает `200 OK`, потому что только отражает запрос.

## 4. Добавьте query-параметры — 8 минут

Откройте вкладку **Params** и добавьте две включённые строки:

| Name | Value | Назначение |
|---|---|---|
| `lesson` | `2` | номер занятия |
| `mode` | `practice` | режим учебного запроса |

Bruno должен собрать URL:

```text
{{baseUrl}}/post?lesson=2&mode=practice
```

Отправьте запрос и найдите в ответе:

```json
{
  "args": {
    "lesson": "2",
    "mode": "practice"
  }
}
```

Важно: значение `lesson` в `args` — строка `"2"`, даже если в таблице Params было введено `2`. Query-параметры передаются в URL как текст. API может преобразовать их в число, но тест не должен предполагать такое преобразование без контракта.

## 5. Добавьте заголовки — 5 минут

Откройте вкладку **Headers** и добавьте две включённые строки:

| Name | Value | Назначение |
|---|---|---|
| `Content-Type` | `application/json` | формат тела запроса |
| `X-Lesson` | `2` | учебный пользовательский заголовок |

После отправки запроса откройте поле `headers` внутри JSON-тела ответа. Echo-сервис обычно приводит имена заголовков к нижнему регистру, поэтому ищите:

```json
{
  "headers": {
    "content-type": "application/json",
    "x-lesson": "2"
  }
}
```

Здесь есть два разных набора заголовков:

- `res.headers` — заголовки, которые сервер отправил в HTTP-ответе;
- `res.body.headers` — заголовки вашего исходного запроса, которые echo-сервис поместил внутрь JSON-ответа.

Проверка `res.headers['content-type']` не доказывает, что клиент отправил JSON. Она доказывает, что сам ответ имеет JSON-формат. Отправленный `Content-Type` нужно искать в `res.body.headers['content-type']`.

## 6. Добавьте JSON-тело — 15 минут

1. Откройте вкладку **Body**.
2. Выберите тип тела **JSON**.
3. Вставьте данные:

```json
{
  "operation": "create-transfer-draft",
  "amountMinor": 1250,
  "currency": "USD",
  "approved": false,
  "tags": ["lesson-2", "practice"]
}
```

4. Сохраните и отправьте запрос.

В поле `json` ответа ожидается объект с теми же значениями:

```json
{
  "json": {
    "operation": "create-transfer-draft",
    "amountMinor": 1250,
    "currency": "USD",
    "approved": false,
    "tags": ["lesson-2", "practice"]
  }
}
```

Проверьте тип каждого значения:

| Поле | JSON-тип | Пример значения |
|---|---|---|
| `operation` | string | `"create-transfer-draft"` |
| `amountMinor` | number | `1250` |
| `currency` | string | `"USD"` |
| `approved` | boolean | `false` |
| `tags` | array | `["lesson-2", "practice"]` |

Кавычки меняют тип. `1250` — число, а `"1250"` — строка. Аналогично, `false` — boolean, а `"false"` — строка.

## 7. Добавьте декларативные проверки — 8 минут

Откройте вкладку **Assert** и добавьте проверки:

| Expression | Operator | Value | Что проверяет |
|---|---|---|---|
| `res.status` | `equals` | `200` | echo-сервис обработал запрос |
| `res.headers['content-type']` | `contains` | `application/json` | ответ имеет формат JSON |
| `res.body.args.lesson` | `equals` | `2` | query-параметр вернулся как строка `"2"` |
| `res.body.args.mode` | `equals` | `practice` | второй query-параметр передан |
| `res.body.headers['x-lesson']` | `equals` | `2` | пользовательский заголовок передан |
| `res.body.json.operation` | `equals` | `create-transfer-draft` | операция передана без изменения |
| `res.body.json.amountMinor` | `equals` | `1250` | сумма передана числом с нужным значением |
| `res.body.json.currency` | `equals` | `USD` | код валюты передан без изменения |
| `res.body.json.approved` | `equals` | `false` | boolean-значение передано без изменения |

Отправьте запрос. Все проверки должны быть зелёными.

Если проверка `approved` не проходит из-за того, что введённое в таблице значение интерпретировалось как текст, оставьте проверку наличия поля в **Assert**, а точный boolean-тип и значение проверьте JavaScript-тестом из следующего раздела. Интерфейс и способ ввода типизированных ожидаемых значений могут различаться между версиями Bruno.

## 8. Добавьте JavaScript-тесты — 7 минут

Откройте вкладку **Tests** и добавьте:

```javascript
test("status is 200 and response is JSON", function () {
  expect(res.getStatus()).to.eql(200);
  expect(res.getHeader("content-type")).to.include("application/json");
});

test("query parameters are echoed as strings", function () {
  const body = res.getBody();

  expect(body.args.lesson).to.eql("2");
  expect(body.args.mode).to.eql("practice");
});

test("custom request header is echoed", function () {
  const body = res.getBody();

  expect(body.headers["x-lesson"]).to.eql("2");
});

test("JSON body preserves values and types", function () {
  const payload = res.getBody().json;

  expect(payload.operation).to.eql("create-transfer-draft");
  expect(payload.amountMinor).to.eql(1250);
  expect(payload.amountMinor).to.be.a("number");
  expect(payload.currency).to.eql("USD");
  expect(payload.approved).to.eql(false);
  expect(payload.approved).to.be.a("boolean");
  expect(payload.tags).to.be.an("array");
  expect(payload.tags).to.include.members(["lesson-2", "practice"]);
});
```

Отправьте запрос ещё раз. Декларативные assertions и JavaScript-тесты должны пройти.

Обратите внимание: проверка значения и проверка типа отвечают на разные вопросы. `amountMinor === 1250` фиксирует ожидаемое значение, а `amountMinor` имеет тип `number` — ожидаемый контракт данных.

## 9. Создайте и исправьте падение типа данных — 8 минут

1. В JSON-теле временно замените число:

   ```json
   "amountMinor": 1250
   ```

   на строку:

   ```json
   "amountMinor": "1250"
   ```

2. Отправьте запрос.
3. Найдите упавший JavaScript-тест.
4. Сравните значения и типы:

   - визуально и число, и строка содержат `1250`;
   - фактический тип теперь `string`;
   - тест ожидает `number`;
   - проблема находится в тестовых данных запроса, а не в echo-сервисе.

5. Верните `"amountMinor": 1250` без кавычек.
6. Повторно отправьте запрос и убедитесь, что все проверки снова зелёные.

Не исправляйте такое падение ослаблением теста до нестрогого сравнения. Если контракт требует число, тест должен продолжать отличать число от строки.

## 10. Запустите коллекцию через Bruno CLI — 7 минут

Сохраните оба запроса в Bruno. Из корня учебного проекта перейдите в папку, где непосредственно находится `bruno.json`, и запустите коллекцию рекурсивно:

```bash
cd bruno/fintech-api
npx bru run -r --env local
cd ../..
```

Ожидаемый результат:

- выполнены `01-echo-get` и `02-echo-post`;
- оба запроса завершились статусом `200`;
- assertions и JavaScript-тесты прошли;
- команда завершилась без ошибки.

Если CLI выполнил только один запрос:

- сохраните новый запрос в Bruno;
- проверьте, что файл `02-echo-post.bru` появился в `00-learning-basics`;
- убедитесь, что указан флаг `-r`.

Если тесты проходят в Bruno Desktop, но падают в CLI, сначала сравните выбранный environment и сохранённую версию `.bru`-файла. CLI читает файлы с диска, а не несохранённое состояние редактора.

## 11. Зафиксируйте результат — 7 минут

Создайте файл `notes/session-02.md` и заполните его своими словами:

```markdown
# Занятие 2

## Из чего состоял мой POST-запрос

- Метод: ...
- Путь: ...
- Query-параметры: ...
- Заголовки: ...
- Тело: ...

## Чем отличаются число и строка в JSON

- Число: ...
- Строка: ...
- Как тест обнаружил разницу: ...

## Что осталось непонятно

- ...
```

Проверьте изменения:

```bash
git status
git diff --check
```

Убедитесь, что в коммит не попали секреты или реальные пользовательские данные. Затем выполните:

```bash
git add bruno/fintech-api/00-learning-basics/02-echo-post.bru notes/session-02.md
git commit -m "test: add POST echo request"
git status
```

Если Bruno изменил дополнительный служебный файл коллекции, сначала изучите его через `git diff`, затем добавьте в тот же коммит, если изменение относится к новому запросу.

## Структура проекта после занятия

Основная структура должна выглядеть так:

```text
fintech-api-tests-bruno/
├── bruno/
│   └── fintech-api/
│       ├── 00-learning-basics/
│       │   ├── 01-echo-get.bru
│       │   └── 02-echo-post.bru
│       ├── environments/
│       │   └── local.bru
│       └── bruno.json
├── notes/
│   ├── session-01.md
│   └── session-02.md
├── .gitignore
├── AGENTS.md
├── package.json
└── package-lock.json
```

## Чек-лист готовности

Занятие завершено, если каждый пункт можно отметить:

- [ ] Запрос `02-echo-post` сохранён в папке `00-learning-basics`.
- [ ] URL использует `{{baseUrl}}`, а не жёстко заданный домен.
- [ ] Query-параметры `lesson` и `mode` добавлены через Params.
- [ ] Заголовки `Content-Type` и `X-Lesson` отправляются.
- [ ] JSON-тело содержит string, number, boolean и array.
- [ ] Вы можете показать, где в ответе находятся `args`, `headers` и `json`.
- [ ] Вы понимаете разницу между `res.headers` и `res.body.headers`.
- [ ] Декларативные assertions проходят.
- [ ] JavaScript-тесты проверяют не только значения, но и типы.
- [ ] Вы увидели падение из-за `1250` против `"1250"` и исправили данные.
- [ ] Оба учебных запроса проходят через Bruno CLI.
- [ ] Заполнен `notes/session-02.md`.
- [ ] Создан коммит `test: add POST echo request`.

## Контрольные вопросы

Ответьте без подсказки:

1. Чем query-параметр отличается от поля JSON-тела?
2. Зачем нужен заголовок `Content-Type`?
3. Почему `lesson=2` возвращается в `args` как строка `"2"`?
4. Чем `res.headers['content-type']` отличается от `res.body.headers['content-type']` в этом запросе?
5. Почему `1250` и `"1250"` не считаются одним и тем же значением контракта?
6. Что проверяет `expect(payload.tags).to.be.an("array")`?
7. Почему статус `200` сам по себе не доказывает, что сервер получил правильное тело?
8. Почему реальный создающий `POST` может возвращать `201`, хотя Postman Echo возвращает `200`?

## Если осталось время

Добавьте в тело вложенный объект:

```json
"metadata": {
  "source": "bruno",
  "attempt": 1
}
```

Затем добавьте JavaScript-тест:

```javascript
test("nested metadata object is preserved", function () {
  const metadata = res.getBody().json.metadata;

  expect(metadata).to.be.an("object");
  expect(metadata.source).to.eql("bruno");
  expect(metadata.attempt).to.eql(1);
});
```

Объясните, как dot notation позволяет пройти от корня ответа к вложенному значению: `body → json → metadata → source`.

## Официальные материалы

- [Обзор REST-запросов в Bruno](https://docs.usebruno.com/send-requests/REST/overview)
- [Параметры запроса в Bruno](https://docs.usebruno.com/send-requests/REST/parameters)
- [Заголовки запроса в Bruno](https://docs.usebruno.com/send-requests/REST/req-header)
- [Данные тела запроса в Bruno](https://docs.usebruno.com/send-requests/REST/body-data)
- [Assertions в Bruno](https://docs.usebruno.com/testing/tests/assertions)
- [Тестовые скрипты в Bruno](https://docs.usebruno.com/testing/tests/introduction)
- [Postman Echo API](https://learning.postman.com/docs/developer/echo-api/)
