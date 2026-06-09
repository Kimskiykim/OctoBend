# User Goals

> Рабочая фиксация целей сессии. Этот файл описывает не сам фреймворк, а задачи пользователя: зачем нужен анализ, какие вопросы он должен закрыть и какой итоговый артефакт нужен.

---

## 1. Базовый анализ AI-кодинга

Главная цель: разложить AI-assisted coding на устойчивые концепты, проблемы, задачи, сущности и контрольные контуры, которые существуют независимо от конкретного инструмента или фреймворка.

Нужно описать то, что в любом случае возникает перед coding agent:

- намерение пользователя и его потеря в чате;
- границы задачи и риск scope drift;
- знания проекта и их authority;
- выбор релевантного контекста;
- impact analysis перед изменениями;
- планирование и execution control;
- verification и evidence;
- freshness docs/specs/contracts/facts после изменения кода;
- learning/process improvement после ошибок;
- permission boundaries и safety;
- tool/model/framework routing;
- handoff между агентами, сессиями и командами.

Ключевой принцип:

```text
Сначала описать концептуальные проблемы и сущности.
Потом маппить на конкретные frameworks/tools/adapters.
```

Это нужно, чтобы:

- менять фреймворки без потери общей модели;
- дружить разные фреймворки, если разные команды используют разные инструменты;
- сравнивать фреймворки по capability coverage, а не по маркетинговым категориям;
- отдельно оценивать каждый процесс: intent, retrieval, impact, execution, verification, freshness, learning;
- видеть, где проблема концептуальная, где процессная, где инструментальная.

Рабочая формула:

```text
Conceptual problem
  -> capability
  -> operation
  -> artifact
  -> policy/check
  -> adapter/tool
```

Пример:

```text
Problem:
  Agent changes public API but OpenAPI is not updated.

Capability:
  Freshness / contract drift control.

Operation:
  detect API-impacting diff and require contract review.

Artifact:
  contracts/openapi/*
  changes/<id>/verification.md
  freshness finding

Policy/check:
  API route changed -> contract update or waiver required.

Adapter/tool:
  CI check, OpenAPI diff, Codex/Claude workflow instruction.
```

---

## 2. Framework-independent evaluation model

Нужна модель, через которую можно оценивать любой agentic coding framework.

Вопросы для оценки:

| Dimension | Question |
|---|---|
| Intent | Как фиксируется пользовательское намерение? |
| Scope | Как задаются boundaries, non-goals и risk level? |
| Knowledge | Где живёт project truth и как определяется authority? |
| Retrieval | Как агент получает контекст и не путает search с truth? |
| Impact | Как определяется, что будет затронуто? |
| Execution | Как контролируется работа агента по шагам? |
| Verification | Как доказывается корректность результата? |
| Evidence | Где сохраняются logs, commands, test outputs, decisions? |
| Freshness | Как docs/specs/contracts/facts остаются актуальными? |
| Learning | Как повторяющиеся ошибки превращаются в улучшения процесса? |
| Governance | Какие действия запрещены или требуют review? |
| Adapter fit | Как framework интегрируется с Codex, Claude Code, CI, MCP, search tools? |

Идеальный результат анализа:

```text
Не "какой framework лучше",
а "какую capability он закрывает,
какую не закрывает,
и как его состыковать с остальными".
```

---

## 3. Итоговая презентация

По итогам сессии нужно подготовить презентацию.

Цель презентации:

- объяснить проблему AI-кодинга не как "агенты иногда ошибаются", а как отсутствие engineering control model;
- показать базовую decomposition: concepts, problems, tasks, entities, artifacts, policies, adapters;
- объяснить отличие project knowledge от agent-specific tasks;
- показать, почему framework-independent model полезнее привязки к одному инструменту;
- дать понятный MVP, который можно применить сразу;
- показать roadmap от markdown/git/CI к более сложным adapters, checks, evals и orchestration.

Предлагаемая структура презентации:

1. Почему raw agentic coding превращается в unmanaged process.
2. Какие проблемы повторяются независимо от агента и фреймворка.
3. Главная модель: project nouns, agent verbs, control policies.
4. Слои: knowledge, change, operations, policies, recipes, adapters.
5. Capability map для AI coding.
6. Как оценивать и совмещать разные frameworks.
7. Instant-apply MVP: что добавить в repo сегодня.
8. Пример flow: API change / auth change / docs drift.
9. Roadmap: solo developer -> team -> org.
10. Итог: repo-native control layer around coding agents.

Минимальный результат для презентации:

```text
10-12 слайдов,
одна центральная схема,
одна таблица capability evaluation,
один concrete MVP example,
один roadmap.
```

---

## 4. Working Outputs Needed From This Repo

К текущим рабочим материалам стоит иметь:

- consolidated framework v0;
- user goals / intent file;
- presentation outline;
- slide draft or deck;
- optional: framework evaluation matrix;
- optional: example mapping for 2-3 tools/frameworks.

Current related files:

```text
ai_agentic_engineering_chat_notes.md
ai_engineering_control_system.md
flowbender_agentic_engineering_framework.md
octobend-architecture.md
octobend_agent_meta_framework_v0.md
user_goals.md
```
