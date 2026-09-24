# Блок 2. Занятие 5 — области видимости переменных, скрипты и связанный сценарий

**Продолжительность:** 90 минут  
**Формат:** 20% объяснение, 80% практика  
**Результат занятия:** в Bruno создан связанный сценарий из двух запросов: первый формирует уникальную ссылку операции и сохраняет её в runtime variable, второй использует сохранённое значение; сценарий проходит в Bruno Desktop и Bruno CLI, а изменения оформлены в отдельной Git-ветке, слиты в `main` и отправлены на GitHub так, чтобы коммит мог учитываться в Contributions.

## Главное правило занятия

На предыдущих занятиях ожидаемые значения часто были записаны прямо в тестах. Теперь входные данные и ожидания связываются через переменные:

```text
request variable
       |
       v
pre-request script создаёт transferRef
       |
       v
10-create-transfer-reference отправляет transferRef
       |
       v
post-response script читает ответ и сохраняет createdTransferRef
       |
       v
11-use-transfer-reference подставляет createdTransferRef
       |
       v
тест сравнивает отправленное и сохранённое значения
```

Работа выполняется в ветке `lesson/05-runtime-chaining`. Коммит из этой ветки должен попасть в `main`: GitHub учитывает коммиты в профиле, когда они находятся в основной ветке или `gh-pages`, а email автора связан с GitHub-аккаунтом.

## Что вы научитесь делать

К концу занятия вы сможете:

- объяснить разницу между environment, request и runtime variables;
- определить область видимости и время жизни переменной;
- использовать синтаксис `{{variableName}}` в URL, query-параметрах и JSON-теле;
- прочитать request variable через `bru.getRequestVar()`;
- создать runtime variable через `bru.setVar()` и прочитать её через `bru.getVar()`;
- отличить pre-request script от post-response script и теста;
- сохранить значение из ответа первого запроса и использовать его во втором;
- понять, почему связанный сценарий зависит от порядка запросов;
- диагностировать запуск второго запроса без подготовленного runtime-состояния;
- запустить связанный сценарий в Bruno Desktop и Bruno CLI;
- оформить изменение через ветку, merge в `main` и push на GitHub;
- проверить условия, при которых коммит отображается в GitHub Contributions.

## Что понадобится

- проект `fintech-api-tests-bruno` после занятия 4;
- изменения занятия 4 слиты в локальный `main`;
- Bruno Desktop;
- локально установленный Bruno CLI;
- environment `local` с `baseUrl = https://postman-echo.com`;
- терминал и Git;
- GitHub-аккаунт и отдельный репозиторий проекта, не являющийся fork;
- доступ к публичному учебному API [Postman Echo](https://postman-echo.com/).

> В сценарии используется только вымышленная ссылка операции. Не добавляйте реальные токены, номера карт, персональные данные и платёжные реквизиты ни в запросы, ни в runtime variables, ни в Git.

## План занятия

| Время | Этап | Результат |
|---:|---|---|
| 0–8 мин | Проверка Git и рабочая ветка | Работа ведётся в `lesson/05-runtime-chaining` |
| 8–20 мин | Области видимости и приоритет | Понятно, где живут environment, request и runtime variables |
| 20–33 мин | Request variables и pre-request script | Перед отправкой создаётся уникальный `transferRef` |
| 33–48 мин | Первый запрос и post-response script | Ответ проверен, `createdTransferRef` сохранён в runtime |
| 48–62 мин | Второй запрос | Runtime-значение использовано и проверено |
| 62–70 мин | Эксперимент с порядком запуска | Видна зависимость связанного сценария от состояния |
| 70–78 мин | Полный запуск через CLI | Оба запроса стабильно проходят в нужном порядке |
| 78–84 мин | Заметки и учебный коммит | Артефакты сохранены в рабочей ветке |
| 84–90 мин | Merge, push и проверка GitHub | Коммит находится в `main` на GitHub |

## 1. Создайте рабочую ветку — 8 минут

### 1.1. Проверьте исходное состояние

Из корня проекта выполните:

```bash
git status --short
git branch --show-current
git log --oneline -5
```

Ожидается:

- `git status --short` ничего не выводит;
- текущая ветка — `main`;
- в истории виден merge занятия 4.

Если есть незакоммиченные файлы, не переносите их вслепую в новое занятие. Сначала определите их происхождение и завершите предыдущую работу.

### 1.2. Проверьте email автора коммитов

Выполните:

```bash
git config user.name
git config user.email
```

Email должен быть:

- добавлен и подтверждён в GitHub-аккаунте; или
- GitHub noreply-адресом вашего аккаунта.

Если GitHub скрывает ваш email, откройте **GitHub → Settings → Emails** и скопируйте предложенный noreply-адрес. Настройте его только для учебного репозитория:

```bash
git config user.email "YOUR_GITHUB_NOREPLY_EMAIL"
```

Не копируйте адрес другого пользователя. Изменение применяется к будущим коммитам и не переписывает старую историю.

### 1.3. Создайте ветку занятия

```bash
git switch -c lesson/05-runtime-chaining
git branch --show-current
```

Ожидаемый результат:

```text
lesson/05-runtime-chaining
```

До раздела **«10. Слейте ветку и опубликуйте результат»** не создавайте учебные коммиты непосредственно в `main`.

## 2. Разберите области видимости переменных — 12 минут

В этом занятии используются три вида переменных:

| Вид | Где задаётся | Где доступен | Время жизни | Пример |
|---|---|---|---|---|
| Environment | environment `local` | Во многих запросах выбранного environment | Сохраняется в файле | `baseUrl` |
| Request | вкладка **Vars** конкретного запроса | Только в этом запросе | Сохраняется вместе с запросом | `referencePrefix` |
| Runtime | Скрипт через `bru.setVar()` | Во всех запросах текущей коллекции | Временное состояние запуска | `createdTransferRef` |

Главное различие:

- environment variable описывает окружение или общую конфигурацию;
- request variable относится к одному запросу;
- runtime variable передаёт временные данные между запросами.

Runtime variable имеет самый высокий приоритет среди обычных областей Bruno. Если одно и то же имя существует в runtime, request и environment, при подстановке будет использовано runtime-значение. Не создавайте одинаковые имена без учебной причины: совпадение легко скрывает ошибку конфигурации.

Для этого занятия используйте разные имена:

```text
baseUrl             — environment
referencePrefix     — request
transferRef         — runtime, создана до первого запроса
createdTransferRef  — runtime, сохранена из ответа
```

### Контрольный вопрос

Где следует хранить каждое значение?

| Значение | Правильная область |
|---|---|
| Адрес учебного API | Environment |
| Префикс только одного запроса | Request |
| ID, полученный из ответа и нужный следующему запросу | Runtime |
| Секретный production-токен | Не в публичном Git; используйте безопасное локальное хранилище |

## 3. Создайте первый запрос и request variable — 8 минут

В папке `00-learning-basics` создайте запрос:

- имя: `10-create-transfer-reference`;
- метод: `POST`;
- URL: `{{baseUrl}}/post`;
- `Content-Type`: `application/json`.

Во вкладке **Vars** найдите секцию **Pre Request Vars** и добавьте:

| Name | Value | Назначение |
|---|---|---|
| `referencePrefix` | `LESSON5` | Постоянный префикс только для этого запроса |

Request variable можно использовать как `{{referencePrefix}}`, но в этом сценарии она будет прочитана в JavaScript через:

```javascript
bru.getRequestVar("referencePrefix")
```

Пока не отправляйте запрос: сначала pre-request script должен подготовить динамическое значение.

## 4. Добавьте pre-request script — 5 минут

Откройте вкладку **Script → Pre Request** первого запроса и добавьте:

```javascript
const prefix = bru.getRequestVar("referencePrefix");
const timePart = Date.now().toString(36).toUpperCase();
const randomPart = Math.random().toString(36).slice(2, 8).toUpperCase();
const transferRef = `${prefix}-${timePart}-${randomPart}`;

bru.setVar("transferRef", transferRef);
```

Скрипт выполняется до отправки HTTP-запроса:

1. читает request variable `referencePrefix`;
2. формирует достаточно уникальную учебную строку;
3. сохраняет её как runtime variable `transferRef`;
4. делает `{{transferRef}}` доступной при сборке URL, заголовков и тела.

Это не криптографический идентификатор и не пример банковского алгоритма. Он нужен только для наглядной передачи данных внутри тестового запуска.

## 5. Добавьте JSON-тело и проверки первого запроса — 15 минут

### 5.1. Подготовьте тело

Во вкладке **Body → JSON** добавьте:

```json
{
  "operation": "create-transfer-reference",
  "transferRef": "{{transferRef}}",
  "amountMinor": 1250,
  "currency": "USD"
}
```

Сохраните и отправьте запрос.

Postman Echo должен вернуть отправленное тело внутри поля `json`:

```json
{
  "json": {
    "operation": "create-transfer-reference",
    "transferRef": "LESSON5-...-...",
    "amountMinor": 1250,
    "currency": "USD"
  }
}
```

### 5.2. Добавьте tests

Во вкладке **Tests** добавьте:

```javascript
test("echo returns the generated transfer reference", function () {
  const body = res.getBody();
  const generatedReference = bru.getVar("transferRef");

  expect(res.getStatus()).to.eql(200);
  expect(body.json).to.be.an("object");
  expect(body.json.transferRef).to.eql(generatedReference);
  expect(body.json.transferRef).to.match(/^LESSON5-[A-Z0-9]+-[A-Z0-9]+$/);
});

test("echo returns the transfer draft data", function () {
  const payload = res.getBody().json;

  expect(payload.operation).to.eql("create-transfer-reference");
  expect(payload.amountMinor).to.eql(1250);
  expect(payload.amountMinor).to.be.a("number");
  expect(payload.currency).to.eql("USD");
});
```

Здесь ожидание не дублирует динамическую строку. Тест читает значение, которое создал pre-request script, и сравнивает его с тем, что вернул сервер.

### 5.3. Сохраните значение из ответа

Откройте **Script → Post Response** и добавьте:

```javascript
const body = res.getBody();
const returnedReference = body?.json?.transferRef;

if (res.getStatus() !== 200) {
  throw new Error(`Cannot save transfer reference: status ${res.getStatus()}`);
}

if (!returnedReference) {
  throw new Error("Cannot save transfer reference: response field json.transferRef is missing");
}

bru.setVar("createdTransferRef", returnedReference);
```

Post-response script выполняется после получения ответа. Он:

- не придумывает новое значение;
- проверяет минимальные предусловия;
- извлекает `json.transferRef`;
- сохраняет его в runtime variable `createdTransferRef` для следующего запроса.

Отправьте запрос ещё раз. Откройте просмотр runtime variables через значок глаза в верхней части Bruno и найдите:

```text
transferRef = LESSON5-...
createdTransferRef = LESSON5-...
```

Значения должны совпадать.

## 6. Создайте второй запрос — 14 минут

В той же папке создайте запрос:

- имя: `11-use-transfer-reference`;
- метод: `GET`;
- URL: `{{baseUrl}}/get`;
- тело отсутствует.

Во вкладке **Params** добавьте включённую строку:

| Name | Value |
|---|---|
| `reference` | `{{createdTransferRef}}` |

Сохраните и отправьте запрос сразу после успешного первого запроса.

Ожидаемый фрагмент ответа:

```json
{
  "args": {
    "reference": "LESSON5-...-..."
  }
}
```

Добавьте тесты:

```javascript
test("second request receives the saved reference", function () {
  const body = res.getBody();
  const savedReference = bru.getVar("createdTransferRef");

  expect(res.getStatus()).to.eql(200);
  expect(savedReference).to.be.a("string").and.not.be.empty;
  expect(body.args.reference).to.eql(savedReference);
});

test("saved reference keeps the lesson prefix", function () {
  const reference = res.getBody().args.reference;

  expect(reference).to.match(/^LESSON5-/);
});
```

Теперь сценарий доказывает не только то, что два запроса по отдельности отвечают `200`. Он доказывает передачу конкретного значения между ними.

## 7. Проведите эксперимент с порядком — 8 минут

Runtime variables — временное состояние. Это полезно, но создаёт зависимость от подготовки сценария.

### 7.1. Убедитесь, что правильный порядок работает

Запустите запросы по порядку:

1. `10-create-transfer-reference`;
2. `11-use-transfer-reference`.

Оба должны пройти.

### 7.2. Удалите runtime-состояние

Перезапустите Bruno или очистите runtime variables через интерфейс. Не удаляйте environment `local`.

После очистки выполните только `11-use-transfer-reference`.

Ожидается диагностируемая проблема:

- `createdTransferRef` отсутствует;
- query-параметр не получает корректное значение;
- тест второго запроса падает на проверке непустой строки или равенства.

Это полезное падение: второй запрос не должен тихо становиться зелёным без результата первого шага.

После эксперимента снова выполните запросы в порядке `10 → 11` и верните сценарий в зелёное состояние.

### Почему не сохраняем значение в environment

Технически скрипт может изменять environment variable, но для этого сценария это неверный выбор:

- значение относится к одному запуску;
- оно не должно оставаться в файле после завершения;
- случайно закоммиченное динамическое значение создаст шум в Git;
- следующий запуск должен формировать новую ссылку, а не наследовать старую.

## 8. Запустите связанный сценарий через Bruno CLI — 8 минут

Проверьте порядок запросов в папке: `10-create-transfer-reference` должен находиться перед `11-use-transfer-reference`.

Из корня коллекции выполните:

```bash
cd bruno/fintech-api/fintech-api
npx bru run 00-learning-basics --env local
cd ../../..
```

Проверьте:

- процесс завершился с кодом `0`;
- запрос `10-create-transfer-reference` прошёл;
- post-response script сохранил значение;
- запрос `11-use-transfer-reference` получил то же значение;
- старые запросы занятий 1–4 также остались зелёными.

Если второй запрос падает только в CLI, проверьте:

1. порядок файлов и поле `seq` в `.bru`-запросах;
2. точное имя `createdTransferRef` без различий в регистре;
3. что сохранение находится в **Post Response**, а не только в Tests;
4. что команда запускает папку целиком, а не один второй файл;
5. что выбран environment `local`.

Не исправляйте проблему постоянным значением в environment: это скроет дефект цепочки.

## 9. Заполните заметки и создайте коммит — 6 минут

Создайте `notes/session-05.md`:

```markdown
# Session 05 — variables and request chaining

## Variable map

| Variable | Scope | Created by | Used by | Lifetime |
|---|---|---|---|---|
| baseUrl | environment | local environment | requests 10–11 | saved |
| referencePrefix | request | request 10 Vars | request 10 pre-script | saved |
| transferRef | runtime | request 10 pre-script | request 10 body and tests | current run |
| createdTransferRef | runtime | request 10 post-script | request 11 | current run |

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
```

Проверьте изменения:

```bash
git status --short
git diff --check
git diff --stat
```

В учебном проекте ожидаются:

- два новых `.bru`-файла;
- при необходимости изменения порядка запросов;
- `notes/session-05.md`;
- отсутствие секретов, токенов и реальных платёжных данных.

Создайте коммит в рабочей ветке:

```bash
git add bruno/fintech-api/fintech-api/00-learning-basics notes/session-05.md
git commit -m "test: add runtime variable request chain"
git status
git log -1 --format=fuller
```

В `git log -1 --format=fuller` проверьте:

- сообщение коммита;
- имя автора;
- email автора, связанный с GitHub;
- актуальную дату автора.

## 10. Слейте ветку и опубликуйте результат — 6 минут

### 10.1. Выполните merge в локальный main

Слияние выполняется только после зелёного CLI-запуска и при чистом рабочем дереве:

```bash
git switch main
git merge --no-ff lesson/05-runtime-chaining -m "merge: complete lesson 5"
git log --oneline --graph --decorate -8
```

Учебный коммит должен быть достижим из `main`. Именно это важно для GitHub Contributions; коммит, оставшийся только в несмерженной feature-ветке, обычно не учитывается в профиле.

### 10.2. Настройте remote при первой публикации

Проверьте:

```bash
git remote -v
```

Если `origin` уже указывает на правильный отдельный репозиторий GitHub, ничего не меняйте.

Если remote отсутствует:

1. создайте на GitHub новый репозиторий `fintech-api-tests-bruno`;
2. не выбирайте шаблон README, `.gitignore` или license, потому что локальная история уже существует;
3. убедитесь, что это обычный репозиторий, а не fork;
4. добавьте показанный GitHub адрес:

```bash
git remote add origin git@github.com:YOUR_LOGIN/fintech-api-tests-bruno.git
```

Для HTTPS используйте адрес, показанный GitHub, без токена в строке команды. Никогда не сохраняйте personal access token в remote URL.

### 10.3. Отправьте main

```bash
git push -u origin main
```

При желании опубликуйте и учебную ветку, но для Contributions это не требуется после merge:

```bash
git push -u origin lesson/05-runtime-chaining
```

### 10.4. Проверьте GitHub

На странице репозитория проверьте:

- default branch — `main`;
- файл `lessons/block-02/lesson-05.md` или артефакты занятия доступны в `main`;
- учебный коммит виден в истории `main`;
- автор коммита связан с вашим GitHub-профилем;
- репозиторий не является fork.

На странице профиля Contributions запись может появиться не мгновенно. GitHub указывает, что обновление иногда занимает до 24 часов.

## Почему коммит может не появиться в Contributions

Проверяйте причины по порядку:

| Проверка | Команда или место | Ожидание |
|---|---|---|
| Коммит отправлен | `git status -sb` | Нет `ahead N` после push |
| Коммит в default branch | `git branch --contains COMMIT_SHA` и GitHub | Среди веток есть `main` |
| Email автора | `git show -s --format='%an <%ae>' COMMIT_SHA` | Email связан с GitHub |
| Репозиторий не fork | Страница GitHub | Нет пометки `forked from` |
| Default branch | Settings → Branches | `main` |
| Приватная активность | Profile → Contribution settings | Включена видимость private contributions, если repo private |
| Время обновления | Профиль GitHub | При необходимости подождать до 24 часов |

Не создавайте пустые коммиты и не меняйте даты задним числом ради зелёных квадратов. Contributions должны отражать реальную, проверяемую работу над проектом.

## Структура проекта после занятия

```text
fintech-api-tests-bruno/
├── bruno/
│   └── fintech-api/
│       └── fintech-api/
│           ├── 00-learning-basics/
│           │   ├── 01-echo-get.bru
│           │   ├── 02-echo-post.bru
│           │   ├── 03-get-post-by-id.bru
│           │   ├── 04-put-post.bru
│           │   ├── 05-patch-post.bru
│           │   ├── 06-delete-post.bru
│           │   ├── 07-get-missing-post.bru
│           │   ├── 08-get-posts-by-user.bru
│           │   ├── 09-get-empty-posts-filter.bru
│           │   ├── 10-create-transfer-reference.bru
│           │   └── 11-use-transfer-reference.bru
│           ├── environments/
│           │   └── local.bru
│           └── bruno.json
├── lessons/
│   ├── block-01/
│   └── block-02/
│       └── lesson-05.md
├── notes/
│   └── session-05.md
└── ROADMAP.md
```

## Чек-лист готовности

- [ ] Работа начата в ветке `lesson/05-runtime-chaining`, а не в `main`.
- [ ] Email автора будущего коммита связан с GitHub или является корректным noreply-адресом.
- [ ] В request 10 создана request variable `referencePrefix`.
- [ ] Pre-request script формирует `transferRef` до отправки запроса.
- [ ] `transferRef` подставляется в JSON-тело request 10.
- [ ] Тест request 10 сравнивает ответ с runtime-значением.
- [ ] Post-response script проверяет ответ перед сохранением данных.
- [ ] `createdTransferRef` сохраняется через `bru.setVar()`.
- [ ] Request 11 использует `{{createdTransferRef}}` в query-параметре.
- [ ] Тест request 11 читает переменную через `bru.getVar()`.
- [ ] Эксперимент подтвердил, что request 11 не должен запускаться первым.
- [ ] После эксперимента полный сценарий снова зелёный.
- [ ] В `notes/session-05.md` заполнена карта переменных.
- [ ] Все запросы папки прошли через Bruno CLI.
- [ ] Создан содержательный коммит `test: add runtime variable request chain`.
- [ ] Коммит создан в рабочей ветке.
- [ ] Ветка слита в локальный `main` после проверок.
- [ ] `main` отправлен в отдельный GitHub-репозиторий.
- [ ] На GitHub default branch — `main`.
- [ ] Учебный коммит виден в истории `main` и связан с профилем автора.

## Контрольные вопросы

1. Чем request variable отличается от environment variable?
2. Почему `createdTransferRef` является runtime variable?
3. Что произойдёт с runtime variables после перезапуска клиента?
4. В какой момент выполняется pre-request script?
5. В какой момент выполняется post-response script?
6. Чем post-response script в этом сценарии отличается от Tests?
7. Что делает `bru.setVar("name", value)`?
8. Что возвращает `bru.getVar("name")`, если runtime variable существует?
9. Зачем отдельно используются `transferRef` и `createdTransferRef`?
10. Почему второй запрос должен падать без выполнения первого?
11. Почему нельзя исправлять этот сценарий постоянным значением в environment?
12. От чего зависит порядок запуска запросов через CLI?
13. Почему коммит в несмерженной feature-ветке может не появиться в Contributions?
14. Как GitHub связывает локальный коммит с профилем пользователя?
15. Почему отдельный репозиторий подходит для Contributions, а коммиты в fork — нет?
16. Что проверить, если push успешен, но вклад ещё не виден в профиле?

## Если осталось время

### Вариант 1. Проверьте приоритет переменных

Временно добавьте в environment:

```text
transferRef = ENVIRONMENT-VALUE
```

Запустите request 10. Pre-request script создаст runtime variable с тем же именем, и в тело попадёт runtime-значение, потому что оно имеет более высокий приоритет.

После эксперимента удалите `transferRef` из environment. Не оставляйте конфликт имён в основном сценарии.

### Вариант 2. Добавьте защиту во второй pre-request script

Во втором запросе добавьте pre-request script:

```javascript
const reference = bru.getVar("createdTransferRef");

if (!reference) {
  throw new Error(
    "createdTransferRef is missing: run 10-create-transfer-reference first"
  );
}
```

Снова очистите runtime variables и запустите второй запрос. Сравните новое сообщение с прежней ошибкой. Хорошая диагностика должна объяснять не только факт падения, но и нужное действие.

## Официальные материалы

- [Bruno: переменные и их приоритет](https://docs.usebruno.com/variables/overview)
- [Bruno: request variables](https://docs.usebruno.com/variables/request-variables)
- [Bruno: runtime variables](https://docs.usebruno.com/variables/runtime-variables)
- [Bruno: порядок выполнения скриптов](https://docs.usebruno.com/testing/script/script-flow)
- [Bruno: request chaining](https://docs.usebruno.com/testing/script/request-chaining)
- [GitHub: что учитывается в Contributions](https://docs.github.com/en/account-and-profile/reference/profile-contributions-reference)
- [GitHub: поиск причин отсутствующих Contributions](https://docs.github.com/en/account-and-profile/how-tos/contribution-settings/troubleshooting-missing-contributions)
- [GitHub: настройка email для коммитов](https://docs.github.com/en/account-and-profile/how-tos/email-preferences/setting-your-commit-email-address)
