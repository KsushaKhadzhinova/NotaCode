# Модуль 05. Деплой и DevOps

Контейнеризация (Docker, Docker Compose, Kubernetes), CI/CD (GitHub Actions, Jenkins), облачные модели и платформы, хостинг фронтенда, домены и SSL; лабораторная №8 — контейнеризация курсового проекта.

## Источники

| № | Файл | Статус |
|---|------|--------|
| 01 | Контейнеризация (Docker, Kubernetes) | прочитан полностью |
| 02 | Оркестрация контейнеров (Docker Compose) | прочитан полностью |
| 03 | CI/CD (Непрерывная интеграция / Непрерывный деплоймент) | прочитан полностью |
| 04 | GitHub Actions | прочитан; пример `ci.yml` в PDF представлен изображением, в тексте отсутствует |
| 05 | Jenkins | прочитан (текст и PDF); картинки с командами установки и примером Jenkinsfile в самом PDF битые |
| 06 | Виртуализация | прочитан полностью |
| 07 | Облачные технологии и подходы | прочитан полностью |
| 08 | Модели развертывания облачных сервисов | прочитан полностью |
| 09 | Модели облачных услуг (IaaS, PaaS, SaaS) | прочитан; сравнительная таблица представлена изображением |
| 10 | Создание современного корпоративного портала (Microsoft 365) | прочитан полностью |
| 11 | SOA (Service-Oriented Architecture) | прочитан; схема SOA представлена изображением |
| 12 | Облачные платформы AWS, Google Cloud, Azure, Firebase | прочитан; таблица сравнения AWS/GCP/Azure представлена изображением |
| 13 | Облачные решения Vercel, Heroku, GitHub Pages | прочитан полностью |
| 14 | GitHub Pages | прочитан; листинги git-команд и примеры HTML/CSS представлены изображениями |
| 15 | Настройка доменов и SSL | прочитан полностью |
| 16 | Вопросы для самоподготовки (30 вопросов) | прочитан полностью |
| 17 | Лабораторная работа №8 «Контейнеризация приложения с Docker» | прочитан полностью |
| — | Генератор вариантов .html | **нечитаем**: сохранённая страница ошибки Moodle (`invalidrecord`), данных вариантов нет |

---

## Ключевая теория

### 1. Контейнеризация и Docker (лекция 01)

**Контейнеризация** — лёгкий вид виртуализации: приложение упаковывается вместе с зависимостями в изолированный контейнер, но все контейнеры **используют ядро хостовой ОС**.

- Плюсы: переносимость (одинаково работает везде), изоляция, быстрый старт, мало ресурсов, лёгкая миграция локально → облако, масштабируемость.
- Минусы: изоляция слабее, чем у ВМ; зависимость от ОС хоста (Linux-контейнеры требуют Linux-ядра).

**Компоненты Docker:** Docker Engine (демон, запускающий контейнеры), Image (неизменяемый шаблон), Container (запущенный экземпляр образа), Docker Hub (реестр образов).
Применение: микросервисы, CI/CD, изолированные тестовые окружения.

**Образ и контейнер.** Образ — read-only набор слоёв, хранится в реестре. Контейнер — запущенный экземпляр образа с тонким записываемым слоем; при удалении контейнера этот слой теряется, если данные не вынесены в volume.

#### Инструкции Dockerfile

| Инструкция | Назначение |
|---|---|
| `FROM` | базовый образ (`alpine`/`slim` — минимальные) |
| `WORKDIR` | рабочая директория внутри образа (создаёт при отсутствии) |
| `COPY` | копирует файлы из контекста сборки в образ |
| `RUN` | команда **во время сборки** (установка зависимостей); каждый RUN = слой |
| `EXPOSE` | документирует порт; **сам по себе не публикует** его |
| `CMD` | команда по умолчанию при запуске контейнера; легко переопределить аргументами `docker run` |
| `ENTRYPOINT` | «исполняемый файл» контейнера; переопределяется только `--entrypoint`; аргументы `CMD` передаются ему как параметры |
| `USER` | пользователь, от которого работает процесс (не root — безопаснее) |

Идиома кэширования слоёв: сначала копировать только манифест зависимостей (`package*.json` / `requirements.txt`), установить зависимости, и лишь затем `COPY . .` — при изменении кода слой зависимостей берётся из кэша.

Примеры из лекции (в сжатом виде):
- Node.js: `node:18-alpine` → `npm ci --only=production` → `CMD ["node","server.js"]`.
- **Python/FastAPI**: `python:3.11-slim` → `pip install --no-cache-dir -r requirements.txt` → `CMD ["uvicorn","main:app","--host","0.0.0.0","--port","8000"]`.
- React + Nginx (multi-stage): этап `builder` на `node:18-alpine` выполняет `npm run build`, финальный этап `nginx:alpine` получает только `/app/dist` через `COPY --from=builder`, плюс свой `nginx.conf`.

#### Команды Docker CLI

```bash
docker build -t my-app:1.0 .          # сборка; -t имя:тег, . — контекст
docker images                          # список образов
docker run -d -p 8080:3000 --name c1 my-app:1.0   # -d фон, -p хост:контейнер
docker ps / docker ps -a               # запущенные / все контейнеры
docker stop|start|rm c1                # управление
docker rmi my-app                      # удалить образ
docker logs -f c1                      # логи (follow)
docker exec -it c1 sh                  # shell внутри (bash, если есть)
docker system prune -a                 # удалить неиспользуемое
docker save -o app.tar my-app / docker load -i app.tar
```

#### Docker Hub и реестры
`docker login` → `docker tag my-app:1.0 user/my-app:1.0` → `docker push user/my-app:1.0` → на другом сервере `docker pull` + `docker run`. Публичные образы бесплатны, приватные — по подписке. Возможна автосборка при push в GitHub (Docker Hub Builds).
Для другого реестра имя образа включает хост: `registry.gitlab.com/<group>/<project>/<image>:tag` (`docker login registry.gitlab.com`, затем tag/push).

#### Оптимизация образов
1. Базовый образ `alpine`/`slim` (node:latest ≈1 ГБ, node:18-alpine ≈170 МБ, node:18-slim ≈220 МБ).
2. Объединять команды в один `RUN` через `&&` (меньше слоёв).
3. Удалять временные файлы в **том же** слое (`rm -rf /var/lib/apt/lists/*`).
4. `.dockerignore` (node_modules, .git, логи, tmp); копировать манифест зависимостей отдельно.
5. Multi-stage build — в финальный образ попадает только артефакт (пример с Go-бинарником на alpine).
6. Без кэша менеджеров пакетов: `pip --no-cache-dir`, `npm ci --only=production`.
7. Анализ слоёв: `dive`, `docker-slim`.
8. Оптимизированный Node-образ: стадия `deps` с prod-зависимостями → финальная стадия копирует `node_modules`, `USER node`.

#### Kubernetes (обзор)
Система оркестрации контейнеров в кластере: размещение по узлам, автомасштабирование, самовосстановление (перезапуск упавших), поддержание желаемого состояния.
Компоненты: **Pod** (минимальная единица, 1+ контейнеров), **Node** (рабочий узел), **Master/Control plane** (управление кластером), **Service** (стабильный доступ к группе подов), **ConfigMap/Secret** (конфигурация и секреты).

### 2. Оркестрация: Docker Compose (лекция 02)

Compose описывает многоконтейнерное приложение в одном YAML-файле и поднимает его одной командой. Предназначен для локальной разработки, тестов, CI и небольших продакшенов на **одном хосте**; кластерная оркестрация — Kubernetes (или Docker Swarm).

Версии: `docker-compose` (v1, отдельный бинарник) и `docker compose` (v2, плагин). Поле `version:` ('3.8'/'3.9') в современном Compose игнорируется, но в курсе используется.

| Команда | Действие |
|---|---|
| `up` (`-d`, `--build`, `--scale svc=N`) | запуск всех сервисов |
| `down` / `down -v` | остановить и удалить контейнеры и сети / плюс тома |
| `logs [svc]`, `ps`, `exec <svc> <cmd>` | логи, список, команда внутри |
| `build`, `pull` | пересобрать / обновить образы |
| `--profile cache up` | запустить сервисы из профиля |

**Разделы файла:** `services`, `networks`, `volumes`.
**Поля сервиса:** `build` (путь или `{context, dockerfile}`), `image`, `ports: "хост:контейнер"`, `environment`, `env_file`, `depends_on`, `volumes`, `networks`, `container_name`, `restart`, `healthcheck`, `profiles`.

**Сети.** По умолчанию Compose создаёт общую сеть проекта; сервисы видят друг друга по **DNS-имени сервиса** (`postgresql://user:pass@db:5432/mydb`). Пользовательские сети — для изоляции (пример: `web` в сетях `frontend`+`backend`, `db` только в `backend`). Также `external: true` (существующая сеть другого проекта) и `ipam` для своей подсети.

**Тома.**
- Именованные (`pgdata:/var/lib/postgresql/data`) — хранятся в `/var/lib/docker/volumes`, для прод-данных.
- Bind mount (`./backend:/app`) — для dev (live reload); анонимный том `/app/node_modules` защищает зависимости образа от перезаписи.
- Без тома данные БД живут в слое контейнера: переживают `stop/start`, но **теряются при удалении/пересоздании** контейнера (`down`, `up --build` с изменениями).

**Переменные.** Файл `.env` рядом с compose-файлом автоматически подставляется в `${VAR}`; `env_file: .env` передаёт переменные внутрь контейнера.

**Порядок запуска.** `depends_on` задаёт только порядок старта, не ждёт готовности. Для ожидания: `healthcheck` у БД (`pg_isready -U user`, interval/timeout/retries) + `depends_on: db: condition: service_healthy`.

**Масштабирование.** `docker compose up --scale backend=3`. Ограничения: нельзя жёстко пробрасывать один хост-порт для всех реплик (конфликт порта) и задавать `container_name`; нужен балансировщик (nginx) перед репликами; всё на одном хосте; нет автомасштабирования.

**Полный стек из лекции:** nginx (reverse-proxy, порт 80) → frontend + backend → db (postgres с healthcheck) + redis, одна сеть `webnet`, том `db_data`.

**Compose vs Kubernetes:**

| | Compose | Kubernetes |
|---|---|---|
| Назначение | dev, тесты, простой прод | промышленная оркестрация |
| Масштабирование | `--scale`, вручную | HPA, автоматическое |
| Отказоустойчивость | только `restart:` | ReplicaSet, самовосстановление |
| Узлы | один хост | кластер |
| Сети | bridge | CNI, service mesh |

### 3. CI/CD и GitHub Actions (лекции 03–04)

- **CI** — частые слияния в общий репозиторий, на каждое — автоматические сборка, линт (ESLint/Prettier), юнит/интеграционные тесты (Jest, Mocha, PyTest). Раннее обнаружение ошибок, main всегда «зелёный».
- **Continuous Delivery** — артефакт всегда готов к релизу, выкладка в прод по ручному подтверждению.
- **Continuous Deployment** — каждое успешное изменение автоматически уходит в прод.
- Типовой пайплайн: checkout → установка зависимостей → lint → test (с coverage) → build → (docker build/push) → deploy.

**Понятия GitHub Actions:** Workflow (YAML в `.github/workflows/`), Event (`push`, `pull_request`, `schedule` (cron), `workflow_dispatch` (ручной, с `inputs`), релизы, issues), Job (выполняется на одном runner; параллельно или последовательно через `needs`), Step (`run:` — shell, `uses:` — готовое action), Runner (GitHub-hosted Ubuntu/Windows/macOS или self-hosted), Action (переиспользуемый шаг).

Структура: `name`, `on`, `jobs.<id>.runs-on`, `steps[].uses/with/run/env`, `needs`, `if`, `environment`, `permissions`, `strategy.matrix`.

Приёмы:
- Кэш зависимостей: `actions/setup-node@v4` с `cache: 'npm'` (для Python — `actions/setup-python@v5` с `cache: 'pip'`).
- Matrix: несколько версий/ОС.
- Условия: `if: github.ref == 'refs/heads/main'`, `if: startsWith(github.ref, 'refs/tags/v')`, `github.event_name == 'push'`.
- **Секреты**: Settings → Secrets and variables → Actions → New repository secret; в YAML `${{ secrets.NAME }}`. Никогда не хранить токены в коде.
- Docker в CI: `docker/login-action@v3` + `docker/build-push-action@v5` (`push: true`, `tags:`), job `docker` с `needs: test`.
- GitHub Pages (современный способ): `permissions: pages: write, id-token: write`; `actions/upload-pages-artifact@v3` (path `./dist`) → job `deploy` с `environment: github-pages` и `actions/deploy-pages@v4`; в Settings → Pages → Source: GitHub Actions.
- Vercel через Actions: `amondnet/vercel-action@v20` (VERCEL_TOKEN, ORG_ID, PROJECT_ID, `--prod`); Netlify: `nwtgck/actions-netlify@v3`.

Командная работа: обязательные проверки PR (линт, тесты), уведомления (Slack, e-mail), параллельные job'ы, прозрачный статус в UI GitHub.

### 4. Jenkins (лекция 05)

Self-hosted CI/CD сервер с огромной экосистемой плагинов.
- **Master (controller)** — управляет задачами, агентами, историей; **Agent** — машины/контейнеры, выполняющие сборки; **Jenkinsfile** — pipeline-as-code в корне репозитория.
- Установка: Docker-образ (`jenkins/jenkins:lts`, порт 8080) или пакеты Linux; первый вход на `http://localhost:8080` с ключом разблокировки из логов; установка плагинов (Git, Node.js, Docker).
- Типы проектов: Freestyle и Pipeline. SCM: Git + URL + креденшалы.
- Шаги для фронтенда: `npm install` → `npm run build` → `npm test`; уведомления, отчёты тестов, мультибранч-пайплайны, деплой на сервер/облако.
- Типовой декларативный Jenkinsfile (в PDF картинка битая; общий вид): `pipeline { agent any; stages { stage('Install'){ steps { sh 'npm ci' } } stage('Test'){...} stage('Build'){...} } }`.

### 5. Виртуализация, облака, модели (лекции 06–09)

**Виртуализация**: гипервизор (VMware, Hyper-V, Xen, KVM) делит CPU/RAM/диск между ВМ; каждая ВМ со своей гостевой ОС. Плюсы: сильная изоляция, разные ОС на одном железе. Минусы: большой расход ресурсов, медленный старт (загрузка ОС). Контейнер vs ВМ: общий kernel vs собственная ОС; мегабайты vs гигабайты; секунды vs минуты.

**Облачные вычисления** — аренда вычислений, хранения, сетей через интернет с оплатой по факту (pay-as-you-go). Применения: хранилища (S3, GCS, Blob), Big Data (EMR, BigQuery, Data Lake), ML (SageMaker, AI Platform, Azure ML), хостинг приложений (Elastic Beanstalk, App Engine, App Service), Kubernetes (EKS, GKE, AKS).
Ключевые подходы: **виртуализация**, **автоматизация**, **масштабируемость** (горизонтальная — больше экземпляров; вертикальная — больше ресурсов одному), **оркестрация** (Kubernetes, Swarm, Mesos).

**Модели развертывания:** частное (одна организация, контроль и безопасность), публичное (AWS/GCP/Azure, общие ресурсы), общее/community (группа организаций со сходными требованиями), гибридное (частное + публичное, напр. Azure Hybrid).

**Модели услуг:**

| Модель | Провайдер отвечает | Пользователь отвечает | Примеры | Плюсы / минусы |
|---|---|---|---|---|
| IaaS | железо, виртуализация, сеть | ОС, runtime, приложение | EC2, Compute Engine, Azure VM | гибкость, контроль / нужны навыки админа |
| PaaS | + ОС, runtime, платформа | код и данные | Elastic Beanstalk, App Engine, App Service, Heroku | быстро, автоскейл / ограничения, vendor lock-in |
| SaaS | всё | только использование | Gmail, Office 365, Dropbox | не нужна установка / мало контроля, риски данных |

### 6. Корпоративный портал на Microsoft 365 (лекция 10)

Портал — экосистема облачных сервисов (PaaS/SaaS, инфраструктура у Microsoft):
- **Teams** — единая точка входа, каналы, чаты, встречи, Viva Connections как лента портала.
- **SharePoint Online** — Team Sites (автоматически для каждой команды Teams, хранят файлы каналов), Communication Sites (новости, политики), библиотеки документов и списки, веб-части. Файл, отправленный в Teams, хранится в SharePoint.
- **Power Platform** (low-code): Power Apps (формы/приложения), Power Automate (потоки между сервисами), Power BI (дашборды).
- **Поиск Microsoft 365** — единый, персонализированный, с учётом прав доступа.
- **Azure AD (Entra ID)** — SSO, MFA, Conditional Access.
- Инструменты: SharePoint Framework (SPFx, TypeScript/React), M365 Admin Center, PnP PowerShell.

### 7. SOA (лекция 11)

Приложение как набор взаимодействующих сервисов. Компоненты: **сервисы** (независимые поставщики функциональности), **потребители**, **реестр сервисов** (каталог для обнаружения). Топологии: одноранговая (прямые вызовы), централизованная (через **ESB** — шину сервисов: маршрутизация, трансформация сообщений), гибридная. Поставщики инфраструктуры: AWS, Azure, GCP, IBM Cloud, Oracle Cloud. Микросервисы — облегчённое развитие идей SOA без тяжёлой шины.

### 8. Облачные платформы (лекция 12)

| Платформа | Сильные стороны | Ключевые сервисы | Минусы |
|---|---|---|---|
| AWS | 200+ сервисов, глобальная сеть, Auto Scaling | EC2, S3, RDS, Redshift, Athena, Lambda, SageMaker | сложен для новичков, дорог без оптимизации |
| Google Cloud | ML/Big Data, Kubernetes | Compute/App Engine, GKE, BigQuery, Dataflow, Cloud SQL, Firestore | меньше сервисов, менее понятная документация |
| Azure | интеграция с Microsoft, гибрид (Azure Stack) | VM, Blob Storage, Functions, App Service, Synapse, Azure DevOps | неудобен вне экосистемы MS |
| Firebase | быстрый старт для web/mobile | Realtime DB, Firestore, Auth, Hosting, Functions, FCM, Analytics | слаб для классического бэкенда и корпоративных сценариев |

Прочие бесплатные варианты: IBM Cloud (free tier), Oracle Cloud (Always Free: 2 ВМ по 1 ГБ RAM, БД до 20 ГБ), Vercel, Netlify, Heroku (в лекции упомянут лимит часов, но см. ниже — бесплатный план отменён).

### 9. Хостинг и платформы деплоя (лекции 13–14)

**Типы хостинга:** shared (дёшево, мало контроля), VPS (root, гарантированные ресурсы), dedicated (весь сервер), cloud (кластер, автоскейл).
**Тренды:** PaaS, контейнеры и оркестрация (ECS, managed K8s), serverless (Lambda, Cloud Functions — оплата за время выполнения), **IaC** (Terraform, Ansible, Pulumi).
**Фронтенд-платформы:** Vercel/Netlify (деплой из Git, preview для PR, CDN, serverless-функции), GitHub/GitLab Pages, Cloudflare Pages, Render (в т.ч. бэкенд), Surge (CLI для статики).

Выбор: визитка → GitHub Pages/Vercel; магазин → VPS/облако; высоконагруженный API → облако + K8s; SPA → Vercel/Netlify/Cloudflare; функции → serverless/PaaS.

- **Vercel**: подключение репозитория → автоопределение фреймворка → Deploy → `project.vercel.app`; автодеплой на push, preview на каждый PR, SSL + CDN, serverless на Node/Go/Python/Ruby, env-переменные. Hobby: 200 проектов, 100 ГБ трафика/мес, 1 млн вызовов функций, 1 параллельная сборка, без карты.
- **Heroku**: классический PaaS; с конца 2022 бесплатных тарифов нет — Eco $5/мес (1000 dyno-часов, засыпание после 30 мин), Basic/Hobby $7/мес (24/7). Для учёбы рекомендуются Render или Fly.io.
- **GitHub Pages**: только статика, бесплатно, без лимитов build-минут, репозиторий до ~1 ГБ. Деплой из ветки (`main`/`gh-pages`, корень или `/docs`) или через GitHub Actions. Личный сайт — репозиторий `username.github.io`; проектный — `username.github.io/repo`. Для SPA-маршрутизации нужен трюк с `404.html` (или HashRouter). Для Vite под проектный сайт — `base: '/repo/'`. Свой домен через файл `CNAME`, «Enforce HTTPS».
- Производительность и безопасность хостинга: аптайм 99.9%, TTFB, CDN, кэширование, сжатие (WebP, минификация); SSL, защита от DDoS, файрвол, бэкапы, обновления.

### 10. Домены и SSL (лекция 15)

- Домен покупается у регистратора (Reg.ru, Nic.ru, Namecheap, Cloudflare Registrar), управляется через DNS-панель.
- DNS-записи: **A** (домен → IPv4), **AAAA** (IPv6), **CNAME** (псевдоним на другое имя), **TXT** (верификация). Распространение — от минут до 48 ч.
- GitHub Pages: A-записи apex-домена на 185.199.108.153 / .109 / .110 / .111; `www` — CNAME на `username.github.io`; файл CNAME; Enforce HTTPS.
- Vercel: Settings → Domains → Add; CNAME на `cname.vercel-dns.com` (для apex обычно A-запись/ALIAS по подсказке Vercel); SSL автоматически за минуты.
- Render: CNAME на `*.onrender.com`; Heroku: CNAME на `app.herokuapp.com`, ACM на платных.
- **SSL/TLS**: шифрование канала браузер ↔ сервер, https. Без него браузер пишет «Не защищено», падает SEO, **не работают Service Workers** (важно для PWA) и Geolocation. Let's Encrypt — бесплатно, 90 дней, автопродление (на VPS — certbot); платные (EV) — $10–200/год. Проверка — замок в браузере, SSL Labs. Редирект HTTP→HTTPS: тумблер на платформе, на VPS — в Nginx.
- Диагностика: `dig`, `nslookup`, `curl -I`. Типовые проблемы: DNS не распространился; сертификат не выдан (неверный CNAME); нет принудительного HTTPS; старые A-записи.

---

## Лабораторные работы

### Лабораторная работа №8. «Контейнеризация приложения с Docker»

- **Ветка репозитория:** `28` (код обязательно в ветке `28`).
- **Срок:** срок не указан в материалах.
- **Выполнение:** индивидуальное. Вариант темы — по «Генератору вариантов» (страница в материалах нечитаема: ошибка Moodle). По содержанию лаба общая для всех вариантов: порядок выполнения один, различается только проект и выбор БД/доп. требования.

#### Цель
Освоить Docker: Dockerfile для backend и frontend, сборка образов, запуск контейнеров, многоконтейнерное приложение через Docker Compose для упаковки и развёртывания **курсового проекта**.

#### Необходимые знания
Образ/контейнер/Docker Hub; инструкции FROM, WORKDIR, COPY, RUN, EXPOSE, CMD; сети и тома; docker-compose.yml; альтернативные реестры (GitLab Container Registry, зеркала).

#### Особенность для РБ (обязательно отразить в отчёте)
Docker Hub может быть недоступен. Варианты:
- зеркала в `/etc/docker/daemon.json` (в Docker Desktop — Settings → Docker Engine):
  ```json
  { "registry-mirrors": ["https://mirror.gcr.io", "https://dockerhub.timeweb.cloud"] }
  ```
- GitLab Container Registry (`registry.gitlab.com`), если репозиторий в GitLab;
- альтернативные реестры для postgres, mongodb, node, nginx (например, quay.io).
В отчёте указать, **какие меры приняты** для доступа к образам.

#### Задачи
1. Dockerfile для **Node.js/Express** backend курсового проекта.
2. Dockerfile для **React** frontend с **многоэтапной** сборкой.
3. `docker-compose.yml`: backend + frontend + БД (PostgreSQL или MongoDB — по выбору).
4. Сборка и запуск: `docker-compose up --build`.
5. Проверка работы в контейнерах.
6. Минимум **одно** дополнительное требование.

#### Порядок выполнения
1. **Dockerfile backend** (эталон): `FROM node:18-alpine`, `WORKDIR /app`, `COPY package*.json ./`, `RUN npm ci --only=production`, `COPY . .`, `EXPOSE 5000`, `CMD ["node","server.js"]`.
2. **Dockerfile frontend** (multi-stage): `node:18-alpine AS builder` → `npm ci` → `npm run build`; затем `nginx:alpine`, `COPY --from=builder /app/dist /usr/share/nginx/html`, `EXPOSE 80`, `CMD ["nginx","-g","daemon off;"]`.
3. **docker-compose.yml** (эталон для PostgreSQL): `version: '3.8'`; `backend` (`build: ./backend`, `ports "5000:5000"`, `DB_URL=postgresql://user:pass@db:5432/mydb`, `depends_on: db`); `frontend` (`build: ./frontend`, `ports "80:80"`); `db` (`postgres:15-alpine`, POSTGRES_USER/PASSWORD/DB, том `pgdata:/var/lib/postgresql/data`); `volumes: pgdata:`. Для MongoDB рекомендуются образы `mongo:6` / `mongo:7`.
4. **Запуск:** `docker-compose up --build`.
5. **Проверка:** `http://localhost` — фронтенд; `http://localhost:5000` — бэкенд; `docker ps` — список контейнеров.
6. **Доп. требования** (одно или несколько):
   - `.env` для строки подключения и секретов;
   - healthcheck backend по эндпоинту `/health`;
   - volume для логов backend (`/logs`);
   - pgAdmin / mongo-express;
   - Redis для кэширования;
   - изоляция сетей (внутренняя и внешняя);
   - Nginx reverse-proxy (фронтенд + прокси на бэкенд);
   - healthcheck Redis/БД с `condition: service_healthy` в `depends_on`.

#### Структура репозитория (ожидаемая)
```
repo/ (ветка 28)
├── backend/   Dockerfile, .dockerignore, исходники
├── frontend/  Dockerfile, .dockerignore, nginx.conf (опц.)
├── docker-compose.yml
├── .env.example  (если выбран .env)
└── README.md     — инструкция по запуску (требуется критериями)
```

#### Отчёт (PDF в СЭО)
Титульный лист, цель; кратко теория контейнеризации; листинги Dockerfile и docker-compose.yml с пояснениями; скриншот `docker ps`; скриншот приложения в браузере (`http://localhost`); описание выбранного доп. требования и реализации; меры по доступу к образам (зеркало/реестр); ответы на контрольные вопросы; выводы.

#### Сдача
PDF-отчёт в СЭО + ссылка на GitHub-репозиторий (ветка `28`).

#### Критерии оценки
1. Корректные Dockerfile для backend и frontend, **multi-stage для React**.
2. Корректный compose с БД и связью сервисов.
3. Тома для данных БД.
4. Выполнено доп. требование.
5. `docker-compose up --build` запускается без ошибок.
6. Полнота отчёта (скриншоты, листинги, ответы).
7. Ссылка на репозиторий с веткой 28 **и инструкцией по запуску**.

#### Контрольные вопросы (с краткими ответами)
1. Образ vs контейнер — шаблон read-only vs запущенный экземпляр с записываемым слоем.
2. `COPY --from=builder` — берёт артефакты из предыдущей стадии, в финальный образ не попадают инструменты сборки.
3. Связь по имени сервиса — встроенный DNS сети Compose (`db:5432`).
4. Volume — хранилище вне слоя контейнера; данные БД переживают пересоздание.
5. Логи — `docker logs -f <c>`, `docker compose logs <svc>`.
6. Уменьшение образа — alpine/slim, multi-stage, .dockerignore, объединение RUN, очистка кэша, только prod-зависимости.
7. Команда по умолчанию — `CMD`.
8. CMD vs ENTRYPOINT — CMD легко переопределяется; ENTRYPOINT фиксирует исполняемый файл, CMD становится его аргументами.
9. Порядок запуска — `depends_on` (+ healthcheck и `condition: service_healthy`).
10. Команда в контейнере — `docker exec -it <c> sh` / `docker compose exec <svc> sh`.
11. Проброс порта — `ports: ["хост:контейнер"]`.
12. `.env` — автоподстановка `${VAR}` и/или `env_file:`.
13. Healthcheck — периодическая проверка готовности; статус healthy/unhealthy, используется в depends_on и оркестраторах.
14. Публикация в GitLab Registry — `docker login registry.gitlab.com`, `docker tag img registry.gitlab.com/grp/proj/img:tag`, `docker push`.
15. Без volume — данные в слое контейнера: переживают restart, но теряются при удалении/пересоздании.
16. Масштабирование — `docker compose up --scale backend=3`; нельзя фиксированный хост-порт и `container_name`, нужен балансировщик, один хост.

---

## Вопросы для самоподготовки

**Docker:** 1) контейнеризация vs ВМ; 2) образ vs контейнер; 3) минимальный Dockerfile Node.js; 4) FROM/WORKDIR/COPY/RUN/EXPOSE/CMD; 5) build/run, проброс порта; 6) Docker Hub, tag/push; 7) оптимизация образов (alpine, multi-stage, .dockerignore, объединение RUN); 8) ps/stop/rm/logs/exec.
**Compose:** 9) зачем Compose и чем отличается от одиночного Docker; 10) структура файла (version, services, networks, volumes); 11) фрагмент backend + PostgreSQL (порты, env, depends_on); 12) связь по имени сервиса, пользовательские сети; 13) тома, именованный том БД; 14) up -d/down/logs/exec; 15) `--scale backend=3` и ограничения; 16) Compose vs Kubernetes.
**CI/CD:** 17) CI и CD (Delivery/Deployment); 18) этапы пайплайна; 19) GitHub Actions, где лежит workflow и формат; 20) события push/pull_request/schedule/workflow_dispatch; 21) структура YAML (name, on, jobs, runs-on, steps, uses, run); 22) secrets; 23) тесты + сборка Docker-образа при push в main.
**Деплой:** 24) бесплатные платформы и ограничения; 25) React на GitHub Pages, ветка gh-pages, деплой через Actions; 26) Vercel (репозиторий, автодеплой, preview PR); 27) бэкенд-хостинг Node.js + БД (Render, Railway, Cyclic); 28) свой домен на Vercel/GitHub Pages (CNAME, A); 29) SSL, Let's Encrypt, HTTPS; 30) безопасная передача env-переменных при деплое.

---

## Как применить в проекте NotaCode

### Что требует Лаба 8 по стеку

| Пункт | Буквальное требование | Строгость |
|---|---|---|
| Backend | «Dockerfile для **Node.js/Express** приложения (backend вашего курсового проекта)» | Явно Node.js/Express в формулировке задачи; эталонный Dockerfile на `node:18-alpine` |
| Frontend | React, **многоэтапная** сборка (критерий оценки) | Строго: multi-stage обязателен |
| БД | PostgreSQL **или** MongoDB | Выбор студента |
| Compose | backend + frontend + БД, том для БД, связь сервисов | Строго |
| Доп. требование | ≥1 из списка | Строго |

**Разрешены ли замены?** Прямого разрешения заменить Node.js на Python в лабе нет. Однако:
- задача сформулирована как «backend **вашего курсового проекта**», а цель — упаковать именно курсовой проект;
- лекция 01 сама приводит **Dockerfile для Python (FastAPI)** с uvicorn — то есть FastAPI в контейнере признан в материалах курса;
- критерии оценки говорят о «корректных Dockerfile для backend и frontend» без указания языка.

**Рекомендация (безопасный вариант):** держать FastAPI основным бэкендом и добавить тонкий **Node.js/Express API-gateway (BFF)**, который тоже контейнеризуется. Тогда буквальное требование «Dockerfile для Node.js/Express» выполнено, а архитектура NotaCode не ломается. Заодно это закрывает доп. требование «изоляция сетей» (FastAPI и БД только во внутренней сети). Если преподаватель подтвердит, что достаточно FastAPI, gateway можно исключить профилем Compose (`profiles: ["gateway"]`). В отчёте явно обосновать: «backend курсового проекта — FastAPI; Node.js-шлюз выполняет роль Express-backend из задания».

### Предлагаемая структура репозитория (ветка `28`)

```
notacode/
├── frontend/          # React + Vite + PWA
│   ├── Dockerfile
│   ├── nginx.conf
│   └── .dockerignore
├── backend/           # FastAPI
│   ├── Dockerfile
│   ├── requirements.txt
│   └── .dockerignore
├── gateway/           # Node.js/Express BFF (для буквального требования лабы)
│   ├── Dockerfile
│   ├── server.js
│   └── .dockerignore
├── docker-compose.yml
├── .env.example
├── .github/workflows/ci.yml
└── README.md          # как запустить: cp .env.example .env && docker compose up --build
```

### frontend/Dockerfile (React + Vite → nginx, multi-stage)

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
ARG VITE_API_URL=/api
ENV VITE_API_URL=$VITE_API_URL
RUN npm run build            # Vite кладёт сборку в dist/

FROM nginx:alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Замечание: переменные `VITE_*` вшиваются **на этапе сборки**, поэтому передаются через `ARG`, а не через `environment` в runtime. Лучше использовать относительный `/api` и проксировать через nginx — тогда нет CORS и не нужно пересобирать образ под другой хост.

### frontend/nginx.conf (SPA + reverse-proxy + PWA)

```nginx
server {
  listen 80;
  root /usr/share/nginx/html;
  index index.html;

  # API: через gateway (или сразу на backend:8000, если gateway не используется)
  location /api/ {
    proxy_pass http://gateway:3000/;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  # Service Worker и манифест не кэшировать, иначе PWA не обновится
  location = /sw.js              { add_header Cache-Control "no-cache"; }
  location = /manifest.webmanifest { add_header Cache-Control "no-cache"; }

  # хэшированные ассеты Vite — долгий кэш
  location /assets/ { add_header Cache-Control "public, max-age=31536000, immutable"; }

  # SPA fallback для клиентского роутинга (/projects/123 и т.п.)
  location / { try_files $uri $uri/ /index.html; }
}
```

### backend/Dockerfile (FastAPI + uvicorn)

```dockerfile
FROM python:3.12-slim
ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1
WORKDIR /app
RUN apt-get update && apt-get install -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/*          # curl для healthcheck
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
RUN useradd -m app && mkdir -p /app/logs && chown -R app /app
USER app
EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Миграции (Alembic) запускаются отдельной командой: `docker compose run --rm backend alembic upgrade head` или entrypoint-скриптом. Эндпоинт `GET /health` в FastAPI: возвращает `{"status":"ok"}` и проверяет соединение с БД (`SELECT 1`).

Если рендер диаграмм делается на сервере (например, PlantUML через Java или mermaid-cli через Node/Chromium), лучше вынести рендерер в **отдельный сервис** compose (образ `plantuml/plantuml-server` и т.п.), а не раздувать образ FastAPI — это же хороший пример multi-service архитектуры для отчёта.

### gateway/Dockerfile (Node.js/Express, как в эталоне лабы)

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev          # современный аналог --only=production
COPY . .
EXPOSE 3000
USER node
CMD ["node", "server.js"]
```

`server.js` — минимум: `express` + `http-proxy-middleware` проксирует `/` на `http://backend:8000`, плюс собственные `GET /health`, логирование запросов (morgan), rate limit (`express-rate-limit`) и, например, агрегирующий эндпоинт `GET /dashboard` (проекты + последние файлы пользователя за один запрос). Так у gateway есть смысловая роль, а не только формальная.

### docker-compose.yml (PostgreSQL, healthchecks, сети, .env)

```yaml
services:
  frontend:
    build: { context: ./frontend, args: { VITE_API_URL: /api } }
    ports: ["80:80"]
    depends_on: [gateway]
    networks: [public]

  gateway:
    build: ./gateway
    environment:
      BACKEND_URL: http://backend:8000
    depends_on:
      backend: { condition: service_healthy }
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:3000/health"]
      interval: 10s
      retries: 5
    networks: [public, internal]

  backend:
    build: ./backend
    env_file: .env                     # DATABASE_URL, JWT_SECRET, ...
    ports: ["8000:8000"]               # для проверки из лабы; в проде убрать
    volumes: ["backend_logs:/app/logs"]
    depends_on:
      db: { condition: service_healthy }
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks: [internal]

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes: ["pgdata:/var/lib/postgresql/data"]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks: [internal]

  pgadmin:
    image: dpage/pgadmin4
    profiles: ["tools"]                # docker compose --profile tools up
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_EMAIL}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_PASSWORD}
    ports: ["5050:80"]
    networks: [internal]

networks:
  public:
  internal:
    internal: true                     # БД и backend не видны снаружи

volumes:
  pgdata:
  backend_logs:
```

Замечание: сеть с `internal: true` не публикует порты наружу, поэтому для проверки `http://localhost:8000` из лабы либо временно подключите backend и к сети `public`, либо проверяйте его через gateway (`http://localhost/api/health`). В отчёте опишите это как осознанную изоляцию.

`.env.example` (коммитится; `.env` — в `.gitignore`):
```
POSTGRES_USER=notacode
POSTGRES_PASSWORD=change_me
POSTGRES_DB=notacode
DATABASE_URL=postgresql+asyncpg://notacode:change_me@db:5432/notacode
JWT_SECRET=change_me
PGADMIN_EMAIL=admin@example.com
PGADMIN_PASSWORD=change_me
```

Эта конфигурация закрывает сразу несколько доп. требований: `.env`, healthcheck backend `/health`, volume для логов, pgAdmin, изоляция сетей, Nginx reverse-proxy, healthcheck БД с `service_healthy`. Опционально — Redis для кэша отрендеренных SVG (ключ = хэш исходника DSL + тема).

`.dockerignore` (frontend/gateway): `node_modules`, `dist`, `.git`, `*.log`, `.env`. Для backend: `__pycache__`, `.venv`, `.pytest_cache`, `.git`, `.env`.

### GitHub Actions CI (`.github/workflows/ci.yml`)

```yaml
name: CI
on:
  push: { branches: [main, "28"] }
  pull_request: { branches: [main] }

jobs:
  frontend:
    runs-on: ubuntu-latest
    defaults: { run: { working-directory: frontend } }
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm, cache-dependency-path: frontend/package-lock.json }
      - run: npm ci
      - run: npm run lint
      - run: npm test -- --run          # Vitest
      - run: npm run build

  backend:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env: { POSTGRES_USER: test, POSTGRES_PASSWORD: test, POSTGRES_DB: test }
        ports: ["5432:5432"]
        options: >-
          --health-cmd "pg_isready -U test" --health-interval 5s --health-retries 10
    defaults: { run: { working-directory: backend } }
    env:
      DATABASE_URL: postgresql+asyncpg://test:test@localhost:5432/test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.12", cache: pip }
      - run: pip install -r requirements.txt
      - run: ruff check .
      - run: pytest -q

  compose-smoke:
    needs: [frontend, backend]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: cp .env.example .env
      - run: docker compose up -d --build --wait    # ждёт healthy
      - run: curl -f http://localhost/ && curl -f http://localhost/api/health
      - if: always()
        run: docker compose logs && docker compose down -v

  images:
    needs: compose-smoke
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    permissions: { contents: read, packages: write }
    strategy:
      matrix: { service: [frontend, backend, gateway] }
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with: { registry: ghcr.io, username: "${{ github.actor }}", password: "${{ secrets.GITHUB_TOKEN }}" }
      - uses: docker/build-push-action@v5
        with:
          context: ./${{ matrix.service }}
          push: true
          tags: ghcr.io/${{ github.repository }}/${{ matrix.service }}:latest
```

GHCR (`ghcr.io`) — альтернативный реестр, удобный при ограничениях Docker Hub; логин через встроенный `GITHUB_TOKEN`, отдельных секретов не нужно.

### Как применить остальную теорию

- **Деплой фронтенда:** статическую сборку PWA можно выложить на Vercel (preview на каждый PR, SSL автоматически) или GitHub Pages (нужны `base: '/repo/'` в `vite.config`, fallback `404.html` или HashRouter; сложнее с PWA-scope). Vercel удобнее: `vercel.json` с rewrite `/api/*` на бэкенд.
- **Деплой бэкенда:** FastAPI + PostgreSQL — Render / Railway / Fly.io (Docker-образ из GHCR), managed Postgres (Neon, Supabase, Render PG). Heroku — только платно.
- **SSL обязателен для PWA:** Service Worker регистрируется только по HTTPS (кроме localhost). Для публичного демо нужен домен с Let's Encrypt (автоматически на Vercel/Render).
- **Домен:** например, `notacode.xyz` → фронтенд CNAME на `cname.vercel-dns.com`, `api.notacode.xyz` → CNAME на Render.
- **Секреты:** `JWT_SECRET`, `DATABASE_URL`, OAuth-ключи — в GitHub Secrets и env-переменных платформы, не в репозитории.
- **Модели облака:** NotaCode как продукт — **SaaS**; фронтенд на Vercel и бэкенд на Render — **PaaS**; VPS с docker compose — **IaaS**.
- **SOA/микросервисы:** frontend, gateway, api (FastAPI), renderer (PlantUML/Mermaid), db — пример сервис-ориентированного разделения; gateway играет роль единой точки входа.
- **Kubernetes:** для курсового не нужен; в отчёте можно упомянуть, что compose-сервисы 1:1 переводятся в Deployment + Service, а секреты — в K8s Secret.

---

## Нечитаемые материалы

- **«Генератор вариантов .html»** — сохранённая страница ошибки Moodle («Не удается найти данную запись…», код `invalidrecord`); данных о вариантах нет. Вариант темы узнать в СЭО или у преподавателя.
- Частично: в PDF лекций 04 (пример `ci.yml`), 05 (команды установки Jenkins, пример Jenkinsfile), 09 (таблица сравнения IaaS/PaaS/SaaS), 11 (схема SOA), 12 (таблица AWS/GCP/Azure), 14 (git-команды, пример сайта) иллюстрации и листинги были картинками или битыми изображениями; суть восстановлена по окружающему тексту.
