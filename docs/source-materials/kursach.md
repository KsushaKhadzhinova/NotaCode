# Курсач (рабочие заметки по курсовому проекту)

> **Источник:** `E:\Downloads\Курсач.docx`. Извлечено из `word/document.xml` (python zipfile) и конвертировано в Markdown без изменения содержания.
> **Примечания к конвертации:** стили «Title» → заголовки `##`, «heading 2» → `###`; нумерованные/маркированные абзацы Word → списки Markdown (уровень вложенности сохранён); таблицы → Markdown-таблицы (переносы внутри ячеек → `<br>`); встроенные изображения заменены пометкой `[Изображение: …]` (в документе одно изображение `word/media/image1.png`; сам файл не переносился, его содержимое кратко описано в пометке; 4 декоративные горизонтальные линии-разделители Word опущены). Артефакты ссылок из ответов ассистента (например, `mermaideditor+1`) сохранены как в исходнике. HTML-теги в тексте обёрнуты в обратные кавычки.

## Инструменты

| Инструмент | Лицензия / статус | Тип инструмента | Тип генерации | Вход (описание/код) | Выход (что генерирует) | Что именно можно воспроизвести в своём сервисе |
|---|---|---|---|---|---|---|
| PlantUML | GPL‑3.0, open source | Diagram as Code | DSL → диаграмма | Текстовый DSL PlantUML (описание диаграммы) | PNG/SVG/ASCII, UML/BPMN и другие диаграммы | Синтаксис DSL, правила парсинга, варианты запуска через CLI/HTTP‑сервер |
| Mermaid (JS) | MIT, open source | Diagram as Code (JS) | DSL → диаграмма | Markdown‑подобный текст Mermaid | SVG/HTML‑диаграммы (flowchart, sequence, ER, Gantt и др.) | Синтаксис диаграмм, JS‑API рендеринга (init, render), встройка в web‑просмотрщик |
| Graphviz (DOT) | Open source | Layout‑движок / Diagram as Code | DOT → граф | Текстовый язык DOT (узлы, рёбра, подграфы) | Графы с авто‑layout в PNG/SVG и других форматах | Формат DOT, алгоритмы layout (вызовы dot/neato), CLI/библиотечный интерфейс |
| D2 (d2lang) | MPL‑2.0, open source | Diagram as Code | DSL → диаграмма | Текстовый DSL D2 (узлы, связи, группы, стили) | Диаграммы с авто‑layout (SVG/PNG/HTML) | Описание языка, CLI‑интерфейс (d2 in.d2 out.svg), формат вывода, парсер/рендер |
| Kroki | Open source | HTTP‑агрегатор рендеринга | DSL разных движков → диаграмма | Текст PlantUML/Mermaid/Graphviz и др. + тип | Рендер диаграмм по HTTP (SVG/PNG/…) | HTTP‑API (эндпойнты, параметры), схему вызова внешних движков; аналогичный backend |
| Text2Diagram | Open source | AI text → diagram | Natural language → диаграмма | Натуральный язык (описание системы/процесса) | Диаграммы (UML/flowchart/mind‑map и др., зависит от модели) | Архитектуру пайплайна: LLM → внутреннее представление → генерация диаграммы |
| AI‑Diagram | MIT, open source | AI text → Graphviz | Text → DOT → диаграмма | Текстовое описание архитектуры/системы | Graphviz DOT + изображение диаграммы | Пайплайн: запрос к модели → генерация DOT‑текста → вызов Graphviz |
| Gleek (DSL) | Freemium (есть Free) | Diagram as Code (SaaS) | DSL → диаграмма в вебе | Собственный текстовый DSL для диаграмм | Диаграммы в веб‑редакторе (UML‑подобные, ER, flowchart) | Публичный синтаксис DSL, интерфейс «textarea → диаграмма»; свой рендер и фронтенд |

## Диаграммы

| Нотация / стандарт | Кто стандартизует и чем является | Документация и формализация требований |
|---|---|---|
| UML (Unified Modeling Language) | Стандарт OMG, дополнительно формализован как ISO/IEC 19505 (UML 2.x) | Формальная спецификация метамодели, синтаксиса и семантики всех 14 типов диаграмм; официальные спецификации UML 2.5/2.5.1 с жёсткими правилами построения моделей |
| UML Activity Diagrams (как часть UML) | Часть стандарта UML (OMG/ISO), отдельный раздел спецификации | Формально заданы как разновидность поведенческих диаграмм UML с чётким набором элементов и ограничений; используются как стандартизованная нотация бизнес‑процессов наряду с BPMN |
| BPMN 2.0 (Business Process Model and Notation) | Стандарт OMG BPMN 2.0, нормативный язык для бизнес‑процессов | Официальная спецификация BPMN 2.0 определяет полный перечень элементов, правила композиции, а также маппинг к языкам исполнения; существует большой корпус работ по формализации через сети Петри и другие модели |
| Petri Nets (сети Петри) | Формальный математический аппарат, де‑факто стандарт в научной литературе и ряде ISO/IEC‑подходов | Имеют строгое формальное определение (множества мест, переходов, дуг, маркировок) и свойства (живость, достижимость и т.п.); используются для формальной верификации моделей BPMN/UML и анализа процессов |
| IDEF0 / IDEF3 | Семейство стандартизованных методологий IDEF для моделирования функций и процессов | Имеют детализированные руководства по нотации, правилам декомпозиции и согласованности моделей; применяются как формальные методики описания процессов, в том числе в гос‑ и корпоративных стандартах |
| DFD (Data Flow Diagrams) в рамках структурного анализа / IDEF‑подходов | Используются как часть формальных нотаций структурного анализа (Structured Analysis, варианты IDEF/ГОСТ‑подходов) | Имеют устоявшийся набор элементов (процесс, хранилище, внешний объект, поток данных) и жёсткие ограничения на уровни декомпозиции и топологию диаграмм |
| ERD (Entity‑Relationship Diagrams) | Графическое представление формальной ER‑модели (Чен и последующие расширения), де‑факто стандарт моделирования данных в СУБД‑ и CASE‑инструментах | Основаны на формальной ER‑модели с чётко определёнными сущностями, атрибутами, связями и кардинальностями; имеют устойчивые правила трансляции в реляционные схемы и нормализации моделей данных |

## План

Предлагаю план уровня «от курсового до реального продукта», разбитый по этапам. Его можно свернуть до MVP или развернуть в диплом.

### 1. Определить рамки и требования

- Сформулировать минимальный функционал первой версии (MVP):
- текстовая консоль → превью диаграммы (1–2 нотации: например, UML Activity + BPMN),
- сохранение диаграмм в БД, загрузка, экспорт в SVG/PNG,
- простая история версий (хранить все ревизии текста DSL).mermaideditor+1
- Выбрать стек:
- backend: Python (FastAPI) или Node.js,
- frontend: React/Vue/чистый JS + Monaco/CodeMirror для редактора,
- хранение: PostgreSQL/SQLite (курсовой).
- Зафиксировать перечень нотаций первой версии и ссылаться на их стандарты (UML 2.x, BPMN 2.0, Petri).standards.iteh+3

### 2. Спроектировать DSL и внутреннюю модель

- Спроектировать внутреннюю метамодель:
- базовый граф (узел, ребро, атрибуты),
- профиль под BPMN (типы событий, задач, шлюзы),
- профиль под UML Activity и/или Petri (place/transition, токены).cmi+2
- Спроектировать собственный DSL:
- выбрать стиль (Mermaid‑подобный, YAML‑подобный, «блоки с фигурными скобками»),
- продумать ключевые слова, структуру файлов, комментарии,
- описать формальную грамматику (EBNF/ANTLR) в РПЗ.
- Добавить в модель понятие «workspace/diagram» как у Structurizr: один набор элементов — несколько разных представлений (видов диаграмм).structurizr+1

### 3. Реализовать ядро: парсер, валидация, layout

- Реализовать парсер DSL → внутренний граф:
- лексер/парсер (можно взять готовую библиотеку),
- обработка ошибок (позиция, подсказка, что не так).
- Реализовать валидатор нотаций:
- отдельные проверки для BPMN/UML/Petri (запрещённые связи, обязательные элементы).sciencedirect+2
- Реализовать layout и рендеринг:
- для начала использовать Graphviz (DOT) как layout‑движок: внутренний граф → DOT → SVG,
- затем постепенно добавлять собственный layout/рендер на стороне фронтенда (Canvas/SVG).[sciencedirect]​

### 4. Frontend: редактор + превью

- Собрать одностраничный UI «консоль + диаграмма»:
- слева редактор кода DSL (подсветка синтаксиса, базовый autocomplete),
- справа панель с SVG/Canvas‑диаграммой,
- кнопка Run / автообновление по debounce.
- Добавить UX‑фичи:
- шаблоны диаграмм (кнопки «пример BPMN», «пример UML»),
- масштабирование/перемещение диаграммы,
- подсветка элементов при наведении на код (и наоборот — по желанию).mermaidonline+1

### 5. Хранение, версияция и история запросов

- Спроектировать схему БД:
- таблица diagrams (id, owner, title, current_version_id, created_at),
- таблица diagram_versions (id, diagram_id, dsl_text, metadata, created_at),
- таблица requests (id, diagram_id, request_text, type: user/AI, created_at).esri+2
- Реализовать API:
- создать диаграмму,
- получить/обновить DSL,
- получить историю версий, откатиться к старой версии,
- экспортировать диаграмму (SVG/PNG).
- Добавить просмотр истории в UI:
- список версий,
- diff двух версий DSL (минимально: поблочный текстовый diff),
- кнопка «откатиться».

### 6. AI‑слой: текст/картинка → DSL (поэтапно)

- Версия 1: только текст → DSL (без картинок):
- сделать endpoint /ai/describe-to-dsl,
- продумать схему JSON‑описания (узлы, типы, связи) и маппинг JSON → DSL,
- подключить LLM (через готовый API) для генерации JSON по тексту, затем детерминированно сериализовать в DSL.mermaidonline+1
- Версия 2: улучшение качества:
- добавить валидацию и autocorrect: валидатор нотации исправляет мелкие нарушения (добавляет start/end, т.п.),
- логировать пары (описание → DSL → картинка) как датасет.
- Версия 3 (опционально для курсового, скорее для диплома): картинка → DSL:
- использовать VLM/модель распознавания диаграмм,
- шаги: детекция форм → построение графа → маппинг в нотацию → DSL.

### 7. Экспорт/импорт и интеграция с другими инструментами

- Реализовать экспорт:
- DSL → Graphviz DOT,
- DSL → PlantUML/Mermaid подмножество (для совместимости с существующими тулзами),
- DSL → BPMN XML/XMI (минимальный поднабор).omg+2
- Реализовать импорт:
- читать DOT/часть PlantUML/Mermaid и строить твой внутренний граф,
- сохранять как DSL (re‑serialize).
- Описать в РПЗ, как твой язык соотносится с существующими (таблица «элемент BPMN → элемент DSL/внутренней модели»).

### 8. Нефункциональные аспекты и оформление курсового

- Прописать нефункциональные требования:
- ограничения по размеру диаграмм,
- требования к отклику (рендер ≤ N мс для диаграммы до M узлов),
- безопасность (ограничение сторонних вызовов, если используешь внешние AI‑API).
- Сделать раздел тестирования:
- тесты парсера и валидатора (набор эталонных DSL‑файлов для UML/BPMN/Petri),
- нагрузочные тесты рендеринга (простая генерация больших графов).structurizr+1
- Подготовить документацию:
- спецификация DSL (формальное описание синтаксиса и семантики),
- user‑guide для web‑сервиса (как писать код, как сохранять/экспортировать),
- раздел «анализ аналогов и позиционирование» (PlantUML, Mermaid, Structurizr, Kroki и др.).diagrams.mingrammer+3

Если хочешь, дальше могу:

- развернуть один из этапов (например, «спроектировать DSL» или «схема БД и API») в более детальный чек‑лист прямо под твою РПЗ,
- или собрать это в структуру оглавления курсового (введение → аналитический раздел → проектный → технологический → экономический/безопасность).

## Глянуть мамин план

https://www.perplexity.ai/search/web-servis-vizualnogo-modeliro-SDcQiUaKQoa06xR5NgI5Bw

[Изображение: `word/media/image1.png` — шаблон диаграммы Исикавы («рыбья кость»): «Our current problem», Cause 1–6, Sub-cause 1–3]

## лр 3 4

Для выполнения Лабораторных работ №3 и №4 по вашему проекту «Visual DSL Diagrammer» (Вариант 13 — с адаптацией под SaaS-платформу), вам необходимо подготовить 7 артефактов.

Поскольку вы выбрали Вариант 13, согласно методичке, ваша микроразметка должна включать типы Person и Organization. Мы интегрируем их в структуру SaaS-сервиса: Organization будет владельцем (разработчиком) платформы, а Person — автором конкретной диаграммы или пользователем в системе.

Ниже приведены максимально подробные промпты, учитывающие ваши требования к селекторам DiagramEngine и Notation, шрифтам (Inter, Source Code Pro) и цветовой палитре.

1.    Лабораторная работа №3: Семантика и Доступность

1. Макет «Desktop Workspace + Footer» (Базовый визуальный макет)

Этот макет демонстрирует полную семантическую структуру Landmark-регионов (header, main, section, figure, footer).

Prompt: High-fidelity UI/UX design of a professional SaaS platform for "Diagram as Code" modeling, desktop view (1920x1080). Split-screen architecture.

Top Navigation Bar (`<header>`): Solid white background, minimalist logo "DiagramCode". Includes two functional dropdown menus: "DiagramEngine" (with "Mermaid" selected) and "Notation" (Optional, with "BPMN" selected). Action buttons: "Run" (solid black), "Save", "Export", and a glowing "AI Prompt" button.

Main Area (`<main>`): 40% width left pane is a dark code editor (#1E1E1E) with line numbers and technical DSL script in Source Code Pro font. 60% width right pane is a light canvas (#F8F9FA) with a dot grid showing a rendered BPMN flowchart.

Footer Area (`<footer>`): A thin status bar at the bottom. Left: "Engine: Connected". Center: "Line 12, Col 4". Right: "Notation: BPMN 2.0 | Version 1.0.4".

Visual Style: Monochromatic gray-scale palette, high contrast, clean Inter font for UI, 8k resolution.

2. Макет «AI Generation Modal» (Визуальный макет формы)

Необходим для описания семантики элементов форм (form, label, textarea) и ARIA-ролей доступности.

Prompt: Focused UI mockup of the diagram editor with a centered white modal dialog window titled "AI Diagram Generator". The background workspace is blurred and darkened. The modal contains a labeled form: a text label "Describe your process logic:", a large technical `<textarea>` with a dashed border and placeholder text, and a primary solid black button "Generate DSL Code". Typography uses Inter font. Monochromatic design, professional SaaS aesthetic, 4k.

3. Схема «Semantic Map» (Технический артефакт)

Инструкция: Возьмите Макет №1 и наложите на него рамки с подписями тегов.

Prompt: A technical documentation infographic showing the semantic HTML5 layout of a web-based IDE. Background is a dimmed version of the "Diagram-as-Code" interface. Overlay layer: Semi-transparent color-coded blocks covering functional areas with clear white labels: "`<header>`", "`<main>`", "`<section class='editor'>`", "`<figure class='preview'>`", and "`<footer>`". Blueprint style, flat vector, professional engineering report aesthetic.

4. Схема «Accessibility & Schema.org» (Технический артефакт для Варианта 13)

Демонстрирует роли role="dialog", role="img" и внедрение микроразметки Organization и Person.

Prompt: Technical UI diagram showing accessibility roles and Schema.org properties. It highlights the modal window from mockup 2. Callout labels point to elements: "role='dialog'" for the modal, "itemprop='publisher' (Organization)" for the logo, and "itemprop='author' (Person)" for a user profile icon in the corner. High-contrast schematic, clean lines, monochromatic palette, 8k.

2.    Лабораторная работа №4: Адаптивная верстка

5. Макет «Mobile Stacked View» (Визуальный макет адаптива)

Демонстрирует переход от 2 колонок к 1 (Stacked Layout) при ширине 375px.

Prompt: Mobile responsive UI of a diagram-as-code web app on a smartphone screen (375x812px). Vertical Stacked Layout (Column Drop pattern): compact header with logo and a hamburger menu icon. Middle section is a dark code editor window (35% height) showing a few lines of DSL code. Bottom section is a light diagram preview area (65% height) with a grid background and a flowchart fit-to-screen. Floating "Run" action button in the corner. Inter and Source Code Pro fonts, sleek modern mobile UI.

6. Макет «Tablet Split View» (Визуальный макет сетки)

Демонстрирует промежуточное состояние (768px) с равным распределением пространства (1fr 1fr).

Prompt: Tablet UI design for a technical modeling tool, landscape orientation (1024px). The interface uses a flexible grid: left code editor and right diagram preview share equal space (50/50 split). Header buttons "Save" and "Export" are replaced by minimalist icons to save space. Clean borders, light gray canvas with a grid, dark editor. Professional tool aesthetic, fonts: Inter and Source Code Pro.

7. Схема «CSS Grid Logic» (Технический артефакт)

Визуализирует области grid-area и логику перестроения.

Prompt: Technical schematic comparison of a CSS Grid layout. Left side "Desktop": 2-column grid with labeled areas 'header', 'editor', 'preview', 'footer'. Right side "Mobile": single-column stacked sequence of the same areas. Arrows show the layout transformation. Mathematical labels: "grid-template-columns: 400px 1fr" vs "1fr". Technical blueprint design, white background with blue grid lines.

3.    Как использовать это в отчете (РПЗ):

1.    ЛР №3 (Семантика): Используйте Макеты 1, 2 и Схемы 3, 4. Обоснуйте в тексте: «Для выполнения требований Варианта 13 в разметку внедрена информация об организации-разработчике (Organization) и авторе модели (Person) через атрибуты Microdata».

2.    ЛР №4 (Адаптив): Используйте Макеты 5, 6 и Схему 7. Опишите стратегию Mobile-First: «Базовая сетка приложения спроектирована как одноколоночная, а профессиональный сплит-режим активируется только на десктопных разрешениях (>1024px) через медиа-запросы».

3.    Drive E: Укажите, что все проектные ресурсы и макеты хранятся в рабочей директории E:\VisualDSL_Project\design\.
