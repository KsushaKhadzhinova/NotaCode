# Условия лабораторных ИТиВП (выжимка из `E:\ИТиВП`)

Дедлайнов в методичках нет, есть только критерий «своевременная сдача». Оформление — СТП 01–2024: A4, поля 30/15/20/20 мм, Times New Roman 14, интервал 1,0, абзацный отступ 1,25 см. Защита индивидуальная, по 10-балльной шкале, «защищено» — от 4 баллов. Без защищённых лаб к экзамену не допускают.

## Часть 1 (6 семестр, «Интернет-технологии»)

| ЛР | Тема | Ветка | Главное | Коммит |
|---|---|---|---|---|
| 1 | Сетевые протоколы, DevTools Network | — | Трафик сайта по теме; 3 запроса разных типов; ≥ 5 заголовков запроса и ≥ 5 ответа; заголовки кэша, cookie, безопасности; Timing/TTFB; Slow 3G; фильтры; экспорт HAR; 16 контрольных вопросов | — |
| 2 | Рабочее окружение | `main` | Node LTS, Git, VS Code (+Live Server, Auto Rename Tag, Prettier, GitLens, ES6 String HTML, Figma to Code), Figma; `mkdir css js assets images design`; `index.html`, `css/style.css`, `js/script.js`; `.gitignore`; `npm init -y`, `lodash`, `-D nodemon`; `.prettierrc` `{"semi":true,"singleQuote":true}`; `design/figma-link.txt`; мудборд, палитра, шрифты, прототип; `README.md` | `Initial commit: project structure` |
| 3 | Семантический HTML | `feature/semantic-markup` | ≥ 8 семантических тегов, h1–h6, figure/figcaption/time, alt, skip-link, БЭМ, Schema.org Microdata (вариант 13 — Person/Organization), W3C Validator, Rich Results Test, `design/designtokens.txt` | `feat: add semantic HTML markup with Schema.org for [тип проекта]` |
| 4 | Адаптивный CSS | `feature/responsive-layout` | `variables.css`, `base.css`, `header.css`, `main.css`, `footer.css`, `adaptive.css`, `style.css`; Grid areas + Flex; mobile-first 576/768/1200; бургер-меню | `feat: implement responsive layout according to Figma design` |
| 5 | Интерактивный JS | `feature/javascript-interactivity` | `js/components/`, `js/utils/storage.js`, `validation.js`; ≥ 3 типов событий, ≥ 2 компонентов; LocalStorage; валидация; делегирование | `feat: add JavaScript interactivity for [тип проекта]` |
| 6 | Async + API + storage | `feature/async-api-storage` | `js/api/apiService.js`, `config.js`, `js/storage/localStorage.js`, `js/utils/dataParser.js`; fetch + таймаут + AbortController; кэш, офлайн, индикатор загрузки; ≥ 2 доп. функций; ключ не коммитить | `feat: integrate external API, caching and localStorage for [тип проекта]` |
| 7 | React-компоненты | `feature/react-components` | Vite, react-router-dom; `components/{ui,layout,features}`, `pages`, `data`; ≥ 5 компонентов, props, моки; **без useState и useEffect** | — |
| 8 | Тестирование фронта | `feature/frontend-testing` | Jest + RTL + jest-dom + jsdom + user-event; `jest.config.js`, `setupTests.js`; 3–4 утилиты и 3 компонента; DevTools; Lighthouse до/после; порог покрытия 70/60 (в NotaCode — 100%) | `feat: add unit tests, component tests, and performance audit with Lighthouse` |

**Итоговая работа ч.1:** React SPA, 5–6 страниц, ≥ 4 маршрутов, брейкпоинты 320/768/1024, LocalStorage, ≥ 5 тестов, доп. функция (например, PWA), Lighthouse ≥ 90 во всех категориях, деплой на Netlify, Vercel или GitHub Pages.

**ИПР №1:** прототип Figma (3 брейкпоинта) + дизайн-система, ≥ 3 тестов в `tests/`, `TZ.md`, ветка `feature/ipr1-prototype`.

**КР №1:** та же вёрстка на чистых HTML/CSS/JS, ветка `feature/kr1-implementation`, Lighthouse accessibility > 90.

## Часть 2 (7 семестр, «Веб-программирование»)

Стек задан жёстко: Node.js + Express + PostgreSQL/Sequelize, MongoDB/Mongoose, Socket.IO, Docker. FastAPI есть только в теории модуля 03. Отчёт сдаётся в PDF плюс ссылка на GitHub с README.

| Работа | Ветка | Требования |
|---|---|---|
| ЛР1 Express CRUD | `lab21` | `server.js`, `express.json()`, порт 3000; GET/POST/PUT/DELETE по сущности темы; данные в памяти; 404/400 в JSON; глобальный обработчик ошибок; `routes/`, `controllers/`, `models/`; Postman |
| ЛР2 PostgreSQL + Sequelize | `lab12` | `sequelize pg pg-hstore`, `-D sequelize-cli`, `init`, `use_env_variable: DATABASE_URL`; `model:generate`; CRUD через findAll/findByPk/create/update/destroy; миграция addColumn; сид; схема БД в отчёте |
| ЛР3 JWT + bcrypt | `23` | User (email unique, passwordHash); `/auth/register` (201), `/auth/login` (JWT 1h); middleware Bearer; `GET /profile`; один доп. механизм из 16 (RBAC, refresh, блокировка, 2FA…) |
| ЛР4 React state | `24` | useState/useEffect, localStorage, фильтр/сортировка, доп. эффект (debounce, title…) |
| ЛР5 React ↔ сервер | `25` | axios, `VITE_API_URL`, интерцептор Bearer, loading/error/retry, оптимистичные обновления с откатом, одно улучшение (пагинация, AbortController…), CORS; папки `server/`, `client/` |
| ЛР6 MongoDB | `26` | Mongoose, `MONGO_URI`; вложенные документы или массивы; CRUD; эндпоинт изменения элемента массива; сравнение с реляционной моделью |
| ЛР7 WebSocket | `27` | Socket.IO сервер и клиент в React; сообщения, connect/disconnect; доп. функции (комнаты, typing, история…) |
| ЛР8 Docker | `28` | backend `node:18-alpine`; frontend multi-stage → nginx; compose с postgres/mongo и томом; одно доп. требование (healthcheck, reverse proxy, Redis…); указать зеркало Docker Hub |
| ПЗ1 EJS | `pz21` | views: layout/index/item/add/404/500; middleware логирования и `?auth=1` |
| ПЗ2 REST + Swagger | `pz22` | ≥ 3 ресурсов, CRUD + PATCH, валидация, `API.md` или Swagger, ≥ 5 скриншотов Postman |
| ПЗ3 React Router v6 | `pz23` | `/`, `/catalog`, `/catalog/:id`, `/about`, `/login`, `/dashboard` (приватный), `*` |
| ПЗ4 Context + useReducer | `pz24` | показать prop drilling; ThemeContext (тема, язык) + бизнес-контекст на useReducer; localStorage |

**Курсовая:** 20–30 страниц без приложений.
- Введение.
- Глава 1: анализ, Use Case, архитектура/C4, эндпоинты, ER на 4–6 сущностей.
- Глава 2: стек, backend, frontend, интеграция, Docker, деплой, тесты.
- Глава 3: руководство пользователя, контрольные примеры.
- Заключение, 15–20 источников, приложения А–Г.

Рекомендованный backend — Node.js/Express. Python-сервисы нужно согласовать с преподавателем и обосновать в разделе 2.1 записки.

## Вариант

Тема проекта — NotaCode (web-сервис визуального моделирования на основе собственного DSL).
