# AI-assisted development: артефакты проекта, операции агента и control model

> Рабочий markdown по итогам чата.
> Фокус: не название репозитория, а проблематика AI-разработки, разделение проектных артефактов и задач кодингового агента, а также критическая оценка задумки.

---

## 1. Исходная проблема

Современная разработка с AI coding agents часто выглядит как ускоренный `vibe coding`:

```text
чат → промпт → агент пишет код → тесты зелёные → merge
```

Главная проблема не в том, что агент плохо пишет код. Главная проблема в том, что вокруг агента часто нет инженерной системы управления:

- намерение живёт в чате и исчезает;
- знания проекта размазаны по README, коду, ADR, docs, issues и памяти людей;
- агент теряет контекст на длинных задачах;
- search / RAG / vector index ошибочно принимаются за источник истины;
- specs, docs, ADR, contracts и код расходятся;
- verification часто сводится к “tests passed”;
- агентские runs не превращаются в reusable knowledge;
- повторяющиеся ошибки процесса не приводят к улучшению процесса;
- временные prompts/scripts/checklists создаются ad hoc и потом теряются.

Изначальная идея была: построить repo-native control system для AI-assisted software development.

Но в ходе обсуждения стало понятно, что нельзя всё сваливать в одну коробку под названием `SDD framework`, `agent framework` или `AI dev framework`.

---

## 2. Первый важный разворот: не гоняться за фреймворками

Была сформулирована идея:

```text
Не строить систему вокруг OpenSpec, GSD, Codex, Claude Code или другого конкретного инструмента.
Строить систему вокруг концептуальных проблем AI-разработки.
```

Фреймворки меняются. Сегодня OpenSpec, завтра GSD, послезавтра Codex или Claude Code встроят часть этого поведения внутрь себя.

Но концептуальные проблемы остаются:

- intent loss;
- context loss;
- authority confusion;
- impact blindness;
- premature implementation;
- execution drift;
- shallow verification;
- knowledge drift;
- process non-learning;
- unsafe agency;
- tooling friction.

Правильная иерархия на этом этапе выглядела так:

```text
Conceptual Problem
  ↓
Capability
  ↓
Recipe
  ↓
Artifact Contract
  ↓
Adapter
  ↓
Concrete Tool
```

Пример:

```text
Problem:
  Intent disappears in chat.

Capability:
  Intent Management.

Artifacts:
  change-spec.md
  acceptance.md
  open-questions.md

Adapters:
  OpenSpec
  Spec Kit
  Markdown templates
  GitHub issue templates
```

Или:

```text
Problem:
  Agent execution must be controlled.

Capability:
  Execution Control.

Artifacts:
  execution-plan.md
  run-log.md
  changed-files.md
  commands-run.md

Adapters:
  GSD
  Codex
  Claude Code
  Cline/Roo
  worktree-per-agent
```

Ключевая мысль:

```text
OpenSpec и GSD не конкурируют.
Они закрывают разные conceptual problems.
```

OpenSpec ближе к:

```text
Intent Management / Change Spec / Stable Spec
```

GSD ближе к:

```text
Execution Control / Run Management / Agent Workflow
```

Codex / Claude Code ближе к:

```text
Executor / Implementation Adapter
```

CI ближе к:

```text
Verification Adapter
```

MCP / RAG ближе к:

```text
Context Access Adapter
```

---

## 3. Черновая таксономия проблем AI-разработки

На этом этапе были выделены концептуальные проблемы.

### 3.1. Intent Loss

Намерение живёт только в чате и исчезает.

Симптомы:

- нет scope;
- нет out-of-scope;
- нет acceptance criteria;
- непонятно, почему решение принято;
- будущий агент не может восстановить контекст.

Решение:

```text
change spec
requirements
acceptance criteria
open questions
out-of-scope
```

---

### 3.2. Scope Ambiguity

Агент не понимает границы задачи.

Симптомы:

- делает больше, чем просили;
- трогает соседние модули;
- начинает рефакторить по пути;
- меняет архитектуру без явного решения.

Решение:

```text
scope contract
non-goals
allowed files
blocked files
approval-required areas
```

---

### 3.3. Knowledge Authority Ambiguity

Агент не понимает, чему верить:

```text
README говорит одно,
код другое,
ADR третье,
старый spec четвёртое,
тесты пятое.
```

Решение:

```text
source-priority.md
authority matrix
fact lifecycle
status fields
evidence links
```

Важная мысль: authority зависит от вопроса.

```text
Вопрос: “что реально работает?”
  → code, tests, runtime evidence.

Вопрос: “что должно быть сохранено?”
  → contracts, stable specs, ADR.

Вопрос: “почему так решили?”
  → ADR, decision trace.

Вопрос: “что можно безопасно менять?”
  → policy, ownership, protected paths.
```

---

### 3.4. Context Selection Failure

Агент не загрузил нужный контекст или загрузил слишком много мусора.

Симптомы:

- не увидел важный тест;
- не нашёл ADR;
- пропустил contract;
- тащит половину репозитория в context;
- не может отличить релевантное от шумового.

Решение:

```text
context-pack.md
retrieval-report.md
gaps.md
source citations
context budget
```

Важное правило:

```text
Retrieval finds candidates.
It does not decide truth.
```

---

### 3.5. Impact Blindness

Агент меняет локальный участок, но не понимает blast radius.

Симптомы:

- изменил auth, но не обновил session docs;
- изменил route, но не проверил OpenAPI;
- изменил schema, но не проверил migration path;
- изменил доменную модель, но не обновил specs/tests.

Решение:

```text
impact-report.md
affected-areas.yaml
test-selection.yaml
spec-to-code-map.yaml
owner map
dependency map
```

Важно:

```text
Retrieval finds related things.
Impact analysis decides what is affected.
```

---

### 3.6. Premature Implementation

Агент слишком рано начинает писать код.

Симптомы:

- нет acceptance criteria;
- нет design choice;
- нет risk analysis;
- нет affected areas;
- агент уже меняет файлы.

Решение:

```text
No implementation before:
  - change intent exists;
  - acceptance criteria exist;
  - impact analysis exists for risky changes;
  - verification plan exists.
```

---

### 3.7. Execution Drift

Агент начал правильно, но в процессе ушёл в сторону.

Симптомы:

- scope creep;
- unplanned refactor;
- looping;
- changing unrelated files;
- fixing failures by weakening tests.

Решение:

```text
execution-plan.md
phase checkpoints
deviation log
changed-files.md
stop conditions
human review points
```

Дополнительное правило:

```text
Every deviation must be classified:
  expected
  harmless
  requires approval
  rollback required
```

---

### 3.8. Verification Shallowness

“Tests passed” считают достаточным доказательством.

Симптомы:

- unit tests passed, but contract broken;
- typecheck passed, but acceptance not satisfied;
- docs not updated;
- security not checked;
- migration not verified.

Решение:

```text
verification-report.md
evidence.yaml
acceptance checklist
contract check
security check
docs drift check
manual review when needed
```

---

### 3.9. Evidence Gap

Даже если агент сделал правильную вещь, нет доказательств.

Симптомы:

- “готово”;
- “я проверил”;
- “должно работать”;
- нет команд, логов, diff summary, test output, CI links.

Решение:

```text
evidence ledger
commands-run.md
test-results.md
decision-trace.md
verification-report.md
```

Правило:

```text
No final answer / PR completion without claim-evidence summary.
```

---

### 3.10. Knowledge Drift

Код поменялся, а знания проекта нет.

Симптомы:

- docs stale;
- facts stale;
- stable specs stale;
- ADR partially obsolete;
- OpenAPI not updated.

Решение:

```text
freshness watcher
stale-facts.yaml
docs/spec update proposal
waiver
reconciliation PR
```

Важное правило:

```text
Freshness watcher should be deterministic-first, LLM-second.
```

Детерминированные сигналы:

```text
git diff
changed paths
CODEOWNERS
ownership.yaml
contract diff
schema validation
status fields
file dependencies
```

LLM может помогать объяснять impact или черновить patch, но не должен сам молча переписывать trusted memory.

---

### 3.11. Conflict Resolution Problem

Источники проекта противоречат друг другу.

Примеры:

```text
code vs spec
spec vs ADR
contract vs implementation
tests vs new intended behavior
docs vs runtime behavior
two agents propose incompatible changes
```

Решение:

```text
conflict-report.md
source-comparison.md
decision-needed.md
resolution-options.md
```

Правило:

```text
Agent must not resolve authority conflicts by vibes.
```

Он должен показать варианты:

```text
Option A: code is wrong → fix code
Option B: spec is stale → update spec
Option C: behavior changed intentionally → update contract + ADR
Option D: conflict requires human/product decision
```

---

### 3.12. Process Non-Learning

Агент повторяет одну и ту же ошибку, но процесс не улучшается.

Симптомы:

- пять раз забыли OpenAPI;
- три раза начали кодить без acceptance;
- регулярно не загружается нужный ADR;
- постоянно не хватает одного check.

Решение:

```text
retrospective.md
failure taxonomy
process-patch.md
eval fixture
template update
new gate proposal
```

Правило:

```text
Learning is not silent mutation.
```

Агент не должен сам переписывать trusted memory или AGENTS.md. Он должен предложить patch с evidence.

---

### 3.13. Tooling Friction

Агент снова и снова делает одно и то же вручную.

Симптомы:

- каждый раз ищет одни и те же файлы;
- каждый раз вручную строит context pack;
- каждый раз сравнивает docs и code;
- каждый раз пишет однотипные migration scripts.

Решение:

```text
JIT automation
scratch tool
codemod
audit script
check proposal
tool promotion lifecycle
```

Критерий promotion:

```text
Tool is promoted only if:
  - used at least N times;
  - has test/validation;
  - reduces measurable friction;
  - has owner;
  - has deletion policy.
```

---

### 3.14. Governance & Safety

Агент может сделать опасное действие.

Симптомы:

- лезет в secrets;
- меняет prod config;
- трогает auth/security без review;
- удаляет migration;
- исполняет risky shell commands;
- верит malicious instruction внутри repo.

Решение:

```text
policy.yaml
protected-paths.yaml
tool-permissions.yaml
approval-rules.yaml
security-review.md
threat-model.md
```

---

### 3.15. Runtime Feedback Gap

Агент сделал change, verification прошёл, но после релиза всё иначе.

Симптомы:

- tests green, production metric падает;
- feature flag не описан;
- rollback plan отсутствует;
- runtime incident не превращается в lessons/checks.

Решение:

```text
release-plan.md
rollback-plan.md
runtime-feedback.md
postmortem.md
new checks from incident
```

---

## 4. Вторая важная поправка: мы снова смешали разные сущности

После таксономии проблем была замечена ошибка: в один список начали попадать вещи разной природы.

Например:

```text
Intent Loss
Knowledge Authority
Context Discovery
Impact Analysis
ADR
Tech Stack
Verification
Domain Model
Core Classes
```

Но `ADR`, `tech-stack.md`, `domain.md`, `core-classes.md` — это не агентские задачи. Это артефакты проекта.

А `impact analysis`, `verification`, `context discovery` — это операции агента.

Поэтому нужно разделить две независимые оси:

```text
1. Project Artifact Model
   Что проект знает о себе.

2. Agent Operating Model
   Что агент делает с этими знаниями и где он ошибается.
```

И добавить третий слой:

```text
3. Control / Policy Binding
   Какие операции агента разрешены над какими артефактами.
```

Самая короткая формула:

```text
Project artifacts are nouns.
Agent tasks are verbs.
Policies define which verbs may act on which nouns.
```

По-русски:

```text
Артефакты проекта — это существительные.
Действия агента — это глаголы.
Политики задают, какие глаголы имеют право менять какие существительные.
```

---

## 5. Project Artifact Model

Это карта того, что проект знает, решил, обещает или содержит.

Примерная структура:

```text
project-knowledge/
  domain/
    domain-model.md
    glossary.md
    business-rules.md
    user-journeys.md

  product/
    product-context.md
    capabilities.md
    requirements-principles.md

  architecture/
    architecture-overview.md
    system-boundaries.md
    components.md
    adr/
      0001-use-postgres.md
      0002-refresh-token-model.md

  technical/
    tech-stack.md
    dependency-policy.md
    build-test-commands.md
    environments.md

  code-model/
    modules.md
    core-classes.md
    service-map.md
    ownership.md
    public-interfaces.md

  contracts/
    openapi.yaml
    asyncapi.yaml
    protobuf/
    db-schema.md

  specs/
    current/
      auth.md
      billing.md
      onboarding.md

  operations/
    runbooks/
    deployment.md
    monitoring.md
    incident-response.md

  facts/
    auth-session-model.fact.md
    billing-provider.fact.md
```

Эта часть отвечает на вопрос:

```text
Что проект знает / решил / обещает / содержит?
```

---

## 6. Change artifacts как bridge-zone

`changes/` лучше вынести отдельно.

Это не совсем stable knowledge, но и не просто лог агента.

Это активное состояние изменения проекта.

```text
changes/
  add-refresh-token-auth/
    intent.md
    requirements.md
    design.md
    acceptance.md
    affected-areas.md
    verification-plan.md
    status.yaml
```

Change отвечает на вопрос:

```text
Что мы сейчас хотим изменить?
```

Важная граница:

```text
Stable spec = текущее intended behavior.
Change spec = proposed delta.
```

После merge change не должен вечно оставаться “истиной”. Он либо архивируется, либо его содержание промотируется в stable specs / facts / docs / ADR / contracts.

Формула:

```text
changes/ = bridge between human intent, agent execution, and future stable knowledge
```

---

## 7. Agent Operating Model

Это уже не про домен проекта.

Это про то, как агент должен работать.

Примеры операций агента:

```text
understand intent
retrieve context
analyze impact
plan implementation
edit code
run checks
verify acceptance
detect stale docs
propose knowledge update
write retrospective
create helper tool
```

Они отвечают на вопрос:

```text
Как агент работает с проектом?
```

Возможная структура:

```text
.agent/
  modes/
    intent-clarification.md
    context-discovery.md
    impact-analysis.md
    implementation.md
    verification.md
    reconciliation.md
    retrospective.md
    toolsmith.md

  policies/
    source-authority.md
    write-permissions.md
    protected-paths.md
    evidence-policy.md
    artifact-lifecycle.md

  adapters/
    openspec.md
    gsd.md
    codex.md
    claude-code.md
    github-actions.md
    mcp.md

  templates/
    change-spec.md
    impact-report.md
    verification-report.md
    freshness-report.md
    retrospective.md
```

---

## 8. Три типа артефактов, которые нельзя смешивать

### 8.1. Knowledge artifacts

То, чему агент может учиться:

```text
domain
architecture
tech stack
code model
stable specs
contracts
facts
runbooks
```

---

### 8.2. Work artifacts

То, через что идёт изменение:

```text
changes
tasks
acceptance
design
verification plan
status
```

---

### 8.3. Evidence artifacts

То, что доказывает работу:

```text
runs
commands log
test results
verification report
review notes
freshness report
retrospective
```

---

### 8.4. Derived artifacts

То, что можно пересоздать:

```text
indexes
context packs
repo maps
search results
generated summaries
```

Правило:

```text
curated map = knowledge
generated index = access mechanism
```

---

## 9. Примеры: artifact ≠ task

### 9.1. ADR

`ADR` — это project artifact.

Агентские действия вокруг него:

```text
read ADR
cite ADR
detect ADR conflict
propose ADR update
create new ADR
mark ADR as superseded
```

Агентские проблемы вокруг него:

```text
treats old ADR as current spec
ignores ADR during architecture change
updates ADR without review
uses ADR to override contract
```

Политика:

```text
ADR explains rationale.
ADR is not automatically the strongest source for current runtime behavior.
ADR updates require review.
Architecture-sensitive changes require ADR or waiver.
```

---

### 9.2. tech-stack.md

`tech-stack.md` — это project artifact.

Агентские действия:

```text
read allowed dependencies
check build commands
verify runtime versions
propose dependency update
```

Агентские проблемы:

```text
installs random dependency
uses wrong package manager
runs wrong test command
assumes outdated runtime
```

Политика:

```text
Dependency changes require tech-stack/dependency-policy check.
Agent may not introduce new major dependency without approval.
```

---

### 9.3. core-classes.md

`core-classes.md` — это project artifact.

Агентские действия:

```text
read responsibility map
map class to domain concept
find affected classes
propose class responsibility update
```

Агентские проблемы:

```text
misunderstands ownership
adds behavior to wrong class
duplicates responsibility
ignores existing abstraction
```

Политика:

```text
Core class responsibility changes require code-model update or waiver.
```

Важное ограничение:

```text
core-classes.md should describe conceptual responsibility, not every method/class detail.
Detailed symbol maps should be generated.
```

---

### 9.4. changes/<id>

`changes/<id>` — это bridge artifact.

Агентские действия:

```text
create change
refine requirements
add acceptance criteria
update affected areas
attach verification plan
mark status
```

Агентские проблемы:

```text
starts coding without change
changes scope silently
does not update acceptance after requirement change
does not archive/promote after merge
```

Политика:

```text
No non-trivial implementation without active change artifact.
After merge, change must be archived or promoted into stable knowledge.
```

---

## 10. Operation header для агента

Чтобы не смешивать всё обратно, агент должен явно объявлять операцию.

Пример:

```yaml
operation:
  mode: impact-analysis
  target_artifacts:
    - changes/add-refresh-token-auth/
    - project-knowledge/specs/current/auth.md
    - project-knowledge/contracts/openapi.yaml
    - project-knowledge/code-model/core-classes.md
  allowed_actions:
    - read
    - search
    - create impact-report
  forbidden_actions:
    - edit production code
    - rewrite stable specs
    - update ADR
  outputs:
    - changes/add-refresh-token-auth/affected-areas.md
    - changes/add-refresh-token-auth/verification-plan.md
```

Это разделяет:

```text
mode = что делает агент
target_artifacts = с какими объектами проекта он работает
allowed_actions = права
outputs = какие новые artifacts появляются
```

---

## 11. Control Binding Layer

Итоговая архитектура должна иметь три независимых карты.

### 11.1. Project Artifact Registry

Пример:

| Artifact | Type | Source status | Updated by | Review | Freshness trigger |
|---|---|---|---|---|---|
| `docs/domain/*.md` | domain knowledge | curated | human/agent proposal | required | domain behavior change |
| `docs/adr/*.md` | decision rationale | curated historical | human/agent proposal | required | architecture-sensitive change |
| `specs/current/*.md` | intended behavior | curated | PR | required | behavior change |
| `contracts/openapi.yaml` | contract | authoritative contract | PR | required | API route change |
| `docs/tech-stack.md` | technical policy | curated | PR | required | dependency/runtime change |
| `changes/<id>/*` | active work state | temporary | agent/human | depends | archived after merge |
| `runs/<id>/*` | evidence/history | append-only | agent/tool | optional/reviewed | archived |
| `generated/*` | derived cache | not truth | tool | no | regenerate |

---

### 11.2. Agent Operation Matrix

Пример:

| Operation | Reads | Writes | Must not write | Required output |
|---|---|---|---|---|
| Intent clarification | project knowledge, issues | `changes/<id>/intent.md` | production code | change spec |
| Context discovery | docs, specs, code, tests | context pack/report | stable knowledge | retrieval report |
| Impact analysis | code, specs, contracts | affected areas, verification plan | production code | impact report |
| Implementation | change spec, context pack | code/tests | trusted memory | diff + run log |
| Verification | code/tests/contracts/specs | verification report | specs/contracts silently | evidence |
| Reconciliation | diff, docs, specs | proposed updates | direct trusted overwrite | freshness report |
| Retrospective | runs/evidence | lesson/proposal | policy silently | process patch proposal |

---

### 11.3. Forbidden Confusions

Это надо вынести как сильные принципы:

```text
ADR is not current behavior spec.
Search result is not truth.
Run log is not memory.
Generated context pack is not source of truth.
Change spec is not stable spec.
Test pass is not full verification.
LLM judgment is not freshness trigger.
Agent proposal is not accepted knowledge.
Code diff is not evidence by itself.
```

---

## 12. Итоговая строгая модель

```text
AI-assisted development system has two independent models:

1. Project Ontology
   Durable artifacts that describe the project:
   domain, product, architecture, tech stack, code model, contracts, specs, changes, facts.

2. Agent Protocol
   Operating modes for acting on those artifacts:
   clarify, retrieve, analyze, implement, verify, reconcile, learn, automate.

The control system binds them with authority, permissions, lifecycle, and evidence rules.
```

По-русски:

```text
Система состоит из онтологии проекта и протокола агента.
Онтология описывает, что существует в проекте.
Протокол описывает, как агент имеет право с этим работать.
```

Схема:

```text
                 PROJECT ARTIFACT MODEL
┌────────────────────────────────────────────────────────┐
│ domain │ architecture │ tech │ code model │ contracts  │
│ specs  │ facts        │ ops  │ changes    │ evidence   │
└────────────────────────────────────────────────────────┘
                         ▲
                         │ reads / writes / verifies / updates
                         │ under policy
                         ▼
                    AGENT PROTOCOL
┌────────────────────────────────────────────────────────┐
│ clarify │ retrieve │ analyze │ implement │ verify      │
│ reconcile │ learn │ automate │ review │ handoff        │
└────────────────────────────────────────────────────────┘
                         ▲
                         │ constrained by
                         ▼
                    CONTROL RULES
┌────────────────────────────────────────────────────────┐
│ authority │ lifecycle │ permissions │ gates │ evidence  │
│ freshness │ review requirements │ adapter mapping        │
└────────────────────────────────────────────────────────┘
```

---

## 13. Критическая оценка задумки

### 13.1. Что реально сильное

Сильное ядро:

```text
Project artifacts are durable nouns.
Agent operations are controlled verbs.
Controls define which verbs may act on which nouns, under what evidence and review rules.
```

Это лучше, чем очередной pipeline:

```text
memory → intent → execution → verification → freshness
```

Потому что pipeline снова начинает смешивать:

- знания проекта;
- действия агента;
- временные артефакты;
- evidence;
- generated context;
- policies.

Главная ценность задумки — **artifact authority model**.

Агент должен понимать:

```text
Что это за артефакт?
Насколько он авторитетен?
Можно ли ему верить?
Можно ли его менять?
Кто должен review?
Когда он устаревает?
Чем его надо подтверждать?
```

---

### 13.2. Главный риск

Главный риск — построить слишком красивую онтологию, которой никто не будет пользоваться.

Можно легко создать:

```text
docs/domain/
docs/product/
docs/architecture/
docs/technical/
docs/code-model/
docs/contracts/
docs/specs/
docs/facts/
docs/operations/
.agent/modes/
.agent/policies/
.agent/adapters/
schemas/
recipes/
```

И получить кладбище markdown-файлов.

Проблема:

```text
Каждый новый артефакт имеет стоимость поддержки.
```

Если артефакт не обновляется автоматически или не имеет lifecycle, он начнёт врать.

А для AI-агента устаревший уверенный markdown опаснее, чем отсутствие markdown.

Жёсткое правило:

```text
Не добавлять артефакт, если непонятно:
1. кто его обновляет;
2. когда он устаревает;
3. какой check ловит drift;
4. кто его review;
5. когда его удалять или архивировать.
```

---

### 13.3. Риск core-classes.md

`core-classes.md` полезен, но опасен.

Если заносить туда реальные классы, методы, связи и зависимости, он быстро протухнет.

Правильнее:

```text
core-classes.md = conceptual responsibility map
generated/symbol-index.json = actual class/function index
```

То есть руками описывать только смысловые якоря:

```yaml
AuthService:
  responsibility: authentication flow orchestration
  owns:
    - login
    - refresh
    - logout
  must_not:
    - directly manage billing permissions
  related_specs:
    - specs/current/auth.md
  related_contracts:
    - contracts/openapi.yaml
```

А подробные symbol maps генерировать из кода.

---

### 13.4. Риск self-classification агента

Идея “агент должен выбрать operation/mode” полезна, но нельзя полностью доверять его самооценке.

Агент может написать:

```yaml
mode: implementation
```

хотя на самом деле нужно было:

```yaml
mode: intent-clarification
```

или:

```yaml
mode: impact-analysis
```

Поэтому mode classification должен быть связан с gates.

Например:

```text
Нельзя перейти в implementation, если:
- нет change intent;
- нет acceptance criteria;
- риск medium/high и нет impact analysis;
- нет verification plan.
```

Иначе агент будет честно писать неправильный mode и продолжать делать глупости.

---

### 13.5. Риск тяжёлого процесса

Если для каждого изменения требовать:

```text
intent.md
requirements.md
design.md
tasks.md
acceptance.md
affected-areas.md
verification.md
freshness-report.md
retrospective.md
evidence-ledger.md
```

то человек плюнет и вернётся к обычному чату.

Нужно risk-based управление.

```text
Low risk:
  typo, small test fix, local refactor
  → short change note + test evidence

Medium risk:
  feature, behavior change, module refactor
  → change spec + acceptance + verification report

High risk:
  auth, payments, public API, DB migration, infra, security
  → design + impact + contract/security checks + rollback/freshness
```

Правило:

```text
Control intensity should match change risk.
```

---

### 13.6. Риск слишком абстрактной framework-independence

Правильно не строить вокруг OpenSpec/GSD/Codex.

Но если слишком уйти в абстракции, получится universal conceptual model, который непонятно как применять завтра в реальном репозитории.

Нужна связка:

```text
Concept → Artifact → Check → Tool adapter
```

Пример:

```text
Problem:
  public API drift

Project artifact:
  contracts/openapi.yaml

Agent operation:
  impact-analysis / verification / reconciliation

Control:
  route changed → OpenAPI diff or waiver required

Adapter:
  GitHub Action + openapi-diff + LLM summary
```

Это рабочая единица.

---

## 14. Что стоит отложить

Не начинать с:

```text
полной problem taxonomy
полного adapter registry
полных schemas для всех artifact types
сложного lifecycle для всего
automatic self-learning
graph/rag memory
много agent modes
полного CLI
```

Это преждевременно.

Сначала надо доказать, что базовая модель реально уменьшает хаос.

Минимально достаточно:

```text
1. Project artifact registry
2. Agent operation matrix
3. Source authority rules
4. Evidence policy
5. Freshness rules
6. 4–5 templates
7. 2–3 checks
8. 3 demo scenarios
```

---

## 15. Минимальный MVP

Не делать сразу “framework”.

Сделать:

```text
control handbook + templates + checks
```

Возможная структура:

```text
README.md
AGENTS.md

.agent/
  artifact-registry.md
  operation-matrix.md
  source-authority.md
  evidence-policy.md
  freshness-rules.md

templates/
  change.md
  impact-report.md
  verification-report.md
  freshness-report.md
  run-log.md

docs/
  domain.md
  architecture.md
  tech-stack.md
  code-map.md
  adr/
  specs/current/

changes/
  example-change/

runs/
  example-run/
```

---

## 16. Минимальный agent protocol

В `AGENTS.md` стоит сделать жёсткое правило:

```text
Before doing non-trivial work, declare:

1. Operation
   What are you doing?
   intent / context / impact / implementation / verification / reconciliation

2. Target artifacts
   What project artifacts are involved?

3. Authority check
   Which sources are trusted for this decision?

4. Write boundary
   What are you allowed to modify?

5. Evidence
   What proof will be produced?

6. Freshness
   Which artifacts may need update after the change?
```

---

## 17. Хорошие proof-of-concept сценарии

### 17.1. API endpoint change

Без системы:

```text
Агент меняет route.
Тесты проходят.
OpenAPI не обновлён.
Docs устарели.
```

С системой:

```text
Operation: impact-analysis
Target artifacts:
  - contracts/openapi.yaml
  - specs/current/api.md
  - docs/code-map.md

Control:
  route changed → OpenAPI check required

Verification:
  openapi diff attached

Reconciliation:
  docs/specs checked

Result:
  agent cannot mark task complete without contract evidence or waiver
```

---

### 17.2. Agent tries to edit ADR directly

Без системы:

```text
Агент переписывает ADR под новый код.
```

С системой:

```text
ADR update requires proposal/review.
Agent may create ADR update proposal, but not silently rewrite trusted rationale.
```

---

### 17.3. Agent starts implementation without acceptance criteria

Без системы:

```text
Агент сразу пишет код по vague request.
```

С системой:

```text
Implementation gate fails.
Agent must enter intent clarification first.
```

---

## 18. Оценка идеи

```text
Conceptual clarity:        8/10
Practical MVP shape:       6/10
Risk of overengineering:   9/10
Potential usefulness:      8/10
Adoption clarity:          5/10
Framework independence:    8/10
```

Вердикт:

```text
Идея правильная.
Формулировка стала лучше.
Но в текущем виде она всё ещё слишком широкая.
Нужно резко сузить MVP и сделать систему проверяемой на практике.
```

Главный риск:

```text
Построить слишком умную систему, которую никто не будет использовать каждый день.
```

---

## 19. Самая точная формулировка проекта

Не так:

```text
Framework for AI-assisted software engineering.
```

И не так:

```text
Problem taxonomy for AI development.
```

Лучше так:

```text
A repo-native control model that separates project artifacts from agent operations,
then defines how agents may read, change, verify, and reconcile those artifacts.
```

По-русски:

```text
Repo-native модель управления AI-разработкой,
которая разделяет артефакты проекта и операции агента,
а затем задаёт правила: что агент может читать, менять, проверять и согласовывать.
```

---

## 20. Итоговая формула

```text
Project artifacts are durable nouns.
Agent operations are controlled verbs.
Controls define which verbs may act on which nouns, under what evidence and review rules.
```

Или короче:

```text
Do not mix project nouns with agent verbs.
```

Практический следующий шаг:

```text
Сделать одну artifact registry,
одну operation matrix,
один source authority file,
один evidence policy,
один freshness rules file,
и 3 demo scenarios, где система предотвращает типичные ошибки агента.
```

Если это сработает, уже потом можно наращивать capabilities, modes, recipes, adapters, schemas и CLI.
