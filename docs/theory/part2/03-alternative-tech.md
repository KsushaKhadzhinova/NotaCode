# Модуль 03. Альтернативные технологии

Цель модуля: познакомиться с альтернативами Node.js/PostgreSQL. Это Python + FastAPI (Pydantic, OpenAPI), документная СУБД MongoDB (Atlas, Mongoose) и работа в реальном времени (WebSocket, Socket.IO). Практика модуля: ЛР №6 (MongoDB + Mongoose) и ЛР №7 (WebSocket / Socket.IO).

> **Главный вывод по стеку (подробности в последнем разделе).** Теория про FastAPI в модуле есть, а лабораторной по FastAPI нет. ЛР №6 и ЛР №7 в формулировках задач прямо требуют **Node.js + Express**, а также **Mongoose** (ЛР6) и **Socket.IO** (ЛР7). Явного разрешения заменить их на Python/FastAPI в материалах нет. Есть только послабления: ЛР6 можно делать «как отдельный проект», а для ЛР7 разрешён упрощённый клиент в отдельной ветке.

---

## Источники

| № | Файл | Статус |
|---|------|--------|
| 01 | Сравнение Node.js и Python/FastAPI | прочитан полностью. Внутри также есть разделы 2–5: создание API, OpenAPI, Pydantic |
| 02 | Создание API на FastAPI | прочитан полностью |
| 03 | Автоматическая документация OpenAPI | прочитан полностью |
| 04 | Валидация данных в FastAPI | прочитан полностью |
| 05 | Документная модель данных | прочитан полностью |
| 06 | MongoDB Atlas | прочитан полностью |
| 07 | ODM Mongoose: схемы, модели | прочитан полностью |
| 08 | CRUD-операции с MongoDB через Mongoose | прочитан полностью |
| 09 | Веб-сокеты и реальное время | прочитан полностью |
| 10 | Вопросы для самоподготовки | прочитан полностью |
| 11 | Лабораторная работа №6 «Интеграция с NoSQL базой данных MongoDB» | прочитан полностью |
| 12 | Лабораторная работа №7 «Реализация реального времени с WebSocket» | прочитан полностью |
| — | «Генератор вариантов .html», «Генератор вариантов (391477).html» | **нечитаемы**: сохранённые страницы ошибки Moodle («Ошибка \| СЭО»), данных о вариантах нет |

---

## Ключевая теория

### 1. Node.js и Python/FastAPI: сравнение (лекция 01)

| Аспект | Node.js | Python + FastAPI |
|---|---|---|
| Модель | один поток + event loop, неблокирующий I/O | воркеры (процессы/потоки) и/или asyncio. Потоки ограничены GIL, выход через multiprocessing |
| I/O-bound | очень высокий RPS при множестве соединений | в async-режиме сопоставимо, в sync-режиме уступает |
| CPU-bound | тяжёлый расчёт блокирует event loop, нужны worker threads | multiprocessing, сильная экосистема для ML/науки |
| Память | лёгкий в простое, быстро стартует, есть риск утечек | больше памяти на старте, поведение предсказуемее |
| Пакеты | npm: больше 1,5 млн пакетов, частые breaking changes, качество разное | pip: около 300 тыс., стабильнее, стандарты PEP |
| Менеджеры | npm / yarn / pnpm | pip / poetry / pipenv |
| Типы | TypeScript, проверка только при компиляции | type hints + Pydantic: валидация во время выполнения |
| Документация API | нужны swagger-jsdoc и swagger-ui-express | OpenAPI из коробки: `/docs`, `/redoc` |
| Безопасность | вручную: helmet, cors, rate-limit | встроенная валидация Pydantic |

**Матрица выбора из лекции:**
- real-time (чат, игры, уведомления): Node.js (WebSocket, socket.io);
- высоконагруженный REST с большим I/O: Node.js или async FastAPI;
- ML/DS/CV: Python (FastAPI как обёртка над моделью);
- сложная enterprise-логика: Python;
- микросервисы: стеки можно комбинировать, например шлюз на Node.js и ML на Python;
- выбор зависит от того, что уже знает команда.

В выводе лекции прямо сказано: «В нашем курсе мы фокусируемся на FastAPI». Однако ЛР этого модуля написаны под Node.js (см. ниже).

### 2. Создание API на FastAPI (лекции 01 §2, 02)

**Три принципа FastAPI:** высокая производительность (на уровне Node/Go), автодокументация OpenAPI, современный Python (type hints и async/await). Разработчик описывает, «что» нужно, а FastAPI сам валидирует данные и строит документацию.

**Окружение и запуск:**
```bash
python -m venv venv && venv\Scripts\activate    # Linux: source venv/bin/activate
pip install fastapi uvicorn
pip freeze > requirements.txt
uvicorn main:app --reload --host 0.0.0.0 --port 8000   # dev
uvicorn main:app --workers 4                           # prod
gunicorn -w 4 -k uvicorn.workers.UvicornWorker app.main:app   # prod (Linux)
```
В `main:app` первая часть означает модуль `main.py`, вторая переменная `app`. Флаг `--reload` нужен только для разработки.

**Маршруты и параметры:**
```python
app = FastAPI(title="My API", version="1.0.0", docs_url="/docs", redoc_url="/redoc")

@app.get("/items/{item_id}")              # path-параметр, тип из аннотации
async def read_item(item_id: int, q: str | None = None):   # q: query ?q=...
    return {"item_id": item_id, "q": q}
```
- Методы: `@app.get/post/put/patch/delete`.
- Тело POST/PUT передаётся Pydantic-моделью.
- `async def` подходит для I/O (БД, внешние API), обычный `def` для простых вычислений.
- Ошибки: `raise HTTPException(status_code=404, detail="Not found")`.

**Рекомендуемая структура проекта:**
```
app/
  main.py
  api/endpoints/{users,items,auth}.py, api/dependencies.py
  core/{config,security}.py
  models/        # ORM (SQLAlchemy)
  schemas/       # Pydantic
  services/      # бизнес-логика
requirements.txt, README.md
```

**Паттерн схем:** `UserBase` содержит общие поля. От него наследуются `UserCreate` (+password), `UserUpdate` (все поля Optional) и `UserInDB` (+id, created_at…, `Config.from_attributes = True` для ORM). Для статусов используется `str, Enum` (draft/published/archived). Вложенность выражается типами `List[Item]` и `Dict[str, Any]`.

**CRUD:**
- `GET /items/` принимает skip/limit/search и возвращает `response_model=List[Item]`;
- `GET /items/{id}` отдаёт 404, если элемент не найден;
- `POST` отвечает `status_code=201`;
- `PUT` делает полную замену;
- `PATCH` использует `item.dict(exclude_unset=True)` и `setattr`;
- `DELETE` отвечает `204`.

**Dependency Injection (`Depends`):** в зависимости выносят общую логику. Примеры: `get_database`, `get_pagination_params` и цепочка авторизации `get_current_user` (заголовок `Authorization: Bearer ...`, иначе 401) → `get_current_active_user` (403) → `require_admin`. Сервисный слой оформляется классом `ItemService` с фабрикой `get_item_service`.

**Ошибки:**
- кастомные исключения наследуются от `HTTPException` (`detail={"error":..., "code":"ITEM_NOT_FOUND"}`);
- глобальные обработчики задаются через `@app.exception_handler(...)`;
- формат 422 можно переопределить через `RequestValidationError`;
- бизнес-правила на несколько полей проверяются в `@root_validator`.

**Production-настройки:**
- `pydantic.BaseSettings` читает `.env`: database_url, secret_key, allowed_origins и т.д.;
- `CORSMiddleware`;
- в production `/docs` отключают (`docs_url=None`).

### 3. Автоматическая документация OpenAPI (лекция 03)

- FastAPI собирает схему OpenAPI из путей и методов, аннотаций (path/query/body/header/cookie), `response_model`, docstring и параметров `summary`, `description`, `tags`, `responses`.
- Интерфейсы: **Swagger UI** на `/docs` и **ReDoc** на `/redoc`. Сырая схема лежит в `/openapi.json`. Все адреса меняются через `docs_url`, `redoc_url`, `openapi_url`, например `/api/docs`.
- Метаданные приложения: `title`, `description` (поддерживает Markdown), `version`, `terms_of_service`, `contact`, `license_info`, `openapi_tags=[{"name":..., "description":...}]`.
- Эндпоинт описывают через docstring (Markdown попадает в description), `summary=`, `description=`, `response_description=`, `tags=[Tags.ITEMS]` (теги хранятся в Enum).
- Параметры: `Query(..., description=, example=)`, `Path(...)`, `Body(..., example={...})`.
- Ответы: `response_model` плюс `responses={404: {"description":..., "model": ErrorResponse}}`.
- Security: `HTTPBearer()` или `OAuth2PasswordBearer` в `dependencies=[Depends(security)]`. Второй путь: кастомная `app.openapi` с `components.securitySchemes.BearerAuth` (`type: http`, `scheme: bearer`, `bearerFormat: JWT`).
- Примеры данных: `Field(..., example=...)` или `Config.schema_extra = {"example": {...}}`.
- `curl .../openapi.json > openapi.json` сохраняет схему для openapi-generator (генерация клиента).
- В production документацию отключают.

### 4. Валидация данных в FastAPI / Pydantic (лекции 01 §4, 04)

- Зачем нужна: безопасность, меньше ручных проверок, понятные ошибки (422 с массивом `detail`), правила сразу попадают в OpenAPI.
- Типы: `int, str, float, bool, List[str], Dict[str,float], Optional[...], Union[int,str]`.
- `Field`:
  - `...` означает обязательное поле, `default` задаёт значение по умолчанию;
  - `min_length/max_length` для строк;
  - `gt/ge/lt/le` для чисел;
  - `regex` (в Pydantic v2 называется `pattern`);
  - `description`, `example`;
  - `min_items` для списков.
- Специальные типы: `EmailStr`, `HttpUrl`, `UUID4`, `PositiveInt`, `constr(min_length=3, to_lower=True)`.
- `@validator('field')` проверяет одно поле. Он получает значение `v`, а через `values` видит уже проверенные поля (например, совпадение `password_confirm`). Должен вернуть значение или бросить `ValueError`. Типовой пример: проверка сложности пароля (длина не меньше 8, есть заглавная буква и цифра).
- `@root_validator` проверяет связи между полями: скидка требует не меньше 3 товаров, в списке нет дублей и т.п.
- `Query/Path/Header/Cookie` поддерживают те же ограничения, например `Header(..., regex="^Bearer .+$")`.
- Вложенные модели валидируются рекурсивно (`User.address: Address`).
- `response_model` отфильтровывает лишние поля в ответе.
- Замечание: в лекциях используется синтаксис **Pydantic v1** (`@validator`, `@root_validator`, `regex=`, `schema_extra`, `.dict()`). В актуальном Pydantic v2 им соответствуют `@field_validator`, `@model_validator`, `pattern=`, `json_schema_extra`, `.model_dump()`, а `BaseSettings` переехал в пакет `pydantic-settings`.

### 5. Документная модель данных (лекция 05)

- Данные хранятся как **документы** в JSON/BSON (BSON добавляет типы Date, ObjectId, Binary и др.). Документы объединяются в **коллекции** (аналог таблиц), и у документов одной коллекции может быть разный набор полей.
- Свойства модели:
  - гибкая схема;
  - вложенность: объекты и массивы вместо JOIN;
  - самодостаточность: заказ хранится вместе с позициями и адресом;
  - документ естественно отображается на объект в коде.
- Разница с SQL:
  - SQL: строгая схема, нормализация, JOIN, миграции при изменении схемы;
  - MongoDB: денормализация и вложенные массивы, новое поле добавляется без миграции.
- `_id` обязателен. Если его не передать, MongoDB сгенерирует `ObjectId`.
- Когда подходит: прототипы, иерархические или изменчивые данные, нагрузка на чтение и горизонтальное масштабирование, логи, профили, каталоги, блоги.
- Когда лучше SQL: сложные транзакции по многим сущностям (финансы), строгая ссылочная целостность, стабильная нормализованная схема.

### 6. MongoDB Atlas (лекция 06)

- **Atlas**: управляемое облако MongoDB с бэкапами, мониторингом, шардированием, глобальными кластерами, шифрованием, IP-whitelist и ролями.
- Бесплатный тариф **M0**: около 512 MB, примерно 5 GB хранилища.
- Шаги:
  1. Зарегистрироваться (по лекции с включённым VPN; можно войти через Google/GitHub).
  2. Создать кластер Shared → M0, выбрать провайдера и регион (например, eu-central-1).
  3. Создать пользователя БД (логин и пароль).
  4. Настроить Network Access: для разработки `0.0.0.0/0`, в production только свои IP.
  5. Нажать Connect → «Connect your application» и получить строку `mongodb+srv://<user>:<pass>@cluster0.xxx.mongodb.net/<db>?retryWrites=true&w=majority`.
- **MongoDB Compass**: GUI для просмотра и редактирования документов, визуальных агрегаций и анализа запросов. Подключается через «Connect using Compass».
- Локальная альтернатива: MongoDB Community Edition (`mongod`, оболочка `mongosh`). В курсе рекомендуется Atlas.

### 7. ODM Mongoose: схемы и модели (лекция 07)

- **Mongoose**: ODM для MongoDB в Node.js. Задаёт схему на уровне приложения, встроенную валидацию, удобный CRUD и преобразование между объектами и документами.
- Подключение: `npm install mongoose`, затем `mongoose.connect(process.env.MONGO_URI)`.
- **Schema** описывает документ (поля, типы, валидаторы, значения по умолчанию). **Model** строится по схеме и выполняет CRUD.
```js
const userSchema = new Schema({
  name:  { type: String, required: [true, 'обязательно'], minlength: 3, maxlength: 50 },
  email: { type: String, required: true, unique: true, lowercase: true, match: /^\S+@\S+\.\S+$/ },
  age:   { type: Number, min: 0, max: 120 },
  isActive: { type: Boolean, default: true },
}, { timestamps: true });            // createdAt / updatedAt
module.exports = mongoose.model('User', userSchema);   // коллекция 'users'
```
- Имя модели пишется в единственном числе, коллекция получает имя во множественном числе в нижнем регистре (`User` → `users`).
- Встроенные валидаторы: `required`, `unique` (строго говоря, это индекс), `minlength/maxlength`, `min/max`, `enum: {values, message: '{VALUE} ...'}`, `match`, `default`.
- Типы: String, Number, Date, Boolean, ObjectId, Array, а также вложенные объекты.
- Связи: `{ type: Schema.Types.ObjectId, ref: 'User' }` и `.populate('field')`. Эта тема упомянута в вопросах, в лекциях не раскрыта.

### 8. CRUD через Mongoose (лекция 08)

| Операция | Методы | Замечания |
|---|---|---|
| Create | `new M(d).save()`, `M.create(obj \| [objs])`, `M.insertMany([...])` | чаще всего используется `create`, `insertMany` для массовой вставки |
| Read | `find(filter, projection)`, `findById`, `findOne`, `countDocuments` | цепочки `.sort({name:1}).skip(10).limit(5)`, проекция вида `'name email -_id'`, операторы вроде `{age:{$gte:18}}` |
| Update | `updateOne(f, {$set})`, `updateMany`, `findByIdAndUpdate(id, d, {new:true, runValidators:true})`, `findOneAndUpdate` | `new:true` возвращает обновлённый документ. Валидация при update срабатывает только с `runValidators: true` |
| Delete | `deleteOne`, `deleteMany`, `findByIdAndDelete`, `findOneAndDelete` | `findByIdAndDelete` возвращает удалённый документ (или null) |

Типовой Express-роутер: `GET /` с пагинацией (`page`, `limit`, `countDocuments`), `POST /` (201, при ошибке валидации 400), `GET /:id` (404, если не найден), `PUT /:id`, `DELETE /:id` (204). Каждый обработчик обёрнут в `try/catch` с `res.status(500)`. В современном коде используется `async/await`.

### 9. WebSocket и реальное время (лекция 09)

**Почему HTTP плохо подходит для real-time:**
- сервер не может сам отправить данные клиенту;
- short polling создаёт много пустых запросов и задержку до N секунд;
- long polling по сути костыль: множество висящих соединений, обрывы на таймаутах прокси;
- каждый запрос несёт около 800 байт заголовков, иногда ещё и новый TCP handshake;
- HTTP не хранит состояние;
- браузер держит не больше ~6 соединений на домен.

**WebSocket (RFC 6455, 2011):** полный дуплекс поверх одного TCP-соединения.
1. Handshake: клиент шлёт HTTP-запрос с `Upgrade: websocket`, сервер отвечает `101 Switching Protocols`.
2. Соединение остаётся открытым.
3. Данные идут фреймами с заголовком 2–14 байт, текстовыми (UTF-8) или бинарными.
4. Любая сторона может закрыть соединение close-фреймом.

Плюсы: низкая задержка и накладные расходы, push от сервера. Минусы: прокси и балансировщики должны поддерживать upgrade, нет автопереподключения, нет комнат, авторизации и broadcast «из коробки».

```js
// браузер
const ws = new WebSocket('ws://localhost:8080');
ws.onopen = () => ws.send('hi'); ws.onmessage = e => console.log(e.data);
ws.onerror = ...; ws.onclose = ...;
// сервер: npm i ws
const wss = new (require('ws').Server)({ port: 8080 });
wss.on('connection', s => { s.on('message', m => s.send(`Эхо: ${m}`)); s.on('close', ...); });
```
Применения: чаты, игры, финансовые тикеры, **совместное редактирование**, ленты уведомлений.

**Socket.IO:**
- автопереподключение с экспоненциальной задержкой;
- комнаты и namespaces;
- middleware для авторизации;
- fallback на long polling;
- именованные события вместо «голых» строк;
- бинарные данные и acknowledgements;
- масштабирование через Redis-адаптер.

```js
const io = new Server(http.createServer(app), { cors: { origin: '*' } });
io.use((socket, next) => {                      // auth middleware
  const t = socket.handshake.auth.token; valid(t) ? next() : next(new Error('Unauthorized'));
});
io.on('connection', socket => {
  socket.on('chat message', msg => {
    socket.broadcast.emit('chat message', msg);  // всем, кроме отправителя
    // io.emit(...) — всем; socket.emit(...) — только этому клиенту
  });
  socket.join('room1'); io.to('room1').emit('message', '...'); socket.leave('room1');
  socket.on('disconnect', () => {});
});
```
Клиент: `<script src="/socket.io/socket.io.js">`, CDN или `socket.io-client`. Далее `const socket = io(url); socket.on('connect', ...); socket.emit(...)`.

**Полный пример чата из лекции:**
- сервер хранит последние 100 сообщений и при подключении отправляет `history`;
- событие `new message` превращается в `{id, username, text, timestamp}` и рассылается как `io.emit('message added')`;
- `typing` рассылается через `socket.broadcast.emit('user typing')`, клиент гасит индикатор по таймауту в 1 с;
- статика отдаётся через `express.static('public')`.

**Альтернативы:**
- **SSE** (`EventSource`): односторонний push от сервера по HTTP. Подходит для уведомлений и тикеров.
- **WebRTC**: P2P-передача аудио, видео и данных (`getUserMedia`, `RTCPeerConnection`, `RTCDataChannel`). Для signalling нужен сервер, часто на WebSocket.

Вывод лекции: для двусторонней real-time связи в учебных проектах Socket.IO оптимален.

---

## Лабораторные работы

### Лабораторная работа №6. «Интеграция с NoSQL базой данных MongoDB»

- **Ветка репозитория:** `26`.
- **Срок:** срок не указан в материалах.
- **Выполнение:** индивидуальное. Тема варианта определяется по «Генератору вариантов». Сохранённые страницы генератора пустые (ошибка Moodle), поэтому варианты неизвестны. По формулировкам заданий вариант, судя по всему, задаёт предметную область, а курсовой проект сам ориентирует на неё схему.
- **Цель:** освоить MongoDB: подключение, схемы и модели через ODM Mongoose, CRUD, документную модель (вложенные документы, массивы). Задача сделать альтернативную реализацию хранения данных курсового проекта **в дополнение к PostgreSQL**.
- **Необходимые знания:** документные БД; MongoDB (коллекции, документы, BSON, ObjectId); Mongoose (схема, модель, методы); строка подключения и Atlas; отличия от PostgreSQL.
- **Стек:** MongoDB Atlas (M0) или локальная MongoDB Community, **Node.js**, **Express**, **Mongoose**, dotenv (рекомендуется), Postman, MongoDB Compass или Atlas Data Explorer.

**Задачи:**
1. Создать бесплатный кластер Atlas (или поставить MongoDB локально).
2. Подключить Mongoose к **Node.js-приложению** (из прошлых работ или новому).
3. Определить схему и модель коллекции по предметной области курсового проекта.
4. Реализовать CRUD через **Express-маршруты**.
5. Использовать вложенные документы или массивы по логике проекта: теги, комментарии, корзина и т.п.
6. Показать отличие от реляционной модели: те же данные одним документом вместо нескольких таблиц.

**Порядок выполнения:**
- **Часть 1. Настройка MongoDB.** Регистрация на Atlas, кластер M0, пользователь БД (пароль сохранить), Connection String. Альтернатива: локальная Community Edition, в коде меняется только URI.
- **Часть 2. Зависимости.** `npm install mongoose`. Рекомендуется также `dotenv`, строку подключения хранить в `.env` (`MONGO_URI`).
- **Часть 3. Подключение.** В `server.js` или `db.js`: `mongoose.connect(process.env.MONGO_URI).then(...).catch(...)`.
- **Часть 4. Схема и модель.** Папка `models/`, файл `<ModelName>.js`. **Обязательно** хотя бы один вложенный документ или массив. Пример из методички: Product с полями `name` (required), `price`, `category`, `reviews: [{userId, rating, comment}]`, `tags: [String]`, `createdAt` (default `Date.now`).
- **Часть 5. CRUD-маршруты Express:**

| Маршрут | Mongoose |
|---|---|
| `GET /items` | `Model.find()` |
| `GET /items/:id` | `Model.findById(id)` |
| `POST /items` | `new Model(body).save()` или `Model.create(body)` |
| `PUT /items/:id` | `Model.findByIdAndUpdate(id, body, { new: true })` |
| `DELETE /items/:id` | `Model.findByIdAndDelete(id)` |

- **Часть 6. Вложенные структуры.** Нужен хотя бы один эндпоинт, работающий с вложенным массивом (добавить отзыв, подзадачу, участника события). Также надо показать обновление элемента **внутри массива**, например увеличение количества товара в корзине (`$push`, позиционный `$` / `$inc`, `arrayFilters`).
- **Часть 7. Проверка.** Запустить сервер, протестировать CRUD в Postman, показать вложенную структуру в Compass или Atlas.

**Отчёт (PDF в СЭО):**
- титульный лист и цель;
- краткое сравнение SQL и NoSQL;
- скриншоты: дашборд Atlas с кластером; URI в `.env`/config; схема и модель (листинг); запросы GET/POST/PUT/DELETE в Postman; данные в Compass или Atlas Data Explorer с вложенной структурой;
- листинги ключевого кода: подключение, модель, маршруты;
- ответы на 16 контрольных вопросов;
- выводы.

**Требования к сдаче:** отчёт PDF в СЭО, ссылка на GitHub (ветка `26`), индивидуальная работа, вариант темы по генератору.

**Примечания методички:**
- ЛР можно делать **отдельным проектом** (чтобы не смешивать с PostgreSQL) или параллельной реализацией того же API на MongoDB в отдельной ветке.
- Рекомендация: ветка `26` с копией сервера, где **Sequelize заменён на Mongoose**.
- В курсовом проекте можно будет выбрать БД, реляционную или документную.
- Atlas должен быть доступен в Беларуси. Если нет, использовать локальную MongoDB.
- Postman для тестов, Compass для просмотра данных.

**Критерии оценки:**
1. Atlas настроен, приложение подключается.
2. Схема и модель соответствуют предметной области и содержат вложенные структуры.
3. Реализованы все CRUD-операции (GET список, GET по id, POST, PUT/PATCH, DELETE) через Mongoose.
4. Вложенные документы или массивы использованы по логике проекта.
5. Качество кода: структура, переменные окружения, обработка ошибок.
6. Полный отчёт: скриншоты, листинги, ответы.
7. Ссылка на репозиторий (ветка 26) с **инструкцией по запуску**.

**Контрольные вопросы (16):**
1. Минимум 4 отличия MongoDB от PostgreSQL/MySQL.
2. Что такое ObjectId, как генерируется, зачем нужен.
3. Поиск по вложенному полю (`{"profile.city": "Минск"}`).
4. Типы BSON (не меньше 5).
5. Aggregation pipeline, пример минимум из 2 стадий (`$match`, `$group`).
6. Вложенный документ (subdocument) в Mongoose без отдельной коллекции, пример из своего проекта.
7. Обновление элемента в массиве (увеличить количество в корзине).
8. Плюсы и минусы вложенных документов против отдельных коллекций.
9. Индекс на поле в Mongoose (например, email).
10. `populate` и когда он нужен.
11. Поиск «массив содержит значение» (`{tags: "важный"}`).
12. Удалить по id и вернуть удалённый документ.
13. Какие методы возвращают Query и поддерживают цепочки (`.where().limit().sort()`).
14. Instance-методы и static-методы модели.
15. Каскадное удаление в MongoDB/Mongoose.
16. Для каких приложений MongoDB лучше или хуже реляционных БД, применительно к курсовому проекту.

### Лабораторная работа №7. «Реализация реального времени с WebSocket»

- **Ветка репозитория:** `27`. Допускается `27-websocket` для упрощённого демо-клиента.
- **Срок:** срок не указан в материалах.
- **Выполнение:** индивидуальное, вариант темы по «Генератору вариантов» (данные о вариантах недоступны).
- **Цель:** освоить WebSocket для двусторонней real-time связи, изучить **Socket.IO**, сделать обмен сообщениями или уведомлениями под нужды курсового проекта. Примеры: чат поддержки, лайвы, уведомления о событиях, **совместное редактирование**.
- **Необходимые знания:** ограничения HTTP; WebSocket (handshake, постоянное соединение); Socket.IO (сервер и клиент); события connection, disconnect, message, broadcast, join, leave.
- **Стек:** **Node.js + Express + Socket.IO** на сервере, `socket.io-client` в React-приложении (или отдельный HTML/JS), опционально MongoDB/Mongoose для истории.

**Задачи:**
1. Создать сервер на **Node.js + Express + Socket.IO**.
2. Создать клиент: React-приложение из прошлых работ или отдельный HTML/JS.
3. Реализовать отправку и получение сообщений или уведомлений в реальном времени.
4. Добавить уведомления о подключении и отключении пользователей.
5. Добавить функции по логике курсового проекта: комнаты, история в MongoDB, приватные сообщения, голосование и т.п.

**Порядок выполнения:**
- **Часть 1. Сервер.**
  1. `npm install socket.io`.
  2. `http.createServer(app)` → `new Server(server, { cors: { origin: '*' } })`.
  3. `io.on('connection', socket => { socket.on('chat message', msg => io.emit('chat message', msg)); socket.on('disconnect', ...) })`.
  4. `server.listen(3000)`.
- **Часть 2. Клиент (React).**
  1. `npm install socket.io-client`.
  2. `import io from 'socket.io-client'; const socket = io('http://localhost:3000')`.
  3. Отправка: `socket.emit('chat message', text)`.
  4. Приём: `socket.on('chat message', msg => ...)`.
- **Часть 3. Дополнительные функции** (одна или несколько):
  - комнаты для групп пользователей (разные заказы или товары);
  - приватные сообщения по `socket.id`;
  - сохранение истории в MongoDB через Mongoose;
  - индикатор набора текста (`typing`);
  - голосования и опросы в реальном времени;
  - **синхронизация состояния**, например совместное редактирование списка задач.
- **Часть 4. Тестирование.** Запустить сервер и клиент, открыть несколько окон или браузеров как разных пользователей. Проверить обмен сообщениями, уведомления о подключении и отключении, комнаты и остальные функции.

**Отчёт (PDF в СЭО):**
- титульный лист и цель;
- схема работы WebSocket (текст или диаграмма);
- листинг сервера (обработчики событий);
- листинг клиента (React-компонент);
- скриншоты с **двумя окнами** (разные пользователи);
- демонстрация дополнительных функций;
- ответы на 16 контрольных вопросов;
- выводы.

**Требования к сдаче:** PDF в СЭО, GitHub (ветка `27`), индивидуальная работа, вариант по генератору.

**Примечания методички:**
- Socket.IO сам использует WebSocket, а при необходимости переключается на long polling.
- История хранится в Atlas M0.
- Клиент можно сделать отдельным React-компонентом основного проекта (ветка 27) или упрощённым клиентом в ветке `27-websocket`.

**Критерии оценки:**
1. Базовый сервер и клиент: соединение, отправка и приём.
2. Уведомления о подключении и отключении.
3. Дополнительные функции по логике проекта.
4. Качество кода: структура, обработка ошибок, комментарии.
5. Полный отчёт: скриншоты, листинги, схема взаимодействия.
6. Ссылка на репозиторий (ветка 27) и **README с инструкцией по запуску**.

**Контрольные вопросы (16):**
1. WebSocket против HTTP, почему для real-time подходит именно WebSocket.
2. **Какую библиотеку вы использовали и почему?** Чем Socket.IO лучше нативного WebSocket.
3. Комнаты: как создать и как слать только в комнату.
4. Отправка всем, кроме отправителя.
5. Обработка disconnect на сервере и на клиенте.
6. Событие «печатает» без отправки текста.
7. Передача данных (имени пользователя) при подключении (`handshake.auth` / `query`).
8. История сообщений в MongoDB через Mongoose, какие поля нужны проекту.
9. Приватное сообщение по `socket.id` (`io.to(id).emit`).
10. Acknowledgement: подтверждение получения через callback в `emit`.
11. Масштабирование на несколько серверов (Redis-адаптер), теоретически.
12. Автоматические события при обрыве и переподключении (`disconnect`, `connect_error`, `reconnect`, `reconnect_attempt` и др.).
13. Ограничение размера или частоты сообщений (`maxHttpBufferSize`, rate limiting).
14. Защита от неавторизованного подключения (проверка токена в `io.use`).
15. Backpressure и пауза отправки при медленном соединении (`volatile`, буферизация, ack).
16. Разница `socket.emit`, `io.emit`, `socket.broadcast.emit`.

---

## Вопросы для самоподготовки

**Python и FastAPI:**
1. Отличия Node.js и Python/FastAPI.
2. Ключевые особенности FastAPI.
3. Установка, запуск через Uvicorn, минимальное приложение.
4. Path- и query-параметры, пример.
5. POST с JSON-телом и валидация Pydantic.
6. Pydantic-модель User (name, email).
7. `Field` с ограничениями min_length, gt, regex.
8. `@validator`: кастомная проверка пароля.
9. Автодокументация OpenAPI, адреса Swagger UI и ReDoc.
10. `HTTPException` 404.

**MongoDB:**

11. Документная модель и отличия от РСУБД.
12. Коллекция, документ, поле, пример JSON.
13. Atlas и возможности M0.
14. Подключение из Node.js, строка подключения.
15. Что такое ODM и зачем нужен Mongoose.
16. Schema и Model, пример Product(name, price).
17. Типы Mongoose.
18. CRUD: create, find, findById, findByIdAndUpdate, findByIdAndDelete.
19. Встроенная валидация: required, min, max, enum.
20. `ref` и `populate`.

**WebSocket:**

21. Ограничения HTTP.
22. Протокол WebSocket и handshake.
23. Браузерный WebSocket API и его события.
24. Эхо-сервер на `ws`.
25. Socket.IO и его преимущества.
26. Установка Socket.IO на сервере и клиенте.
27. Пользовательские события (chat message).
28. Комнаты.
29. `socket.broadcast.emit`.
30. Последовательность реализации чата на Socket.IO.

---

## Как применить в проекте NotaCode

### Ключевой вопрос: разрешён ли Python/FastAPI в ЛР6/ЛР7?

**Ответ: теория модуля про FastAPI, а обе лабораторные написаны под Node.js.** Явного разрешения выполнять ЛР6/ЛР7 на Python/FastAPI в материалах нет. Явного запрета тоже нет, но задачи и критерии сформулированы через Node-инструменты.

Фразы, которые задают стек (цитаты короче 15 слов):

| Источник | Фраза | Что из неё следует |
|---|---|---|
| Лекция 01, вывод | «В нашем курсе мы фокусируемся на FastAPI» | FastAPI входит в курс как теория (в этом модуле нет ни одной FastAPI-лабораторной) |
| ЛР6, задачи | «Подключить Mongoose к Node.js-приложению» | требуются **Node.js + Mongoose** |
| ЛР6, задачи | «Реализовать CRUD-операции через Express-маршруты» | требуется **Express** |
| ЛР6, критерии | «Реализация всех CRUD-операций … с использованием Mongoose» | Mongoose входит в критерий оценки |
| ЛР6, примечания | «может выполняться как отдельный проект» | можно отдельный сервис, не трогая основной бэкенд |
| ЛР6, примечания | «создать копию сервера, заменив Sequelize на Mongoose» | предполагается Node-сервер из модуля 01 |
| ЛР7, задачи | «Создать сервер на Node.js + Express + Socket.IO» | требуются **Node.js + Express + Socket.IO** |
| ЛР7, цель | «Изучение библиотеки Socket.IO» | Socket.IO обязателен по смыслу |
| ЛР7, контр. вопрос 2 | «Какую библиотеку вы использовали и почему?» | слабый намёк на свободу выбора библиотеки, но вопрос тут же сравнивает Socket.IO с нативным WS |
| ЛР7, примечания | «создать отдельную ветку 27-websocket с упрощённым клиентом» | допустим отдельный упрощённый демо-клиент |
| ЛР6, примечания | «В курсовом проекте вы сможете выбрать, какую базу данных» | в **курсовом** выбор БД свободный |

**Вывод и рекомендация.** Чтобы гарантированно получить зачёт, обе лабораторные надо делать на **Node.js** (Express + Mongoose, Express + Socket.IO) как **отдельные небольшие сервисы в монорепо NotaCode**. Основной бэкенд FastAPI при этом не меняется. Если хочется сделать ЛР на FastAPI (Motor/Beanie, нативный WebSocket FastAPI), это нужно заранее и письменно согласовать с преподавателем. Материалы такой замены не предусматривают, а критерии прямо называют Mongoose и Socket.IO. Гибридная схема из лекции 01 («Node.js для шлюза, Python для ML», «Можно комбинировать») даёт хорошее обоснование в отчёте, почему в NotaCode два рантайма.

### ЛР6 (ветка `26`): MongoDB в NotaCode

**Требования:** Node.js, Express, Mongoose, MongoDB Atlas (или локальная MongoDB), `.env`, полный CRUD, вложенные массивы или документы, обновление элемента массива, Postman, Compass.

**Предлагаемая реализация (основной вариант, по методичке):** отдельный сервис `services/docs-mongo/` (Node 20+, Express, Mongoose) в ветке `26`, так как ЛР разрешено делать отдельным проектом. Сервис хранит **документы диаграмм** в документной модели, где вложенность выглядит естественно:

```js
// models/Diagram.js
const revisionSchema = new Schema({            // subdocument без отдельной коллекции
  source: { type: String, required: true },     // текст DSL
  message: String, authorId: String,
  createdAt: { type: Date, default: Date.now },
});
const commentSchema = new Schema({ userId: String, line: Number, text: String, resolved: { type: Boolean, default: false } }, { timestamps: true });
const diagramSchema = new Schema({
  projectId: { type: String, index: true },
  ownerId: { type: String, required: true, index: true },   // id пользователя из FastAPI/Postgres
  title: { type: String, required: true, maxlength: 200 },
  engine: { type: String, enum: ['mermaid', 'plantuml', 'graphviz'], default: 'mermaid' },
  source: { type: String, default: '' },
  tags: [String],
  revisions: [revisionSchema],                  // история версий внутри документа
  comments: [commentSchema],
  collaborators: [{ userId: String, role: { type: String, enum: ['viewer', 'editor'] } }],
  settings: { theme: String, direction: String },   // вложенный объект
}, { timestamps: true });
diagramSchema.index({ tags: 1 });
```

**Эндпоинты** (префикс `/api/v2/diagrams` в сервисе на отдельном порту, например 4000):
- `GET /` использует `find()` с фильтрами `?tag=` (массив содержит значение), `?engine=`, `?ownerId=` и пагинацией `sort/skip/limit`;
- `GET /:id` → `findById`; `POST /` → `create`; `PUT /:id` → `findByIdAndUpdate(..., {new:true, runValidators:true})`; `DELETE /:id` → `findByIdAndDelete` (возвращает удалённый документ);
- **вложенные структуры:**
  - `POST /:id/revisions` делает `$push` ревизии (снапшот `source`);
  - `POST /:id/comments` добавляет комментарий;
  - `PATCH /:id/comments/:commentId` использует позиционный `$` / `arrayFilters` и ставит `comments.$.resolved = true`. Это покрывает требование «обновить элемент внутри массива»;
  - `PATCH /:id/collaborators/:userId` меняет роль участника;
- для раздела «отличие от реляционной модели»: в Postgres (FastAPI + SQLAlchemy) то же хранится в таблицах `diagrams`, `diagram_revisions`, `comments`, `project_members` с FK и JOIN, а в MongoDB это один документ `Diagram`. Для отчёта можно добавить пример aggregation: `$unwind: revisions` → `$group` по `authorId`, результат «число ревизий по авторам» (контрольный вопрос 5).

**Интеграция с основным стеком (необязательно, но красиво):**
- аутентификация: Node-сервис проверяет тот же JWT, что выдаёт FastAPI (общий `JWT_SECRET` / публичный ключ). В `ownerId` пишется `sub` из токена;
- фронт (React/Vite) ходит в `/api/v2` через Vite dev proxy, в prod через nginx/Caddy;
- альтернатива: FastAPI проксирует `/api/v2/*` в Node-сервис (httpx) и остаётся единой точкой входа.

**Отчёт:** скриншоты Atlas, `.env` с замаскированным паролем, Postman-коллекция `NotaCode-Mongo.postman_collection.json`, Compass с раскрытыми `revisions[]` и `comments[]`, README с запуском (`npm i && npm run dev`, `MONGO_URI=...`).

**Вариант на будущее, не для зачёта ЛР6:** если в курсовом выбрать MongoDB как основное хранилище, в FastAPI это делается через **Motor** (async-драйвер) или **Beanie** (ODM на Pydantic, ближайший аналог Mongoose: `Document`, вложенные `BaseModel`, `Indexed`, `Link` вместо ref/populate). Модели Beanie переиспользуют Pydantic-схемы из лекций 02–04. Сдать это как ЛР6 можно только по договорённости: в критериях прямо назван Mongoose.

### ЛР7 (ветка `27`): реальное время в NotaCode

**Требования:** Node.js + Express + Socket.IO, клиент React на `socket.io-client`, уведомления connect/disconnect, дополнительные функции по логике проекта, демо в двух окнах.

**Предлагаемая реализация:** сервис `services/realtime/` (Express + Socket.IO, порт 4001) для **совместного редактирования диаграммы и live preview**:
- **комната на диаграмму:** `socket.join('diagram:' + diagramId)`, событие `diagram:join`. Остальным участникам уходит `presence:joined` / `presence:left` с именем. Этим закрываются критерии о подключении и отключении и о комнатах;
- **передача пользователя при подключении:** `io({ auth: { token } })`, на сервере `io.use` проверяет JWT от FastAPI и заполняет `socket.data.user` (контрольные вопросы 7 и 14);
- **синхронизация исходника:**
  - клиент шлёт `source:update {diagramId, text, version}` с debounce 150–300 мс;
  - сервер рассылает `socket.to(room).emit('source:updated', ...)` всем, кроме автора;
  - через ack-callback подтверждается номер версии (контрольный вопрос 10);
  - на уровне ЛР хватает стратегии last-write-wins с версией. Для продвинутого варианта годится **Yjs** (CRDT) с `y-socket.io` или собственный relay бинарных update-сообщений Yjs;
- **курсоры и выделения:** `cursor:move {line, col}` отправляется как `volatile.emit` (устаревшие позиции можно терять, это и есть пример backpressure, контрольный вопрос 15). Плюс индикатор `typing`;
- **live preview:** рендерит клиент (Mermaid в браузере), поэтому по сокету идёт только текст. Для PlantUML сервер после `source:updated` может вызвать FastAPI `/render` и разослать `preview:ready {svgUrl}`;
- **комментарии и уведомления:** `comment:new`, а также приватное уведомление автору через `io.to(socketIdOrUserRoom)` (контрольный вопрос 9). Для приватных сообщений удобнее персональная комната `user:<id>`;
- **история в MongoDB:** переиспользовать модель `Diagram` из ЛР6: сохранять ревизию по `diagram:save` или по таймеру, а комментарии чата комнаты складывать в коллекцию `RoomMessage {diagramId, userId, text, createdAt}` (контрольный вопрос 8);
- **ограничения:** `maxHttpBufferSize` (например, 1 МБ на исходник) и простой rate limit по `socket.id` (не больше N событий в секунду);
- **React-клиент:**
  - хук `useDiagramSocket(diagramId)` с подключением в `useEffect` и `socket.off(...)` / `disconnect()` в cleanup;
  - компонент `CollaboratorsBar` показывает, кто онлайн;
  - редактор (CodeMirror/Monaco) применяет входящие изменения;
  - для скриншотов открыть одну диаграмму в двух окнах (Chrome и Firefox или обычное окно и инкогнито).

**Отчёт:** схема «клиент ⇄ Socket.IO (handshake Upgrade → 101) ⇄ комната `diagram:<id>` ⇄ MongoDB / FastAPI render». Её можно нарисовать в самом NotaCode как sequence diagram на Mermaid. Также нужны листинги обработчиков, React-хук, скриншоты двух окон, README.

**Альтернатива на FastAPI (только по согласованию):** в FastAPI есть нативный `@app.websocket("/ws/diagrams/{id}")` с `WebSocket.accept/receive_text/send_text` и `ConnectionManager` (словарь комнат). Есть также `python-socketio` (ASGI-сервер, совместимый с клиентом `socket.io-client`, с комнатами, ack и Redis-менеджером). Формально это ближе всего к требованию «Socket.IO», поскольку клиентская часть остаётся той же, но методичка требует Node.js + Express. Поэтому для гарантированной сдачи лучше Node-сервис, а `python-socketio` стоит оставить на случай, когда realtime переносят в основной бэкенд в курсовом.

### Как теория модуля применяется к основному бэкенду NotaCode (FastAPI)

- **Структура:** `app/api/endpoints/{auth,projects,diagrams,share,render}.py`, `schemas/`, `services/`, `core/config.py` (`BaseSettings` / `pydantic-settings` с `.env`).
- **Pydantic-валидация:**
  - `DiagramCreate(title: Field(min_length=1, max_length=200), engine: Literal/Enum, source: Field(max_length=200_000))`;
  - `UserCreate` с `@field_validator` сложности пароля и `@model_validator` для совпадения `password_confirm`;
  - `ShareLinkCreate` с `expires_at > now`.
- **DI:** `get_db` (сессия SQLAlchemy), `get_current_user` (JWT Bearer, 401), `require_project_editor` (403), пагинация `skip/limit` через `Query(ge=0, le=100)`.
- **OpenAPI:**
  - теги «Auth», «Projects», «Diagrams», «Sharing», «Render»;
  - `responses={404: {"model": ErrorResponse}}`;
  - security-схема BearerAuth, чтобы работала кнопка Authorize в Swagger;
  - `/docs` отключается в production;
  - `/openapi.json` отдаётся в openapi-typescript / orval для генерации типизированного клиента React.
- **Ошибки:** единый формат `{"error", "code", "details"}` через `exception_handler(RequestValidationError)` и кастомные `DiagramNotFound(HTTPException)`.
- **Сравнение Node и Python** пригодится в отчётах и защите: FastAPI остаётся ядром (валидация, документация, рендер и интеграции), а Node-сервисы отвечают за учебные требования (Mongoose, Socket.IO). Эта схема совпадает со строкой матрицы «микросервисы: можно комбинировать».

---

## Нечитаемые материалы

- `Генератор вариантов .html` и `Генератор вариантов (391477).html` — сохранённые HTML-страницы ошибки СЭО (Moodle, заголовок «Ошибка | СЭО»). Данных о вариантах нет, поэтому распределение тем для ЛР6/ЛР7 неизвестно. Его нужно смотреть в СЭО непосредственно.
