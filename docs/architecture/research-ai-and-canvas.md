# Исследование: AI-провайдеры и библиотеки холста (23.09.2026)

Отметка **[?]** — не подтверждено официальной документацией, перепроверить перед использованием.

## 1. Бесплатные и платные LLM API

Большинство провайдеров предоставляют OpenAI-совместимый API. Для них используется единый адаптер на пакете `openai` с параметрами `base_url`, `api_key`, `model`. Anthropic и собственный SDK Gemini подключаются отдельными адаптерами.

| Провайдер | Бесплатно | Vision | Переменная | Base URL (OpenAI-совм.) | Где взять ключ |
|---|---|---|---|---|---|
| Google Gemini | да, лимиты в AI Studio [?] | да | `GEMINI_API_KEY` | `https://generativelanguage.googleapis.com/v1beta/openai/` | https://aistudio.google.com/apikey → Create API key |
| Groq | да, 30 RPM / 1000 RPD | `qwen/qwen3.8-27b` | `GROQ_API_KEY` | `https://api.groq.com/openai/v1` [?] | https://console.groq.com/keys → Create API Key |
| OpenRouter `:free` | да, 20 RPM / 50 RPD (1000 при кредитах от $10) | зависит от модели | `OPENROUTER_API_KEY` | `https://openrouter.ai/api/v1` | openrouter.ai → Settings → Keys [?] |
| Mistral (Experiment) | да, без карты | да [?] | `MISTRAL_API_KEY` | `https://api.mistral.ai/v1` | console.mistral.ai → API Keys → Create new key |
| Hugging Face | $0.10 в месяц | да | `HF_TOKEN` | `https://router.huggingface.co/v1` | huggingface.co/settings/tokens/new (fine-grained, Inference Providers) |
| Cohere Trial | 1000 вызовов в месяц, не для продакшна | `command-a-vision-07-2025` | `CO_API_KEY` | `https://api.cohere.ai/compatibility/v1` | dashboard.cohere.com/api-keys |
| Cloudflare Workers AI | 10 000 Neurons в день | да [?] | `CLOUDFLARE_API_KEY`, `CLOUDFLARE_ACCOUNT_ID` | `https://api.cloudflare.com/client/v4/accounts/{id}/ai/v1` | dash → Account API Tokens → шаблон Workers AI [?] |
| Ollama (локально) | да, без ключа | да | — | `http://localhost:11434/v1/` | установить Ollama |
| DeepSeek | нет, дёшево | `deepseek-flash` [?] | `DEEPSEEK_API_KEY` | `https://api.deepseek.com` | platform.deepseek.com/api_keys |
| Anthropic | нет | да | `ANTHROPIC_API_KEY` | собственный API | platform.claude.com/settings/keys |
| OpenAI | нет | да | `OPENAI_API_KEY` | `https://api.openai.com/v1` | platform.openai.com → API keys |
| Together AI | нет, от $5 | да | `TOGETHER_API_KEY` | `https://api.together.ai/v1` | api.together.ai |
| GitHub Models | **закрыт 30.07.2026** | — | — | — | не использовать |

## 2. Холст и раскладка

| Библиотека | Лицензия | Для чего годится | Риски |
|---|---|---|---|
| maxGraph `@maxgraph/core` | Apache-2.0 | ближе всего к draw.io: swimlane, порты, ортогональные рёбра, редактирование | версия 0.x, API меняется, мало документации |
| elkjs | EPL-2.0 | layered-раскладка с портами FIXED_SIDE (ICOM у IDEF0), составные узлы, ортогональная маршрутизация | только раскладка, отрисовки нет |
| React Flow `@xyflow/react` | MIT | идиоматичный React, sub-flows | своей раскладки и ортогонального обхода узлов нет |
| JointJS core | MPL-2.0 | строгие фигуры, порты, manhattan-роутинг | удобный редактор только в платной JointJS+ |
| `@hpcc-js/wasm-graphviz` | Apache-2.0 | статический DOT → SVG в браузере | только рендер, интерактива нет |
| bpmn-js | MIT с условием водяного знака | эталонный BPMN + BPMN XML | только BPMN, водяной знак обязателен |
| draw.io embed (`embed.diagrams.net`, `jgraph/drawio`) | Apache-2.0 | открыть и отредактировать диаграмму в полном draw.io через postMessage JSON | торговая марка; стенсилы нельзя использовать в продуктах Atlassian |
| Kroki (`yuzutech/kroki` + компаньоны) | MIT | PlantUML, Mermaid, Graphviz, D2, BPMN, Structurizr и др. → SVG/PNG/PDF, можно развернуть у себя | внешний рендер, связи «строка кода ↔ элемент» нет |

## 3. Monaco Editor

`@monaco-editor/react` даёт:

- `Editor` и `DiffEditor` (side-by-side, как в VS Code);
- Monarch-токенайзер для своего DSL;
- `setModelMarkers` для волнистых подчёркиваний;
- `registerCodeActionProvider` для quick fix (автоисправление);
- `registerCompletionItemProvider` для подсказок.
