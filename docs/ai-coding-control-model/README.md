# AI Coding Control Model: карта файлов

> Аудит рабочей папки после раскладки материалов.
> Статус `ядро` означает, что файл нужен для дальнейшей подготовки презентации.
> Статус `поддержка` означает источник, цель или исследовательскую базу.
> Статус `черновик/архив` означает, что файл полезен как история мысли, но не должен быть основой текущей презентации.

---

## 1. Текущая структура

```text
docs/ai-coding-control-model/
├── README.md
├── core/
│   ├── components-catalog-ru.md
│   ├── glossary-and-semantic-map-ru.md
│   ├── component-model-ru.md
│   ├── context-model-ru.md
│   └── slides-outline-ru.md
├── research/
│   ├── AI_DISRUPT_PDLC_v3.pdf
│   ├── agentic_software_engineering_operating_model_research.md
│   ├── ai-disrupt-pdlc-terms-review-ru.md
│   ├── components-catalog-extra-high-review-ru.md
│   └── chat-notes-ai-agentic-engineering.md
├── drafts/
│   ├── ai-engineering-control-system.md
│   ├── flowbender-agentic-engineering-framework.md
│   └── octobend-agent-meta-framework-v0.md
├── octobend/
│   └── octobend-architecture.md
└── meta/
    └── user-goals.md
```

Сгенерированные презентационные артефакты вынесены отдельно:

```text
artifacts/presentations/generated/
```

---

## 2. Что является актуальным ядром

| Файл | Статус | Зачем нужен |
|---|---|---|
| `core/components-catalog-ru.md` | ядро | Консолидированный каталог: слайдовая карта компонентов, детальные карточки, lifecycle/authority/risk таблицы, единый глоссарий |
| `core/glossary-and-semantic-map-ru.md` | ядро | Главный опорный файл: глоссарий, смысловая таблица, диаграмма, термины из PDF |
| `core/component-model-ru.md` | ядро | Детальное описание компонентов модели: контекст, память, спецификация, исполнение, проверка, объяснение, эффективность |
| `core/context-model-ru.md` | ядро | Глубокий разбор контекста: широкий/узкий смысл, виды контекста, извлечение, навигация, память |
| `core/slides-outline-ru.md` | ядро | Текущий markdown-черновик презентации на 5-6 слайдов |

Практический порядок чтения:

```text
1. components-catalog-ru.md
2. glossary-and-semantic-map-ru.md
3. component-model-ru.md
4. context-model-ru.md
5. slides-outline-ru.md
```

---

## 3. Поддерживающие материалы

| Файл | Статус | Зачем нужен |
|---|---|---|
| `meta/user-goals.md` | поддержка | Исходные цели: анализ AI-кодинга и подготовка презентации |
| `research/AI_DISRUPT_PDLC_v3.pdf` | поддержка | Внешний PDF-источник терминов и идей |
| `research/agentic_software_engineering_operating_model_research.md` | поддержка | Research по внешним терминам и аналогам: Agentic Software Engineering, SDD, context engineering, governance, evals |
| `research/ai-disrupt-pdlc-terms-review-ru.md` | поддержка | Разбор PDF: какие термины взять, какие использовать осторожно |
| `research/components-catalog-extra-high-review-ru.md` | поддержка / review | Сводный extra-high review от БА, СА и Архитектора по каталогу компонентов |
| `research/chat-notes-ai-agentic-engineering.md` | поддержка | Большая выжимка ранних диалогов и проблематики; полезна как source material |

---

## 4. Черновики и архив

Эти файлы не являются основой текущей презентации. Их лучше не удалять сейчас: в них есть ранние формулировки, которые могут пригодиться при развитии framework или appendix.

| Файл | Статус | Почему не ядро |
|---|---|---|
| `drafts/ai-engineering-control-system.md` | черновик/архив | Ранняя более широкая control-system модель; частично поглощена глоссарием и component model |
| `drafts/flowbender-agentic-engineering-framework.md` | черновик/архив | Старый framework-oriented текст на английском; текущая презентация должна быть не про Flowbender/OctoBend |
| `drafts/octobend-agent-meta-framework-v0.md` | черновик/архив | OctoBend-specific meta-framework; полезен позже, но не нужен как центральная тема презентации |
| `octobend/octobend-architecture.md` | project-specific | Архитектура OctoBend; держим отдельно, чтобы не смешивать с универсальной AI-coding моделью |

Кандидаты на удаление позже, если нужна чистка:

- `drafts/flowbender-agentic-engineering-framework.md`;
- `drafts/octobend-agent-meta-framework-v0.md`;
- `artifacts/presentations/generated/manual-20260609-ai-coding-entities/` — неполный/промежуточный presentation scaffold без финального `.pptx`.

Пока они сохранены намеренно.

---

## 5. Сгенерированные артефакты

| Путь | Статус | Комментарий |
|---|---|---|
| `artifacts/presentations/generated/manual-20260608-ai-coding-control/` | generated | Полная предыдущая PPTX-сборка для IT-лидеров с preview/layout/source |
| `artifacts/presentations/generated/manual-20260609-ai-coding-entities/` | generated / partial | Промежуточная заготовка deep-dive презентации по сущностям |

Главный готовый `.pptx` из предыдущей итерации:

```text
artifacts/presentations/generated/manual-20260608-ai-coding-control/presentations/ai-coding-control-model/output/ai-coding-control-model-it-leaders.pptx
```

---

## 6. Рекомендация по дальнейшей работе

Для следующей итерации презентации лучше брать за основу только:

```text
core/components-catalog-ru.md
core/glossary-and-semantic-map-ru.md
core/component-model-ru.md
core/context-model-ru.md
research/ai-disrupt-pdlc-terms-review-ru.md
```

Логика:

```text
Каталог компонентов даёт consolidated view.
Глоссарий даёт общий язык.
Component model даёт структуру.
Context model даёт глубину.
PDF review даёт внешнюю терминологию и validation.
```

`slides-outline-ru.md` можно использовать как черновой slide flow, но его лучше пересобрать после стабилизации глоссария.
