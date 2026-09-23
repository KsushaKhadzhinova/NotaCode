# Модуль 01. Серверная разработка на Node.js

Конспект модуля ИТиВП ч.2 (БГУИР): Node.js и Express, маршрутизация, middleware и EJS, Sequelize/PostgreSQL, REST/GraphQL, аутентификация (JWT, bcrypt, OAuth), валидация и веб-безопасность, DevSecOps. Здесь же полный разбор ЛР 1–3 и ПЗ 1–2 и план, как сдать их в проекте NotaCode.

---

## Источники

Папка: `E:\ИТиВП\Часть 2\Модуль 01. Серверная разработка на Node.js\` (40 PDF). Тексты извлечены pypdf и прочитаны. Лабораторные, практические и вопросы прочитаны полностью, теория прочитана или внимательно просмотрена.

| № | Файл (кратко) | Стр. |
|---|---|---|
| 01 | SSR — рендеринг на сервере | 4 |
| 02 | Node.js. Установка | 4 |
| 03 | Модульная система (CommonJS, ES Modules) | 6 |
| 04 | Глобальные объекты в Node.js | 7 |
| 05 | Менеджеры пакетов npm и Yarn | 11 |
| 06 | Создание сервера (Express) | 10 |
| 07 | Маршрутизация (Routing) | 9 |
| 08 | Обработка параметров URL и строк запроса | 8 |
| 09 | Статические файлы | 8 |
| 10 | Middleware и шаблонизаторы | 12 |
| 11 | Работа с базами данных. ORM Sequelize | 24 |
| 12 | Принципы REST | 11 |
| 13 | Пример REST API на Express.js (с Mongoose-моделью) | 2 |
| 14 | Проектирование эндпоинтов | 10 |
| 15 | Создание GraphQL-эндпоинтов | 8 |
| 16 | Коды состояния HTTP | 11 |
| 17 | Реализация CRUD API | 12 |
| 18 | Документирование API | 9 |
| 19 | Сессии vs JWT-токены | 14 |
| 20 | Регистрация и аутентификация пользователей | 14 |
| 21 | Хеширование паролей (bcrypt) | 15 |
| 22 | Middleware для проверки прав доступа | 20 |
| 23 | Аутентификация и авторизация (JWT, OAuth 2.0, OIDC) | 4 |
| 24 | JWT (JSON Web Token) | 2 |
| 25 | OAuth (Open Authorization) | 4 |
| 26 | Валидация входящих данных | 17 |
| 27 | Основы безопасности: инъекции, XSS, CSRF | 15 |
| 28 | Протоколы защиты от атак (DDoS, фишинг, SQL-инъекции) | 5 |
| 29 | Защитные меры: санитизация, CORS | 13 |
| 30 | Защита данных в сети: TLS/SSL, VPN, IPsec | 5 |
| 31 | Брандмауэры и IDS/IPS | 5 |
| 32 | Защита пользователей и данных в облаке | 5 |
| 33 | Безопасность на клиенте при работе с API | 2 |
| 34 | DevSecOps | 9 |
| 35 | Вопросы для самоподготовки (30 вопросов) | 2 |
| 36 | **ЛР 1** «Разработка серверного приложения на Node.js» | 5 |
| 37 | **ЛР 2** «Работа с базами данных и ORM Sequelize» | 5 |
| 38 | **ЛР 3** «Реализация системы аутентификации и авторизации» | 7 |
| 39 | **ПЗ 1** «Middleware и шаблонизаторы» | 5 |
| 40 | **ПЗ 2** «Проектирование и реализация REST API» | 5 |

Нечитаемых файлов нет. В лекциях 15, 24, 25, 33 есть картинки (скриншоты кода и схемы), но текст извлёкся нормально. В PDF лабораторных и практических картинок нет, текст полный.

---

## Ключевая теория

### 1. SSR, SPA и SSG (лекция 01)

- **SSR**: сервер на каждый запрос выполняет компоненты (в React это `ReactDOMServer.renderToString()`), вставляет HTML в шаблон и отдаёт готовую страницу. Затем на клиенте идёт **гидрация**: JS не пересоздаёт DOM, а навешивает обработчики на существующую разметку. После неё приложение работает как SPA.
- Плюсы SSR: быстрый первый показ контента, SEO, слабые устройства, Open Graph и Twitter Cards. Минусы: нужен JS-сервер, нагрузка на CPU, Time-to-Interactive позже первого показа, нет `window` и `document` на сервере.
- Фреймворки: Next.js (React), Nuxt (Vue), Angular Universal, SvelteKit.

| | SPA | SSR | SSG |
|---|---|---|---|
| Где HTML | браузер | сервер, на каждый запрос | сервер, при сборке |
| Первая загрузка | медленная | быстрая | очень быстрая |
| SEO | низкое | высокое | высокое |
| Нагрузка на сервер | низкая (только API) | высокая | очень низкая |
| Сценарий | админки, дашборды, IDE | магазины, соцсети | блоги, документация, лендинги |

### 2. Node.js: среда, установка, модули, глобальные объекты, npm (лекции 02–05)

- Node.js — это **среда выполнения** JS на движке V8, а не язык. Асинхронная событийная модель: event loop и неблокирующий ввод-вывод. Много I/O-операций (БД, файлы, сеть) обслуживаются без блокировки главного потока.
- Установка: LTS-версия с nodejs.org, `brew install node` / `apt install nodejs npm` или **nvm** (`nvm install --lts`, `nvm use`, `nvm alias default`). Проверка: `node -v`, `npm -v`.
- Сервер без фреймворка: `http.createServer((req,res)=>{res.writeHead(200,{...}); res.end('...')}).listen(3000)`.
- **CommonJS**: `module.exports = {...}` / `exports.fn = ...`, импорт через `require('./x')`. Загрузка синхронная.
- **ES Modules**: `"type": "module"` в package.json или расширение `.mjs`. Синтаксис `export`, `export default`, `import x, {a} from './x.js'`. Загрузка асинхронная, есть статический анализ и tree-shaking. Совместимость: `createRequire(import.meta.url)` в ESM и динамический `await import()` в CJS. Для новых проектов рекомендуется ESM. Расширения: `.js`, `.mjs`, `.cjs`. Двойной пакет собирается через поле `exports: {import, require}`.
- **Глобальные объекты**: `global` (аналог `window`), `console` (`log/error/warn/table/time`), `process` (`version`, `platform`, `pid`, `cwd()`, `env`, `argv`, `exit()`, `on('uncaughtException'|'unhandledRejection'|'SIGINT'|'SIGTERM')`), `__filename` и `__dirname` (в ESM их получают через `fileURLToPath(import.meta.url)`), `Buffer` (`from`, `alloc`, base64 и hex), таймеры `setTimeout/setInterval/setImmediate`, `queueMicrotask`, `URL`. Конфигурацию лучше хранить в `process.env`, а не в `global`.
- **npm**: `npm init -y`, `npm i pkg`, `-D`/`--save-dev`, `-g`, `pkg@версия`, `uninstall`, `outdated`, `update`, `list --depth=0`, `view`, `audit` / `audit fix`, `npm run <script>` (`npm start` пишется без `run`). Разделы `dependencies`, `devDependencies`, `peerDependencies` и `engines`. Версии по semver (`^`, `~`). `package-lock.json` фиксирует точные версии, `node_modules` добавляется в `.gitignore`. **Yarn** — альтернатива (`yarn add`, `yarn.lock`), для учебных проектов не обязателен. Scope-пакеты `@org/pkg`.

### 3. Express: сервер, маршрутизация, параметры, статика (лекции 06–09)

- Express — минималистичный фреймворк: простой API, middleware, быстрая разработка.
- Каркас: `const app = express(); app.use(express.json()); app.get(...); app.listen(PORT, cb)`. `PORT = process.env.PORT || 3000`. Порт `0` означает случайный свободный порт (используют в тестах). Варианты запуска: через `http.createServer(app)` или `https.createServer({key, cert}, app)`. Health-check `/health`, конфигурация для разных сред (`NODE_ENV`). Ошибки запуска ловят через событие `'error'` (например, `EADDRINUSE`).
- **Graceful shutdown**: на `SIGTERM`/`SIGINT` вызвать `server.close(() => { закрыть БД; process.exit(0) })`.
- **Маршрут** — это метод, путь и обработчик. Методы: GET (чтение), POST (создание), PUT (полная замена), PATCH (частичное изменение), DELETE, `app.all`. Для одного пути с разными методами — `app.route('/x').get().post()`. Модульные роутеры: `express.Router()` подключается через `app.use('/api/users', router)`. Порядок важен: конкретные маршруты ставят раньше параметризованных, а **404-обработчик — последним**. Асинхронные обработчики оборачивают в try/catch или в функцию-обёртку, которая передаёт ошибку в `next(err)`.
- **Параметры**:
  - route params: `/users/:id` читается как `req.params.id` (строка, её надо приводить к числу и проверять). Бывают несколько параметров (`/users/:userId/posts/:postId`), опциональные (`:id?`), с regex-ограничением (`/:id(\\d+)`), wildcard.
  - query: `/users?sort=asc&page=2` читается из `req.query`. Массивы: `?tag=a&tag=b` или `tag[]=`. На их основе строят динамические фильтры, пагинацию и сортировку.
  - body: `express.json()` (JSON) и `express.urlencoded({extended:true})` (формы) заполняют `req.body`.
  - Валидация и санитизация параметров — отдельным middleware, 400 при ошибке.
- **Статика**: `app.use(express.static('public'))`, виртуальный префикс `app.use('/static', express.static(path.join(__dirname,'public')))`, можно подключить несколько папок. Опции: `maxAge`, `etag`, `index`, `dotfiles`. Защита от directory traversal, кэширование, gzip (`compression`), 404 для отсутствующих файлов. Статику подключают **до** маршрутов.

### 4. Middleware и шаблонизатор EJS (лекция 10; основа ПЗ 1)

- Middleware — функция `(req, res, next)`. Она может изменить `req` и `res`, завершить ответ или вызвать `next()`. **Если не вызвать `next()` и не ответить, запрос повиснет.** `next(err)` передаёт ошибку в обработчики ошибок и пропускает обычные middleware.
- Типы: уровня приложения (`app.use`), роутера (`router.use`), для одного маршрута (`app.get('/x', mw1, mw2, handler)`), встроенные (`express.json`, `express.urlencoded`, `express.static`), сторонние (`morgan`, `cors`, `helmet`, `cookie-parser`, `compression`), обработчики ошибок.
- **Error-handling middleware** принимает 4 аргумента `(err, req, res, next)` и подключается после всех маршрутов. 404 — обычный middleware в самом конце: `app.use((req,res)=>res.status(404)...)`.
- Порядок: логирование, парсеры, статика, auth, маршруты, 404, обработчик ошибок.
- Примеры собственных middleware: логгер (метод, URL, время; длительность через `res.on('finish')`), auth (проверка заголовка или сессии), валидация, общий заголовок ответа, блокировка IP.
- **EJS**: `npm i ejs`, `app.set('view engine','ejs')`, `app.set('views','./views')`, рендер `res.render('index', {title, items})`.
  - `<%= x %>` — вывод с экранированием HTML (защита от XSS).
  - `<%- x %>` — вывод **без** экранирования (опасно, только для доверенного HTML, `include` и `body`).
  - `<% код %>` — управляющий код: `if`, `forEach`.
  - Partials: `<%- include('partials/header') %>`. Layout-каркас с `<%- body %>` (пакет `express-ejs-layouts`) или header/footer через partials.
  - Хелперы (форматирование даты и т.п.) передают через `res.locals` или `app.locals`.
  - Страница ошибки рендерится из error-handler: `res.status(code).render('error', {...})`.
- `res.render` отдаёт HTML из шаблона, `res.send` — строку или буфер, `res.json` — JSON.

### 5. Базы данных и ORM Sequelize (лекция 11; основа ЛР 2)

- **ORM** отображает таблицы на классы и объекты. Плюсы: абстракция над SQL, несколько СУБД (PostgreSQL, MySQL, SQLite, MSSQL), валидация, связи, миграции, транзакции, защита от SQL-инъекций за счёт параметризации. Минусы: накладные расходы, сложные запросы, проблема N+1.
- Установка: `npm i sequelize pg pg-hstore` (для MySQL — `mysql2`), `npm i -D sequelize-cli`.
- Подключение: `new Sequelize(process.env.DATABASE_URL, {dialect:'postgres', logging:false, pool:{max,min,acquire,idle}, define:{timestamps:true, underscored:true}})`, проверка через `await sequelize.authenticate()`.
- **Модель**: `sequelize.define('User', {...}, {tableName, timestamps, indexes, hooks})` или `class User extends Model {}` + `User.init(...)`. Атрибуты: `type`, `allowNull`, `defaultValue`, `primaryKey`, `autoIncrement`, `unique`, `validate` (`notEmpty`, `len`, `isEmail`, `min`, `max`, собственные сообщения `msg`).
- Имена: модель `User` по умолчанию превращается в таблицу `Users` (множественное число). Это меняется через `tableName` или `freezeTableName`. `timestamps` добавляет `createdAt` и `updatedAt`, `underscored` включает snake_case, `paranoid` даёт мягкое удаление (`deletedAt`).
- **Типы**: `STRING(n)`, `TEXT`, `CHAR`, `INTEGER`, `BIGINT`, `FLOAT`, `DECIMAL(p,s)`, `DATE`, `DATEONLY`, `TIME`, `BOOLEAN`, `JSON`, `JSONB` (PostgreSQL), `ENUM(...)`, `UUID` + `UUIDV4`, `ARRAY` (PostgreSQL), `VIRTUAL` (вычисляемое поле, в БД не хранится).
- **Хуки**: `beforeValidate`, `beforeCreate`, `beforeUpdate` (например, генерация slug или хеширование пароля), `instance.changed('field')`.
- **Ассоциации**:
  - 1:N — `User.hasMany(Post,{foreignKey:'authorId', as:'posts', onDelete:'CASCADE'})` + `Post.belongsTo(User,{foreignKey:'authorId', as:'author'})`.
  - 1:1 — `hasOne` / `belongsTo`.
  - N:M — `Post.belongsToMany(Category,{through: PostCategory, foreignKey, otherKey, as})`.
- **CRUD**: `create`, `bulkCreate`, `findAll({where, attributes, order, limit, offset, include})`, `findByPk(id)`, `findOne({where})`, `findOrCreate`, `findAndCountAll` (для пагинации), `count`, `update(values,{where})` (возвращает `[affectedCount]`), `instance.update()` / `save()`, `destroy({where})`, `increment`.
- **Операторы** `Op`: `eq`, `ne`, `gt`, `gte`, `lt`, `lte`, `in`, `notIn`, `like`, `iLike`, `between`, `or`, `and`. Пример: `where:{ [Op.or]: [{a:1},{b:2}], price:{[Op.between]:[10,50]} }`.
- **Eager loading**: `include: [{model: User, as: 'author', attributes: [...]}]`, условия по ассоциации, `required` (INNER или LEFT JOIN).
- **Миграции** (sequelize-cli):
  - `npx sequelize-cli init` создаёт `config/`, `models/`, `migrations/`, `seeders/`.
  - `model:generate --name X --attributes a:string,b:integer` создаёт модель и миграцию.
  - `migration:generate --name ...`, методы `up`/`down` через `queryInterface.addColumn / removeColumn / createTable / dropTable`.
  - `db:migrate`, `db:migrate:undo`.
  - `seed:generate --name ...`, `bulkInsert`, `db:seed:all`.
  - Конфигурация `config/config.json` по средам, для облака `"use_env_variable": "DATABASE_URL"`.
- **Структура Express-проекта**: `config/`, `models/index.js` (инициализация и связи), `controllers/`, `routes/`, `middleware/`.
- **Транзакции**: управляемые `sequelize.transaction(async t => {... {transaction:t}})` или ручные `t.commit()` / `t.rollback()`. Ошибки: `SequelizeValidationError` отдают как 400, `SequelizeUniqueConstraintError` — как 409 или 400.
- Кастомные методы: метод экземпляра `Model.prototype.fn = function(){}` или метод класса `static fn()`. `findByPk` ищет по первичному ключу, `findOne` — по любому `where`.

### 6. REST: принципы, проектирование, CRUD, коды, документация (лекции 12–14, 16–18)

- **REST** — архитектурный стиль, который предложил Рой Филдинг (2000). Ограничения:
  1. клиент-сервер;
  2. **stateless** (каждый запрос самодостаточен, аутентификация через токен в каждом запросе);
  3. кэшируемость (`Cache-Control`, `ETag`, `Last-Modified`, 304);
  4. **единообразный интерфейс**: ресурсы идентифицируются URI, манипуляции идут через представления, сообщения самоописательны, HATEOAS;
  5. многоуровневая система (прокси, шлюзы, балансировщики);
  6. код по требованию (необязательно).
- Ресурсо-ориентированность: в URI существительные во множественном числе (`GET /users`, `POST /users`, `PUT /users/123`, `DELETE /users/123`), а не глаголы (`/getUsers`, `/createUser`). Нижний регистр, дефисы, без завершающего слеша. Вложенность не глубже 2 уровней (`/posts/1/comments`). Фильтры, сортировка, поиск и пагинация передаются через query (`?status=active&sort=-createdAt&page=2&limit=20&q=...`).
- Нестандартные действия: подресурс-действие (`POST /users/1/block`, `POST /orders/1/cancel`) или отдельный ресурс (`POST /auth/login`, `POST /payments`).
- **Идемпотентность**: повторный запрос даёт тот же эффект. GET, HEAD, OPTIONS, PUT, DELETE идемпотентны. POST — нет, PATCH в общем случае тоже нет. **Безопасные** (не меняют данные): GET, HEAD, OPTIONS. Чтобы POST можно было безопасно повторять, используют Idempotency-Key.
- **Пагинация**: offset (`page/limit`, `limit/offset`; в ответе total, pages, текущая позиция) или cursor (стабильна на больших и меняющихся данных).
- **Версионирование**: в URI (`/api/v1/...`, самое распространённое), в заголовке (`Accept: application/vnd.company.v1+json` или свой заголовок), в query (`?version=1`). Добавлять поля можно без смены версии, удалять и переименовывать — только с новой версией и deprecation-периодом.
- **CRUD ↔ HTTP**: Create = POST (201 + заголовок `Location` + тело), Read = GET (200), Update = PUT (полная замена, идемпотентен) / PATCH (частичное изменение, экономит трафик), Delete = DELETE (204 или 200; мягкое удаление или жёсткое). Оптимистичная блокировка через `ETag` + `If-Match` (412 при конфликте). Неизменяемые поля (id, createdAt) при обновлении не принимать.
- **Коды состояния**:
  - 2xx: 200 OK, 201 Created, 202 Accepted (фоновые или долгие задачи), 204 No Content.
  - 3xx: 301, 302, 303, 304 Not Modified (кэш), 307, 308.
  - 4xx: 400 Bad Request (невалидный JSON, нет обязательных полей), 401 Unauthorized («кто ты?», нет или неверный токен, заголовок `WWW-Authenticate`), 403 Forbidden («знаю кто ты, но нельзя»), 404 Not Found (иногда вместо 403, чтобы скрыть существование ресурса), 405 Method Not Allowed, 406 Not Acceptable, 409 Conflict (дубликат, конфликт версии), 410 Gone, 412, 415, 422 Unprocessable Entity (семантическая ошибка валидации), 429 Too Many Requests (+ `Retry-After`).
  - 5xx: 500, 501 Not Implemented (неизвестный метод), 502, 503 (+ `Retry-After`), 504.
- **Единый формат ошибки**, например: `{ "error": { "code": "VALIDATION_ERROR", "message": "...", "details": [{ "field": "email", "message": "..." }] } }`. Стектрейсы в продакшене не отдают.
- **Пример структуры (лекция 13)**: `app.use('/api/users', require('./routes/users'))`, контроллер с `exports.getAllUsers = async (req,res)=>{try{...res.json()}catch{res.status(500)}}`, `router.get('/:id', ctrl.getUserById)` и т.д. Модель в лекции написана на Mongoose; в ЛР вместо неё используется Sequelize.
- **Документация API**. Читатели: интеграторы (быстрый старт, примеры), техлиды (лимиты, стоимость), архитекторы и DevOps. Состав: Quick Start («Hello World» за 5–10 минут: требования, получение ключа, первый запрос, следующие шаги), аутентификация, руководства по сценариям, справочник эндпоинтов (параметры, заголовки, тела, примеры ответов и ошибок), коды ошибок, лимиты, changelog, troubleshooting. **OpenAPI/Swagger**: `swagger-jsdoc` (спецификация из JSDoc-комментариев) + `swagger-ui-express` (UI на `/api-docs`). Альтернативы: Redoc, Stoplight, коллекции Postman.

### 7. GraphQL (лекция 15)

- Язык запросов с **одним эндпоинтом** (обычно `POST /graphql`): клиент сам выбирает поля, нет over- и under-fetching, несколько ресурсов в одном запросе. Строгая типизированная схема (SDL), интроспекция и интерактивная документация (GraphiQL, Apollo Sandbox).
- Операции: **query** (чтение), **mutation** (аналог POST/PUT/DELETE), **subscription** (push по WebSocket).
- Создание эндпоинта: схема `typeDefs` (`type User { id: ID!, name: String }`, `type Query { users: [User] }`), **резолверы** (функции, возвращающие данные), сервер (`new ApolloServer({typeDefs, resolvers})`), обработка запросов.
- Минусы: сложность на сервере, риск тяжёлых вложенных запросов (нужны ограничение глубины и сложности), сложнее HTTP-кэширование, контроль доступа на уровне полей. Инструменты: Apollo Server/Client, GraphiQL, Relay.

### 8. Аутентификация: сессии, JWT, регистрация, bcrypt, OAuth/OIDC (лекции 19–21, 23–25)

- **Аутентификация** отвечает на вопрос «кто ты?», **авторизация** — «что тебе можно?».
- **Сессии (stateful)**: сервер хранит состояние (память, Redis через `connect-redis`, БД), клиент хранит `sessionId` в cookie (`express-session`). Плюсы: мгновенный logout и отзыв, маленькая cookie. Минусы: нужно общее хранилище при масштабировании, CSRF.
- **JWT (stateless)**: `header.payload.signature` в Base64URL. Header: `alg` (HS256/RS256) и `typ`. Payload содержит claims: registered (`iss`, `sub`, `aud`, `exp`, `iat`, `nbf`, `jti`), public и private (`id`, `email`, `role`). Payload **не шифруется**, секреты в него не кладут. Подпись проверяется секретом или открытым ключом. Плюсы: не нужен запрос к хранилищу, удобно для микросервисов и мобильных клиентов. Минусы: нельзя отозвать до `exp`, токен больше по размеру.
  - Создание: `jwt.sign({id, email}, process.env.JWT_SECRET, {expiresIn: '1h'})`. Проверка: `jwt.verify(token, secret)` (бросает `TokenExpiredError` / `JsonWebTokenError`).
  - Передача: заголовок `Authorization: Bearer <token>`.
  - Best practice: короткий access-токен (15 мин) + refresh-токен (7 дней) в **httpOnly + Secure + SameSite** cookie, ротация refresh-токенов.
  - Отзыв до истечения срока: blacklist по `jti` (Redis), `tokenVersion` у пользователя, короткий TTL + refresh, смена секрета (разлогинит всех).
- **Уязвимости**:
  - сессии: hijacking (кража ID через XSS; защита — HttpOnly, Secure), fixation (регенерировать ID после логина), CSRF (токены, SameSite);
  - JWT: подделка (`alg:none`, слабый секрет; явно задавать алгоритмы), кража из localStorage через XSS, невозможность отзыва.
- **Регистрация**: минимум трения (email + пароль, подтверждение пароля; имя и телефон по желанию). Многоуровневая валидация (формат email, сложность пароля), проверка уникальности email (409), хеширование с солью. Верификация email по токену со сроком действия, onboarding.
- **Вход**: найти пользователя, сравнить хеш, выдать токен. Ошибка всегда общая «Неверные учётные данные», чтобы не раскрывать, существует ли email. Защита: rate limiting (`express-rate-limit`), временная блокировка аккаунта, мониторинг аномалий, логирование.
- **Хеширование паролей**: не хранить пароль открытым текстом и не шифровать обратимо. MD5 и SHA-1 устарели. SHA-256/512 слишком быстрые, их легко перебирать на GPU, а без соли ломаются rainbow-таблицами. **bcrypt** медленный намеренно, соль у него встроена, work factor настраивается.
  - Формат хеша: `$2b$<cost>$<22 символа соли><31 символ хеша>`.
  - API: `await bcrypt.hash(pw, saltRounds)` или `genSalt(n)` + `hash`, проверка `await bcrypt.compare(pw, hash)`.
  - Скорость по cost: 10 ≈ 100 мс, 12 ≈ 400 мс, 14 ≈ 1,6 с. Рекомендуется 10–12, а по мере роста мощности железа — повышать (при входе проверить cost и перехешировать).
  - **Pepper** — секрет из env, который добавляется к паролю. Альтернативы: argon2, scrypt. `bcryptjs` — версия на чистом JS.
- **OAuth 2.0** — фреймворк **делегированной авторизации**, а не аутентификации.
  - Роли: Resource Owner, Client, Authorization Server, Resource Server.
  - Потоки: Authorization Code (+ **PKCE** для SPA и мобильных), Client Credentials (сервис-сервис), устаревшие Implicit и Password.
  - Токены: access (короткий), refresh.
  - Типичный поток «Войти через GitHub/Google»: редирект на провайдера, `code` в callback, обмен на токен на сервере, запрос к API. `client_secret` нельзя хранить во фронтенде.
- **OIDC** — слой поверх OAuth 2.0 для аутентификации: `id_token` (JWT с данными пользователя), `/userinfo`, scope `openid`. На практике: вход через OIDC-провайдера, после чего бэкенд выпускает свой JWT.

### 9. Контроль доступа: middleware прав (лекция 22)

- Модели: **RBAC** (роли и набор разрешений; просто и прозрачно, но при росте системы ролей становится слишком много), **ABAC** (решение по атрибутам субъекта, ресурса и контекста; гибко, но сложно), ACL, ownership.
- Принципы: одна ответственность у middleware, fail-safe (запрет по умолчанию), композиция, понятные ошибки (401 или 403), логирование.
- Идиомы:
  - `authenticate` проверяет JWT и ставит `req.user`;
  - `authorize(...roles)` возвращает 403, если `req.user.role` не входит в список;
  - `requirePermission('project:write')`;
  - **проверка владельца** (`resource.ownerId === req.user.id || admin`, защищает от IDOR);
  - доступ по времени;
  - rate limit в зависимости от роли;
  - композиция цепочек `[authenticate, authorize('admin'), checkOwnership]`.
- Разрешения кэшируют (Redis), чтобы не ходить в БД на каждый запрос. Рекомендуемая структура: `middleware/auth.js`, `middleware/rbac.js`, `config/permissions.js`.

### 10. Валидация входящих данных (лекция 26)

- Уровни:
  1. клиентская (только UX, серверную не заменяет);
  2. синтаксическая (типы, формат, структура);
  3. семантическая (бизнес-правила, связи полей, например `endDate > startDate`);
  4. системная (права, квоты, лимиты).
- Библиотеки: **Joi** (`Joi.object({ email: Joi.string().email().required(), password: Joi.string().min(8).pattern(...), confirmPassword: Joi.valid(Joi.ref('password')) })`, `schema.validate(body, {abortEarly:false, stripUnknown:true})`), **express-validator** (`body('email').isEmail().normalizeEmail()`, `validationResult(req)`), а также Yup и Zod.
- Паттерн: `validate(schema)` как middleware возвращает 400 или 422 со списком ошибок по полям. Whitelist вместо blacklist, лимиты размера тела, отбрасывание неизвестных полей, санитизация. Схемы лежат в `validators/`.

### 11. Веб-безопасность: инъекции, XSS, CSRF, санитизация, CORS (лекции 27–29, 33)

- **SQL-инъекция**: склейка строк (`"... WHERE id = " + id`) позволяет выполнить `' OR 1=1 --`, `UNION` и т.п. Бывают classic, blind, time-based. Защита: **параметризованные запросы / ORM** (Sequelize экранирует параметры; в `sequelize.query` использовать `replacements` или `bind`), валидация, минимальные привилегии пользователя БД, скрытие ошибок СУБД. **NoSQL-инъекция** (`{"$gt": ""}`) лечится приведением типов и санитизацией.
- **XSS**:
  - виды: reflected, stored, DOM-based;
  - защита: экранирование вывода (`<%= %>` в EJS; React экранирует по умолчанию, опасен `dangerouslySetInnerHTML`), санитизация HTML (**DOMPurify**, `sanitize-html`), **CSP** (`helmet.contentSecurityPolicy`), HttpOnly cookies, проверка входных данных.
- **CSRF**: чужой сайт отправляет запрос с cookie жертвы. Защита: анти-CSRF токен (synchronizer token или double submit cookie; заголовок `X-CSRF-Token`), `SameSite=Lax/Strict`, проверка `Origin`/`Referer`. Токен в заголовке `Authorization` сам по себе не подвержен CSRF.
- **Стек защитных middleware**: `helmet()` (заголовки безопасности: CSP, HSTS, X-Frame-Options, noSniff), `cors({origin: whitelist, credentials: true})`, `express-rate-limit`, `hpp` (дублирование параметров), лимит тела `express.json({limit:'10kb'})`, `xss-clean` (устарел), логирование security-событий.
- **Санитизация контекстно-зависимая**: HTML, атрибуты, URL, JS, SQL и имена файлов обрабатываются по-разному.
- **CORS**: браузерная Same-Origin Policy (одинаковые схема, хост и порт). CORS разрешает кросс-доменные запросы заголовками `Access-Control-Allow-Origin/Methods/Headers/Credentials`, `Max-Age`. Простые запросы (GET/HEAD/POST со стандартными заголовками) идут сразу, остальные сначала отправляют **preflight OPTIONS**. `*` нельзя сочетать с credentials. В Express: `app.use(cors({ origin: ['https://app.example.com'], methods: [...], credentials: true }))` или функция-проверка origin.
- **Клиент (лекция 33)**: секретные ключи не должны попадать в бандл (`.env` на клиенте — не секрет, переменные `VITE_*` публичны). Запросы к сторонним API с секретом идут через свой сервер или прокси. CORS настраивается на стороне сервера.

### 12. Сетевая и инфраструктурная безопасность (лекции 28, 30–32)

- **DDoS**: объёмные атаки, атаки на протоколы, атаки уровня L7. Защита: CDN и anycast (Cloudflare), балансировка, rate limiting, WAF, автомасштабирование. **Фишинг**: обучение пользователей, SPF/DKIM/DMARC, MFA, фильтры. **SQL-инъекции** — см. выше.
- **TLS/SSL**: handshake, сертификаты от CA, асимметричная криптография для обмена ключом и симметричная для данных. HTTPS, HSTS. **VPN**: удалённый доступ и site-to-site, протоколы OpenVPN, WireGuard, IPsec, L2TP. **IPsec**: шифрование на сетевом уровне, режимы transport и tunnel, протоколы AH, ESP, IKE.
- **Брандмауэры**: пакетные фильтры, stateful, прокси и прикладной уровень (WAF), программные и аппаратные. **IDS** обнаруживает, **IPS** блокирует; бывают сетевые (NIDS) и хостовые (HIDS), по сигнатурам и по аномалиям, гибридные.
- **Облако**: шифрование at rest и in transit, KMS, управление доступом (RBAC, принцип наименьших привилегий, **MFA**), изоляция виртуальных машин и контейнеров, стандарты GDPR, PCI DSS, ISO 27001, HIPAA, облачные средства мониторинга и защиты.

### 13. DevSecOps (лекция 34)

- Безопасность встраивается в каждый этап (**shift left**): планирование, разработка, сборка, тестирование, релиз, развёртывание, мониторинг.
- **SAST** анализирует код без запуска: рано, быстро, указывает на строку, но даёт ложные срабатывания и не видит runtime. Инструменты: SonarQube, Semgrep, Snyk Code, Checkmarx.
- **DAST** атакует запущенное приложение: находит проблемы runtime и конфигурации, но поздно и без указания строки. Инструменты: OWASP ZAP, Burp Suite.
- **SCA** проверяет зависимости на CVE и лицензии: `npm audit`, Snyk, Dependabot.
- Дополнительно: поиск секретов в репозитории, сканирование контейнеров, IaC. Всё это встраивается в CI с порогами, которые ломают сборку.

---

## Лабораторные работы

> Общее для всех ЛР модуля: работа **индивидуальная**; **тема = тема курсового проекта**, вариант темы определяется «Генератором вариантов»; **отчёт в PDF загружается в СЭО** + **ссылка на GitHub-репозиторий с кодом в указанной ветке**. **Сроки сдачи в материалах модуля не указаны** (смотреть СЭО и методические указания).
>
> **Названия веток в методичках записаны именно так (не исправлять без согласования):** ЛР 1 — `lab21`, ЛР 2 — `lab12`, ЛР 3 — `23`, ПЗ 1 — `pz21`, ПЗ 2 — `pz22`. Похоже, что для ЛР 2 и ЛР 3 подразумевались `lab22` и `lab23` (ч.2, работы 1–3). В ЛР 2 явно написано «ветка lab12», в ЛР 3 — «Ветка репозитория: 23» и «ссылка на репозиторий с веткой 23». Безопаснее всего создать ветку ровно с тем именем, что в методичке, или уточнить у преподавателя.

### ЛР 1. «Разработка серверного приложения на Node.js»

- **Ветка:** `lab21`. **Срок:** срок не указан в материалах.
- **Цель:** практика создания серверного приложения на Node.js + **Express.js**: маршрутизация, обработка GET/POST/PUT/DELETE, формирование ответов для будущей интеграции с клиентом курсового проекта (начатого в ч.1).
- **Необходимые знания:** Node.js (установка, модули, event loop), Express (приложение, маршруты, middleware), HTTP-методы (GET, POST, PUT, DELETE, PATCH), параметры (route params, query string, body), форматы JSON и URL-encoded.
- **Задачи:** настроить проект Node.js + Express; сделать REST API для коллекции данных предметной области курсового; CRUD-маршруты; обработка ошибок с корректными статусами; тестирование браузером, Postman или curl.
- **Стек:** Node.js, `express`, dev-зависимость `nodemon`; тестирование в Postman, Insomnia или REST Client для VS Code.

**Порядок выполнения**

1. **Инициализация проекта**
   1. Создать папку проекта, выполнить `npm init -y`.
   2. `npm install express`.
   3. `npm install nodemon --save-dev`.
   4. В `package.json` добавить скрипты: `"start": "node server.js"`, `"dev": "nodemon server.js"`.
2. **Создание сервера**
   1. Создать `server.js`.
   2. Подключить Express: `const express = require('express'); const app = express();`.
   3. Добавить `app.use(express.json())`.
   4. Задать порт (3000 или другой) и запустить: `app.listen(port, () => console.log('Server running...'))`.
3. **Маршруты** (имя ресурса `items` **обязательно заменить** на термин своей предметной области: products, tasks, courses, orders…):

   | Метод | URL | Назначение |
   |---|---|---|
   | GET | `/items` | список всех элементов |
   | GET | `/items/:id` | один элемент по ID |
   | POST | `/items` | добавить элемент |
   | PUT | `/items/:id` | полное обновление |
   | DELETE | `/items/:id` | удаление |

   Данные **временно хранятся в массиве в памяти** сервера.
4. **Обработка ошибок**
   - Элемент не найден: **404** + JSON с ошибкой.
   - Неверные данные запроса: **400**.
   - Добавить **глобальный обработчик ошибок** (error-handling middleware).
5. **Тестирование**
   - `npm run dev`.
   - Проверить все эндпоинты в Postman, Insomnia или REST Client (VS Code).
   - Зафиксировать результаты (скриншоты).

- **Рекомендуемая структура** (с расчётом на следующие ЛР): `routes/`, `controllers/`, `models/` (даже если «модель» пока просто массив).

**Отчёт (PDF в СЭО)**
1. Титульный лист, цель работы.
2. Краткое описание проекта (предметная область курсового, структура маршрутов).
3. Листинг `server.js` (основные части).
4. Скриншоты запросов и ответов **для каждого эндпоинта** (Postman или браузер).
5. Таблица: маршруты, методы, ожидаемые статусы, примеры запросов и ответов.
6. Ответы на контрольные вопросы.
7. Выводы.

**Сдача:** PDF в СЭО + ссылка на GitHub (ветка `lab21`); индивидуально; тема по Генератору вариантов.

**Варианты:** общий порядок для всех; отличается только предметная область (тема курсового из Генератора вариантов), от неё зависит имя ресурса и его поля.

**Контрольные вопросы (16)**
1. Какие HTTP-методы реализованы и для каких операций?
2. Как получить route params и query string в Express?
3. Зачем нужен `express.json()`?
4. Статус при успешном создании? При успешном удалении?
5. Что такое REST API, основные принципы REST?
6. Как организовать глобальную обработку ошибок в Express?
7. Какую структуру ответа выбрали для ошибок (например, `{ error: "message" }`)?
8. Как протестировать DELETE уже удалённого ресурса, какой статус вернуть?
9. Преимущества nodemon?
10. PUT и PATCH: в чём разница, что реализовали и почему?
11. Как добавить CORS (теоретически)?
12. Почему данные в массиве, а не в БД? Когда переходить на БД?
13. Какой `Content-Type` для отправки JSON?
14. Пример curl для своего POST-эндпоинта.
15. Как убедиться, что сервер работает корректно после каждой модификации кода?
16. Какие шаги нужны, чтобы запустить приложение на другом компьютере?

**Примечания методички:** это первая серверная ЛР ч.2, основа для БД, аутентификации и интеграции с React-фронтендом из ч.1. В следующих работах добавятся БД, валидация, JWT и интеграция с фронтендом.

### ЛР 2. «Работа с базами данных и ORM Sequelize»

- **Ветка:** `lab12` (так в методичке). **Срок:** срок не указан в материалах.
- **Цель:** подключить реляционную БД **PostgreSQL** к Node.js-приложению; использовать **ORM Sequelize** для моделей и CRUD; изучить миграции и связи между таблицами. Продолжает серверную часть из ЛР 1.
- **Необходимые знания:** SQL и реляционные БД, установка и настройка PostgreSQL, понятие ORM, базовые операции Sequelize (`define`, `sync`, `create`, `findAll`, `findByPk`, `update`, `destroy`), миграции и сиды.
- **Задачи:**
  - установить PostgreSQL (локально или в облаке: Neon, Supabase);
  - подключить Sequelize;
  - создать модель предметной области курсового (рекомендуется продолжить сущность из ЛР 1);
  - реализовать CRUD через Sequelize вместо массива;
  - создать и выполнить миграцию, изменяющую схему (**добавление нового поля**);
  - наполнить БД тестовыми данными (**seed**).
- **Стек (обязательно):** PostgreSQL, `sequelize`, `pg`, `pg-hstore`, dev-зависимость `sequelize-cli`; Express из ЛР 1.

**Порядок выполнения**

1. **Подготовка БД**
   1. Установить PostgreSQL или создать бесплатный кластер в Neon или Supabase.
   2. Создать БД и пользователя, записать строку подключения (URL).
2. **Зависимости**
   ```bash
   npm install sequelize pg pg-hstore
   npm install --save-dev sequelize-cli
   ```
3. **Настройка Sequelize**
   1. `npx sequelize-cli init` (создаст `config/`, `models/`, `migrations/`, `seeders/`).
   2. Настроить `config/config.json` или переменные окружения (`.env`). Пример для облака:
      ```json
      { "development": { "use_env_variable": "DATABASE_URL", "dialect": "postgres" } }
      ```
   3. Создать модель и миграцию, например:
      `npx sequelize-cli model:generate --name Item --attributes name:string,description:string`
4. **CRUD в Express.** Маршруты ЛР 1 **не удалять**, а заменить работу с массивом на вызовы Sequelize:

   | Маршрут | Вызов |
   |---|---|
   | `GET /items` | `await Item.findAll()` |
   | `GET /items/:id` | `await Item.findByPk(id)` |
   | `POST /items` | `await Item.create(req.body)` |
   | `PUT /items/:id` | `await Item.update(req.body, { where: { id } })` |
   | `DELETE /items/:id` | `await Item.destroy({ where: { id } })` |
5. **Миграция и сиды**
   1. Миграция для нового поля (например, `priority` для задач, `stockQuantity` для товаров): `npx sequelize-cli migration:generate --name add-field-to-items`; в файле добавить колонку через `queryInterface.addColumn` (в `down` — `removeColumn`).
   2. `npx sequelize-cli db:migrate`.
   3. `npx sequelize-cli seed:generate --name demo-items`.
   4. Заполнить seed-файл массивом объектов (`bulkInsert`) и выполнить `npx sequelize-cli db:seed:all`.

**Отчёт (PDF в СЭО)**
1. Титульный лист, цель.
2. Краткие теоретические сведения об ORM (1–2 абзаца).
3. Скриншоты: структура БД (pgAdmin, DBeaver или дашборд облачного сервиса), конфигурация Sequelize.
4. Листинг модели и контроллеров CRUD (ключевые фрагменты).
5. Скриншоты запросов в Postman с ответами из БД.
6. Демонстрация миграции (добавленное поле в таблице).
7. Ответы на контрольные вопросы.
8. Выводы.

**Сдача:** PDF в СЭО + GitHub (ветка `lab12`); индивидуально; тема по Генератору вариантов. **Варианты:** общий порядок, предметная область из темы курсового.

**Контрольные вопросы (10)**
1. Что такое ORM, плюсы и минусы?
2. Как Sequelize преобразует имена моделей в имена таблиц?
3. Поиск с условием WHERE (пример)?
4. Что такое миграции и зачем они?
5. Связь один-ко-многим — пример из своего проекта.
6. Пагинация (`limit`, `offset`)?
7. Обработка ошибки уникальности (duplicate entry)?
8. `findByPk` и `findOne` — в чём разница?
9. Как добавить кастомный метод в модель?
10. Какие типы связей поддерживает Sequelize (`belongsTo`, `hasMany`, `belongsToMany`)?

> Вопрос 5 требует **связи 1:N из своего проекта**, поэтому одной модели мало: нужна как минимум пара связанных моделей (например, User 1:N Project или Project 1:N Share).

**Примечания:** ЛР 2 заменяет массив из ЛР 1 на БД; следующим этапом (ЛР 3) будет аутентификация.

### ЛР 3. «Реализация системы аутентификации и авторизации»

- **Ветка:** `23` (так в методичке, дважды). **Срок:** срок не указан в материалах.
- **Цель:** механизмы аутентификации и авторизации: регистрация и вход с **JWT**, хеширование паролей **bcrypt**, защита маршрутов через **middleware**.
- **Необходимые знания:** аутентификация и авторизация; JWT (структура, подпись, срок действия); bcrypt и соль; middleware Express для проверки токена; хранение токена на клиенте (localStorage или cookies).
- **Задачи:**
  1. модель пользователя (email, хеш пароля), рекомендуется в том же проекте, той же БД PostgreSQL и Sequelize из ЛР 2;
  2. маршрут регистрации (хеширование, сохранение в БД);
  3. маршрут входа (проверка пароля, выдача JWT);
  4. middleware проверки JWT;
  5. защита части маршрутов;
  6. маршрут «текущий пользователь»;
  7. **один дополнительный механизм безопасности** по теме курсового (раздел 7 методички).
- **Стек (обязательно):** `jsonwebtoken`, `bcrypt` (+ `dotenv` при использовании переменных окружения); Express, Sequelize, PostgreSQL из ЛР 1–2.

**Порядок выполнения**

1. **Зависимости:** `npm install jsonwebtoken bcrypt` (и `dotenv`).
2. **Модель User (Sequelize):**
   `npx sequelize-cli model:generate --name User --attributes email:string,passwordHash:string`
   - Обязательные поля: `id` (PK), `email` (string, **unique**, `allowNull: false`), `passwordHash` (string, `allowNull: false`).
   - Дополнительные поля — под выбранный механизм (например, `role`, `refreshToken`, `failedLoginAttempts`, `lockUntil`).
   - `npx sequelize-cli db:migrate`.
3. **`POST /auth/register`** (файл `routes/auth.js`): принимает `email` и `password`; проверяет уникальность email; `bcrypt.hash(password, 10)`; сохраняет пользователя; возвращает **201**.
4. **`POST /auth/login`**: находит пользователя по email; сравнивает пароль с хешем; генерирует `jwt.sign({ id, email }, process.env.JWT_SECRET, { expiresIn: '1h' })`; возвращает токен.
5. **Middleware** `middleware/auth.js`: извлекает токен из `Authorization: Bearer <token>`, верифицирует, выполняет `req.user = decoded` и вызывает `next()` (без токена или с неверным токеном — 401).
6. **Защищённые маршруты:** защитить **`GET /profile`** (данные текущего пользователя); по желанию добавить другие (например, удаление пользователя).
7. **Дополнительный механизм** — выбрать один из списка или согласовать с преподавателем свой:

   | Механизм | Что сделать |
   |---|---|
   | Ролевая модель (RBAC) | поле `role` (user, admin), middleware `isAdmin`, маршруты только для админа |
   | Refresh-токены | поле `refreshToken`, `POST /auth/refresh` выдаёт новую пару токенов |
   | Блокировка после неудачных попыток | поля `failedLoginAttempts`, `lockUntil`; после **5** неудач блок на **5 минут** |
   | Смена пароля | `POST /auth/change-password`: проверка старого пароля, хеширование нового |
   | Логирование входов | модель `LoginLog` (`userId`, `ip`, `userAgent`, `timestamp`, `success`) + эндпоинт просмотра логов для админа |
   | 2FA по email | одноразовый код, «отправка» на email (вывод в консоль), проверка на отдельном эндпоинте |
   | Проверка сложности пароля | при регистрации и смене пароля: длина, заглавные буквы, цифры, спецсимволы |
   | Подтверждение email | токен, ссылка на email (имитация), поле `isVerified`, вход запрещён без подтверждения |
   | Ограничение активных сессий | модель `Session` с `tokenVersion`, не более **3** одновременных сессий |
   | Восстановление пароля через email | эндпоинты `forgot-password` (генерация токена) и `reset-password` (смена по токену) |
   | «Запомнить меня» | параметр `rememberMe` при логине увеличивает время жизни токена |
   | Блокировка пользователей админом | поля `isBanned`, `banReason`, middleware `isNotBanned`, эндпоинты `/ban`, `/unban` |
   | Имитация OAuth (GitHub) | заглушка входа через GitHub, возвращающая JWT |
   | Защита от CSRF | генерация CSRF-токена, проверка заголовка `X-CSRF-Token` (кроме GET и auth-эндпоинтов) |
   | Смена email | `POST /auth/change-email`: проверка пароля и повторное подтверждение по email |
   | SMS-подтверждение | 6-значный код, вывод в консоль, проверка на отдельном эндпоинте |

   Выбор подсказывает предметная область (магазину важны роли, системе задач — восстановление пароля); при необходимости согласовать с преподавателем.

**Отчёт (PDF в СЭО)**
1. Цель, краткая теория о JWT.
2. Листинг модели User и middleware аутентификации.
3. Скриншоты успешной регистрации и входа с получением токена.
4. Скриншот запроса к защищённому маршруту **с валидным токеном**.
5. Скриншот запроса к защищённому маршруту **без токена (401)**.
6. Описание и скриншоты дополнительного механизма.
7. Ответы на контрольные вопросы.
8. Выводы.

**Сдача:** PDF в СЭО + GitHub (ветка `23`); индивидуально; тема по Генератору вариантов; дополнительный механизм выбирается по теме проекта или согласуется с преподавателем.

**Критерии оценки:** работают регистрация и вход; JWT корректно генерируется и проверяется; реализован middleware и защищён маршрут; выполнен дополнительный механизм; оформлен отчёт и даны ответы на вопросы; есть ссылка на репозиторий с веткой `23`.

**Контрольные вопросы (16)**
1. Аутентификация и авторизация — в чём разница?
2. Из каких частей состоит JWT, как проверить подлинность на сервере?
3. Зачем хешировать пароли, что такое соль?
4. Где безопаснее хранить JWT: localStorage или httpOnly cookie, почему?
5. Как защитить маршрут только для админа?
6. Как реализовать logout при JWT?
7. Недостатки localStorage (XSS)?
8. Как работает refresh token?
9. Как ограничить число одновременных сессий?
10. Защита от brute force на уровне API?
11. Что будет при компрометации секрета JWT?
12. Как сделать временную блокировку после неудачных попыток?
13. В каких заголовках передаётся JWT, какой формат?
14. Как задать время жизни токена и обеспечить автоматическое истечение?
15. Можно ли отозвать JWT до истечения срока, какие есть подходы?
16. Сессии с cookie и JWT: когда что лучше?

**Примечания:** продолжает ЛР 1 и 2 (БД подключена, модели предметной области есть). Все инструменты (Node.js, Express, PostgreSQL, Sequelize, bcrypt, jsonwebtoken) локальные или свободные; **облачные Neon и Supabase можно использовать, но это не обязательно**. В **ЛР 4** (следующий модуль) React-приложение интегрируется с этим API, запросы авторизуются через JWT.

---

## Практические занятия

### ПЗ 1. «Middleware и шаблонизаторы»

- **Ветка:** `pz21`. **Срок:** срок не указан в материалах.
- **Цель:** middleware в Express (кастомные: логирование, аутентификация, ошибки) и шаблонизатор **EJS** для динамических HTML-страниц на сервере.
- **Необходимые знания:** middleware (порядок, типы); встроенные `express.json()` и `express.static()`; кастомные middleware `(req, res, next)`; EJS (установка, view engine, рендеринг, передача данных).
- **Задачи (индивидуально):** Express-приложение с EJS; middleware логирования (время и URL); middleware проверки авторизации (имитация); несколько динамических страниц (главная, детальная, форма добавления) с данными из контроллеров; обработка ошибок 404 и 500. Тематика — курсовой проект (Генератор вариантов). **Данные в массиве в памяти, без БД.**
- **Стек:** Node.js, Express, `ejs`.

**Порядок**

1. **Настройка:** взять существующий проект («из лабораторной работы №9» — так в тексте, вероятно, опечатка или сквозная нумерация) или создать новый. `npm install ejs`. В `server.js`:
   ```js
   app.set('view engine', 'ejs');
   app.set('views', './views');
   ```
2. **Шаблоны:** папка `views/`; `layout.ejs` (каркас с `<%- body %>`) или partials. Минимум три страницы:
   - `index.ejs` — главная (список элементов);
   - `item.ejs` — детальная страница по ID;
   - `add.ejs` — форма добавления.
3. **Middleware:**
   - логирующее: выводит в консоль метод, URL и время запроса;
   - имитация авторизации: проверяет query-параметр `?auth=1`; если его нет — редирект на `/login` (или `req.user = { name: 'Гость' }`).
4. **Маршруты и рендеринг:**

   | Метод | URL | Действие |
   |---|---|---|
   | GET | `/` | список элементов |
   | GET | `/item/:id` | детальная страница |
   | GET | `/add` | форма добавления |
   | POST | `/add` | добавить в массив и сделать редирект на `/` |

   Данные в глобальном массиве (`const items = [];`). Для формы понадобится `express.urlencoded()`.
5. **Ошибки:** middleware 404 рендерит `404.ejs`; middleware 500 рендерит `500.ejs` и логирует ошибку.

**Отчёт:** титульный лист, цель; листинги middleware (логирование, проверка авторизации, обработка ошибок); листинг хотя бы одного EJS-шаблона; скриншоты страниц (главная, детальная, форма, 404); **ответы на 30 контрольных вопросов**; выводы. **Сдача:** PDF в СЭО + GitHub (ветка `pz21`), индивидуально, тема по Генератору.

**Критерии оценки:** настроен EJS и есть минимум 3 динамические страницы (список, детали, форма); middleware логирования и авторизации; отдельные страницы 404 и 500; соответствие теме курсового (данные из массива); качество кода (структура, комментарии, именование); полнота отчёта (скриншоты, листинги, 30 ответов); ветка `pz21` **и инструкция по запуску** (README).

**Контрольные вопросы (30, кратко):**
1. Что такое middleware и его сигнатура.
2. Порядок выполнения и что на него влияет.
3. Передача данных в EJS.
4. `next()` и что будет без него.
5. Error-middleware с 4 параметрами.
6. `app.use` и `app.METHOD`.
7. Циклы и условия в EJS.
8. Partials.
9. Массив объектов, отрисованный таблицей.
10. Middleware для одного маршрута.
11. Чем отличается `express.static` от других middleware.
12. Проверка авторизации (имитация по сессии).
13. `res.render` и `res.send`.
14. Ошибки в async-middleware без try/catch.
15. Базовый layout.
16. Middleware до или после маршрутов.
17. Перехват и изменение `req.body`.
18. Несколько middleware подряд.
19. Переменные `res.render` внутри partial.
20. Общий заголовок ответа (`X-Powered-By`).
21. Встроенный middleware для форм (urlencoded).
22. Отключить HTML-ошибки и вернуть JSON.
23. Подключение CSS и JS из статики в EJS.
24. Замер времени выполнения запроса.
25. CSRF через middleware (теоретически).
26. Helper-функция в шаблоне (формат даты).
27. Почему статику подключают до маршрутов.
28. Блокировка по IP.
29. Вывод без экранирования (`<%- %>`, опасно).
30. Error-middleware, который ловит все ошибки.

### ПЗ 2. «Проектирование и реализация REST API»

- **Ветка:** `pz22`. **Срок:** срок не указан в материалах.
- **Цель:** проектирование RESTful API, CRUD-эндпоинты, документирование (Swagger/OpenAPI или ручное описание).
- **Необходимые знания:** принципы REST (ресурсы, методы, идемпотентность, коды); структура URL; коды 200, 201, 204, 400, 401, 404, 500; Swagger UI.
- **Задачи (индивидуально):**
  - спроектировать REST API предметной области, **минимум 3 ресурса**;
  - реализовать все эндпоинты (**CRUD + дополнительные операции по варианту**);
  - добавить **валидацию входных данных**;
  - задокументировать API (файл `API.md` или Swagger/OpenAPI);
  - протестировать в Postman или аналоге.
- **Стек:** **Node.js/Express** (явно: «Часть 2. Реализация (Node.js/Express)»); данные во временном массиве; опционально `swagger-jsdoc` + `swagger-ui-express`.

**Порядок**

1. **Проектирование:** ≥ 3 ресурса; для каждого:

   | Метод | URL | Назначение |
   |---|---|---|
   | GET | `/resource` | список |
   | GET | `/resource/:id` | детали |
   | POST | `/resource` | создание |
   | PUT | `/resource/:id` | полное обновление |
   | PATCH | `/resource/:id` | частичное обновление |
   | DELETE | `/resource/:id` | удаление |

   Определить коды ответов и возможные ошибки.
2. **Реализация:** роуты и контроллеры на каждый ресурс; хранилище — массив (потом его заменит БД); валидация (обязательные поля, типы).
3. **Документация:** `API.md` со всеми эндпоинтами (метод, URL, параметры, пример тела, пример ответа, ошибки). Опционально — `swagger-jsdoc` + `swagger-ui-express`.
4. **Тестирование:** `npm run dev`; проверить каждый эндпоинт в Postman или Insomnia.

**Отчёт:** титульный лист, цель; таблица эндпоинтов (метод, URL, описание, пример тела, ожидаемый ответ); скриншоты документации (Swagger UI или фрагменты `API.md`); скриншоты Postman (**не менее 5 разных операций**); ответы на вопросы; выводы. **Сдача:** PDF в СЭО + GitHub (ветка `pz22`).

**Критерии оценки:** полнота эндпоинтов (≥ 3 ресурса, весь CRUD); корректная реализация на Express.js (роуты, контроллеры, валидация); коды HTTP и обработка ошибок; документация (Swagger или `API.md`); тестирование в Postman (скриншоты); качество ответов; ветка `pz22` **с инструкцией по запуску**.

**Контрольные вопросы (30, кратко):**
1. Идемпотентные методы и почему.
2. Статусы для POST и DELETE.
3. Фильтрация в GET (пример URL).
4. PUT и PATCH.
5. OpenAPI/Swagger и инструменты.
6. Что такое REST и его принципы.
7. Код для «не найден».
8. Именование ресурсов (множественное число).
9. `req.params` и `req.query`.
10. JSON body и нужный middleware.
11. Код при невалидных данных.
12. HATEOAS и обязателен ли он.
13. Вложенные ресурсы (комментарии к посту).
14. Заголовки кэширования.
15. Пагинация (page, limit, offset).
16. Сортировка через query.
17. Версионирование и его способы.
18. Поиск через query.
19. Postman и коллекции.
20. CORS в Express.
21. Единый формат ошибки (пример JSON).
22. Типы аутентификации (Basic, Bearer, OAuth).
23. Схема модели в OpenAPI.
24. Валидация через Joi или express-validator.
25. Межресурсные связи (заказы пользователя).
26. Коды 3xx в REST.
27. Коды ошибок в документации.
28. Idempotency key.
29. Трудности PATCH.
30. Автотесты API (Jest + Supertest).

---

## Вопросы для самоподготовки

Файл 35, 30 вопросов:

1. Event loop; блокирующий и неблокирующий ввод-вывод.
2. Свой модуль в CommonJS: `require` и экспорт функций.
3. `process`, `__dirname`, `__filename`.
4. Зачем `package.json`, как добавить скрипт `start`.
5. Минимальный Express-сервер на порту 3000.
6. POST `/users` и получение JSON из тела.
7. `:id` из `/users/:id` и `sort` из `?sort=asc`.
8. Раздача статики из `public`.
9. Что будет без `next()`; как передать ошибку дальше.
10. Middleware-логгер метода и URL.
11. Установка EJS и настройка view engine.
12. Вывод переменной и `forEach` в EJS.
13. Подключение Sequelize к PostgreSQL; что такое модель.
14. Модель `User`: `name` (string), `email` (string, unique), `isActive` (boolean, default true).
15. `User.create/findAll/findByPk(1)/update/destroy`.
16. Что такое миграция; миграция, добавляющая `age` в `Users`.
17. Принципы REST; метод для списка и для создания.
18. Статусы для успешных GET, POST, PUT, DELETE и для 404.
19. REST API для книг (список, детали, создание, обновление, удаление).
20. Идемпотентность; какие методы идемпотентны и почему.
21. Аутентификация и авторизация; сессия и JWT.
22. Части JWT; проверка подлинности на сервере.
23. Создание JWT при входе (`jsonwebtoken`).
24. Зачем хешировать пароли; bcrypt: hash и compare.
25. Middleware: JWT из `Authorization` в `req.user`.
26. Библиотеки валидации в Express; пример проверки email и пароля.
27. SQL-инъекция; как Sequelize помогает.
28. XSS; экранирование вывода, CSP.
29. CSRF; анти-CSRF токены.
30. CORS; разрешение конкретного домена в Express.

---

## Как применить в проекте NotaCode

### Главное ограничение

Все три ЛР модуля и обе ПЗ **явно требуют Node.js + Express.js**:

- ЛР 1: `npm install express`, `server.js`, скрипты `node`/`nodemon`.
- ЛР 2: **Sequelize + PostgreSQL** (`sequelize`, `pg`, `pg-hstore`, `sequelize-cli`, миграции, сиды).
- ЛР 3: **`jsonwebtoken` + `bcrypt`**, модель `User` в Sequelize, `routes/auth.js`, `middleware/auth.js`.
- ПЗ 1: **EJS**.
- ПЗ 2: заголовок «Реализация (Node.js/Express)».

**Разрешения на замену стека (Python, FastAPI и т.п.) в материалах модуля нет.** Допускаемые вариации:
- PostgreSQL локально **или** в облаке (Neon/Supabase), облако необязательно;
- тестирование через Postman, Insomnia, REST Client VS Code, curl или браузер;
- документация — `API.md` **или** Swagger;
- в ПЗ 1 — `layout.ejs` **или** partials, редирект на `/login` **или** `req.user = {name:'Гость'}`;
- в ЛР 3 дополнительный механизм — из списка **или свой, по согласованию с преподавателем** (это единственное явное «согласуйте» в модуле);
- имя ресурса `items` **обязательно** заменить на термин предметной области.

Итог: FastAPI-бэкенд сам по себе эти работы **не закрывает**. Нужен настоящий Express-сервис, который принимает запросы и работает с данными через Sequelize.

### Предлагаемая схема: Node.js/Express Edge Gateway (BFF) перед FastAPI

Схема согласована с `docs/architecture/README.md` (контейнер «Edge Gateway / BFF», схема БД `identity`, миграции Sequelize).

```
React/Vite PWA ──HTTPS──▶  Express Gateway (Node 22)  ──REST──▶  FastAPI (render/parse/workspace)
                          │ auth (JWT+bcrypt), users,
                          │ projects metadata, shares,
                          │ CORS/helmet/rate-limit,
                          │ proxy /api/render → FastAPI
                          └──Sequelize──▶ PostgreSQL (схема identity/gateway)
```

- **Фреймворк — Express**, а не Fastify (в архитектуре указано «Fastify/Express», а методички требуют Express). Точка входа — `server.js` со скриптами `start`/`dev` ровно как в ЛР 1. TypeScript не запрещён, но в ЛР фигурируют `server.js` и `node server.js`. Надёжнее писать ЛР-ветки на JS (CommonJS как в методичке) или хотя бы оставить `npm start` → `node dist/server.js` и объяснить это в отчёте.
- **Одна PostgreSQL**, у gateway своя схема (`identity`), FastAPI (Alembic) владеет своей. Sequelize-миграции и Alembic-миграции не пересекаются по таблицам.
- **Проксирование**: `http-proxy-middleware` или `fetch` из контроллера. `POST /api/render` → FastAPI `/render`, `POST /api/parse` → `/parse`. Gateway проверяет JWT и передаёт в FastAPI `X-User-Id` (или сервисный токен). FastAPI доверяет только внутренней сети.
- **Расхождение с архитектурой, которое надо решить.** В README таблицы `PROJECT..TEMPLATE` принадлежат Workspace (Python), а gateway владеет только `identity` (users, credentials, refresh_tokens, settings, linked_accounts). Для ЛР 1–2 нужна CRUD-сущность **предметной области** с полем, которое добавит миграция, и со связью 1:N (КВ 5 ЛР 2). Варианты:
  1. **(рекомендуется)** Отдать gateway метаданные проектов: `projects` (id, ownerId → users, name, description, notation, visibility, createdAt/updatedAt). Содержимое файлов, коммиты, парсинг и рендер остаются в FastAPI и ссылаются на `project_id`. Это и есть «auth/users/projects metadata в Node, рендер в FastAPI».
  2. Оставить projects в Python, а gateway дать собственную доменную сущность: **`shares`** (ссылки общего доступа к диаграмме: id, projectId, token, access `view|edit`, expiresAt; миграция добавляет `passwordHash` или `maxViews`) или **`user_settings`/`snippets`**. Связь 1:N: `User hasMany Share`.

  Выбор зафиксировать в ADR.

### ЛР 1 → NotaCode (ветка `lab21`)

- Ресурс: **`projects`** (или `diagrams`), хранение в массиве в памяти.
- Эндпоинты:
  - `GET /projects`, `GET /projects/:id`;
  - `POST /projects` (400, если нет `name` или `notation` не из списка `mermaid|plantuml|...`);
  - `PUT /projects/:id` (400 при неполном теле, 404 при отсутствии);
  - `DELETE /projects/:id` (204, повторно — 404).
- Структура с заделом: `server.js`, `routes/projects.js`, `controllers/projectsController.js`, `models/projects.js` (массив), `middleware/errorHandler.js`. Формат ошибки: `{ "error": "Project not found" }`.
- Можно сразу добавить `POST /render`, который проксирует DSL в FastAPI. В ЛР это не требуется, но показывает интеграцию; в отчёте отметить как дополнительное.
- Отчёт: таблица маршрутов со статусами (200, 201, 204, 400, 404, 500), скриншоты Postman по каждому эндпоинту, curl-пример POST.

### ЛР 2 → NotaCode (ветка `lab12`)

- Код маршрутов ЛР 1 не удалять, заменить массив на Sequelize. `npx sequelize-cli init`, `config/config.json` с `"use_env_variable": "DATABASE_URL"` (та же PostgreSQL, что у FastAPI; в docker-compose или Neon).
- Модели:
  - `User` (появится полноценно в ЛР 3; для связи можно завести уже сейчас);
  - `Project` (`name:string, description:string, notation:string, ownerId:integer`);
  - ассоциация `User.hasMany(Project, {foreignKey:'ownerId'})` / `Project.belongsTo(User)` (закрывает КВ 5 и 10).
- Миграция `add-field-to-projects`: `addColumn('Projects','visibility',{type: ENUM('private','link','public'), defaultValue:'private'})` (или `isArchived`, `lastRenderedAt`).
- Seed `demo-projects`: 3–5 проектов с разными нотациями.
- Пагинация `GET /projects?limit=20&offset=0` (`findAndCountAll`), фильтр `?notation=mermaid` (`where`), обработка `SequelizeUniqueConstraintError` → 409 (уникальность `name` в пределах `ownerId`) — закрывают КВ 3, 6, 7.
- Скриншоты: pgAdmin или DBeaver (схема `identity`), колонка `visibility` после миграции.

### ЛР 3 → NotaCode (ветка `23`)

- `User` (`email` unique, `passwordHash`, плюс поля механизма), `routes/auth.js`: `POST /auth/register` (bcrypt cost 10, 201, 409 при дубликате), `POST /auth/login` (JWT `{id,email}`, `expiresIn:'1h'`).
- `middleware/auth.js`: Bearer, `jwt.verify`, `req.user`, 401.
- `GET /profile` защищён. Дополнительно защитить `POST/PUT/DELETE /projects` и отдавать в `GET /projects` только проекты владельца (проверка `ownerId` от IDOR). `POST /api/render` проксируется в FastAPI только с валидным токеном (гость рендерит локально или через отдельный лимитированный публичный эндпоинт).
- **Дополнительный механизм** — выбраны **Refresh-токены** (`POST /auth/refresh`, refresh в httpOnly cookie, ротация). Они естественны для PWA-IDE с долгими сессиями и совпадают с таблицей `refresh_tokens` в архитектуре. Альтернативы, подходящие NotaCode:
  - **RBAC** (`role: user|admin`, `isAdmin` для модерации публичных шаблонов);
  - **Имитация OAuth (GitHub)** — в ТЗ есть интеграция с GitHub, заглушка потом станет настоящим OAuth в Integration Service;
  - **Восстановление пароля**.

  Выбрать один; если хочется «настоящий GitHub OAuth» вместо заглушки — согласовать с преподавателем.
- Для ЛР 4 (React): фронтенд хранит access-токен в памяти, refresh — в httpOnly cookie, `Authorization: Bearer` в запросах к gateway. CORS в gateway: `origin` = домен Vite/PWA, `credentials: true`.

### ПЗ 1 → NotaCode (ветка `pz21`)

- Отдельное маленькое Express + EJS приложение (или отдельный роутер в gateway, не мешающий API). Страницы: `index.ejs` — список проектов или публичных шаблонов диаграмм; `item.ejs` — карточка с DSL-кодом в `<pre>` (вывод через `<%= %>`, чтобы не было XSS); `add.ejs` — форма «новый шаблон». Данные в массиве.
- Middleware: логгер (метод, URL, время), `?auth=1` → иначе редирект на `/login`. Страницы `404.ejs` и `500.ejs`. README с инструкцией запуска (это критерий оценки).
- Практическая польза для проекта: серверные публичные страницы шэринга или галереи шаблонов (SSR для SEO и Open Graph превью ссылок на диаграммы, лекция 01).

### ПЗ 2 → NotaCode (ветка `pz22`)

- ≥ 3 ресурса на Express (массивы): **`projects`**, **`files`** (вложенно `/projects/:id/files` или плоско `/files?projectId=`), **`templates`** (или `shares`). Для каждого реализовать GET list, GET by id, POST, PUT, PATCH, DELETE.
- Дополнительные операции:
  - `POST /projects/:id/duplicate`;
  - `POST /files/:id/render` (прокси в FastAPI);
  - `GET /templates?notation=plantuml&sort=-createdAt&page=1&limit=10`.
- Валидация через `express-validator` или Joi: `name` обязателен, `notation ∈ {...}`, `content` — строка ≤ N КБ. Ответ 400 или 422 с `details`.
- Документация: `API.md` + Swagger (`swagger-jsdoc`, `swagger-ui-express` на `/api-docs`). Та же спецификация пригодится фронтенду. Postman-коллекция ≥ 5 операций.

### Как теория ложится на NotaCode

| Тема | Применение |
|---|---|
| Event loop, I/O-bound | Gateway в основном ждёт сеть и БД и проксирует, это сильная сторона Node. CPU-тяжёлый парсинг и рендер — в FastAPI |
| ESM и CJS, npm, nvm | `engines.node >= 22`, `package-lock.json` в git, `npm audit` в CI |
| Middleware-цепочка | `helmet → cors → express.json({limit}) → morgan → rateLimit → routes → 404 → errorHandler` |
| REST-дизайн | `/api/v1/projects`, `/api/v1/projects/:id/files`, `POST /api/v1/render`; единый формат ошибки для фронтенда и FastAPI |
| Коды | 201 + `Location` при создании проекта, 202 для долгого экспорта или AI-генерации, 409 при конфликте имени или версии файла (`ETag`/`If-Match`), 413 для слишком большого DSL, 429 при лимите рендера |
| GraphQL | Не нужен для MVP; REST + OpenAPI проще. Можно упомянуть как альтернативу для дерева «проект → файлы → версии» |
| JWT и сессии | Короткий access + refresh в httpOnly cookie; `tokenVersion` для «выйти на всех устройствах» |
| bcrypt | cost 10–12, pepper из env, перехеширование при повышении cost |
| RBAC и ownership | Владелец, редактор и зритель проекта при шэринге; middleware `checkProjectAccess(role)` |
| Валидация | DSL-текст ограничен по размеру; `notation` проверяется по whitelist; неизвестные поля отбрасываются |
| XSS | **Главный риск: SVG из рендера.** Санитизировать SVG (DOMPurify на клиенте, запрет `<script>`/`on*`/`foreignObject`), CSP, не вставлять пользовательский текст без экранирования |
| CSRF и CORS | Refresh-cookie `SameSite=Strict` + CSRF-токен на `/auth/refresh`; CORS только для origin PWA |
| SQL-инъекции | Sequelize с параметрами; никаких `sequelize.query` со склейкой строк |
| Безопасность клиента | Ключи LLM, GitHub и Kroki только на сервере; `VITE_*` не содержит секретов |
| DevSecOps | CI: ESLint + Semgrep (SAST), `npm audit` + Dependabot (SCA), OWASP ZAP baseline против staging (DAST), поиск секретов |
| Graceful shutdown | Gateway закрывает HTTP-сервер и пул Sequelize по SIGTERM (важно для Docker и деплоя в модуле 05) |

### Практические рекомендации по репозиторию

- Монорепо: `services/gateway/` (Express + Sequelize), `services/*` (FastAPI), `web/` (React). Ветки ЛР `lab21`, `lab12`, `23`, `pz21`, `pz22` должны содержать работающий gateway с README по запуску (`npm i`, `.env.example` с `DATABASE_URL`, `JWT_SECRET`, `FASTAPI_URL`, `npx sequelize-cli db:migrate`, `db:seed:all`, `npm run dev`).
- ПЗ 1 и 2 по методичке хранят данные в массиве. Если делать их внутри gateway, держать отдельный in-memory модуль или отдельную папку `practice/pz21`, `practice/pz22`, чтобы не ломать основную ветку с БД.
- В каждом отчёте одной фразой объяснять архитектуру: «Серверная часть курсового на Node.js/Express выполняет роль API-шлюза (аутентификация, пользователи, метаданные проектов), тяжёлая обработка DSL вынесена в сервис на Python/FastAPI». Так прозрачно видно, что требования ЛР (Express, Sequelize, JWT, bcrypt) выполнены именно на Node.

---

## Нечитаемые материалы

Нет. Все 40 файлов извлечены и прочитаны. Изображения в лекциях 15, 24, 25, 33 не несут информации, которой нет в тексте.
