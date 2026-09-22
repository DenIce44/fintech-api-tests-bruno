# Блок 1. Занятие 3 — ресурсы по ID, PUT, PATCH, DELETE и коды ответа

**Продолжительность:** 90 минут  
**Формат:** 20% объяснение, 80% практика  
**Результат занятия:** в Bruno-коллекции есть пять проверенных запросов к ресурсу по ID: получение записи, полная и частичная замена, удаление и ожидаемый сценарий `404 Not Found`; все запросы запускаются в Bruno Desktop и Bruno CLI.

## Что вы научитесь делать

К концу занятия вы сможете:

- объяснить назначение методов `GET`, `PUT`, `PATCH` и `DELETE`;
- отличить полную замену ресурса через `PUT` от частичного изменения через `PATCH`;
- передать идентификатор ресурса как часть URL;
- использовать environment-переменные внутри path;
- различать успешные коды `2xx` и клиентские ошибки `4xx`;
- написать позитивные проверки для изменяющих запросов;
- оформить ожидаемый `404` как проходящий негативный тест;
- объяснить, почему статус `200` не доказывает правильность тела ответа;
- запустить папку учебных запросов из Bruno CLI;
- сохранить результат отдельным Git-коммитом.

## Что понадобится

- проект `fintech-api-tests-bruno` после занятия 2;
- Bruno Desktop;
- локально установленный Bruno CLI;
- environment `local` с переменной `baseUrl = https://postman-echo.com`;
- терминал и Git;
- доступ к публичному учебному API [JSONPlaceholder](https://jsonplaceholder.typicode.com/).

> JSONPlaceholder имитирует создание, изменение и удаление ресурсов, но не сохраняет эти изменения на сервере. Это безопасно для обучения: запрос выглядит реалистично, однако чужие данные не изменяются. Не используйте в примерах реальные персональные или платёжные данные.

## План занятия

| Время | Этап | Результат |
|---:|---|---|
| 0–10 мин | Методы и группы кодов ответа | Понимаете назначение `GET`, `PUT`, `PATCH`, `DELETE`, `2xx` и `4xx` |
| 10–18 мин | Environment и ID в URL | Добавлены адрес API и два учебных идентификатора |
| 18–30 мин | Получение ресурса по ID | `GET` возвращает существующую запись и проходит проверки |
| 30–45 мин | Полная замена через `PUT` | Проверено полное представление изменённого ресурса |
| 45–58 мин | Частичное изменение через `PATCH` | Изменено одно поле, остальные поля проверены отдельно |
| 58–67 мин | Удаление через `DELETE` | Проверены статус и пустой JSON-объект |
| 67–75 мин | Ожидаемый `404` | Негативный сценарий проходит с ожидаемой ошибкой |
| 75–83 мин | Запуск через CLI | Все семь базовых запросов запускаются из терминала |
| 83–90 мин | Итоги и коммит | Заполнены заметки и создан коммит |

## 1. Разберите методы и коды ответа — 10 минут

На предыдущих занятиях вы использовали:

- `GET` для чтения данных;
- `POST` для отправки нового представления на обработку.

Сегодня добавятся три метода:

| Метод | Типичное назначение | Есть JSON-тело |
|---|---|---|
| `GET` | Получить ресурс | Обычно нет |
| `PUT` | Полностью заменить представление ресурса | Обычно да |
| `PATCH` | Частично изменить ресурс | Обычно да |
| `DELETE` | Удалить ресурс | Обычно нет |

Важно: конкретное поведение всегда определяет контракт API. Например, один сервис после `DELETE` возвращает `204 No Content`, другой — `200 OK` с JSON-телом. Тест должен проверять контракт конкретного сервиса, а не универсальное предположение.

Запомните основные группы кодов:

- `2xx` — сервер успешно обработал запрос;
- `4xx` — запрос клиента нельзя выполнить в текущем виде;
- `5xx` — сервер не смог корректно обработать запрос из-за ошибки на своей стороне.

В этом занятии будут два ожидаемых результата:

- `200 OK` для существующего учебного ресурса;
- `404 Not Found` для несуществующего ресурса.

Ожидаемый `404` — не падение теста. Если сценарий специально проверяет отсутствующий ресурс, тест должен пройти только при статусе `404`.

## 2. Подготовьте environment — 8 минут

Откройте environment `local`, созданный на занятии 1. Не удаляйте существующую переменную `baseUrl`. Добавьте ещё три публичные переменные:

| Name | Value | Назначение |
|---|---|---|
| `resourceBaseUrl` | `https://jsonplaceholder.typicode.com` | адрес учебного REST API |
| `postId` | `1` | идентификатор существующей записи |
| `missingPostId` | `999999` | идентификатор отсутствующей записи |

После сохранения environment в нём должны быть как минимум четыре переменные:

```text
baseUrl = https://postman-echo.com
resourceBaseUrl = https://jsonplaceholder.typicode.com
postId = 1
missingPostId = 999999
```

Почему используются разные адреса:

- Postman Echo из занятий 1–2 отражает детали отправленного запроса;
- JSONPlaceholder показывает привычные REST-маршруты вида `/posts/{id}` и имитирует операции над ресурсами.

Переменная в URL:

```text
{{resourceBaseUrl}}/posts/{{postId}}
```

после подстановки превращается в:

```text
https://jsonplaceholder.typicode.com/posts/1
```

Здесь `1` находится в path, потому что является частью пути `/posts/1`. Это не query-параметр: перед ним нет `?`, и он идентифицирует конкретный ресурс.

> Переменная Bruno и path-параметр — не одно и то же понятие. `postId` — переменная, а значение после `/posts/` занимает место path-параметра в URL.

## 3. Получите существующий ресурс — 12 минут

### 3.1. Создайте запрос

В папке `00-learning-basics` создайте запрос:

- имя: `03-get-post-by-id`;
- метод: `GET`;
- URL: `{{resourceBaseUrl}}/posts/{{postId}}`;
- тело: отсутствует.

Сохраните и отправьте запрос.

Ожидаемый результат:

- статус — `200 OK`;
- заголовок `Content-Type` содержит `application/json`;
- тело — JSON-объект;
- в теле есть поля `userId`, `id`, `title`, `body`;
- значение `id` равно `1`.

Пример структуры ответа:

```json
{
  "userId": 1,
  "id": 1,
  "title": "...",
  "body": "..."
}
```

Точный текст `title` и `body` сейчас не является целью занятия. Проверяйте их тип и непустое значение, а не копируйте большой текст в ожидаемый результат.

### 3.2. Добавьте декларативные assertions

Откройте вкладку **Assert** и добавьте:

| Expression | Operator | Value | Что проверяет |
|---|---|---|---|
| `res.status` | `equals` | `200` | существующий ресурс найден |
| `res.headers['content-type']` | `contains` | `application/json` | ответ имеет JSON-формат |
| `res.body.id` | `equals` | `1` | сервер вернул нужный ресурс |

Если интерфейс Bruno интерпретирует ожидаемое значение `1` как строку, оставьте в **Assert** проверки статуса и заголовка, а числовой тип и значение проверьте JavaScript-тестом.

### 3.3. Добавьте JavaScript-тесты

Откройте вкладку **Tests** и добавьте:

```javascript
test("existing post returns 200 and JSON", function () {
  expect(res.getStatus()).to.eql(200);
  expect(res.getHeader("content-type")).to.include("application/json");
});

test("response contains the requested post", function () {
  const post = res.getBody();

  expect(post).to.be.an("object");
  expect(post.id).to.eql(1);
  expect(post.id).to.be.a("number");
  expect(post.userId).to.be.a("number");
  expect(post.title).to.be.a("string").and.not.be.empty;
  expect(post.body).to.be.a("string").and.not.be.empty;
});
```

Отправьте запрос ещё раз. Все проверки должны пройти.

## 4. Выполните полную замену через PUT — 15 минут

### 4.1. Создайте запрос

В папке `00-learning-basics` создайте запрос:

- имя: `04-put-post`;
- метод: `PUT`;
- URL: `{{resourceBaseUrl}}/posts/{{postId}}`.

Добавьте заголовок:

| Name | Value |
|---|---|
| `Content-Type` | `application/json; charset=UTF-8` |

Выберите JSON-тело и добавьте полное представление записи:

```json
{
  "id": 1,
  "userId": 1,
  "title": "Lesson 3: complete replacement",
  "body": "This representation contains every required post field."
}
```

Сохраните и отправьте запрос.

Ожидаемый ответ имеет статус `200` и содержит все четыре поля с отправленными значениями.

### 4.2. Добавьте проверки

Во вкладке **Assert** добавьте как минимум:

| Expression | Operator | Value |
|---|---|---|
| `res.status` | `equals` | `200` |
| `res.headers['content-type']` | `contains` | `application/json` |
| `res.body.title` | `equals` | `Lesson 3: complete replacement` |

Во вкладке **Tests** добавьте:

```javascript
test("PUT returns the complete replacement", function () {
  const post = res.getBody();

  expect(res.getStatus()).to.eql(200);
  expect(post).to.deep.include({
    id: 1,
    userId: 1,
    title: "Lesson 3: complete replacement",
    body: "This representation contains every required post field."
  });
});

test("PUT response keeps field types", function () {
  const post = res.getBody();

  expect(post.id).to.be.a("number");
  expect(post.userId).to.be.a("number");
  expect(post.title).to.be.a("string");
  expect(post.body).to.be.a("string");
});
```

Почему тело называется полным представлением: в нём явно переданы идентификатор, владелец, заголовок и содержимое. В реальном API пропуск обязательного поля в `PUT` может очистить поле или привести к ошибке валидации. Точное правило должно быть описано в контракте.

## 5. Выполните частичное изменение через PATCH — 13 минут

### 5.1. Создайте запрос

В папке `00-learning-basics` создайте запрос:

- имя: `05-patch-post`;
- метод: `PATCH`;
- URL: `{{resourceBaseUrl}}/posts/{{postId}}`;
- заголовок `Content-Type: application/json; charset=UTF-8`.

Добавьте JSON-тело только с изменяемым полем:

```json
{
  "title": "Lesson 3: partial update"
}
```

Сохраните и отправьте запрос.

Ожидаемый результат:

- статус равен `200`;
- `title` содержит новое значение;
- `id`, `userId` и `body` по-прежнему присутствуют в ответе;
- запрос не требовал повторно отправлять полный объект.

### 5.2. Добавьте проверки

Во вкладке **Assert** добавьте:

| Expression | Operator | Value |
|---|---|---|
| `res.status` | `equals` | `200` |
| `res.body.title` | `equals` | `Lesson 3: partial update` |
| `res.body.id` | `equals` | `1` |

Во вкладке **Tests** добавьте:

```javascript
test("PATCH changes the requested field", function () {
  const post = res.getBody();

  expect(res.getStatus()).to.eql(200);
  expect(post.title).to.eql("Lesson 3: partial update");
});

test("PATCH response still contains the resource shape", function () {
  const post = res.getBody();

  expect(post.id).to.eql(1);
  expect(post.userId).to.be.a("number");
  expect(post.body).to.be.a("string").and.not.be.empty;
});
```

Сравните тела двух запросов:

- `PUT` отправляет все поля представления;
- `PATCH` отправляет только поле `title`.

> JSONPlaceholder возвращает правдоподобный результат изменения, но не сохраняет его. Повторный `GET /posts/1` вернёт исходную запись. Не добавляйте тест, который ожидает сохранения нового заголовка между запросами: он противоречит поведению учебного сервиса.

## 6. Выполните удаление через DELETE — 9 минут

В папке `00-learning-basics` создайте запрос:

- имя: `06-delete-post`;
- метод: `DELETE`;
- URL: `{{resourceBaseUrl}}/posts/{{postId}}`;
- тело: отсутствует.

Сохраните и отправьте запрос.

JSONPlaceholder отвечает статусом `200` и пустым JSON-объектом:

```json
{}
```

Во вкладке **Assert** добавьте:

| Expression | Operator | Value |
|---|---|---|
| `res.status` | `equals` | `200` |
| `res.headers['content-type']` | `contains` | `application/json` |

Во вкладке **Tests** добавьте:

```javascript
test("DELETE returns the documented success response", function () {
  expect(res.getStatus()).to.eql(200);
  expect(res.getHeader("content-type")).to.include("application/json");
});

test("DELETE returns an empty JSON object", function () {
  const body = res.getBody();

  expect(body).to.be.an("object");
  expect(Object.keys(body)).to.have.lengthOf(0);
});
```

Не переносите ожидание `200` на любой другой API. В реальном сервисе успешный `DELETE` часто возвращает `204 No Content`; при таком статусе тело ответа должно отсутствовать.

## 7. Оформите ожидаемый 404 — 8 минут

В папке `00-learning-basics` создайте запрос:

- имя: `07-get-missing-post`;
- метод: `GET`;
- URL: `{{resourceBaseUrl}}/posts/{{missingPostId}}`;
- тело: отсутствует.

Сохраните и отправьте запрос.

Ожидаемый результат:

- статус — `404 Not Found`;
- ответ имеет JSON-формат;
- тело — пустой объект `{}`.

Добавьте декларативные assertions:

| Expression | Operator | Value |
|---|---|---|
| `res.status` | `equals` | `404` |
| `res.headers['content-type']` | `contains` | `application/json` |

Добавьте JavaScript-тесты:

```javascript
test("missing post returns the expected 404", function () {
  expect(res.getStatus()).to.eql(404);
  expect(res.getHeader("content-type")).to.include("application/json");
});

test("404 response does not expose unexpected data", function () {
  const body = res.getBody();

  expect(body).to.be.an("object");
  expect(Object.keys(body)).to.have.lengthOf(0);
});
```

Этот запрос должен отображаться как успешно протестированный, хотя HTTP-запрос получил `404`. Причина: фактический результат совпал с ожидаемым результатом негативного сценария.

### Намеренно создайте и исправьте падение

1. Временно измените ожидаемый статус в JavaScript-тесте с `404` на `200`.
2. Отправьте запрос.
3. Прочитайте сообщение об ошибке: тест ожидал `200`, но фактически получил `404`.
4. Верните ожидаемое значение `404`.
5. Повторите запрос и убедитесь, что тест снова проходит.

Не меняйте `missingPostId` на существующий ID только ради зелёного теста. Негативный сценарий должен сохранять своё назначение.

## 8. Запустите базовую папку через Bruno CLI — 8 минут

Сохраните все запросы и environment в Bruno. Из корня учебного проекта выполните:

```bash
cd bruno/fintech-api
npx bru run 00-learning-basics --env local
cd ../..
```

Ожидаемый результат:

- выполнены запросы `01`–`07`;
- позитивные запросы завершились ожидаемыми статусами `200`;
- `07-get-missing-post` получил ожидаемый `404`, но его тесты прошли;
- все assertions и JavaScript-тесты зелёные;
- команда завершилась без ошибки.

Если CLI выполнил не все запросы:

- убедитесь, что каждый запрос сохранён в Bruno;
- проверьте, что файлы `03`–`07` находятся в `00-learning-basics`;
- проверьте порядок запросов в коллекции;
- убедитесь, что команда выполняется из папки, где находится `bruno.json`.

Если CLI не видит новые environment-переменные:

- сохраните environment `local` в Bruno;
- проверьте регистр имён `resourceBaseUrl`, `postId`, `missingPostId`;
- убедитесь, что в команде указан `--env local`;
- не добавляйте пробелы внутрь конструкции `{{variableName}}`.

## 9. Зафиксируйте результат — 7 минут

Создайте файл `notes/session-03.md` и заполните его своими словами:

```markdown
# Занятие 3

## Чем отличаются PUT и PATCH

- PUT: ...
- PATCH: ...
- Что было в телах моих запросов: ...

## Как ID передавался в URL

- Переменная: ...
- Итоговый path: ...

## Почему тест с 404 прошёл

- Сценарий: ...
- Ожидаемый результат: ...
- Фактический результат: ...

## Что осталось непонятно

- ...
```

Проверьте изменения:

```bash
git status
git diff --check
```

Убедитесь, что в коммит не попали токены, реальные персональные данные или платёжные реквизиты. Затем выполните:

```bash
git add bruno/fintech-api/00-learning-basics \
  bruno/fintech-api/environments/local.bru \
  notes/session-03.md
git commit -m "test: cover resource update and delete methods"
git status
```

Если Bruno хранит environment по другому пути, добавьте фактический файл `local.bru`. Перед коммитом изучите его и убедитесь, что в нём только публичные учебные значения.

## Структура проекта после занятия

Основная структура должна выглядеть так:

```text
fintech-api-tests-bruno/
├── bruno/
│   └── fintech-api/
│       ├── 00-learning-basics/
│       │   ├── 01-echo-get.bru
│       │   ├── 02-echo-post.bru
│       │   ├── 03-get-post-by-id.bru
│       │   ├── 04-put-post.bru
│       │   ├── 05-patch-post.bru
│       │   ├── 06-delete-post.bru
│       │   └── 07-get-missing-post.bru
│       ├── environments/
│       │   └── local.bru
│       └── bruno.json
├── notes/
│   ├── session-01.md
│   ├── session-02.md
│   └── session-03.md
├── .gitignore
├── AGENTS.md
├── package.json
└── package-lock.json
```

## Чек-лист готовности

Занятие завершено, если каждый пункт можно отметить:

- [ ] В `local` добавлены `resourceBaseUrl`, `postId` и `missingPostId`.
- [ ] Все URL используют environment-переменные вместо жёстко заданного домена.
- [ ] `03-get-post-by-id` получает существующий ресурс по ID.
- [ ] `04-put-post` отправляет полное представление ресурса.
- [ ] `05-patch-post` отправляет только изменяемое поле.
- [ ] Вы можете своими словами объяснить разницу между `PUT` и `PATCH`.
- [ ] `06-delete-post` проверяет статус и пустой JSON-объект.
- [ ] `07-get-missing-post` ожидает статус `404`, а не `200`.
- [ ] Проверки отличают число от строки для поля `id`.
- [ ] Проверяется не только статус, но и структура тела ответа.
- [ ] Вы увидели намеренное падение из-за неверного ожидаемого статуса и исправили его.
- [ ] Все семь запросов папки проходят через Bruno CLI.
- [ ] Вы понимаете, что JSONPlaceholder не сохраняет изменения.
- [ ] Заполнен `notes/session-03.md`.
- [ ] Создан коммит `test: cover resource update and delete methods`.

## Контрольные вопросы

Ответьте без подсказки:

1. Где находится path-параметр в URL `https://example.test/posts/15`?
2. Чем path-параметр отличается от query-параметра?
3. Почему адрес API хранится отдельно от идентификатора ресурса?
4. В чём практическая разница между `PUT` и `PATCH`?
5. Почему нельзя проверять только статус `200` после `PUT`?
6. Почему ожидаемый `404` должен делать негативный тест зелёным?
7. К какой группе относится код `404` и что означает эта группа?
8. Может ли успешный `DELETE` вернуть `204` вместо `200`?
9. Почему после `PATCH` повторный `GET` к JSONPlaceholder возвращает исходный заголовок?
10. Что обнаруживает проверка `Object.keys(body).to.have.lengthOf(0)`?
11. Почему `id: 1` и `id: "1"` — разные результаты проверки контракта?
12. Какой результат должен дать CLI, если HTTP-статус равен `404`, но тест ожидает `404`?

## Если осталось время

### Вариант 1. Проверьте другой существующий ресурс

Временно измените `postId` с `1` на `2` и выполните только `03-get-post-by-id`.

Тест с жёсткой проверкой `expect(post.id).to.eql(1)` должен упасть. Исправьте тест так, чтобы ожидаемый ID брался из выбранного сценария, либо верните `postId = 1` и объясните, почему данные запроса и ожидание должны быть согласованы.

Не усложняйте основной тест динамическими переменными, если ещё не можете объяснить область видимости переменных. Полноценная работа с request и runtime variables будет отдельной темой.

### Вариант 2. Сравните контракты DELETE

Запишите в `notes/session-03.md` два допустимых варианта контракта:

```text
200 OK + {}
204 No Content + пустое тело
```

Объясните, почему нельзя одновременно ожидать оба варианта, если спецификация конкретного API разрешает только один.

## Официальные материалы

- [Bruno: обзор REST-запросов](https://docs.usebruno.com/send-requests/REST/overview)
- [Bruno: environment variables](https://docs.usebruno.com/variables/environment-variables)
- [Bruno: интерполяция переменных](https://docs.usebruno.com/variables/variables-interpolation)
- [Bruno: assertions](https://docs.usebruno.com/testing/tests/assertions)
- [Bruno: JavaScript-тесты](https://docs.usebruno.com/testing/tests/introduction)
- [Bruno CLI: запуск коллекций и папок](https://docs.usebruno.com/bru-cli/runCollection)
- [JSONPlaceholder: руководство по GET, PUT, PATCH и DELETE](https://jsonplaceholder.typicode.com/guide/)
- [MDN: справочник HTTP-методов](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods)
- [MDN: справочник HTTP-кодов ответа](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status)
