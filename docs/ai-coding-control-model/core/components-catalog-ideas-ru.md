# Идеи и операционные заготовки к каталогу компонентов

> Материалы вынесены из `components-catalog-ru.md`, чтобы основной каталог оставался компактным.
> Это заготовки для handbook, операционных таблиц, MVP внедрения и appendix.

## Project Artifact Registry

Artifact registry нужен, чтобы агент и человек одинаково понимали, с каким типом объекта они работают, кто им владеет, можно ли его менять и какая проверка нужна.

| Artifact class | Что это | Примеры | Agent write rule | Lifecycle / review |
|---|---|---|---|---|
| **System / Executable artifacts** | Исполняемое состояние системы | code, tests, configs, schemas, migrations, build/runtime manifests | только в рамках scope, permissions and risk gates | требуют checks, owner review для protected areas |
| **Knowledge artifacts** | Устойчивые знания проекта | domain docs, architecture, ADR, specs/current, contracts, facts, runbooks | proposal or reviewed update; no silent overwrite | требуют owner, authority, freshness trigger |
| **Work artifacts** | Активное состояние изменения | `changes/<id>/intent.md`, design, acceptance, affected areas, verification plan | можно создавать/обновлять в рамках активной задачи | после merge archived or promoted |
| **Артефакты доказательств** | Доказательства работы | run log, commands, test output, verification report, review notes | append-only or traceable revision | не заменяют specs; retention/audit rule |
| **Control artifacts** | Правила управления агентом и процессом | policies, authority matrix, operation matrix, политика доказательств, lifecycle rules | agent may propose, but not silently update | versioned, owner-reviewed, audit-sensitive |
| **Derived artifacts** | Пересоздаваемые артефакты доступа | generated index, repo map, context pack, search results | regenerate, do not curate as truth | invalidated by freshness triggers |

Forbidden confusions:

```text
ADR is not current behavior spec.
Search result is not truth.
Run log is not memory.
Generated context pack is not source of truth.
Change spec is not stable spec.
Test pass is not full verification.
LLM judgment is not freshness trigger.
Agent proposal is not accepted knowledge.
Policy proposal is not accepted policy.
Code diff сам по себе не является доказательством.
```

---

## Операционные таблицы

### Agent Operation Matrix v2

| Operation | Reads | Writes | Gate / owner | State before -> after | Нужные доказательства |
|---|---|---|---|---|---|
| Clarify | user request, issue, project knowledge | intent, scope, open questions | human owns intent | vague request -> stated intent | captured assumptions and questions |
| Retrieve | code, docs, specs, tests, external sources | retrieval report, context pack | context rules | unknown context -> selected context | source map with authority/freshness notes |
| Analyze | code, specs, contracts, owner map | impact report, risk note, verification plan seed | risk owner if medium/high | selected context -> affected areas | affected areas and risk rationale |
| Plan | intent, context, impact, permissions | execution plan, plan approval request if needed | required before medium/high implementation | analyzed task -> approved route | plan with scope, allowed writes, checks, stop conditions |
| Implement | approved scope, plan, code/tests | code, tests, local docs if allowed, run log | permissions and protected paths | approved route -> diff | diff summary, changed files, command log |
| Verify | code, tests, contracts, acceptance | verification report, пакет доказательств | политика доказательств | diff -> verified/not verified claims | маппинг утверждений и доказательств |
| Validate | intent, acceptance, demo/UAT доказательства | acceptance sign-off or residual gap | human/product owner when needed | verified change -> accepted/rejected intent | validation note or explicit gap |
| Reconcile | diff, docs, specs, contracts, memory | freshness report, update proposal | owner review for trusted knowledge | changed system -> reconciled knowledge | updated artifact or not-affected decision |
| Handoff | plan, run log, доказательства, gaps | handoff summary | next human/agent can continue | active session -> transferable state | done/verified/not-verified summary |
| Learn/propose | runs, review comments, repeated failures | insight, process proposal, eval fixture | process owner review | observation -> reviewed proposal | доказательства повторяющегося сбоя или улучшения |
| Toolsmith | repeated manual steps, run logs | helper/check proposal | engineering owner review | friction -> validated tool candidate | validation result and deletion/owner policy |

### Lifecycle State Table

| Artifact | Typical states | Promotion rule | Invalidation/freshness trigger |
|---|---|---|---|
| Change spec | draft -> proposed -> active -> accepted -> archived/promoted | promoted only into stable docs/specs after review | scope change, rejected acceptance, merge/close |
| Stable spec | active -> updated -> superseded/deprecated | updated through reviewed PR/change | behavior or contract change |
| Пакет доказательств | created -> appended -> referenced -> retained/archived | never silently rewritten; corrections are traceable | rerun checks, failed CI, changed diff |
| Freshness report | draft -> reviewed -> actioned -> archived | action needs owner/reason | changed paths, contract/schema diff, stale source found |
| Derived context pack | generated -> used -> invalidated/regenerated | cannot be promoted directly to truth | source file change, stale summary, context rot |
| Control policy | proposed -> reviewed -> active -> superseded | owner-reviewed only | process incident, recurring failure, governance change |

### Traceability Matrix

| Поле | Что фиксирует | Пример |
|---|---|---|
| Intent ID | зачем делаем изменение | `INT-1: снизить риск ложного ready` |
| Spec / AC ID | что должно быть проверено | `AC-2: contract check attached` |
| Impact area | что затронуто | API, auth, docs, schema |
| Execution artifact / diff | где реализовано | changed files, PR, patch |
| Команда проверки / доказательства | чем доказано | test command, CI link, screenshot, log |
| Freshness action | что обновлено или почему не обновлено | docs updated, not affected |
| Owner / reviewer | кто принял | code owner, architect, product owner |
| Residual gap | что осталось непроверенным | manual UAT not run, staging unavailable |

### Authority & Conflict Matrix

| Тип вопроса | Сильнейшие источники | Слабые источники | Что делать при конфликте | Может ли агент менять | Ревью/доказательства |
|---|---|---|---|---|---|
| Что реально работает? | code, tests, runtime-подтверждения | old docs, generated summaries | conflict report, verify runtime | code/tests only in allowed scope | test/runtime-подтверждения |
| Что должно быть сохранено? | stable spec, contract, policy | chat, old issue, temporary note | owner decision | proposal or reviewed update | spec/contract review |
| Почему так решили? | ADR, decision trace | generated summary, memory note | ADR update proposal or supersede | no silent rewrite | architecture review |
| Что можно менять? | permissions, protected paths, owner map | informal agreement | approval request | only within allowed boundary | audit/approval record |
| Что устарело? | changed paths, contract/schema diff, freshness rules | LLM judgment alone | freshness report | update proposal or not-affected decision | deterministic signal plus owner/review |

---

## Минимальный end-to-end lifecycle

Для low-risk задач модель может быть лёгкой. Для medium/high-risk задач минимальный управляемый маршрут выглядит так:

```text
1. Намерение
   Зафиксировать проблему, expected behavior, scope, non-goals.

2. Интерпретация намерения
   Собрать релевантный context, уточнить смысл, constraints, assumptions и acceptance criteria.

3. Планирование
   Оценить affected areas, owners, contracts, security, risk gates,
   порядок действий, границы записи, checks, stop conditions and путь сбора доказательств.

4. Исполнение
   Выполнить работу через approved plan, boundaries, run log and deviation log.

5. Тестирование
   Проверить acceptance criteria, tests, contracts, security checks и собрать пакет доказательств.

6. Ревью
   Объяснить diff, доказательства, assumptions and residual gaps человеку.
   Проверить результат против intent, acceptance criteria, риска и доказательств.

7. Приёмка / решение по изменению
   Принять изменение, вернуть на доработку или явно зафиксировать gap/residual risk.

8. Check Guardrails
   Проверить, что соблюдены risk gates, protected paths, approvals,
   обязательные checks и stop conditions.

9. Обновление документации и спецификаций
   Обновить или явно не обновить docs/specs/contracts/memory.

10. Улучшение процесса
   Если есть recurring failure или friction, предложить process patch.
```

---

## Минимальная версия для внедрения

Если превращать модель в практический handbook, минимальный набор должен быть таким:

```text
1. Artifact registry
2. Operation matrix
3. Source authority rules
4. Политика доказательств
5. Freshness rules
6. Risk-based control budget
   - Vibe Coding vs SDD decision rule
7. 4-5 templates:
   - change spec
   - impact report
   - verification report
   - freshness report
   - run log
8. 2-3 deterministic checks:
   - contract drift
   - protected paths
   - доказательства обязательны до завершения
9. 3 demo scenarios:
   - API endpoint change
   - ADR update proposal
   - implementation blocked before acceptance criteria
```

Жёсткое правило MVP:

```text
Не добавлять артефакт, если непонятно:
кто его обновляет,
когда он устаревает,
какой check ловит drift,
кто его review,
когда его удалять или архивировать.
```

---

## Примеры адаптеров как appendix

Адаптеры - это заменяемые реализации зон управления. Они помогают показать, как модель применяется на практике, но не являются стабильной частью самой модели.

```text
Component contract stays stable.
Adapter examples are replaceable.
```

Как читать списки адаптеров в карточках:

- coding agents усиливают execution, explanation and handoff;
- search/RAG/MCP усиливают context retrieval, но не определяют truth;
- CI/test/security tools поставляют доказательства, но не заменяют acceptance criteria;
- OpenSpec/Spec Kit усиливают Intent-to-Spec, но не покрывают orchestration, freshness and learning;
- IDP/policy engines могут реализовать governance, gates and audit на enterprise-уровне.

---

## Источники внутри репозитория

Основные файлы, из которых собран каталог:

- `core/component-model-ru.md`;
- `core/context-model-ru.md`;
- `core/glossary-and-semantic-map-ru.md`;
- `core/slides-outline-ru.md`;
- `research/agentic_software_engineering_operating_model_research.md`;
- `research/ai-disrupt-pdlc-terms-review-ru.md`;
- `research/chat-notes-ai-agentic-engineering.md`;
- `research/AI_DISRUPT_PDLC_v3.pdf`.

Этот файл не заменяет исходные материалы. Он служит consolidated view для подготовки презентации и deep-dive описания компонентов.
