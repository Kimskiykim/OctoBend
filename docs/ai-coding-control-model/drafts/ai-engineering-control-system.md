# AI Engineering Control System для AI-assisted / Spec-driven Development

**Статус:** рабочий design-документ
**Назначение:** описать управляемый engineering-процесс разработки с AI-агентами в реальном software project.
**Главная идея:** это не просто SDD-framework и не набор промптов. Это контур управления разработкой: память проекта → намерение изменения → исполнение → проверка → reconciliation знаний → самоулучшение процесса.

---

## 1. Короткая формула

```text
Stable Knowledge tells the agent what is true/intended.
Change Spec tells what should become different.
Execution Run records how the agent tried to do it.
Evidence proves whether it worked.
Freshness reconciles knowledge after reality changed.
Self-Optimization improves the process after observing failures.
Retrieval accelerates access but never becomes truth.
```

Или по-русски:

```text
Стабильное знание говорит агенту, что сейчас считается истиной или намеренным поведением.
Change spec говорит, что должно измениться.
Run фиксирует, как агент пытался выполнить изменение.
Evidence доказывает, что результат работает.
Freshness watcher сверяет память с новой реальностью.
Self-optimization улучшает сам процесс после ошибок.
Retrieval ускоряет доступ к контексту, но не становится источником истины.
```

---

## 2. Что мы вообще строим

Мы строим **AI Engineering Control System** для разработки в репозитории.

Это система, которая отвечает на вопросы:

1. Что проект уже знает?
2. Что мы хотим изменить?
3. Почему мы хотим это изменить?
4. Какие требования и acceptance criteria у изменения?
5. Как агент должен выполнять работу?
6. Как не дать агенту потерять контекст или уйти в сторону?
7. Как проверить, что результат правильный?
8. Как понять, какие docs/specs/facts устарели после изменения кода?
9. Как промотировать новые знания в curated memory?
10. Как улучшать сам AI-development workflow после повторяющихся ошибок?

Это **не** просто:

- vibe coding;
- один AGENTS.md;
- один SDD framework;
- один RAG/vector search;
- один агент-оркестратор;
- набор промптов в чате.

Это связка нескольких независимых слоёв с разными источниками истины, артефактами, lifecycle и проверками.

---

## 3. Главная архитектурная модель

Не стоит рисовать процесс как простой pipeline:

```text
Memory → Intent → Execution → Retrieval → Verification → Self-Optimization → Freshness
```

Так неправильно, потому что **Retrieval используется везде**: при создании change spec, при планировании, при реализации, при ревью, при freshness check.

Более строгая модель:

```text
                         ┌──────────────────────────┐
                         │ Stable Knowledge Plane    │
                         │ facts / ADR / specs / docs│
                         └────────────┬─────────────┘
                                      │
User request / issue                  │
        ↓                             │
┌──────────────────┐       uses       ┌──────────────────────┐
│ Change Control   │◄────────────────►│ Context Access Plane  │
│ proposal/spec    │                  │ search/index/MCP/RAG  │
└────────┬─────────┘                  └──────────────────────┘
         ↓
┌──────────────────┐
│ Execution Plane  │
│ run/agent/phases │
└────────┬─────────┘
         ↓
┌──────────────────┐
│ Evidence Plane   │
│ tests/checks/logs│
└────────┬─────────┘
         ↓
       Merge
         ↓
┌──────────────────────────┐
│ Knowledge Reconciliation │
│ freshness / drift / PRs  │
└────────┬─────────────────┘
         ↓
┌──────────────────────────┐
│ Process Learning Plane   │
│ retros / evals / patches │
└──────────────────────────┘
```

Ключевая мысль: это не линейная фабрика, а **замкнутый engineering control loop**.

---

## 4. Строгая терминология

| Исходный термин | Более строгий термин | Ответственность |
|---|---|---|
| Project Memory Layer | **Stable Knowledge Plane** | Curated, versioned project knowledge |
| Intent / Change Layer | **Change Control Plane** | Delta intent: что меняем, зачем, критерии готовности |
| Execution Orchestration Layer | **Execution Plane** | Как агент исполняет approved change |
| Context Retrieval Layer | **Context Access Plane** | Search, indexing, context packaging, MCP |
| Verification Layer | **Evidence & Quality Plane** | Tests, checks, reviews, proof of correctness |
| Self-Optimization Layer | **Process Learning Plane** | Улучшение prompts, policies, templates, gates |
| Freshness Watcher Layer | **Knowledge Reconciliation Plane** | Drift detection, invalidation, update proposals |

Дополнительные термины:

| Термин | Значение |
|---|---|
| **Stable Spec** | Описание текущего intended behavior / capability системы. |
| **Change Spec** | Предложение изменения: delta от текущего состояния. |
| **Run** | Конкретная попытка выполнить change. |
| **Evidence** | Проверяемые факты: test output, CI logs, diff, command output, screenshots, traces. |
| **Fact** | Маленькое атомарное знание о проекте с owner/evidence/freshness rules. |
| **Index** | Производный артефакт для поиска. Не source of truth. |
| **Gate** | Условие, без прохождения которого нельзя merge/ship. |
| **Reconciliation** | Приведение memory/spec/docs/contracts в соответствие после изменения. |
| **Promotion** | Перенос знания из run/chat/search в curated memory. |
| **Invalidation** | Пометка знания как stale/deprecated/superseded. |
| **Waiver** | Явное объяснение, почему конкретное обновление/check не требуется. |
| **Process Patch** | Предложение изменить workflow, template, AGENTS.md, CI gate или eval. |

---

## 5. Базовые границы

Главные границы:

```text
Memory ≠ Search
Search ≠ Spec
Spec ≠ Execution
Execution ≠ Verification
Verification ≠ Freshness
Freshness ≠ Self-Optimization
Run Log ≠ Trusted Memory
Index ≠ Source of Truth
```

Расшифровка:

| Граница | Почему важна |
|---|---|
| **Memory ≠ Search** | Memory — curated knowledge. Search — способ найти evidence/context. |
| **Search ≠ Spec** | Найденный код или doc не говорит, что мы хотим изменить. |
| **Spec ≠ Execution** | Spec описывает намерение, execution описывает путь выполнения. |
| **Execution ≠ Verification** | То, что агент что-то сделал, не значит, что это правильно. |
| **Verification ≠ Freshness** | Тесты могут пройти, но docs/specs/facts могут протухнуть. |
| **Freshness ≠ Self-Optimization** | Freshness чинит знания проекта. Self-optimization чинит процесс разработки. |
| **Run Log ≠ Trusted Memory** | Run log — evidence/history, но не stable truth. |
| **Index ≠ Source of Truth** | Vector DB/search index можно пересоздать. Ему нельзя доверять как первоисточнику. |

---

## 6. Слои системы

### 6.1. Stable Knowledge Plane

Отвечает за долгосрочную curated-память проекта.

Что хранит:

```text
Project Rules
  AGENTS.md / CLAUDE.md / coding rules / agent protocol

Product Knowledge
  product.md / domain.md / glossary

Architecture Knowledge
  architecture.md / C4 / ADR

Capability Specs
  specs/current/*

Interface Contracts
  OpenAPI / AsyncAPI / protobuf / DB schema

Operational Knowledge
  runbooks / deployment / incidents / operations.md

Atomic Facts
  docs/facts/*.fact.md
```

Важно: это не один тип памяти. У каждого вида знания своя authority.

Примеры артефактов:

```text
AGENTS.md
CLAUDE.md
docs/context/product.md
docs/context/domain.md
docs/context/architecture.md
docs/context/tech.md
docs/context/operations.md
docs/context/active-context.md
docs/facts/auth-session-model.fact.md
docs/facts/billing-provider.fact.md
docs/facts/deployment-model.fact.md
docs/adr/0001-use-postgres.md
docs/adr/0002-use-refresh-tokens.md
specs/current/auth/session-model.md
contracts/openapi/public-api.yaml
```

Правила:

1. Stable knowledge обновляется только через reviewed PR.
2. Агент может предлагать изменения, но не должен молча переписывать trusted memory.
3. Если знание найдено поиском и оказалось важным, его надо промотировать в curated memory.
4. Если memory entry устарела, её надо пометить как `superseded`, `deprecated`, `challenged` или `invalid`.
5. Длинные memory-файлы быстро деградируют. Лучше много маленьких facts, чем один огромный `project-context.md`.

---

### 6.2. Change Control Plane

Отвечает за намерение изменения.

Вопросы:

1. Что именно мы хотим изменить?
2. Зачем?
3. Что входит в scope?
4. Что явно не входит в scope?
5. Какие acceptance criteria?
6. Какие области затронуты?
7. Какие stable specs/contracts/facts/ADR нужно обновить?
8. Как мы поймём, что change можно закрыть?

Пример структуры:

```text
changes/
  add-refresh-token-auth/
    proposal.md
    requirements.md
    design.md
    tasks.md
    acceptance.md
    verification.md
    memory-updates.md
    status.yaml
```

Разделение:

```text
Stable Spec
  Текущее intended behavior системы.

Change Spec
  Delta: что хотим изменить относительно текущего состояния.
```

Важная правка: **OpenAPI / AsyncAPI / protobuf — это не intent layer**. Это stable contracts. Change может предлагать их изменить, но сами они живут в contract/stable layer.

---

### 6.3. Execution Plane

Отвечает за контроль исполнения.

Вопросы:

1. Как агент выполняет change?
2. Какие фазы работы?
3. Какие разрешения есть у агента?
4. Где остановки на review?
5. Как защищаемся от context rot?
6. Как фиксируем решения и отклонения от плана?
7. Где хранится execution evidence?

Примеры подходов:

```text
GSD / phased execution
BMAD-like roles
worktree-per-agent
gstack-like role separation
fresh-context review agents
review gates
```

Execution отвечает:

```text
Как довести изменение до конца?
```

Но не отвечает:

```text
Правильный ли результат?
```

За это отвечает Evidence & Quality Plane.

---

### 6.4. Context Access Plane

Отвечает за поиск, индексирование и упаковку контекста.

Примеры:

```text
ripgrep
ast-grep
Sourcegraph Cody
Continue context providers
Repomix
RepoPrompt
vector search
GraphRAG
MCP servers
custom scripts
```

Функции:

```text
retrieval.find_related(change)
retrieval.find_implementations(symbol)
retrieval.find_tests(module)
retrieval.find_docs(api)
retrieval.find_owners(path)
retrieval.build_context_pack(change)
retrieval.find_impacted_facts(diff)
```

Главное правило:

```text
Context Access Plane ускоряет доступ к источникам истины, но сам не является источником истины.
```

Search result не становится memory. Search result может стать evidence для PR, который обновляет memory.

---

### 6.5. Evidence & Quality Plane

Отвечает за проверку результата.

Проверки:

```text
tests
lint
typecheck
contract tests
snapshot tests
e2e
security scan
AI review
spec compliance check
doc drift check
public API diff
migration safety check
performance regression check
```

Без этого SDD превращается в красивую бюрократию.

Главный вопрос:

```text
Можно ли считать изменение корректным и безопасным для merge/ship?
```

Verification evidence должно быть сохранено в run:

```text
runs/<run-id>/test-results.md
runs/<run-id>/commands.log
runs/<run-id>/review.md
runs/<run-id>/verification.md
```

---

### 6.6. Knowledge Reconciliation / Freshness Plane

Отвечает за актуальность памяти после изменения реальности.

Вопросы:

1. Код изменился — какие docs/specs/ADR/facts могли устареть?
2. API поменялся — обновлён ли OpenAPI?
3. Stable spec всё ещё соответствует коду?
4. README говорит то же, что код?
5. Memory entry всё ещё актуальна?
6. Change закрыт — надо ли промотировать новое знание в stable memory?
7. Старое знание надо архивировать, invalidировать или supersede-ить?

Основные функции:

```text
1. Detect
   Найти изменение: git diff, PR, merged branch, changed spec, changed tests.

2. Impact map
   Понять, какие memory/spec/docs/checks могут быть затронуты.

3. Freshness check
   Проверить, не противоречит ли память текущему коду/spec/contracts.

4. Reindex
   Обновить search/vector/graph index.

5. Propose update
   Создать PR / patch / issue на обновление памяти.

6. Archive / invalidate
   Пометить старое знание как superseded, deprecated, invalid.
```

Важная граница:

```text
Retrieval ищет.
Watcher сравнивает реальность с памятью.
Self-optimization улучшает процесс.
```

Freshness watcher должен быть **deterministic-first, LLM-second**.

Детерминированные сигналы:

```text
git diff
timestamps
changed paths
CODEOWNERS
ownership.yaml
source priority
status: deprecated/superseded/invalid
PR status
schema validation
file dependencies
contract diff
```

LLM можно использовать для:

```text
semantic drift review
summarizing impact
suggesting doc/spec patches
classifying ambiguous freshness findings
```

Но LLM не должен сам молча переписывать trusted memory.

---

### 6.7. Process Learning / Self-Optimization Plane

Отвечает за улучшение самого AI-development workflow.

Это не про продуктовый код. Это про процесс.

Вопросы:

1. Какие задачи агент регулярно проваливает?
2. Какие инструкции работают плохо?
3. Какие specs слишком размытые?
4. Какие checks надо добавить?
5. Какие правила в AGENTS.md / CLAUDE.md надо изменить?
6. Какие memory entries вредят или создают drift?
7. Какие типы задач требуют другого execution protocol?
8. Какие retrieval heuristics часто не находят нужные файлы?
9. Какие gates ловят ошибки слишком поздно?

Примеры подслоёв:

```text
tracing / observability
eval harness
prompt / policy optimization
retrospective miner
process patcher
failure taxonomy
agent benchmark fixtures
```

Примеры инструментов:

```text
Langfuse
LangSmith
Braintrust
Phoenix
Promptfoo
DeepEval
Ragas
OpenEvals
DSPy
TextGrad
custom eval scripts
```

Правило:

```text
Self-optimization не должна сама молча переписывать trusted memory или operating protocol.
Она создаёт proposal/PR с evidence.
```

Пример:

```text
API endpoint changed, OpenAPI not updated
  → freshness problem

Agent 5 раз забывал обновить OpenAPI после API change
  → self-optimization problem
```

---

## 7. Authority model: кто кому главнее

Нужен явный `source-priority.md`, иначе watcher и агент будут принимать странные решения.

Пример модели authority:

```text
1. Runtime evidence / CI / tests
   Показывают, что реально проходит.

2. Public contracts
   OpenAPI, AsyncAPI, protobuf, SDK contracts.
   Если код изменился, а contract нет — это не автоматически stale contract,
   а potential breaking change / drift event.

3. Stable specs
   Описывают intended behavior.

4. Code
   Описывает implemented behavior.

5. ADR
   Описывает rationale на момент решения.
   ADR не обязан полностью описывать текущее поведение.

6. Docs/context/facts
   Curated knowledge for humans/agents.

7. Change specs
   Истина только внутри конкретного change lifecycle.

8. Run logs/chat/search results
   Evidence, но не truth.

9. Indexes/vector DB/GraphRAG
   Derived cache, никогда не truth.
```

Нюанс:

```text
Code главнее docs для вопроса: "что сейчас реализовано?"
Spec/contract главнее code для вопроса: "что должно быть сохранено?"
```

Поэтому конфликт `code vs spec` — это не повод автоматически обновить spec под код. Это drift event, который требует решения:

```text
- code wrong?
- spec stale?
- contract violated?
- docs incomplete?
- behavior intentionally changed but not documented?
```

---

## 8. Классы артефактов

### 8.1. Trusted stable artifacts

То, чему агент может доверять после загрузки контекста.

```text
AGENTS.md
CLAUDE.md
docs/context/*.md
docs/facts/*.fact.md
docs/adr/*.md
specs/current/**/*
contracts/**/*
runbooks/**/*
```

### 8.2. Proposed / delta artifacts

Это намерение изменить истину. Они не являются текущей stable truth.

```text
changes/<change-id>/*
```

### 8.3. Ephemeral execution artifacts

История исполнения, evidence, trace. Не становятся memory автоматически.

```text
runs/<run-id>/*
```

### 8.4. Derived artifacts

Их можно пересоздать.

```text
memory-index/*
repomix-output/*
vector-index/*
graph-index/*
impact-map.generated.json
context-pack.generated.md
```

Главное правило:

```text
Trusted memory обновляется только через reviewed PR.
Run logs, search results и chat summaries не становятся truth автоматически.
```

---

## 9. Предлагаемая структура репозитория

```text
/
  AGENTS.md
  CLAUDE.md

  .ai/
    policy/
      operating-protocol.md
      source-priority.md
      agent-permissions.md
    schemas/
      fact.schema.yaml
      change.schema.yaml
      run.schema.yaml
      freshness-rule.schema.yaml
      retrospective.schema.yaml
    templates/
      fact.md
      change/
        proposal.md
        requirements.md
        design.md
        tasks.md
        acceptance.md
        verification.md
        memory-updates.md
      run/
        plan.md
        execution-log.md
        verification.md
        retrospective.md
    scripts/
      new-change
      validate-change
      impact-map
      freshness-check
      reindex
      promote-memory
      retro-summary

  docs/
    context/
      product.md
      domain.md
      architecture.md
      tech.md
      operations.md
      active-context.md
    facts/
      auth-session-model.fact.md
      billing-provider.fact.md
      deployment-model.fact.md
    adr/
      0001-use-postgres.md
      0002-use-refresh-tokens.md

  specs/
    current/
      auth/
        session-model.md
        token-lifecycle.md
      billing/
      onboarding/
    archived/

  contracts/
    openapi/
      public-api.yaml
    asyncapi/
    protobuf/

  changes/
    add-refresh-token-auth/
      proposal.md
      requirements.md
      design.md
      tasks.md
      acceptance.md
      verification.md
      memory-updates.md
      status.yaml

  runs/
    2026-06-05-add-refresh-token-auth/
      run.yaml
      plan.md
      execution-log.md
      commands.log
      test-results.md
      review.md
      retrospective.md

  checks/
    docs-updated.md
    spec-compliance.md
    public-api-contract.md
    no-architecture-change-without-adr.md
    memory-freshness.md
    security-baseline.md

  memory-index/
    manifest.yaml
    ownership.yaml
    freshness-rules.yaml
    generated/
      context-pack.md
      impact-map.json
      search-index.json
```

---

## 10. MVP для реального проекта

Главная рекомендация: **не начинать с Langfuse + GraphRAG + gstack + OpenSpec одновременно**.

Сначала нужен маленький kernel на markdown/git/CI. Иначе получится автоматизация хаоса.

### 10.1. MVP kernel

Минимальный процесс:

```text
1. Любое нетривиальное изменение начинается с changes/<id>/proposal.md.
2. До кода должны быть requirements.md + acceptance.md.
3. До merge должны быть verification.md + test evidence.
4. Если затронуты architecture/API/domain facts — обновляются specs/docs/facts/ADR или пишется waiver.
5. Watcher проверяет changed paths против ownership/freshness rules.
6. Agent не имеет права напрямую менять trusted memory без memory-updates.md.
7. После merge change архивируется, stable specs/facts обновляются.
8. После проблемного run пишется retrospective.md.
```

### 10.2. Минимальные команды

```bash
make change NAME=add-refresh-token-auth
make validate-change CHANGE=add-refresh-token-auth
make impact CHANGE=add-refresh-token-auth
make verify
make freshness
make reindex
make retro RUN=2026-06-05-add-refresh-token-auth
```

Или как npm scripts:

```json
{
  "scripts": {
    "ai:new-change": "node .ai/scripts/new-change.js",
    "ai:validate-change": "node .ai/scripts/validate-change.js",
    "ai:impact": "node .ai/scripts/impact-map.js",
    "ai:freshness": "node .ai/scripts/freshness-check.js",
    "ai:reindex": "node .ai/scripts/reindex.js",
    "verify": "npm run lint && npm run typecheck && npm test"
  }
}
```

### 10.3. Минимальный CI

```text
ci:
  - lint
  - typecheck
  - unit tests
  - integration tests where relevant
  - validate change schema
  - check PR references change-id
  - check architecture paths require ADR
  - check API paths require OpenAPI diff or explicit waiver
  - check changed code paths have matching freshness rules
  - check memory updates are proposed, not silently applied
```

### 10.4. Минимальный AGENTS.md protocol

```md
# Agent Operating Protocol

1. Before making non-trivial code changes, find the active change under /changes.
2. Read:
   - AGENTS.md
   - .ai/policy/operating-protocol.md
   - active change files
   - related facts/specs/ADR from memory-index/manifest.yaml
3. Do not modify trusted memory directly unless the change has memory-updates.md.
4. Keep execution evidence in /runs/<run-id>/.
5. After implementation, run verification commands from changes/<id>/verification.md.
6. If verification fails, stop and write failure evidence before fixing.
7. If code behavior contradicts docs/specs/facts, create a freshness finding.
8. Never treat search results, chat summaries, or run logs as source of truth.
```

---

## 11. Что можно сделать простыми средствами, а что требует инструментов

### 11.1. Markdown / Git / CI достаточно для

| Capability | Simple implementation |
|---|---|
| Stable memory | Markdown files + frontmatter |
| Change specs | `changes/<id>/*.md` |
| ADR | `docs/adr/*.md` |
| Acceptance criteria | Markdown checklist |
| Execution logs | `runs/<run-id>/*.md` |
| Verification | CI + test commands |
| Freshness MVP | path-based rules in YAML |
| Ownership | CODEOWNERS + `ownership.yaml` |
| Spec validation | script checking required sections |
| Promotion to memory | PR template + `memory-updates.md` |
| Archive/invalidate | frontmatter `status: superseded/deprecated/archived` |
| Context discovery | `rg`, `git grep`, simple AST search |

### 11.2. Специализированные инструменты нужны для

| Capability | When needed | Examples |
|---|---|---|
| Formal SDD workflow | Markdown templates уже не хватает | GitHub Spec Kit, OpenSpec, Kiro Specs |
| Agent orchestration | Длинные multi-phase/multi-agent задачи | GSD Core, gstack, BMAD-like methods |
| Large-codebase retrieval | Большой repo/monorepo/multi-repo | Sourcegraph, Continue providers, RepoPrompt, MCP |
| Structural search | Нужны code-aware queries/refactors | ast-grep |
| Context packaging | Нужно дать LLM controlled repo snapshot | Repomix, RepoPrompt |
| Semantic/graph retrieval | Много docs/specs/issues и plain search слаб | vector search, GraphRAG |
| LLM observability/evals | Много AI runs, надо мерить качество | Langfuse, Braintrust, LangSmith, Phoenix |
| Prompt/security evals | Нужно regression-test prompts/agents/RAG | Promptfoo, OpenEvals, DeepEval, Ragas |

---

## 12. Roadmap внедрения

### 12.1. Solo developer

Цель: discipline without bureaucracy.

Достаточно:

```text
AGENTS.md
changes/<id>/*
runs/<run-id>/*
docs/facts/*.fact.md
docs/adr/*.md
specs/current/*
checks/*
memory-index/manifest.yaml
```

Минимальные правила:

```text
1. Нет change spec — нет большой задачи для агента.
2. Нет acceptance criteria — агент не пишет код.
3. Нет verification evidence — change не закрыт.
4. Изменил architecture/API/domain behavior — обнови ADR/spec/fact или напиши waiver.
5. После каждого провала — retro с process patch.
```

Команды:

```text
/change:new
/change:impact
/run:plan
/run:execute
/verify
/freshness
/retro
```

Инструменты:

```text
markdown
git
CI
test runner
rg
ast-grep
Repomix/RepoPrompt optionally
```

Не нужно на этом этапе:

```text
heavy multi-agent roles
enterprise tracing
GraphRAG
сложный workflow engine
central knowledge platform
```

---

### 12.2. Small team

Цель: shared memory, reviewability, fewer silent agent mistakes.

Добавить:

```text
CODEOWNERS
PR template with change-id
required CI gates
required review for memory/spec/ADR changes
team-owned facts/specs
change approval before implementation
run logs for AI-generated work
freshness findings as PR comments
```

Процесс:

```text
1. Engineer/PM creates change proposal.
2. AI helps refine requirements/design/tasks.
3. Human approves scope.
4. Agent executes in branch/worktree.
5. CI verifies code + spec/freshness.
6. Reviewer checks diff + evidence + memory updates.
7. Merge archives change and promotes stable knowledge.
```

На этом этапе можно подключать:

```text
OpenSpec / Spec Kit / Kiro-like specs
stronger CI gates
CODEOWNERS-based freshness rules
basic evals for repeated agent failures
```

---

### 12.3. Larger engineering org

Цель: governance, compliance, scale.

Добавить:

```text
central schema registry
artifact ownership registry
policy-as-code
cross-repo search/index
contract diff automation
architecture decision workflow
LLM trace/eval platform
agent permission model
audit trail for AI-generated changes
dashboard for freshness debt
dashboard for process failure modes
```

Организационная модель:

```text
Developer repo
  local specs/facts/runs/checks

Platform service
  schemas / policies / evals / dashboards / indexes

CI/CD
  enforcement gates

Knowledge service
  source priority / ownership / freshness / search
```

Здесь уже уместны:

```text
Sourcegraph / MCP / GraphRAG for multi-repo context
Langfuse / Braintrust / LangSmith / Phoenix for observability
Promptfoo / OpenEvals / DeepEval / Ragas for evals
policy-as-code gates
central ownership registry
```

---

## 13. Practical MVP order

Внедрять лучше по стадиям.

```text
Stage 1: Artifact discipline
  - AGENTS.md
  - change template
  - run template
  - fact template
  - ADR template
  - PR template

Stage 2: Verification gates
  - lint/typecheck/test
  - change schema validation
  - required verification.md
  - required acceptance checklist

Stage 3: Freshness MVP
  - memory-index/manifest.yaml
  - path-based watcher
  - API → OpenAPI rule
  - architecture → ADR rule
  - code path → fact/spec review rule

Stage 4: Retrieval MVP
  - rg scripts
  - ast-grep for structural queries
  - repomix/context pack generation
  - optional MCP servers

Stage 5: Execution orchestration
  - phased prompts first
  - GSD-like execution only if tasks span many phases/sessions
  - role separation only when product/review/governance complexity justifies it

Stage 6: Self-optimization
  - retrospective template
  - failure taxonomy
  - process patch PRs
  - small eval suite for recurring agent failures

Stage 7: Advanced platform
  - traces
  - dashboards
  - LLM evals
  - vector/graph search
  - org-level policy enforcement
```

---

## 14. Schema: memory/facts

Факт должен быть маленьким, проверяемым и иметь freshness rules.

```md
---
id: fact.auth.session-model
title: Auth session model
type: fact
domain: auth
status: active # active | proposed | challenged | deprecated | superseded | invalid | archived
authority: curated
owner: team-auth
created: 2026-06-05
updated: 2026-06-05
review_after: 2026-09-05

source_of_truth:
  - path: src/auth/session.ts
    kind: code
  - path: specs/current/auth/session-model.md
    kind: stable-spec
  - path: docs/adr/0002-use-refresh-tokens.md
    kind: adr

evidence:
  - commit: abc123
    note: Session model introduced
  - test: tests/auth/session.test.ts

related:
  specs:
    - specs/current/auth/session-model.md
  adr:
    - docs/adr/0002-use-refresh-tokens.md
  changes:
    - changes/add-refresh-token-auth

freshness:
  watches:
    - src/auth/**
    - tests/auth/**
    - contracts/openapi/**
  trigger_on:
    - code_change
    - contract_change
    - test_change
  check:
    type: deterministic-plus-llm
    rule: "If token/session behavior changes, this fact must be reviewed."
  stale_if:
    - watched_files_changed_without_fact_review
    - related_stable_spec_changed
  required_action: propose_update_pr

supersedes: []
superseded_by: null
confidence: high # high | medium | low
---

# Fact

The system uses short-lived access tokens and long-lived refresh tokens.

# Details

...

# Implications

- Access token validation must not hit the database.
- Refresh token rotation must be persisted.
- Logout invalidates the refresh token family.

# Non-goals

...

# Freshness notes

Last reviewed against `src/auth/session.ts` at commit `abc123`.
```

Правила для facts:

```text
1. Один fact — один смысловой вопрос.
2. Fact должен иметь owner.
3. Fact должен иметь source_of_truth/evidence.
4. Fact должен иметь freshness watches.
5. Если fact стал длиннее 1–2 экранов, его надо разбить.
6. Fact без evidence — подозрительная память.
7. Fact без freshness rules быстро протухает.
```

---

## 15. Schema: change/spec

Машиночитаемый вариант:

```yaml
id: change.add-refresh-token-auth
title: Add refresh token auth
status: draft # draft | proposed | approved | in_progress | implemented | verified | merged | archived | rejected
owner: leonid
created: 2026-06-05
updated: 2026-06-05
risk: medium # low | medium | high
change_type:
  - feature
  - api
  - security

problem:
  summary: "Current sessions expire without a refresh flow."
  user_impact: "Users must log in again after access token expiration."

goals:
  - "Add refresh token issuance."
  - "Rotate refresh tokens on use."
  - "Invalidate refresh token family on suspicious reuse."

non_goals:
  - "Do not add OAuth provider login."
  - "Do not redesign user roles."

affected_areas:
  code:
    - src/auth/**
    - src/users/**
  tests:
    - tests/auth/**
  contracts:
    - contracts/openapi/public-api.yaml
  specs:
    - specs/current/auth/session-model.md
  facts:
    - docs/facts/auth-session-model.fact.md
  adr:
    - docs/adr/0002-use-refresh-tokens.md

requirements:
  - id: req-001
    text: "System must issue refresh token on login."
    acceptance:
      - "Login response contains access_token and refresh_token."
      - "Refresh token is persisted hashed, not plaintext."

design:
  summary: "Introduce refresh token table and rotation endpoint."
  alternatives_considered:
    - option: "JWT-only refresh token"
      rejected_because: "No server-side revocation."
  migrations:
    - "Create refresh_tokens table."
  security:
    - "Hash refresh tokens at rest."
    - "Detect token reuse."
  observability:
    - "Log refresh token reuse events."

tasks:
  - id: task-001
    title: "Add refresh token persistence model"
    depends_on: []
    files_expected:
      - src/auth/refresh-token.ts
      - tests/auth/refresh-token.test.ts
  - id: task-002
    title: "Add /auth/refresh endpoint"
    depends_on: [task-001]

verification:
  required_commands:
    - "npm run lint"
    - "npm run typecheck"
    - "npm test -- --runInBand"
  required_tests:
    - "refresh token rotation"
    - "reuse detection"
    - "logout invalidation"
  required_reviews:
    - security
    - api-contract

memory_update_plan:
  stable_specs:
    - specs/current/auth/session-model.md
  facts:
    - docs/facts/auth-session-model.fact.md
  adr:
    - docs/adr/0002-use-refresh-tokens.md
  contracts:
    - contracts/openapi/public-api.yaml

exit_criteria:
  - "All acceptance criteria checked."
  - "Verification commands pass."
  - "OpenAPI updated or explicit no-contract-change note exists."
  - "Memory updates proposed."
```

Markdown-структура:

```text
proposal.md       why / scope / non-goals
requirements.md   user stories / acceptance
design.md         architecture / data / security / alternatives
tasks.md          executable plan
acceptance.md     user-visible acceptance criteria
verification.md   commands / tests / gates
memory-updates.md expected updates to stable knowledge
status.yaml       machine-readable state
```

---

## 16. Schema: run/execution log

Run — это не spec. Это запись конкретной попытки.

```yaml
run_id: run.2026-06-05.add-refresh-token-auth.001
change_id: change.add-refresh-token-auth
status: completed # planned | running | blocked | failed | completed | abandoned
branch: feature/add-refresh-token-auth
worktree: ../worktrees/add-refresh-token-auth
base_commit: abc123
head_commit: def456

agent:
  tool: claude-code # or codex/cursor/etc
  model: unknown-or-declared
  mode: supervised
  permissions:
    can_edit_code: true
    can_edit_memory: false
    can_run_tests: true
    can_push: false

loaded_context:
  required:
    - AGENTS.md
    - changes/add-refresh-token-auth/*
    - specs/current/auth/session-model.md
    - docs/facts/auth-session-model.fact.md
  retrieved:
    - src/auth/session.ts
    - tests/auth/session.test.ts

phases:
  - id: phase-001
    name: impact-analysis
    started_at: 2026-06-05T15:00:00Z
    completed_at: 2026-06-05T15:08:00Z
    outputs:
      - runs/2026-06-05-add-refresh-token-auth/impact.md
  - id: phase-002
    name: implementation
    started_at: 2026-06-05T15:09:00Z
    completed_at: 2026-06-05T16:20:00Z

decisions:
  - id: decision-001
    text: "Use opaque refresh tokens stored hashed."
    evidence:
      - docs/adr/0002-use-refresh-tokens.md

commands:
  - command: "npm test -- auth"
    exit_code: 0
    output_file: test-results.md

files_touched:
  code:
    - src/auth/refresh-token.ts
  tests:
    - tests/auth/refresh-token.test.ts
  docs:
    - specs/current/auth/session-model.md

deviations:
  - expected: "No DB migration"
    actual: "DB migration required"
    reason: "Refresh token storage needed"
    approved_by: leonid

verification_summary:
  lint: pass
  typecheck: pass
  unit_tests: pass
  integration_tests: not_run
  security_review: required

freshness_findings:
  - id: fresh-001
    artifact: docs/facts/auth-session-model.fact.md
    status: update_proposed

result:
  pr: null
  ready_for_review: true
```

Минимальный `execution-log.md`:

```md
# Execution Log

## Run metadata

- Run ID:
- Change ID:
- Branch:
- Base commit:
- Agent/tool:
- Permissions:

## Loaded context

- Required:
- Retrieved:

## Plan

...

## Actions

### Step 1

- What was done:
- Files changed:
- Evidence:

### Step 2

...

## Deviations from plan

...

## Verification

...

## Freshness findings

...

## Result

...
```

---

## 17. Schema: freshness watcher

Freshness watcher должен быть скучным и детерминированным в основе.

```yaml
version: 1

artifacts:
  - id: fact.auth.session-model
    path: docs/facts/auth-session-model.fact.md
    owner: team-auth
    status: active
    watches:
      - src/auth/**
      - tests/auth/**
      - contracts/openapi/**
      - specs/current/auth/**
    required_when_changed:
      - review_fact
      - update_or_waive
    severity: high

  - id: spec.auth.session-model
    path: specs/current/auth/session-model.md
    owner: team-auth
    watches:
      - src/auth/**
      - contracts/openapi/**
    severity: high

rules:
  - id: rule.api-change-requires-contract-review
    description: "API route changes require OpenAPI update or explicit waiver."
    trigger:
      changed_paths:
        - src/routes/**
        - src/controllers/**
    require_one_of:
      - changed_paths:
          - contracts/openapi/**
      - waiver:
          file: changes/*/verification.md
          heading: "API contract waiver"
    severity: blocking

  - id: rule.architecture-change-requires-adr
    description: "Architecture-sensitive paths require ADR update/proposal."
    trigger:
      changed_paths:
        - src/infrastructure/**
        - src/db/migrations/**
        - deployment/**
    require_one_of:
      - changed_paths:
          - docs/adr/**
      - waiver:
          file: changes/*/design.md
          heading: "ADR not required because"
    severity: blocking

  - id: rule.fact-review-on-watched-change
    description: "Facts watching changed files must be reviewed."
    trigger:
      changed_paths_from_manifest: true
    require:
      - reviewed_artifacts_listed_in: changes/*/memory-updates.md
    severity: warning
```

Watcher output:

```yaml
freshness_run_id: freshness.2026-06-05.001
base_commit: abc123
head_commit: def456

changed_paths:
  - src/auth/session.ts
  - src/auth/refresh-token.ts

impacted_artifacts:
  - path: docs/facts/auth-session-model.fact.md
    reason: "watches src/auth/**"
    severity: high
    status: needs_review
  - path: specs/current/auth/session-model.md
    reason: "watches src/auth/**"
    severity: high
    status: update_proposed

findings:
  - id: fresh-001
    type: stale_or_unreviewed_memory
    artifact: docs/facts/auth-session-model.fact.md
    evidence:
      - "src/auth/refresh-token.ts added"
      - "fact last reviewed before this commit"
    proposed_action: "Update fact or add explicit waiver."
```

---

## 18. Schema: self-optimization retrospective

Self-optimization не должна писать в `AGENTS.md` напрямую. Она создаёт patch proposal.

```md
---
id: retro.2026-06-05.add-refresh-token-auth
run_id: run.2026-06-05.add-refresh-token-auth.001
change_id: change.add-refresh-token-auth
status: proposed # proposed | accepted | rejected | implemented | measured
owner: leonid
created: 2026-06-05
severity: medium
---

# Summary

Agent completed implementation but initially forgot to update OpenAPI and stable auth spec.

# What went well

- Unit tests were added before endpoint implementation.
- Refresh token reuse case was covered.

# What failed

- API contract update was missed.
- Existing fact `auth-session-model.fact.md` was not loaded during first implementation phase.

# Evidence

- `contracts/openapi/public-api.yaml` unchanged while `src/routes/auth.ts` changed.
- Freshness watcher finding `fresh-001`.
- Run log phase `implementation` did not list related fact in loaded context.

# Failure mode classification

- context-selection-failure
- missing-contract-gate
- memory-not-promoted

# Root cause

The change template asks for affected code/tests but does not explicitly ask for affected contracts/facts.

# Proposed process patches

## Patch 1: change template

Add required section:

```md
## Affected contracts and stable knowledge

- Contracts:
- Stable specs:
- Facts:
- ADR:
```

## Patch 2: CI gate

Add blocking rule:

```yaml
api-change-requires-contract-review: blocking
```

## Patch 3: AGENTS.md

Add rule:

```md
When editing API routes/controllers, inspect contracts/openapi and update or write waiver.
```

# Proposed eval

Create a fixture task:

```text
Given an API route change, agent must identify OpenAPI as impacted.
```

# Expected measurement

Next 5 API-related changes should have zero missing contract findings.
```

---

## 19. Lifecycles

### 19.1. Change lifecycle

```text
draft
  ↓
proposed
  ↓
approved
  ↓
in_progress
  ↓
implemented
  ↓
verified
  ↓
merged
  ↓
archived
```

Rules:

```text
draft/proposed:
  agent may edit change spec

approved:
  scope locked unless explicit scope-change note

in_progress:
  run log required

implemented:
  code done, verification not yet passed

verified:
  evidence attached

merged:
  code in main

archived:
  change converted into stable specs/facts/ADR where needed
```

### 19.2. Fact lifecycle

```text
proposed → active → challenged → superseded/deprecated/invalid → archived
```

Rules:

```text
active:
  agent may read and rely on it

challenged:
  agent may read but must verify against source files

superseded:
  must point to new fact/spec/ADR

invalid:
  should not be used as context

archived:
  historical only
```

### 19.3. Run lifecycle

```text
planned → running → blocked/failed/completed → reviewed → archived
```

Run logs should be mostly append-only. Не нужно переписывать историю, чтобы агент выглядел умнее.

### 19.4. Freshness lifecycle

```text
detect → impact map → check → finding → proposed patch/waiver → review → resolved
```

### 19.5. Self-optimization lifecycle

```text
retro → failure classification → process patch proposal → eval/check added → measured → accepted/rejected
```

---

## 20. Checks / gates

Примеры проверок в `checks/`:

```text
checks/docs-updated.md
checks/spec-compliance.md
checks/public-api-contract.md
checks/no-breaking-public-api.md
checks/no-architecture-change-without-adr.md
checks/memory-freshness.md
checks/security-baseline.md
checks/no-unreviewed-trusted-memory-change.md
checks/change-id-required.md
checks/verification-evidence-required.md
```

Пример `checks/public-api-contract.md`:

```md
# Public API Contract Check

## Purpose

Ensure public API behavior is not changed without contract review.

## Trigger

Changed paths:

- `src/routes/**`
- `src/controllers/**`
- `src/api/**`

## Required evidence

One of:

1. OpenAPI/contract file updated.
2. Explicit waiver in `changes/<id>/verification.md` under heading `API contract waiver`.
3. Reviewer approval from API owner.

## Blocking?

Yes.
```

Пример `checks/no-architecture-change-without-adr.md`:

```md
# Architecture Change Requires ADR

## Trigger

Changed paths:

- `src/infrastructure/**`
- `src/db/migrations/**`
- `deployment/**`
- `terraform/**`
- `k8s/**`

## Required evidence

One of:

1. New ADR.
2. Updated ADR.
3. Waiver in `changes/<id>/design.md`.

## Blocking?

Yes for team/org, warning for solo developer.
```

---

## 21. PR template

```md
# PR Summary

## Change ID

- `changes/<id>`:

## What changed

...

## Why

...

## Acceptance criteria

- [ ] ...
- [ ] ...

## Verification evidence

Commands run:

- [ ] `npm run lint`
- [ ] `npm run typecheck`
- [ ] `npm test`

Evidence files:

- `runs/<run-id>/test-results.md`
- `runs/<run-id>/review.md`

## Stable knowledge updates

- [ ] Specs updated or waived
- [ ] Facts updated or waived
- [ ] ADR updated or waived
- [ ] Contracts updated or waived

## Freshness findings

- Finding IDs:
- Resolution:

## AI execution disclosure

- Agent/tool used:
- Run log:
- Human review performed:
```

---

## 22. Operating protocol для агента

Минимальный агентский protocol:

```md
# Agent Operating Protocol

## Before implementation

1. Locate active change in `/changes`.
2. Read `proposal.md`, `requirements.md`, `design.md`, `tasks.md`, `acceptance.md`, `verification.md`.
3. Build impact map:
   - code
   - tests
   - specs
   - contracts
   - facts
   - ADR
4. Read relevant stable knowledge.
5. Create or update `/runs/<run-id>/plan.md`.
6. Do not start code changes until acceptance criteria are clear.

## During implementation

1. Work task-by-task.
2. Keep changes small.
3. Record deviations from plan.
4. Add tests close to changed behavior.
5. Do not silently expand scope.
6. Do not rewrite trusted memory without `memory-updates.md`.

## After implementation

1. Run verification commands.
2. Save outputs into run folder.
3. Update `verification.md` evidence.
4. Run freshness check.
5. Propose memory/spec/contract updates or waivers.
6. Write retrospective if there was failure, scope drift, repeated confusion, missing context, or broken check.
```

---

## 23. Failure taxonomy для retrospectives

Пример классификации ошибок:

```text
context-selection-failure
  Агент не загрузил нужный файл/spec/fact/ADR.

spec-ambiguity
  Change spec был слишком размытым.

acceptance-missing
  Не было проверяемых acceptance criteria.

verification-gap
  Не было test/check, который ловит ошибку.

contract-drift
  API/contract изменился без обновления contract artifact.

memory-drift
  Docs/facts/specs устарели относительно кода или intended behavior.

architecture-drift
  Изменение затронуло архитектуру без ADR/design review.

scope-creep
  Агент сделал больше, чем было в change spec.

premature-implementation
  Агент начал кодить до уточнения design/requirements.

agent-looping
  Агент повторял одну и ту же неудачную стратегию.

tool-misuse
  Агент использовал неправильный инструмент или команду.

insufficient-evidence
  Change был объявлен готовым без доказательств.

unsafe-memory-write
  Агент изменил trusted memory без review/evidence.
```

---

## 24. Как выглядит один полный flow

```text
User request / Issue
  ↓
Create changes/<id>/proposal.md
  ↓
Clarify requirements + acceptance criteria
  ↓
Create design + tasks + verification plan
  ↓
Impact map finds related code/specs/facts/contracts/ADR
  ↓
Human approves change scope
  ↓
Agent creates run folder
  ↓
Agent executes task-by-task
  ↓
Agent runs tests/checks
  ↓
Verification evidence saved
  ↓
Freshness watcher checks impacted stable knowledge
  ↓
Agent proposes memory/spec/contract/ADR updates or waivers
  ↓
Human review
  ↓
Merge
  ↓
Archive change/run
  ↓
Retrospective if process failed
  ↓
Process patch/eval/check if repeated failure mode detected
```

---

## 25. Example: add refresh token auth

```text
Change:
  changes/add-refresh-token-auth/

Touched code:
  src/auth/session.ts
  src/auth/refresh-token.ts
  src/routes/auth.ts

Touched tests:
  tests/auth/refresh-token.test.ts

Impacted stable knowledge:
  specs/current/auth/session-model.md
  docs/facts/auth-session-model.fact.md
  docs/adr/0002-use-refresh-tokens.md
  contracts/openapi/public-api.yaml

Verification:
  npm run lint
  npm run typecheck
  npm test -- auth
  security review
  OpenAPI diff review

Freshness findings:
  fact.auth.session-model needs review
  spec.auth.session-model needs update
  public-api contract needs update

Self-optimization possible finding:
  If agent forgot OpenAPI update, add API contract gate to CI/template.
```

---

## 26. Anti-patterns

### 26.1. One giant memory file

Плохо:

```text
docs/project-memory.md
```

Почему плохо:

```text
- быстро протухает
- трудно ревьюить
- трудно ownership назначить
- агент тащит лишний контекст
- нет freshness granularity
```

Лучше:

```text
docs/facts/auth-session-model.fact.md
docs/facts/billing-provider.fact.md
docs/facts/deployment-model.fact.md
specs/current/auth/session-model.md
docs/adr/0002-use-refresh-tokens.md
```

### 26.2. Search as truth

Плохо:

```text
Агент нашёл старую заметку через vector search и поверил ей.
```

Лучше:

```text
Агент проверяет status/evidence/source_of_truth/freshness before use.
```

### 26.3. Spec as bureaucracy

Плохо:

```text
Написали красивый spec, потом агент всё равно сделал по-своему, tests нет.
```

Лучше:

```text
Spec → tasks → verification plan → checks → evidence.
```

### 26.4. Self-optimization as hallucinated improvement

Плохо:

```text
LLM решил, что AGENTS.md надо переписать, и сам переписал.
```

Лучше:

```text
Retro → evidence → proposed process patch → eval/check → human review.
```

### 26.5. Freshness as vibes

Плохо:

```text
LLM "на глаз" решил, что docs вроде актуальны.
```

Лучше:

```text
changed paths + ownership + manifest + deterministic rules + optional LLM semantic review.
```

---

## 28. Возможная лицензия

Для open-source framework/tooling разумные варианты:

```text
MIT
  Максимально permissive. Хорошо для adoption.

Apache-2.0
  Permissive, но с явным patent grant. Лучше для engineering/platform tooling.

MPL-2.0
  Weak copyleft. Хорошо, если хочется, чтобы изменения самого framework-файла/ядра возвращались назад,
  но без заражения всего пользовательского проекта.

AGPL-3.0
  Сильный copyleft, включая network use. Может отпугнуть компании.
```

Практичная рекомендация:

```text
Apache-2.0
```

Почему:

```text
- хорошо воспринимается компаниями
- permissive
- есть patent grant
- подходит для devtools/platform/framework проекта
- не мешает adoption
```

Если цель — максимально быстрое распространение среди разработчиков, можно взять MIT. Если хочется выглядеть серьёзнее для devtools/org adoption — Apache-2.0 лучше.

---

## 29. Минимальный README skeleton для такого проекта

```md
# Octobend

Repo-native control framework for AI-assisted software development.

Octobend helps teams move from vibe coding to controlled AI engineering by managing:

- stable project memory
- change specs
- execution runs
- context retrieval
- verification gates
- freshness checks
- process retrospectives

## Core idea

AI agents should not work from chat memory alone. Important project knowledge, change intent, execution evidence, and process improvements should be versioned in the repository.

## Planes

1. Stable Knowledge Plane
2. Change Control Plane
3. Execution Plane
4. Context Access Plane
5. Evidence & Quality Plane
6. Knowledge Reconciliation Plane
7. Process Learning Plane

## Repository layout

...

## Quickstart

```bash
make change NAME=add-refresh-token-auth
make impact CHANGE=add-refresh-token-auth
make verify
make freshness
```

## Principles

1. Retrieval is not truth.
2. Run logs are not memory.
3. Trusted memory changes require review.
4. Every non-trivial change needs acceptance criteria.
5. Every completed change needs evidence.
6. Every stale knowledge finding needs update or waiver.
7. Every repeated failure needs a process patch or eval.
```

---

## 30. Итоговая оценка архитектуры

Архитектура сильная, но с важными правками:

```text
1. Retrieval — не этап после execution, а сервис для всех слоёв.
2. Memory надо разбить по authority: rules, facts, ADR, stable specs, contracts.
3. OpenAPI/AsyncAPI — это stable contracts, не intent layer.
4. Verification и Freshness — это gates/feedback loops, а не просто этапы.
5. Self-optimization должна менять процесс только через proposed patches/evals.
6. Watcher должен быть deterministic-first, LLM-second.
7. Change spec после merge должен либо архивироваться, либо промотировать stable specs/facts.
8. Run logs — evidence, не memory.
```

Самое короткое определение системы:

```text
AI Engineering Control System is a repo-native framework that turns AI-assisted coding into a controlled loop of stable knowledge, explicit change intent, supervised execution, evidence-based verification, knowledge reconciliation, and process learning.
```

Это уже не vibe coding. Это нормальный engineering-процесс для AI-assisted development.
