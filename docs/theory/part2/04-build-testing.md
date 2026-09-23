# Модуль 04. Инструменты сборки и тестирование

Цель модуля: разобраться, как модульные системы JS лежат в основе сборщиков (Webpack, Vite, Parcel), как устроены менеджеры пакетов и таск-раннеры, что такое SSG, и как тестировать серверные приложения (Jest, Supertest, Playwright, coverage, CI).

## Источники

Папка: `E:\ИТиВП\Часть 2\Модуль 04. Инструменты сборки и тестирование\`

| # | Файл | Тип | Статус |
|---|------|-----|--------|
| 01 | Модульные системы.pdf | лекция | прочитан полностью |
| 02 | Инструменты сборки и автоматизации.pdf | лекция | прочитан полностью; PDF открыт и визуально, см. «Нечитаемые материалы» (пустые блоки) |
| 03 | Сборщики Webpack и Vite.pdf | лекция | прочитан полностью |
| 04 | Инструменты сборки пакетов (бандлеры).pdf | лекция | прочитан полностью |
| 05 | SSG (Static Site Generation).pdf | лекция | прочитан полностью |
| 06 | Тестирование серверных приложений.pdf | лекция | прочитан полностью |
| 07 | Вопросы для самоподготовки.pdf | вопросы | прочитан полностью (30 вопросов) |

**В модуле нет ни одного файла лабораторной работы или практического занятия** (в папке 7 PDF: 6 лекций и список вопросов).

---

## Ключевая теория

### 1. Модульные системы JavaScript (лекция 01)

**Зачем нужны модули.** Когда весь код лежал в глобальной области видимости, возникали четыре проблемы:
- конфликты имён между библиотеками;
- неявные зависимости и ручной порядок тегов `<script>`;
- много тегов `<script>`, из-за чего страница медленно грузится;
- нет встроенной инкапсуляции (пространств имён).

Модуль — изолированный фрагмент кода. Он явно экспортирует свой API и явно импортирует зависимости.

**Эволюция:**

1. **IIFE** (протомодуль): замыкание даёт приватную область видимости. Проблемы зависимостей и загрузки это не решает.
   ```js
   const counter = (function () {
     let n = 0;                        // приватное состояние
     return { inc: () => ++n };        // публичный API
   })();
   ```
2. **CommonJS (CJS)** — стандарт Node.js:
   - синхронная загрузка: подходит для сервера, где файлы лежат на локальном диске;
   - модуль кэшируется после первого `require`;
   - `require` можно вызывать где угодно, в том числе внутри условия (динамическая загрузка);
   - браузер не понимает CJS без сборки.
   ```js
   module.exports = { add };           // или exports.add = add;
   const { add } = require('./math');
   ```
3. **AMD** (реализация RequireJS): асинхронная загрузка для браузера через `define([...deps], factory)` и `require([...], cb)`. Синтаксис громоздкий. Сегодня AMD устарел.
4. **ES Modules (ESM)** — стандарт с ES2015. Работает в браузерах и в Node.js (с флагами с v12, стабильно с v14):
   - **статическая структура**: импорты и экспорты известны на этапе парсинга, без выполнения кода. Поэтому возможны **tree shaking** (удаление неиспользуемых экспортов) и оптимизация загрузки;
   - асинхронная параллельная загрузка в браузере;
   - строгий режим (`use strict`) включён автоматически;
   - в браузере: `<script type="module">`;
   - в Node.js: расширение `.mjs` или `"type": "module"` в `package.json`.
   ```js
   export const PI = 3.14;
   export default class Renderer {}
   import Renderer, { PI } from './renderer.js';
   import * as utils from './utils.js';
   ```

**Сравнение:**

| Характеристика | CommonJS | AMD | ESM |
|---|---|---|---|
| Среда | Node.js | браузер (RequireJS) | браузер + Node.js |
| Загрузка | синхронная | асинхронная | асинхронная, структура статическая |
| Синтаксис | `require` / `module.exports` | `define` / `require` | `import` / `export` |
| Статический анализ | нет | нет | да (tree shaking) |
| Кэширование | да | да | да (по URL) |
| Статус | по умолчанию в Node.js | устарел | стандарт для нового кода |

**Почему нативных ESM в браузере недостаточно для production:**
- каждый импорт порождает отдельный HTTP-запрос. HTTP/2 смягчает проблему, но не снимает её;
- нельзя импортировать CSS, картинки и шрифты;
- нет минификации и обфускации;
- очень старые браузеры модули не поддерживают;
- нет удобного способа подключать npm-пакеты: остаются CDN или неудобные пути в `node_modules`.

Эти проблемы закрывают сборщики (Webpack, Vite, Rollup, Parcel). Они:
- объединяют модули в бандлы;
- обрабатывают ресурсы (CSS, изображения, шрифты);
- транспилируют код через Babel;
- минифицируют сборку;
- дают HMR в режиме разработки.

### 2. Менеджеры пакетов, таск-раннеры, загрузчики модулей (лекция 04)

Лекция делит инструменты сборки на три класса.

- **Менеджеры пакетов** управляют зависимостями: установка, обновление, версии.
  - **npm** — официальный менеджер. Поставляется вместе с Node.js, самая большая экосистема.
  - **Yarn** создан в Facebook. Преимущества: быстрее установка, параллельная загрузка, кэш зависимостей, надёжнее разрешение конфликтов версий.
- **Node.js** — среда выполнения JS на движке V8:
  - однопоточная событийная модель (event loop), операции ввода-вывода не блокируют поток;
  - модульность, высокая производительность, большая экосистема;
  - применение: API, микросервисы, real-time-приложения, чаты.
  - **Express.js** — минималистичный фреймворк поверх Node.js: маршрутизация, обработка запросов и ответов, middleware, обработка ошибок.
- **Исполнители задач (task runners)** автоматизируют рутину: компиляцию CSS/JS, оптимизацию изображений, запуск тестов.
  - **Grunt** — старше, задачи описываются декларативным JSON-подобным конфигом.
  - **Gulp** — новее, задачи пишутся JS-кодом в виде потоков (streams), проще в использовании.
- **Загрузчики модулей / сборщики** собирают модули в исполняемый файл.
  - **Browserify** — старше и проще: даёт использовать `require` (CommonJS) в браузере.
  - **Webpack** — гибче и мощнее, поддерживает ESM.
- **Выбор инструмента** зависит от языка, фреймворка, набора задач для автоматизации и используемой модульной системы.
- **Что дают такие инструменты:** автоматизацию рутины, контроль качества (оптимизация, безопасность) и производительность (меньше файлов и запросов).

### 3. Инструменты сборки: Vite, Webpack, Parcel (лекция 02)

| | Vite | Webpack | Parcel |
|---|---|---|---|
| Идея | нативные ESM в dev, компилирует файлы «по требованию» | полный контроль через `webpack.config.js`, лоадеры и плагины | ноль конфигурации |
| Dev | мгновенный холодный старт, быстрый HMR | `webpack-dev-server`, HMR | HMR, многопоточная сборка |
| Оптимизация | tree shaking, code splitting (Rollup/esbuild) | tree shaking, code splitting, lazy loading | минификация, code splitting автоматически |
| Плюсы | скорость, простота, современные стандарты | гибкость, огромная экосистема, большие проекты | простота, скорость, прототипы |
| Минусы | ориентирован на современные проекты | сложный конфиг, медленный на малых проектах | меньше гибкости и настроек |

- Parcel запускается одной командой без конфига: `parcel index.html`.
- В лекции 02 сказано, что production-сборку Vite делает через esbuild. Это неточность: в лекции 03 уточнено, что production-сборкой занимается **Rollup**, а esbuild используется для pre-bundling зависимостей и транспиляции. На вопрос 13 правильный ответ — Rollup.

### 4. Webpack и Vite подробно (лекция 03)

**Задачи сборщика:**
- объединить модули и сократить число запросов;
- транспилировать ES6+, JSX и TypeScript через Babel;
- обработать CSS/SCSS, изображения и шрифты;
- оптимизировать сборку: минификация, tree shaking, code splitting;
- дать HMR при разработке;
- собрать production-вариант.

#### Webpack

Webpack строит **граф зависимостей** по `import`/`require`. Ключевые понятия:
- **entry** — точка входа;
- **output** — путь и имя бандла. `[contenthash]` добавляет в имя хеш для кэширования, `clean: true` очищает `dist` перед сборкой;
- **mode**: `development` / `production` / `none`;
- **module.rules (loaders)** — преобразуют файлы на лету:
  - `babel-loader` с пресетами `@babel/preset-env` и `@babel/preset-react`;
  - `css-loader` + `style-loader`. Лоадеры применяются **справа налево**, поэтому пишут `['style-loader','css-loader']`;
  - `type: 'asset/resource'` для изображений (встроенные asset-модули Webpack 5);
- **plugins** — более сложные задачи. Пример: `HtmlWebpackPlugin` генерирует HTML по шаблону. Сюда же относятся минификация и копирование файлов;
- **devServer** (`webpack-dev-server`): `static`, `hot: true` (HMR), `port`;
- **resolve.extensions**: `['.js', '.jsx']`.

Скелет конфига (набросок):
```js
module.exports = {
  entry: './src/index.js',
  output: { path: path.resolve(__dirname, 'dist'), filename: 'bundle.[contenthash].js', clean: true },
  mode: 'production',
  module: { rules: [
    { test: /\.jsx?$/, exclude: /node_modules/, use: 'babel-loader' },
    { test: /\.css$/, use: ['style-loader', 'css-loader'] },
    { test: /\.(png|svg|jpg|gif)$/, type: 'asset/resource' },
  ]},
  plugins: [new HtmlWebpackPlugin({ template: './src/index.html' })],
  devServer: { hot: true, port: 3000 },
};
```

**Что включает режим production:**
- минификация через TerserPlugin;
- хеши в именах файлов (если задан `[contenthash]`);
- без отладочной информации, source maps по желанию;
- tree shaking для ESM.

Запуск: `npx webpack --mode production` или npm-скрипт `"build"`.

- Плюсы: максимальная гибкость, огромная экосистема, подходит проектам любого масштаба, хорошая поддержка старых браузеров.
- Минусы: сложный и громоздкий конфиг, медленный старт на больших проектах, долго изучать.

#### Vite

Vite создал Эван Ю, автор Vue. Принцип работы:
- в **dev** Vite не бандлит проект. Браузер запрашивает нативные ES-модули, Vite преобразует их по требованию. Отсюда мгновенный холодный старт и быстрый HMR;
- **pre-bundling зависимостей**: esbuild заранее собирает npm-пакеты в эффективные ESM;
- **production**: сборку делает `vite build` через **Rollup** (tree shaking, code splitting и минификация по умолчанию), результат попадает в `dist`;
- TypeScript, JSX, CSS Modules и препроцессоры работают из коробки;
- для старых браузеров нужен плагин `@vitejs/plugin-legacy`.

```js
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
export default defineConfig({
  plugins: [react()],
  resolve: { alias: { '@': path.resolve(__dirname, './src') } },
  server: { port: 3000, open: true },
  build: {
    outDir: 'dist', sourcemap: false,
    rollupOptions: { output: { manualChunks: { vendor: ['react', 'react-dom'] } } },
  },
});
```

Команды:
- `npm create vite@latest my-app -- --template react` — создать проект;
- `npm run dev` — режим разработки;
- `npm run build` — production-сборка;
- `npm run preview` — локальный предпросмотр сборки.

**Webpack и Vite:**

| Критерий | Webpack | Vite |
|---|---|---|
| Скорость dev | средняя/низкая | очень высокая |
| Скорость production-сборки | средняя | высокая (esbuild + Rollup) |
| Настройка | сложная | конфиг опционален |
| Гибкость | максимальная | достаточная |
| Экосистема | огромная | растущая |
| Старые браузеры | полная поддержка (Babel, полифиллы) | через `@vitejs/plugin-legacy` |
| Dev-подход | бандлинг в память | нативные ESM без бандлинга |

- **Webpack** выбирают для большого legacy-проекта, кастомных требований или очень старых браузеров.
- **Vite** выбирают для нового проекта на React, Vue или Svelte.

**Термины, которые спрашивают в вопросах:**
- **Code splitting** — разбиение бандла на чанки, которые грузятся по требованию (`import()` / `React.lazy`, `manualChunks`). Уменьшает начальную загрузку.
- **Tree shaking** — удаление неиспользуемых экспортов. Работает только благодаря статической структуре ESM.
- **Source maps** — соответствие минифицированного кода исходному. Нужны для отладки и для стектрейсов ошибок из production.
- **HMR** — замена изменённого модуля в работающем приложении без полной перезагрузки и с сохранением состояния.

### 5. SSG — Static Site Generation (лекция 05)

**Суть:** все HTML-страницы генерируются заранее, во время сборки (`npm run build`), и раздаются как статические файлы. На каждый запрос серверная логика не выполняется.

**Процесс:**
1. **Подготовка.** Компоненты пишутся на React, Vue или Svelte. Источники данных: Markdown, headless CMS (Contentful, Strapi), JSON/YAML, API во время сборки.
2. **Сборка.** Фреймворк читает все источники данных, выполняет компонент для каждой страницы, подставляет данные и рендерит чистый HTML в `dist/` или `out/`.
3. **Хостинг.** Файлы раздаются через CDN (Cloudflare, CloudFront), статический хостинг (Vercel, Netlify, GitHub Pages) или Nginx.

**Плюсы:**
- производительность: 90–100 баллов в PageSpeed, статика полностью кэшируется на CDN;
- безопасность: нет серверного кода и БД, значит нет SQL-инъекций и RCE;
- SEO: краулер получает готовый HTML;
- дешёвый хостинг и практически неограниченная масштабируемость;
- простой CI/CD: откат означает возврат к предыдущей сборке.

**Минусы:**
- контент обновляется только после пересборки;
- нет real-time;
- интерактивность ограничена, нужен клиентский JS;
- долгая сборка на сайтах от 10 тыс. страниц.

**Решения этих проблем:**
- **ISR** (Next.js): `getStaticProps` возвращает `revalidate: N`, и страница перегенерируется не чаще раза в N секунд.
- **On-Demand Revalidation**: вебхук из CMS запускает пересборку конкретных страниц.
- **Гибрид**: статика для «О компании», блога и документации, SSR для личного кабинета и поиска, SPA-логика для корзины и фильтров.

Пример на Next.js (Pages Router):
- `getStaticPaths` возвращает список `params` и `fallback: false`;
- `getStaticProps({ params })` возвращает `props` и `revalidate`;
- компонент страницы рендерит пост.

В примере лекции `revalidate: 3600` подписан как «каждые 2 часа». На самом деле 3600 с — это 1 час.

**Инструменты:**
- универсальные фреймворки: Next.js (SSG/SSR), Nuxt, SvelteKit;
- специализированные генераторы: Gatsby, VitePress (документация), Astro (мультифреймворк, минимум JS), Hugo (Go, очень быстрый), Jekyll (Ruby, блоги).

**Когда подходит:**
- подходит: лендинги, блоги, документация, портфолио, каталоги магазинов, новости (с ISR);
- не подходит: соцсети, мессенджеры, real-time-системы, сильно персонализированные приложения.

| | SSG | SSR | SPA |
|---|---|---|---|
| Загрузка | самая быстрая | быстрая | медленная |
| SEO | максимальное | высокое | низкое |
| Сложность | низкая | высокая | средняя |
| Хостинг | очень дешёвый | дорогой | дешёвый |
| Обновление контента | после сборки | при запросе | на клиенте |
| Масштабируемость | бесконечная | ограниченная | бесконечная |

### 6. Тестирование серверных приложений (лекция 06)

**Уровни тестирования:**
- **unit** — функция, модуль или класс в изоляции;
- **integration** — связка компонентов (запрос → контроллер → БД);
- **E2E** — полный пользовательский сценарий через браузер или API.

Стек из лекции для Node.js: Jest + Supertest + Playwright.

#### Jest (unit)

**Возможности:**
- встроенный `expect`;
- моки: `jest.fn()`, `jest.mock()`;
- асинхронные тесты;
- покрытие через `--coverage`;
- снапшоты.

**Установка и скрипты:** `npm i -D jest`, затем в `package.json`:
- `"test": "jest"`;
- `"test:watch": "jest --watch"`;
- `"test:coverage": "jest --coverage"`.

CommonJS работает сразу. Для ESM и TypeScript нужны Babel или ts-jest.

**Основные приёмы:**
- **Базовые матчеры:** `toBe`, `toEqual`, `toBeTruthy`/`toBeFalsy`, `toHaveProperty`, `toMatchObject`, `toBeInstanceOf`.
- **Асинхронный тест:** функция теста объявляется `async`, внутри `await`. Также возможны возврат промиса и колбэк `done` (упоминается в вопросах).
- **Мок функции:** подменить `global.fetch = jest.fn(() => Promise.resolve({ json: () => Promise.resolve({...}) }))`, затем проверить `expect(fetch).toHaveBeenCalledWith(url)`.
- **Мок модуля целиком:** `jest.mock('axios')`.
- **Группировка тестов:** `describe('...', () => { test(...); })`.

```js
const { add } = require('./math');
test('1 + 2 = 3', () => expect(add(1, 2)).toBe(3));
```

#### Supertest (integration)

Supertest отправляет HTTP-запросы в экспортированное Express-приложение (`module.exports = app`) без запуска сервера на порту.

```js
const request = require('supertest');
const app = require('./app');
test('POST /users', async () => {
  const res = await request(app).post('/users').send({ name: 'Bob' }).expect(201);
  expect(res.body).toMatchObject({ name: 'Bob' });
});
```

- **Цепочки проверок:** `.expect(200)`, `.expect('Content-Type', /json/)`.
- **Тестовая БД:**
  - берут отдельную БД: SQLite в памяти, отдельную схему PostgreSQL или отдельный `TEST_DB_URL` для MongoDB;
  - хуки: `beforeAll` подключается, `afterAll` отключается, `beforeEach` очищает данные (`User.deleteMany({})`);
  - после запроса проверяют, что запись действительно появилась в БД;
  - в вопросах упомянута ещё и изоляция через транзакции с откатом.
- **Плюсы:** не нужен запущенный сервер, цепочки ассертов, интеграция с Jest, работает с Express, Fastify, Koa и Nest.

#### Playwright (E2E)

Playwright — инструмент Microsoft для автоматизации Chromium, Firefox и WebKit. Умеет тестировать API и эмулировать мобильные устройства.

- **Установка:** `npm init playwright@latest` создаёт `tests/` и `playwright.config.js` и ставит браузеры.
- **Тест:** `test('...', async ({ page }) => {...})`. Основные действия:
  - `page.goto(url)`;
  - `page.fill('#email', ...)`;
  - `page.click('button[type="submit"]')`;
  - `await expect(page).toHaveURL(/login/)`;
  - `await expect(page.locator('h1')).toContainText('Dashboard')`.
- **API-тест:** фикстура `request`, например `await request.post(url, { data })`, затем проверяют `response.status()` и `await response.json()`.
- **Запуск:** `npx playwright test`, `--ui` (графический интерфейс), `--headed` (с открытым браузером).
- **Прочее:** параллельный запуск в разных браузерах, скриншоты, видео, трейсы (trace viewer).

#### Покрытие кода

- **Запуск:** `npx jest --coverage`. HTML-отчёт появляется в `coverage/index.html`, краткая таблица выводится в консоль.
- **Метрики:**
  - **Statements** — выполненные операторы;
  - **Branches** — ветви if/else и switch;
  - **Functions** — вызванные функции;
  - **Lines** — выполненные строки.
- **Пороги в `jest.config.js`:**
  ```js
  module.exports = {
    collectCoverageFrom: ['src/**/*.js', '!src/**/*.test.js'],
    coverageThreshold: { global: { statements: 80, branches: 70, functions: 80, lines: 80 } },
  };
  ```
  Если порог не достигнут, прогон тестов падает.
- Playwright тоже умеет собирать покрытие (через V8 и v8-to-istanbul), но это сложнее. Обычно покрытие меряют на unit- и integration-уровне. Альтернатива — nyc (Istanbul).

#### CI/CD

GitHub Actions, файл `.github/workflows/test.yml`. Триггер: `on: [push, pull_request]`. Шаги:
1. `actions/checkout`;
2. `actions/setup-node` (node 18);
3. `npm ci`;
4. `npm test`;
5. `npx playwright install --with-deps`;
6. `npm run test:e2e`;
7. `npm run test:coverage`.

**Итоговые рекомендации лекции:**

| Уровень | Инструменты | Что проверяем |
|---|---|---|
| Unit | Jest (или Mocha + Chai) | функции, модули, классы |
| Integration | Jest + Supertest | эндпоинты, работа с БД |
| E2E | Playwright (или Cypress) | полные сценарии |
| Coverage | `jest --coverage`, nyc | процент покрытия |

- начинать с unit-тестов бизнес-логики;
- интеграционные тесты писать на критичные API;
- E2E писать только на основные сценарии (happy path), потому что они дорогие;
- держать покрытие ключевых модулей на уровне 70–80%.

---

## Лабораторные работы

**В модуле 04 лабораторных работ нет.** В папке модуля нет файла лабораторной или практической работы: только 6 лекций и список вопросов для самоподготовки. Поэтому нечего указывать о сроках, вариантах, отчёте и критериях оценки, в материалах модуля этого нет.

**Практических заданий и упражнений в лекциях тоже нет.** Ни одна лекция не содержит формулировок «выполните», «задание» или «упражнение». В лекциях есть только **пошаговые примеры**, которые можно повторить для самоконтроля:

| Лекция | Пример, который можно воспроизвести |
|---|---|
| 01 | Один и тот же модуль в CJS и в ESM; подключение через `<script type="module">`; `"type": "module"` в Node.js |
| 03 | `webpack.config.js` с babel-loader, css/style-loader, asset/resource, HtmlWebpackPlugin, devServer; проект `npm create vite@latest -- --template react`, `vite.config.js` с alias и `manualChunks`, команды dev/build/preview |
| 05 | SSG-страница блога на Next.js: `getStaticPaths` + `getStaticProps` с `revalidate` |
| 06 | Jest-тесты `math.js` (add/isEven); мок `fetch`; Supertest для `GET/POST /users`; тест с реальной БД и хуками; Playwright-сценарий «регистрация → вход → Dashboard»; API-тест через Playwright; `coverageThreshold`; workflow GitHub Actions |

Вероятно, тестирование и сборку оценят в рамках лаб других модулей или курсового проекта. Например, в лабах по Node.js и React из модулей 01–02 могут требовать тесты; это нужно сверить с их заметками. **Из материалов модуля 04 это не следует.**

---

## Вопросы для самоподготовки

**Модульные системы**
1. Что такое модульные системы, CommonJS и ESM, в чём их принципиальное отличие.
2. `require`/`module.exports`: где эта система используется по умолчанию (Node.js).
3. `import`/`export`: `.mjs` или `"type": "module"` в Node.js.
4. Статическая структура ESM, tree shaking.

**Сборщики Webpack и Vite**

5. Что такое сборщик и какие задачи он решает (объединение, транспиляция, оптимизация).
6. Концепции Webpack: entry, output, loaders, plugins.
7. Лоадеры: настройка babel-loader и css-loader/style-loader.
8. webpack-dev-server и HMR.
9. Режимы development и production, оптимизации в production.
10. Vite: ключевое отличие в dev (нативные ESM).
11. Почему у Vite быстрый холодный старт и мгновенный HMR.
12. `vite.config.js`, плагин `@vitejs/plugin-react`.
13. Команда production-сборки Vite (`vite build`) и бандлер под капотом (Rollup).
14. Code splitting и tree shaking: как уменьшают бандл.
15. Source maps: зачем нужны при отладке production.

**Тестирование серверных приложений**

16. Виды тестов (unit, integration, e2e): объём и цель.
17. Jest: установка, тест функции сложения.
18. Асинхронные тесты: async/await, промисы, `done`.
19. Моки: мок функции, проверка аргументов вызова.
20. `jest.mock()` для целого модуля (axios).
21. Supertest: тест эндпоинта Express.
22. Интеграционный тест: POST создаёт пользователя, затем проверка в БД.
23. Изоляция тестовой БД: отдельная БД, транзакции с откатом.
24. Playwright и поддерживаемые браузеры (Chromium, Firefox, WebKit).
25. E2E-тест: открыть страницу, заполнить форму, проверить сообщение об успехе.
26. Отладка в Playwright: headed-режим, trace viewer, видео.
27. Покрытие в Jest и его метрики.
28. Пороги покрытия (например, 80% строк).
29. Тесты в CI/CD (GitHub Actions): как и зачем.
30. Стратегия: когда писать unit-, integration- и E2E-тесты.

---

## Как применить в проекте NotaCode

NotaCode — web-IDE diagram-as-code: текстовый DSL (Mermaid, PlantUML и др.) превращается в диаграмму. Фронтенд: React + Vite + PWA. Бэкенд: Python FastAPI.

### Требования к стеку и допустимые замены

- Лабораторных работ в модуле нет, поэтому **обязательных технологий модуль не задаёт**.
- Примеры в лекциях написаны на Node.js (Jest, Supertest, Express, Mongoose). Но лекция 06 прямо называет серверную часть «Node.js + Express/FastAPI и т.д.»: принципы тестирования подаются как независимые от стека.
- Поэтому для Python-бэкенда оправданы **прямые аналоги**:

| Лекция (Node) | Аналог для NotaCode (Python) |
|---|---|
| Jest | pytest |
| Supertest | `httpx.AsyncClient` + `ASGITransport(app=app)` или `fastapi.testclient.TestClient` (тоже без запуска сервера на порту) |
| `jest.fn()` / `jest.mock()` | `unittest.mock` / `pytest-mock`, `app.dependency_overrides` |
| `jest --coverage`, `coverageThreshold` | `pytest-cov`: `--cov=app --cov-fail-under=80` |
| `beforeAll`/`beforeEach` | фикстуры pytest (`scope="session"`/`"function"`) |

- Если какая-то лаба другого модуля потребует тесты именно на Jest/Supertest, есть два пути:
  - покрыть ими **тонкий Node.js/Express BFF (API-gateway)** перед FastAPI, если он появится ради требований модуля 01;
  - покрыть Jest-тестами (через Vitest) фронтенд-логику.

### Сборка: Vite-конфиг для React PWA

Что использовать из лекций 01–03:
- **ESM и tree shaking.** Импортировать точечно: `import { debounce } from 'lodash-es'`, а не весь `lodash`. Держать `"type": "module"` в `package.json` фронтенда.
- **Code splitting рендереров диаграмм.** Mermaid, PlantUML-энкодер (plantuml-encoder / Kroki-клиент), Graphviz (`@hpcc-js/wasm`) и Monaco/CodeMirror тяжёлые. Их не должно быть в стартовом бандле:
  ```js
  // renderers/index.ts — движок грузится только при открытии файла с этим DSL
  const loaders = {
    mermaid:  () => import('./mermaidRenderer'),
    graphviz: () => import('./graphvizRenderer'),
    plantuml: () => import('./plantumlRenderer'),
  };
  export const getRenderer = (lang) => loaders[lang]().then(m => m.default);
  ```
  Страницы (Editor, Projects, SharedView) подключать через `React.lazy` + `<Suspense>`.
- **Конфиг `vite.config.ts`:**
  ```ts
  import { defineConfig } from 'vite';
  import react from '@vitejs/plugin-react';
  import { VitePWA } from 'vite-plugin-pwa';
  export default defineConfig({
    plugins: [
      react(),
      VitePWA({
        registerType: 'autoUpdate',
        manifest: { name: 'NotaCode', short_name: 'NotaCode', display: 'standalone', theme_color: '#1e1e2e',
                    icons: [{ src: '/icon-192.png', sizes: '192x192', type: 'image/png' },
                            { src: '/icon-512.png', sizes: '512x512', type: 'image/png' }] },
        workbox: {
          globPatterns: ['**/*.{js,css,html,svg,png,woff2,wasm}'],
          maximumFileSizeToCacheInBytes: 5 * 1024 * 1024,   // mermaid/graphviz-чанки крупные
          runtimeCaching: [{ urlPattern: /\/api\/projects/, handler: 'NetworkFirst',
                             options: { cacheName: 'api-projects' } }],
        },
      }),
    ],
    resolve: { alias: { '@': '/src' } },
    server: { port: 5173, proxy: { '/api': 'http://localhost:8000' } },   // dev-прокси на FastAPI
    build: {
      sourcemap: true,          // source maps для стектрейсов (вопрос 15); можно 'hidden' + загрузка в Sentry
      rollupOptions: { output: { manualChunks: {
        react: ['react', 'react-dom', 'react-router-dom'],
        editor: ['@monaco-editor/react'],
        mermaid: ['mermaid'],
      } } },
    },
  });
  ```
  - `server.proxy` устраняет CORS в dev;
  - `vite build` (Rollup) кладёт статику в `dist`, FastAPI или Nginx её раздают;
  - `vite preview` проверяет PWA и service worker локально, потому что в dev SW по умолчанию выключен.
- **Webpack и Parcel** не нужны. Их можно упомянуть в отчёте для сравнения, а выбор Vite обосновать скоростью HMR, нативными ESM и встроенной PWA через плагин (таблицы из лекций 02–03).
- **Менеджер пакетов:** npm (или pnpm/yarn), с фиксированным `package-lock.json` и `npm ci` в CI (лекция 04). Роль task runner (Grunt/Gulp) выполняют npm-скрипты: `dev`, `build`, `preview`, `test`, `test:e2e`, `lint`.

### SSG

- Сам IDE — это SPA/PWA, его нельзя делать SSG: он персонализирован, требует авторизации и пересчитывает диаграммы в реальном времени.
- SSG подходит для **публичной части**:
  - лендинг;
  - документация по синтаксису DSL и примеры (VitePress или Astro, отдельный сайт `docs/`);
  - галерея публичных шаблонов диаграмм.
- **Публичные расшаренные диаграммы** (`/s/{slug}`) можно пререндерить в статический SVG/HTML с open-graph-превью (гибридный подход из лекции). Инвалидация — при сохранении файла, по аналогии с on-demand revalidation.

### Тестирование: фронтенд (Vitest + React Testing Library)

- **Vitest** — Jest-совместимый API (`describe/test/expect`, `vi.fn()`, `vi.mock()`), использует тот же `vite.config`. Настройка: `test: { environment: 'jsdom', setupFiles: './src/test/setup.ts', coverage: { provider: 'v8', thresholds: { lines: 80, branches: 70 } } }`.
- **Unit-тесты:**
  - парсер и валидатор DSL (определение языка по расширению `.mmd` / `.puml`);
  - утилиты сериализации проекта;
  - хуки `useAutosave` и `useDebouncedRender` с `vi.useFakeTimers()`.
- **Компонентные тесты (RTL):**
  - ввод текста в редактор → вызов рендерера (рендерер мокается через `vi.mock('./renderers/mermaidRenderer')`, аналог `jest.mock` из вопроса 20);
  - показ ошибки синтаксиса;
  - формы логина и регистрации;
  - диалог «Поделиться».
- **Мок сети:** `vi.fn()` для `fetch` (как в лекции) или MSW для `/api/*`.
- **Скрипты:** `"test": "vitest run"`, `"test:watch": "vitest"`, `"test:coverage": "vitest run --coverage"`.

### Тестирование: бэкенд (pytest + httpx)

- **Unit-тесты:**
  - сервисы: хеширование паролей, выпуск и проверка JWT, генерация share-slug;
  - прав доступа (owner / editor / viewer);
  - обёртка рендера (вызов Kroki или PlantUML-сервера мокается через `unittest.mock.patch` или `respx`).
- **Интеграционные тесты (аналог Supertest):**
  ```python
  import pytest, httpx
  from app.main import app

  @pytest.fixture
  async def client():
      transport = httpx.ASGITransport(app=app)
      async with httpx.AsyncClient(transport=transport, base_url="http://test") as c:
          yield c

  async def test_create_project(client, auth_headers):
      r = await client.post("/api/projects", json={"name": "Demo"}, headers=auth_headers)
      assert r.status_code == 201
      assert r.json()["name"] == "Demo"
  ```
  Эндпоинты для покрытия:
  - `POST /api/auth/register`, `POST /api/auth/login`;
  - `GET/POST/PATCH/DELETE /api/projects`, `/api/projects/{id}/files`;
  - `POST /api/render` (DSL → SVG);
  - `POST /api/share`, `GET /api/share/{slug}`;
  - негативные кейсы: 401 без токена, 403 для чужого проекта, 422 при невалидном теле.
- **Тестовая БД** (вопрос 23):
  - отдельный `TEST_DATABASE_URL` (PostgreSQL в Docker или SQLite in-memory);
  - таблицы создаются в фикстуре уровня сессии, а каждый тест выполняется в транзакции с откатом;
  - get-db подменяется через `app.dependency_overrides[get_db]`.
- **Покрытие:** `pytest --cov=app --cov-report=html --cov-fail-under=80`. Это аналог `coverageThreshold`.

### E2E (Playwright)

Playwright запускает фронтенд (`vite preview`) и бэкенд (`uvicorn`) через `webServer` в `playwright.config`. Можно писать на TS (`@playwright/test`) или на Python (`pytest-playwright`).

**Happy-path сценарии:**
1. Регистрация → вход → создание проекта → создание файла `.mmd` → ввод `graph TD; A-->B` → в превью появился `<svg>`.
2. «Поделиться» → открыть ссылку в новом контексте без авторизации → диаграмма отображается read-only.
3. PWA: `context.setOffline(true)` → ранее открытый проект доступен из кэша.

### Если используется Node.js-шлюз (Jest + Supertest)

Если ради требований модуля 01 перед FastAPI будет стоять тонкий Express BFF (проксирование `/api/*`, агрегация, rate limiting, выдача share-страниц):
- `app.js` экспортирует `app` без `listen`, как требует Supertest;
- Jest unit-тесты: middleware проверки JWT, rate-limiter;
- Supertest integration-тесты: `request(app).get('/api/projects')`, а FastAPI мокается через `nock` или `jest.mock('axios')`;
- `jest.config.js` с `coverageThreshold` — один в один с примером лекции.

### CI (GitHub Actions)

Файл `.github/workflows/test.yml`, `on: [push, pull_request]`. Три job:
- **frontend:** `setup-node` → `npm ci` → `npm run lint` → `npm run test:coverage` → `npm run build`;
- **backend:** `setup-python` → `pip install -r requirements-dev.txt` → `pytest --cov --cov-fail-under=80` (service-контейнер `postgres`);
- **e2e** (зависит от первых двух): `npx playwright install --with-deps` → `npx playwright test`; при падении трейсы и видео загружаются как артефакты.

---

## Нечитаемые материалы

Нечитаемых файлов нет, весь текст извлечён. Одна оговорка:
- **02. Инструменты сборки и автоматизации.pdf.** Блоки «Пример конфигурации Vite», «Пример конфигурации Webpack» и таблица «Сравнение Vite, Webpack и Parcel» (стр. 2, 3, 5) **пусты в самом PDF**: я открыл его визуально, под заголовками пустое место. Вероятно, при экспорте потерялись изображения или код. Сравнение восстановлено по тексту лекции 02 (см. таблицу в теории), примеры конфигураций есть в лекции 03.
