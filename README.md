# NotaCode

Web-IDE «diagram as code» для формальных нотаций моделирования: UML, BPMN 2.0, ERD, IDEF0, IDEF1X, IDEF3, DFD, сети Петри, свободная нотация. Диаграмма описывается на собственном языке (DSL), отображается на холсте, проверяется на соответствие правилам нотации, версионируется и экспортируется.

## Документация

| Раздел | Путь |
|---|---|
| Архитектура (C4, сценарии, модель данных) | [docs/architecture](docs/architecture/README.md) |
| Требования | [docs/requirements](docs/requirements/README.md) |
| План работ (Scrum + Kanban, Гант, MVP) | [docs/planning/agile-plan.md](docs/planning/agile-plan.md) |
| Условия лабораторных работ | [docs/labs/lab-conditions.md](docs/labs/lab-conditions.md) |
| Теория курса | [docs/theory](docs/theory) |
| Литература | [docs/literature](docs/literature) |
| Исходные материалы и аудит предыдущих версий | [docs/source-materials](docs/source-materials) |
| Контекст и решения | [docs/context/dialogue-decisions.md](docs/context/dialogue-decisions.md) |

## Стек

React 19 + TypeScript + Vite (PWA), Monaco Editor, elkjs · Node.js + Express (Gateway, Sequelize, Mongoose, Socket.IO) · Python + FastAPI (Workspace, Language, Conversion, AI, Integration) · PostgreSQL, MongoDB, Redis, RabbitMQ, MinIO · Docker, GitHub Actions, OpenTelemetry.
