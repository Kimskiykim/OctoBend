# Extra-high review: каталог компонентов AI-кодинга

> Объект ревью: `core/components-catalog-ru.md`
> Дата: 2026-06-09
> Роли ревью: БА, СА, Архитектор
> Фокус: формулировки, аналитическая целостность, архитектурные границы, пригодность для слайда и detailed model.

---

## 1. Итоговый вердикт

Критичных противоречий, которые ломают модель целиком, ревью не нашло.

Документ сильный как deep-dive каталог и концептуальная база, но пока слабее как:

- один понятный слайд для IT-лидеров;
- практически применимый handbook/templates/checks;
- формальная control model с registry, lifecycle, operation matrix, traceability and evidence contract.

Общий вывод трёх ролей:

```text
9 компонентов можно оставить как слайдовую карту,
но detailed model должна опираться не на список компонентов,
а на три архитектурные плоскости:

1. Project Artifact Model
2. Agent Operating Model
3. Control Binding Layer
```

Самое сильное ядро модели:

```text
Артефакты проекта = существительные.
Операции агента = глаголы.
Правила управления связывают, какие глаголы могут действовать
на какие существительные, с какими правами, проверками и evidence.
```

---

## 2. Главные решения по итогам ревью

### 2.1. Для слайда

Слайдовая карта должна быть на русском языке. Английские термины лучше оставить в скобках, глоссарии или speaker notes.

Рекомендуемые короткие названия:

| # | Название для слайда | Смысл |
|---:|---|---|
| 1 | **Правила и ответственность** | что агенту можно, кто владелец, где согласование |
| 2 | **Контекст и память** | что агент видит, чему доверяет, что устарело |
| 3 | **Намерение и спецификация** | зачем меняем, границы, критерии приёмки |
| 4 | **Влияние и риск** | какие системы, данные, клиенты и команды затронуты |
| 5 | **Управляемое исполнение** | план, права, среда, журнал действий |
| 6 | **Проверка и доказательства** | тесты, CI, review, evidence для приёмки |
| 7 | **Актуальность знаний** | обновить docs/specs/contracts |
| 8 | **Решения человека** | вопросы, approvals, эскалации, принятие риска |
| 9 | **Эффективность и обучение** | rework, review effort, cost, повторные ошибки |

Более управленческая рамка для слайда:

```text
5 этапов AI-изменения:
спецификация -> риск -> исполнение -> доказательства -> актуальность знаний

4 сквозных контура контроля:
правила -> контекст -> решения человека -> эффективность и обучение
```

### 2.2. Для detailed model

В detailed model нужно явно отделить:

- архитектурные плоскости;
- capability areas;
- типы артефактов;
- операции агента;
- control rules;
- lifecycle states;
- evidence contract.

Дополнительно нужно явно показать, что модель не требует всегда тяжёлый SDD. Она должна объяснять управляемый спектр:

```text
Vibe Coding <-> SDD
```

Зрелость процесса не в запрете быстрого режима, а в выборе правильной глубины спецификации, planning, verification and evidence под риск изменения.

Рекомендуемая строгая карта:

```text
Primary planes:
1. Project Artifact Model
2. Agent Operating Model
3. Control Binding Layer

Artifact types:
1. System / Executable
2. Knowledge
3. Work
4. Evidence
5. Control
6. Derived

Operations:
clarify, retrieve, analyze, plan, implement, verify, validate,
reconcile, handoff, learn/propose, toolsmith

Controls:
authority, lifecycle, permissions, risk gates, owner review,
evidence, audit, freshness triggers
```

---

## 3. Consolidated findings

### P0. Слайдовая версия пока не готова для IT-лидеров

**Источник:** БА.

**Проблема:** верхнеуровневая карта использует много англоязычных и технических названий: `Governance`, `Control Binding`, `Intent-to-Spec`, `Harness`, `Freshness`, `Evidence`, `HITL`.

**Почему важно:** IT-лидер должен понять карту ответственности за 20-30 секунд. Сейчас ему нужно расшифровывать термины.

**Решение:**

- сделать русские названия основными;
- английские термины оставить в скобках или глоссарии;
- добавить управленческую рамку: боль -> решение -> риск без контроля -> метрика.

### P0. Не хватает явной бизнес-рамки

**Источник:** БА.

**Проблема:** в карточках есть “проблема” и “типовые сбои”, но нет явного ответа, какое управленческое решение поддерживает компонент.

**Почему важно:** без этого модель выглядит как инженерная методология, а не как способ снизить риск, rework, review effort и defect escape.

**Решение:** в каждую карточку добавить:

```text
Управленческое решение
Риск без компонента
Метрика
```

Пример:

```text
Проверка и доказательства
Управленческое решение: можно ли принимать результат агента.
Риск без компонента: ложное "готово".
Метрики: evidence completeness, defect escape rate, review effort.
```

### P1. Три архитектурные плоскости должны быть главным каркасом

**Источники:** Архитектор, СА.

**Проблема:** раздел про `Project Artifact Model / Agent Operating Model / Control Binding Layer` сейчас выглядит как пояснение после 9 компонентов, хотя это более строгий фундамент модели.

**Почему важно:** иначе модель снова читается как pipeline, а не как separation of concerns.

**Решение:**

- вынести три плоскости перед 9 компонентами;
- назвать 9 компонентов capability areas;
- отдельно показать, что capability areas проходят поверх трёх плоскостей.

### P1. Artifact taxonomy неполна

**Источники:** СА, Архитектор.

**Проблема:** текущая taxonomy содержит `Knowledge`, `Work`, `Evidence`, `Derived`, но не выделяет:

- `System / Executable artifacts`: code, tests, configs, schemas, migrations, build/runtime manifests;
- `Control artifacts`: policies, authority matrix, operation matrix, evidence policy, lifecycle rules.

**Почему важно:** code/tests/configs являются главными target artifacts для permissions, protected paths, impact and evidence. Control artifacts нельзя обрабатывать как обычную проектную память.

**Решение:** добавить типы:

| Тип | Примеры | Особое правило |
|---|---|---|
| System / Executable | code, tests, configs, schemas, migrations | особые permissions, checks, runtime impact |
| Control | policies, authority matrix, operation matrix, lifecycle rules | agent may propose, but not silently update |

Добавить forbidden confusion:

```text
Policy proposal is not accepted policy.
```

### P1. Нужна operation matrix v2

**Источник:** СА.

**Проблема:** матрица операций короче собственной модели: нет planning, handoff, approval/escalation, deviation/replan, validation.

**Почему важно:** operation matrix должна быть основой policy/checks. Сейчас она не задаёт, что агент читает, пишет и доказывает на каждой фазе.

**Решение:** расширить матрицу до:

```text
clarify
retrieve
analyze
plan
implement
verify
validate
reconcile
handoff
learn/propose
toolsmith
```

Добавить колонки:

```text
Gate
Owner/HITL
Allowed actions
Artifact state before/after
Evidence required
```

### P1. Lifecycle заявлен, но не смоделирован

**Источники:** СА, Архитектор.

**Проблема:** lifecycle описан как маршрут работы, но не как state model для артефактов.

**Почему важно:** ключевой риск модели - временная заметка становится trusted knowledge, change spec не архивируется, evidence перезаписывается, stale summary живёт дальше.

**Решение:** добавить lifecycle state table:

```text
draft -> proposed -> approved -> active -> accepted
      -> promoted / archived / superseded / invalidated
```

Отдельно описать lifecycle для:

- change spec;
- stable spec;
- evidence bundle;
- freshness report;
- derived context pack;
- control policy.

### P1. Не хватает traceability matrix

**Источник:** СА.

**Проблема:** traceability декларируется, но нет структуры, связывающей intent, acceptance criteria, impact, changed files, verification evidence и freshness decision.

**Почему важно:** без этого человек ревьюит narrative, а не проверяемый contract.

**Решение:** добавить таблицу:

| Поле | Что фиксирует |
|---|---|
| Intent ID | зачем делаем изменение |
| Spec / AC ID | что должно быть проверено |
| Impact area | что затронуто |
| Execution artifact / diff | где реализовано |
| Verification command / evidence | чем доказано |
| Freshness action | что обновлено или почему не обновлено |
| Owner / reviewer | кто принял |
| Residual gap | что осталось непроверенным |

### P1. Authority нужно формализовать как matrix

**Источники:** СА, БА, Архитектор.

**Проблема:** `authority` правильно оставлена свойством, а не названием компонента, но пока нет матрицы приоритетов источников по типу вопроса.

**Почему важно:** source of truth зависит от вопроса. Code, tests, runtime evidence, stable spec, ADR и docs могут конфликтовать.

**Решение:** добавить `Authority & Conflict Matrix`:

| Тип вопроса | Сильнейшие источники | Слабые источники | Что делать при конфликте | Может ли агент менять | Review/evidence |
|---|---|---|---|---|---|
| Что реально работает? | code, tests, runtime evidence | old docs, summaries | conflict report | code/tests only in allowed scope | test/runtime evidence |
| Что должно быть сохранено? | stable spec, contract | chat, old issue | owner decision | proposal only | spec/contract review |
| Почему так решили? | ADR, decision trace | generated summary | ADR update proposal | no silent rewrite | architecture review |

### P1. Risk budget должен стать binding control

**Источник:** Архитектор.

**Проблема:** risk table задаёт уровни, но не связывает их с operation matrix, permissions, approvals and evidence requirements.

**Почему важно:** без binding риск остаётся рекомендацией.

**Решение:** Risk component должен производить `control-decision`:

```text
risk level
allowed writes
required approvals
required checks
evidence minimum
reconciliation triggers
```

Добавить бизнес-критерии риска:

| Риск | Признаки задачи |
|---|---|
| Low | локальный refactor, typo, test-only change без контракта |
| Medium | изменение поведения одного модуля или сервиса |
| High | публичный API, данные, безопасность, миграции, downstream consumers |
| Critical | auth, payments, PII, production recovery, критичные миграции |

Также добавить связку с режимами работы:

| Риск | Базовый режим |
|---|---|
| Low | Vibe Coding с границами |
| Medium | облегчённый SDD |
| High | SDD с owner gates |
| Critical | human-led, agent-assisted |

Главная формулировка:

```text
Vibe Coding и SDD - не враги.
Это два конца спектра контроля.
Control model выбирает точку на спектре по риску задачи.
```

### P1. Evidence нужно оформить как claim-evidence contract

**Источник:** Архитектор.

**Проблема:** формула готовности сильная, но не требует маппинга каждого claim к evidence.

**Почему важно:** “готово” должно проверяться не по summary агента, а по inspectable proof.

**Решение:** добавить инвариант:

```text
Every completion claim maps to:
evidence id,
command/check,
timestamp/environment,
result,
or explicit "not verified".
```

### P2. Context & Memory Management требует внутренних подграниц

**Источники:** БА, СА, Архитектор.

**Проблема:** в одном компоненте смешаны retrieval/window management, session memory, project memory, generated summaries, authority metadata and freshness metadata.

**Почему важно:** это место с наибольшим риском “context pack стал истиной” и “session note стал project memory”.

**Решение:** для слайда оставить `Контекст и память`, но в detailed model выделить:

- Context Selection;
- Context Window / TokenOps;
- Session Memory;
- Project Memory;
- Generated / Derived Context;
- Authority / Freshness Metadata.

Коротко зафиксировать границу:

```text
Governance задаёт правила авторитетности.
Context & Memory применяет их при выборе источников.
Freshness проверяет, не устарели ли источники до, во время или после изменения.
```

### P2. Freshness/Reconciliation нельзя описывать только как этап после изменения

**Источник:** Архитектор.

**Проблема:** drift может быть найден во время context discovery, impact analysis или verification.

**Решение:** переформулировать:

```text
Freshness & Reconciliation detects, classifies, reconciles, waives or invalidates
drift across code/docs/specs/contracts/memory before, during and after change.
```

### P2. Human Control Loop не должен забирать governance responsibilities

**Источник:** Архитектор.

**Проблема:** approvals/gates описаны и в Human Loop, и в Governance/Risk.

**Почему важно:** approval должен быть gate, а не просто чат.

**Решение:**

- Human Loop отвечает за interaction protocol: questions, status, explanation, escalation, handoff;
- authority to approve задаётся Governance/Risk/Owner rules.

### P2. Verification и Validation нужно связать с lifecycle

**Источник:** СА.

**Проблема:** validation разведён в глоссарии, но отсутствует в lifecycle.

**Почему важно:** можно получить verified change, который не решает исходный intent.

**Решение:** добавить gate:

```text
Validation / Acceptance sign-off:
кто подтверждает, что intent решён;
какими evidence/demo/UAT это подтверждается.
```

### P2. Learning & Effectiveness слишком широкий catch-all

**Источники:** СА, Архитектор.

**Проблема:** в одной карточке смешаны retrospectives, evals, template updates, tool promotion lifecycle, metrics and TokenOps.

**Почему важно:** метрика не является improvement, а proposal не является accepted policy.

**Решение:** для слайда оставить `Эффективность и обучение`, но внутри разделить:

```text
Observation -> Insight -> Proposal -> Review -> Policy/Template/Check/Eval update
```

И отдельно:

```text
Measurement model:
owner, cadence, numerator, denominator, threshold, decision supported.
```

### P2. Карточки компонентов требуют единого интерфейса

**Источник:** СА.

**Проблема:** компонент определён как область с входами и артефактами, но карточки не имеют одинаковой структуры входов/выходов.

**Решение:** нормализовать карточки:

```text
Purpose
Inputs
Owned outputs
Consumes
Controls
Failure modes
Checks
Adapters
Metrics
Management decision / Risk without component
```

### P2. Глоссарий нужен в двух слоях

**Источник:** БА.

**Проблема:** глоссарий перегружен английскими терминами и смешанными формулировками.

**Решение:**

- добавить “10 терминов для презентации”;
- полный технический глоссарий оставить как appendix;
- в основном тексте использовать русский термин + английский при первом упоминании.

### P3. Adapter lists ослабляют framework-independence

**Источник:** Архитектор.

**Проблема:** списки конкретных инструментов в конце карточек визуально смещают модель к tool catalog.

**Решение:** вынести adapters в appendix или явно пометить:

```text
Adapter examples are replaceable.
Component contract stays stable.
```

### P3. “Forbidden confusions” стоит оформить как русский слайд

**Источник:** БА.

**Проблема:** блок сильный, но оформлен на английском.

**Решение:** сделать таблицу “Что нельзя подменять”:

| Подмена | Почему опасно |
|---|---|
| Зелёные unit-тесты = готовность | acceptance, contracts, docs and risk могут быть не проверены |
| Diff = доказательство | diff показывает изменение, но не доказывает корректность |
| Search result = истина | search даёт кандидатов, не authority |
| Change spec = stable spec | proposed delta не становится intended behavior без promotion |
| Proposal агента = policy | policy требует owner review |

---

## 4. Рекомендуемый план исправлений

### Шаг 1. Быстро привести слайдовую карту

Цель: сделать документ пригодным для презентации.

Изменения:

- заменить слайдовые названия на русские;
- показать “5 этапов + 4 контура”;
- добавить различение `Vibe Coding` и `SDD` как спектр контроля;
- добавить бизнес-рамку `боль -> решение -> риск -> метрика`;
- переоформить “Что нельзя подменять” как русскую таблицу.

### Шаг 2. Усилить архитектурный каркас

Цель: закрепить модель как framework-independent operating model.

Изменения:

- вынести три плоскости перед 9 компонентами;
- назвать 9 компонентов capability areas;
- добавить `System/Executable artifacts` и `Control artifacts`;
- добавить lifecycle state model.

### Шаг 3. Сделать модель операциональной

Цель: перейти от каталога к handbook/templates/checks.

Изменения:

- добавить `Agent Operation Matrix v2`;
- добавить `Traceability Matrix`;
- добавить `Authority & Conflict Matrix`;
- добавить `Risk Control Decision`;
- добавить `Claim-Evidence Contract`;
- нормализовать карточки компонентов.

### Шаг 4. Почистить detailed model

Цель: убрать смешение и дубли.

Изменения:

- внутри `Контекст и память` выделить поддомены;
- внутри `Эффективность и обучение` разделить learning flow and measurement model;
- уточнить Freshness as before/during/after drift control;
- отделить Human Loop от approval policy;
- вынести adapters в appendix;
- сделать глоссарий двухуровневым.

---

## 5. Что не менять

Ревью не предлагает выбрасывать 9 компонентов.

Лучшее решение:

```text
9 компонентов оставить как presentation/capability map.
Три плоскости сделать архитектурным основанием detailed model.
```

Переименование `Context & Memory Authority` в `Context & Memory Management` подтверждено как корректное.

При этом `authority` нужно сохранить как:

- свойство источников;
- часть governance;
- часть context selection;
- часть freshness/conflict handling.
