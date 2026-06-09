# OctoBend

**AI Engineering Control Loop for agent-assisted software development**.

---

## 0. Что это такое

**OctoBend** — это не просто SDD framework и не набор промптов для coding agent. Это архитектура управляемого engineering-процесса для разработки с AI-агентами.

Цель: сделать так, чтобы работа с coding agents была не `vibe coding`, а воспроизводимым процессом:

```text
stable knowledge → change intent → controlled execution → evidence → freshness → learning
```

Но важная поправка: это не линейный pipeline. Это **control loop**, где retrieval, verification, freshness и learning работают вокруг всего процесса.

OctoBend должен отвечать на вопросы:

- что проект уже знает;
- что именно мы хотим изменить;
- какие требования и критерии готовности есть у изменения;
- какой контекст агент должен загрузить перед работой;
- как агент исполняет задачу;
- чем доказать, что результат правильный;
- какие docs/specs/facts устарели после изменения кода;
- какие ошибки процесса повторяются;
- какие инструкции, шаблоны, проверки и инструменты надо улучшить;
- какие повторяющиеся действия стоит автоматизировать.

---

## 1. Модули

### Внутренние модули

```text
octobend-memory
octobend-intent
octobend-run
octobend-checks
octobend-freshness
octobend-retro
octobend-index
octobend-tools
```

---

## 2. Главная идея

Нельзя складывать всё в одну коробку под названием `SDD framework`.

В реальном AI-assisted development есть несколько независимых плоскостей:

1. **Stable Knowledge Plane** — что проект уже знает.
2. **Change Control Plane** — что именно мы хотим изменить.
3. **Execution Plane** — как агент выполняет изменение.
4. **Context Access Plane** — как агент находит нужный контекст.
5. **Evidence & Quality Plane** — как проверяется результат.
6. **Knowledge Reconciliation Plane** — как проверяется свежесть памяти/docs/specs.
7. **Process Learning Plane** — как улучшается сам процесс разработки с агентами.
8. **Toolsmith / JIT Automation Capability** — как агент создаёт вспомогательные артефакты, prompts и tools под повторяющиеся задачи.

Ключевая формула:

```text
Stable Knowledge tells the agent what is true/intended.
Change Spec tells what should become different.
Execution Run records how the agent tried to do it.
Evidence proves whether it worked.
Freshness reconciles knowledge after reality changed.
Self-Optimization improves the process after observing failures.
Retrieval accelerates access but never becomes truth.
JIT Automation turns repeated friction into repo-local tools and templates.
```

---

## 3. Architecture model

### Неправильная модель

```text
Memory
  ↓
Intent / Change
  ↓
Execution Control
  ↓
Retrieval / Context Engine
  ↓
Verification / Quality Gates
  ↓
Self-Optimization
  ↓
Freshness Watcher
```

Проблема: **retrieval не является этапом после execution**. Retrieval используется везде: при создании spec, при планировании, при исполнении, при ревью, при freshness check и при retrospective.

### Более правильная модель

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
         ↓                                      ▲
┌──────────────────┐                            │
│ Execution Plane  │────────────────────────────┘
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

### Event-driven loop

```text
Change requested
  ↓
Change spec created
  ↓
Impact map generated
  ↓
Relevant memory/specs/docs/tests retrieved
  ↓
Execution run starts
  ↓
Agent implements in controlled phases
  ↓
Verification gates run
  ↓
Freshness watcher detects impacted knowledge
  ↓
Memory/spec/docs update proposed
  ↓
Human/code review
  ↓
Merge
  ↓
Change archived
  ↓
Retrospective mines failures
  ↓
Process patches proposed
```

---

## 4. Strict terminology

| Old / informal term | Strict term | Meaning |
|---|---|---|
| Project Memory Layer | **Stable Knowledge Plane** | Curated, versioned project knowledge |
| Intent / Change Layer | **Change Control Plane** | Delta intent: what should change and why |
| Execution Orchestration | **Execution Plane** | How agents execute approved changes |
| Context Retrieval | **Context Access Plane** | Search, indexing, context packaging, MCP/RAG |
| Verification | **Evidence & Quality Plane** | Tests, checks, reviews, proof |
| Freshness Watcher | **Knowledge Reconciliation Plane** | Drift detection, invalidation, update proposals |
| Self-Optimization | **Process Learning Plane** | Improve prompts, policies, templates, gates |
| Dynamic tools/prompts | **Toolsmith / JIT Automation** | Create repo-local tools and dynamic artifacts |

### Core terms

```text
Stable Spec
  Description of current intended behavior/capability.

Change Spec
  Proposal for a delta from current behavior.

Run
  A concrete execution attempt for a change.

Evidence
  Verifiable proof: tests, CI logs, command output, diffs, traces, reviews.

Fact
  Small atomic curated knowledge entry with owner, evidence and freshness rules.

Index
  Derived search/cache artifact. Never source of truth.

Gate
  Condition that must pass before merge/ship.

Reconciliation
  Process of making memory/spec/docs/contracts consistent with changed reality.

Promotion
  Moving knowledge from chat/run/search into curated stable memory.

Invalidation
  Marking old knowledge as deprecated/superseded/invalid.

Waiver
  Explicit documented exception to a required check.
```

---

## 5. What must not be mixed

### 5.1 Memory is not search

```text
Memory = curated knowledge.
Search = access mechanism.
```

Search can find useful information, but if it is important and reusable, it should be promoted into curated memory.

### 5.2 Search is not spec

```text
Search result = possible evidence/context.
Spec = explicit intended behavior.
```

### 5.3 Spec is not execution

```text
Spec says what and why.
Execution says how the agent performs the work.
```

### 5.4 Execution is not verification

```text
Execution produces a diff.
Verification proves whether the diff is acceptable.
```

### 5.5 Run logs are not memory

```text
Run logs = evidence and history.
Memory = reviewed stable project knowledge.
```

### 5.6 Self-optimization is not freshness

```text
Freshness checks whether project knowledge matches code/spec/contracts.
Self-optimization checks whether the AI development process itself is improving.
```

Example:

```text
API endpoint changed, OpenAPI not updated
  → freshness problem

Agent repeatedly forgets to update OpenAPI after API changes
  → process learning problem
```

### 5.7 OpenAPI / AsyncAPI are not change intent

OpenAPI, AsyncAPI, protobuf, DB schema are **stable contracts**. A change can propose modifying them, but the contract files themselves are stable artifacts.

---

## 6. Planes / capabilities

## 6.1 Stable Knowledge Plane

### Responsibility

Stores what the project already knows.

This is not random memory, not chat history, not vector DB. This is curated, reviewed, versioned knowledge.

### Includes

```text
Project rules
  AGENTS.md
  CLAUDE.md
  coding conventions
  agent permissions

Product knowledge
  docs/context/product.md
  docs/context/domain.md
  glossary

Architecture knowledge
  docs/context/architecture.md
  C4 docs
  ADRs

Technical knowledge
  docs/context/tech.md
  dependency policy
  build/test commands

Operational knowledge
  docs/context/operations.md
  runbooks
  deployment model
  incident notes

Stable capability specs
  specs/current/**

Contracts
  OpenAPI
  AsyncAPI
  protobuf
  DB schema

Atomic facts
  docs/facts/*.fact.md
```

### Rules

```text
1. Trusted memory is updated only through reviewed PRs.
2. Facts must have owners and evidence.
3. Stable specs describe current intended behavior.
4. ADRs explain why decisions were made.
5. Contracts describe promises to external/internal consumers.
6. Chat logs and run logs are not stable memory.
7. Vector/graph indexes are derived caches, not sources of truth.
```

---

## 6.2 Change Control Plane

### Responsibility

Turns vague intent into a versioned change artifact.

A user request should not live only in chat. Any non-trivial change should become:

```text
changes/<change-id>/
```

### Includes

```text
proposal.md
requirements.md
design.md
tasks.md
acceptance.md
verification.md
memory-updates.md
status.yaml
```

### Change spec answers

```text
What are we changing?
Why are we changing it?
What is explicitly out of scope?
What behavior must exist after the change?
What are acceptance criteria?
What files/modules/contracts/specs/facts may be impacted?
What tests/checks must pass?
What stable knowledge must be updated?
```

### Boundary

```text
Change spec = proposed delta.
Stable spec = current intended behavior.
```

After merge, change spec is archived and relevant knowledge is promoted into stable specs/facts/docs/contracts.

---

## 6.3 Execution Plane

### Responsibility

Controls how the agent performs work.

Execution is not “agent, go code”. It is phased work with constraints, logs, stops and review points.

### Includes

```text
run creation
plan generation
phase control
worktree/branch isolation
context loading
implementation steps
command execution
intermediate verification
deviation logging
human review points
```

### Typical phases

```text
1. Load protocol
2. Load active change
3. Impact analysis
4. Retrieve relevant context
5. Plan implementation
6. Implement small steps
7. Run verification
8. Fix failures
9. Produce evidence
10. Propose memory/spec updates
11. Write retrospective if needed
```

### Execution is allowed to create

```text
runs/<run-id>/plan.md
runs/<run-id>/execution-log.md
runs/<run-id>/commands.log
runs/<run-id>/test-results.md
runs/<run-id>/review.md
runs/<run-id>/retrospective.md
```

### Execution is not allowed to do silently

```text
- rewrite trusted memory without review;
- change architecture without ADR or waiver;
- change API without contract review;
- treat search results as truth;
- merge without verification evidence.
```

---

## 6.4 Context Access Plane

### Responsibility

Finds and packages relevant context.

This is an infrastructure service used by every other plane.

### Includes

```text
ripgrep / grep-style search
AST search
symbol search
repo maps
context packs
vector search
graph search
MCP servers
issue/docs/test retrieval
ownership lookup
impact map generation
```

### Important boundary

```text
Context Access finds evidence.
It does not decide truth.
```

### Common functions

```text
retrieval.find_related(change)
retrieval.find_implementations(symbol)
retrieval.find_tests(module)
retrieval.find_docs(api)
retrieval.find_contracts(endpoint)
retrieval.find_owners(path)
retrieval.find_facts_for_paths(paths)
retrieval.build_context_pack(change)
```

### Derived artifacts

```text
memory-index/generated/context-pack.md
memory-index/generated/impact-map.json
memory-index/generated/search-index.json
repomix-output.md
repo-map.json
```

These can be deleted and regenerated.

---

## 6.5 Evidence & Quality Plane

### Responsibility

Proves whether the result is acceptable.

Without verification, SDD becomes pretty bureaucracy.

### Includes

```text
unit tests
integration tests
e2e tests
contract tests
snapshot tests
lint
typecheck
security scan
migration checks
API diff
AI review
spec compliance check
doc drift check
freshness check
manual review
```

### Evidence types

```text
command output
CI run link
coverage report
test result file
contract diff
security scan report
review comment
screenshot
trace
benchmark result
manual checklist
```

### Gate examples

```text
no code change without tests or explicit waiver
no public API change without contract update or waiver
no architecture-sensitive change without ADR or waiver
no merged change without verification evidence
no trusted memory update without review
no closed change without acceptance criteria checked
```

---

## 6.6 Knowledge Reconciliation Plane

### Responsibility

Checks whether project knowledge is still fresh after code/spec/contracts change.

This is the **freshness watcher**.

### Freshness loop

```text
1. Detect
   Find changed files, merged PRs, changed specs, changed contracts, changed tests.

2. Impact map
   Identify which facts/specs/docs/ADR/contracts may be affected.

3. Freshness check
   Compare stable knowledge with current code/contracts/specs.

4. Reindex
   Regenerate search/vector/graph indexes.

5. Propose update
   Create PR/patch/issue with evidence.

6. Archive/invalidate
   Mark old knowledge as superseded/deprecated/invalid when needed.
```

### Deterministic first, LLM second

Freshness watcher should not rely only on LLM judgment.

It should use deterministic rules:

```text
timestamps
changed paths
ownership
source priority
status fields
superseded_by links
CODEOWNERS
PR metadata
contract diffs
schema validation
required sections
```

LLM can help explain impact or draft updates, but the trigger logic should be boring and inspectable.

---

## 6.7 Process Learning Plane

### Responsibility

Improves the AI-development process itself.

This is not about product code. It is about prompts, policies, templates, checks, retrieval strategy and agent operating protocol.

### Questions

```text
Which tasks does the agent repeatedly fail?
Which instructions are ambiguous?
Which specs are too vague?
Which checks are missing?
Which AGENTS.md rules need updating?
Which memory entries cause drift?
Which context packs are too large or too small?
Which repeated manual steps should become tools?
```

### Subsystems

```text
tracing / observability
eval harness
prompt/policy optimization
retrospective miner
process patcher
failure taxonomy
agent behavior regression tests
```

### Rule

Self-optimization must not silently rewrite trusted memory or project policy.

It should create:

```text
process patch proposal
PR
issue
eval fixture
new check proposal
template update proposal
AGENTS.md patch proposal
```

---

## 6.8 Toolsmith / JIT Automation Capability

### Responsibility

Detects repeated friction and turns it into repo-local tools, templates, prompts or checks.

This is important because the agent should not only use the repo; it should gradually improve the repo as a working environment.

### Examples

```text
Repeated task:
  Agent often searches the same files before auth changes.

Possible automation:
  obend context auth
  Generates a context pack for auth-related changes.

Repeated task:
  Agent forgets OpenAPI updates.

Possible automation:
  CI check: API route changed → OpenAPI update or waiver required.

Repeated task:
  Agent writes similar change specs.

Possible automation:
  dynamic markdown template generator for change type.

Repeated task:
  Agent manually compares docs and code.

Possible automation:
  freshness-check script with path ownership rules.
```

### Tool lifecycle

```text
observed friction
  ↓
retro finding
  ↓
tool proposal
  ↓
review
  ↓
experimental tool under .ai/tools/experimental
  ↓
measured usefulness
  ↓
promotion to .ai/scripts or tools
```

### Rule

Generated tools are not automatically trusted.

They should start as proposed/experimental and become stable only after review.

---

## 7. Authority model

The system needs an explicit model of source authority. Otherwise agents and watchers will make bad guesses.

### Source priority

```text
1. Runtime evidence / CI / tests
   Shows what currently passes or fails.

2. Public/internal contracts
   OpenAPI, AsyncAPI, protobuf, SDK contracts, DB migration contracts.
   If code changes but contract does not, that is not automatically stale contract.
   It may be an unintended breaking change.

3. Stable specs
   Describe intended current behavior.

4. Code
   Describes implemented behavior.

5. ADR
   Describes rationale at the time of decision.
   ADR is not always a full current behavior spec.

6. Docs/context/facts
   Curated knowledge for humans and agents.

7. Change specs
   Truth only inside a particular change lifecycle.

8. Run logs/chat/search results
   Evidence/history, not source of truth.

9. Indexes/vector DB/GraphRAG
   Derived cache, never source of truth.
```

### Important nuance

Code is authoritative for:

```text
what is currently implemented
```

Spec/contract is authoritative for:

```text
what is intended or promised
```

So conflict between code and spec is not an automatic docs update. It is a **drift event**.

---

## 8. Repository structure

Recommended structure:

```text
/
  AGENTS.md
  CLAUDE.md

  .ai/
    policy/
      operating-protocol.md
      source-priority.md
      agent-permissions.md
      change-lifecycle.md
      memory-lifecycle.md
    schemas/
      fact.schema.yaml
      change.schema.yaml
      run.schema.yaml
      freshness-rule.schema.yaml
      retrospective.schema.yaml
      tool-proposal.schema.yaml
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
      tool/
        tool-proposal.md
        dynamic-prompt.md
    scripts/
      new-change
      validate-change
      impact-map
      freshness-check
      reindex
      promote-memory
      retro-summary
      propose-tool

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

  tools/
    ai/
      experimental/
      stable/
```

---

## 9. Lifecycle models

## 9.1 Change lifecycle

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

### Rules

```text
draft/proposed:
  agent may edit change spec

approved:
  scope is locked unless explicit scope-change note exists

in_progress:
  run log is required

implemented:
  code is done but verification may not be complete

verified:
  evidence is attached

merged:
  code is in main

archived:
  change is converted into stable specs/facts/ADR where needed
```

---

## 9.2 Fact lifecycle

```text
proposed
  ↓
active
  ↓
challenged
  ↓
superseded / deprecated / invalid
  ↓
archived
```

### Rules

```text
active:
  agent may read and rely on it

challenged:
  agent may read but must verify against source files

superseded:
  must point to new fact/spec/ADR

deprecated:
  should not be used for new decisions

invalid:
  should not be loaded as context

archived:
  historical only
```

---

## 9.3 Run lifecycle

```text
planned
  ↓
running
  ↓
blocked / failed / completed
  ↓
reviewed
  ↓
archived
```

Run logs should be mostly append-only. Do not rewrite history to make the agent look smarter.

---

## 9.4 Freshness lifecycle

```text
detect
  ↓
impact map
  ↓
check
  ↓
finding
  ↓
proposed patch / waiver
  ↓
review
  ↓
resolved
```

---

## 9.5 Process learning lifecycle

```text
retro
  ↓
failure classification
  ↓
process patch proposal
  ↓
eval/check added
  ↓
measured
  ↓
accepted / rejected
```

---

## 9.6 JIT tool lifecycle

```text
repeated friction observed
  ↓
tool idea written in retrospective
  ↓
tool proposal created
  ↓
experimental script/prompt/template generated
  ↓
reviewed
  ↓
used in real runs
  ↓
measured
  ↓
promoted to stable or deleted
```

---

## 10. MVP process

The first implementation should be boring.

Do not start with GraphRAG, Langfuse, multi-agent orchestration and complex evals. Start with markdown, git and CI.

### MVP rules

```text
1. Any non-trivial change starts with changes/<id>/proposal.md.
2. Before code, there must be requirements.md and acceptance.md.
3. Before merge, there must be verification.md and evidence.
4. If architecture/API/domain behavior changes, update specs/docs/facts/ADR or write waiver.
5. Watcher checks changed paths against ownership/freshness rules.
6. Agent cannot silently rewrite trusted memory.
7. After merge, change is archived.
8. Useful knowledge is promoted into stable memory.
9. Repeated failures create retrospective and process patch proposal.
10. Repeated manual work becomes JIT automation proposal.
```

### MVP commands

```bash
obend init
obend change new add-refresh-token-auth
obend change validate add-refresh-token-auth
obend impact add-refresh-token-auth
obend run start add-refresh-token-auth
obend verify
obend freshness check
obend memory promote
obend retro create
obend tool propose
```

### MVP using `make`

```bash
make change NAME=add-refresh-token-auth
make validate-change CHANGE=add-refresh-token-auth
make impact CHANGE=add-refresh-token-auth
make verify
make freshness
make reindex
make retro RUN=2026-06-05-add-refresh-token-auth
```

### Minimal CI

```text
ci:
  - lint
  - typecheck
  - unit tests
  - integration tests where relevant
  - validate change schema
  - check PR references change-id
  - check architecture paths require ADR or waiver
  - check API paths require OpenAPI diff or waiver
  - check changed code paths have matching freshness review
  - check memory updates are proposed, not silently applied
```

---

## 11. What can be done with markdown/git/CI vs special tools

## 11.1 Markdown/git/CI is enough for

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
| Archive/invalidate | frontmatter status fields |
| Dynamic prompt packs | generated `.md` files under `memory-index/generated` |
| JIT tool proposals | `.ai/templates/tool/tool-proposal.md` |

## 11.2 Specialized tools become useful for

| Capability | When needed | Examples |
|---|---|---|
| Formal SDD workflow | Markdown templates become too loose | Spec Kit, OpenSpec, Kiro Specs |
| Agent orchestration | Long multi-phase/multi-agent tasks | GSD Core, gstack, BMAD |
| Large-codebase retrieval | Big repo/monorepo/multi-repo | Sourcegraph, Continue, RepoPrompt, MCP |
| Structural search/refactor | Need AST-aware code queries | ast-grep |
| Context packaging | Need controlled repo snapshot for LLM | Repomix, RepoPrompt |
| Semantic/graph retrieval | Many docs/specs/issues, keyword search weak | vector search, GraphRAG |
| LLM observability/evals | Many AI runs, quality must be measured | Langfuse, Braintrust, LangSmith, Phoenix |
| Prompt/security evals | Need regression tests for prompts/agents/RAG | Promptfoo, OpenEvals, DeepEval, Ragas |
| Prompt optimization | Need systematic prompt/program tuning | DSPy, TextGrad |

---

## 12. Schemas

## 12.1 Memory/fact schema

A fact should be small, owned, reviewable and linked to evidence.

```md
---
id: fact.auth.session-model
title: Auth session model
type: fact
domain: auth
status: active # proposed | active | challenged | deprecated | superseded | invalid | archived
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

### Fact rules

```text
1. One fact should answer one important question.
2. Every fact needs owner, status and evidence.
3. Every fact should have freshness watches.
4. Deprecated/superseded facts must point to replacement if possible.
5. Facts should be short enough for agents to load safely.
```

---

## 12.2 Change/spec schema

Machine-readable version:

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
    depends_on:
      - task-001

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

Markdown file layout:

```text
proposal.md
  why / scope / non-goals

requirements.md
  user stories / acceptance criteria

design.md
  architecture / data / security / alternatives / migrations

tasks.md
  implementation plan

acceptance.md
  checkable done criteria

verification.md
  commands / tests / gates / waivers

memory-updates.md
  expected updates to stable specs/facts/ADR/contracts

status.yaml
  machine-readable state
```

---

## 12.3 Run/execution log schema

```yaml
run_id: run.2026-06-05.add-refresh-token-auth.001
change_id: change.add-refresh-token-auth
status: completed # planned | running | blocked | failed | completed | abandoned
branch: feature/add-refresh-token-auth
worktree: ../worktrees/add-refresh-token-auth
base_commit: abc123
head_commit: def456

agent:
  tool: claude-code # codex/cursor/etc
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

---

## 12.4 Freshness watcher schema

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

## 12.5 Self-optimization retrospective schema

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

`Affected contracts and stable knowledge`

## Patch 2: CI gate

Add blocking rule:

`api-change-requires-contract-review: blocking`

## Patch 3: AGENTS.md

Add rule:

`When editing API routes/controllers, inspect contracts/openapi and update or write waiver.`

# Proposed eval

Create a fixture task:

`Given an API route change, agent must identify OpenAPI as impacted.`

# Expected measurement

Next 5 API-related changes should have zero missing contract findings.
```

---

## 12.6 Dynamic prompt / markdown artifact schema

Dynamic markdown artifacts are useful, but they must not become trusted memory by accident.

```md
---
id: dynamic-context.auth-change.2026-06-05
kind: dynamic-context-pack
status: generated # generated | reviewed | promoted | discarded
generated_by: obend context auth
change_id: change.add-refresh-token-auth
run_id: run.2026-06-05.add-refresh-token-auth.001
created: 2026-06-05
source_files:
  - AGENTS.md
  - specs/current/auth/session-model.md
  - docs/facts/auth-session-model.fact.md
  - src/auth/session.ts
  - tests/auth/session.test.ts
valid_for:
  - change.add-refresh-token-auth
expires_after: merge
trusted: false
---

# Context pack: auth change

## Must-read project rules

...

## Relevant stable knowledge

...

## Relevant code

...

## Relevant tests

...

## Open questions

...
```

### Rules

```text
1. Dynamic artifacts are generated context, not truth.
2. They should declare source files.
3. They should expire after the run/change.
4. Useful knowledge must be promoted into facts/specs through review.
```

---

## 12.7 JIT tool proposal schema

```md
---
id: tool-proposal.auth-context-pack
title: Generate auth context pack
status: proposed # proposed | experimental | accepted | rejected | stable | removed
owner: leonid
created: 2026-06-05
source:
  retro: retro.2026-06-05.add-refresh-token-auth
problem_type: repeated-context-selection
risk: low
---

# Problem

Agent repeatedly needs the same auth-related files before auth changes.

# Proposed tool

```bash
obend context auth
```

# Behavior

The tool generates:

```text
memory-index/generated/auth-context-pack.md
```

using:

```text
AGENTS.md
specs/current/auth/**
docs/facts/*auth*.fact.md
src/auth/**
tests/auth/**
contracts/openapi/**
```

# Acceptance criteria

- Generated pack lists all source files.
- Generated pack marks itself as untrusted derived context.
- Generated pack fits configured token budget.
- Tool can run locally and in CI.

# Risks

- Context pack may become stale if source files change.
- Agent may treat generated summary as truth.

# Guardrails

- Include source file list.
- Include `trusted: false`.
- Include generation timestamp and commit.
- Require regeneration after watched file changes.

# Promotion criteria

Promote from experimental to stable after 5 real auth changes where it reduced missing-context findings.
```

---

## 13. AGENTS.md operating protocol

Minimal version:

```md
# Agent Operating Protocol

## Core rule

Do not do vibe coding. Work through project artifacts.

## Before non-trivial code changes

1. Find or create active change under `/changes`.
2. Read:
   - `AGENTS.md`
   - `.ai/policy/operating-protocol.md`
   - active change files
   - related specs/facts/ADR from `memory-index/manifest.yaml`
3. Build or inspect impact map.
4. Identify affected tests, contracts, specs and facts.

## During execution

1. Work in small steps.
2. Keep run evidence under `/runs/<run-id>/`.
3. Log deviations from the approved plan.
4. Do not silently expand scope.
5. Do not treat search results as source of truth.

## Verification

1. Run commands from `changes/<id>/verification.md`.
2. Attach command results to the run.
3. If verification fails, record failure before fixing.
4. If a required check is skipped, write a waiver.

## Memory and docs

1. Do not modify trusted memory unless the change includes `memory-updates.md`.
2. If code behavior contradicts docs/specs/facts, create freshness finding.
3. Promote reusable knowledge into stable memory through reviewed PR.
4. Dynamic context packs and run logs are not trusted memory.

## After task

1. Ensure acceptance criteria are checked.
2. Ensure freshness check is clean or has explicit findings.
3. Write retrospective when the run failed, drift was found, or process friction repeated.
```

---

## 14. Checks and gates

## 14.1 Required checks for MVP

```text
check.change-exists
  PR must reference changes/<id>.

check.change-schema-valid
  Required sections exist.

check.acceptance-complete
  Acceptance checklist is checked or explicitly deferred.

check.verification-evidence
  verification.md contains commands and results.

check.api-contract
  API route/controller changes require OpenAPI/contract update or waiver.

check.architecture-adr
  architecture-sensitive changes require ADR update/proposal or waiver.

check.memory-freshness
  Changed watched paths require fact/spec review or waiver.

check.no-silent-memory-update
  Trusted memory changes require explicit memory-updates.md.
```

## 14.2 PR template

```md
# Change

Change ID: `changes/...`

# Summary

...

# Acceptance criteria

- [ ] ...
- [ ] ...

# Verification evidence

Commands run:

```bash
...
```

Results:

...

# Impacted stable knowledge

- Specs:
- Facts:
- ADR:
- Contracts:

# Freshness

- [ ] `obend freshness check` passed
- [ ] findings attached
- [ ] waivers documented

# AI execution

Run ID: `runs/...`

# Retrospective

- [ ] not needed
- [ ] added under `runs/.../retrospective.md`
```

---

## 15. Roadmap

## 15.1 Solo developer

Goal: discipline without bureaucracy.

Start with:

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

Rules:

```text
1. No change spec → no big agent task.
2. No acceptance criteria → no implementation.
3. No verification evidence → change not closed.
4. Architecture/API/domain behavior changed → update ADR/spec/fact/contract or write waiver.
5. After repeated failure → retrospective and process patch.
6. After repeated manual action → JIT tool proposal.
```

Recommended commands:

```bash
obend init
obend change new <name>
obend impact <change>
obend run start <change>
obend verify
obend freshness check
obend retro create
```

Good enough tools:

```text
markdown
git
CI
ripgrep
ast-grep
test runner
simple Python/Node scripts
optional Repomix/RepoPrompt for context packs
```

Avoid at this stage:

```text
large multi-agent role system
complex vector DB
heavy observability platform
automatic memory rewriting
```

---

## 15.2 Small team

Goal: shared memory, reviewability, fewer silent agent mistakes.

Add:

```text
CODEOWNERS
PR template with change-id
required CI gates
required review for memory/spec/ADR changes
team-owned facts/specs
change approval before implementation
run logs for AI-generated work
freshness findings as PR comments
basic evals for repeated agent failures
```

Workflow:

```text
1. Engineer/PM creates change proposal.
2. AI helps refine requirements/design/tasks.
3. Human approves scope.
4. Agent executes in branch/worktree.
5. CI verifies code + spec/freshness.
6. Reviewer checks diff + evidence + memory updates.
7. Merge archives change and promotes stable knowledge.
8. Retro creates process patch if needed.
```

Specialized tools may become useful:

```text
OpenSpec / Spec Kit / Kiro Specs for structured specs
GSD Core for disciplined phase loops
Sourcegraph / Continue / MCP for better context access
Promptfoo/OpenEvals for basic prompt regression tests
```

---

## 15.3 Larger engineering org

Goal: governance, compliance, scale.

Add:

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
central prompt/template registry
org-wide eval harness
```

Org-level architecture:

```text
Developer repo
  local specs/facts/runs/checks

Platform service
  schemas / policies / evals / dashboards / indexes

CI/CD
  enforcement gates

Knowledge service
  source priority / ownership / freshness / search

Observability service
  traces / evals / failure modes / prompt performance
```

Specialized tools likely needed:

```text
Sourcegraph / GraphRAG / MCP for multi-repo retrieval
Langfuse / Braintrust / LangSmith / Phoenix for observability/evals
Promptfoo / DeepEval / Ragas / OpenEvals for regression and RAG quality
policy-as-code for governance
contract diff tooling
security scanning
```

---

## 16. Implementation stages

## Stage 1: Artifact discipline

Create:

```text
AGENTS.md
.ai/policy/operating-protocol.md
.ai/policy/source-priority.md
.ai/templates/change/*
.ai/templates/run/*
.ai/templates/fact.md
.ai/templates/tool/tool-proposal.md
```

Done when:

```text
A real change can be represented in markdown before implementation.
```

---

## Stage 2: Verification gates

Add:

```text
lint/typecheck/test integration
change schema validation
required verification.md
required acceptance checklist
PR template
```

Done when:

```text
A PR cannot be merged without change ID and verification evidence.
```

---

## Stage 3: Freshness MVP

Add:

```text
memory-index/manifest.yaml
memory-index/ownership.yaml
memory-index/freshness-rules.yaml
path-based watcher script
API → contract rule
architecture → ADR rule
code path → fact/spec review rule
```

Done when:

```text
Changing watched code produces impacted docs/specs/facts list.
```

---

## Stage 4: Retrieval MVP

Add:

```text
rg-based search scripts
ast-grep patterns
context pack generator
impact map generator
optional RepoPrompt/Repomix integration
```

Done when:

```text
Agent can load a bounded context pack for a change instead of random repo browsing.
```

---

## Stage 5: Execution orchestration

Add:

```text
run IDs
run templates
phase logs
worktree-per-agent convention
stop/review points
deviation logging
```

Done when:

```text
Every agent task leaves a useful execution trail.
```

---

## Stage 6: Process learning

Add:

```text
retrospective template
failure taxonomy
process patch proposals
small eval suite for recurring failures
prompt/template versioning
```

Done when:

```text
Repeated agent mistakes produce checks/templates/prompts instead of just frustration.
```

---

## Stage 7: JIT automation

Add:

```text
friction mining from retros
experimental generated tools
promotion criteria
measurement of tool usefulness
stable tool registry
```

Done when:

```text
The repo starts accumulating small tools that remove repeated manual work.
```

---

## Stage 8: Advanced platform

Add only when needed:

```text
LLM traces
dashboards
vector/graph search
multi-repo indexing
org-level policy enforcement
agent eval harness
observability platform
```

Done when:

```text
The process is measurable across many agents, repos and teams.
```

---

## 17. Failure taxonomy

Useful classification for retrospectives:

```text
context-selection-failure
  Agent did not load relevant files/specs/facts.

spec-ambiguity
  Change spec was too vague or contradictory.

acceptance-gap
  Acceptance criteria did not cover important behavior.

verification-gap
  No test/check caught the issue.

contract-drift
  Code changed but contract was not updated/reviewed.

doc-drift
  Code/spec changed but docs/facts remained stale.

architecture-drift
  Design changed without ADR/update.

memory-harm
  Existing memory was misleading or stale.

execution-scope-creep
  Agent expanded scope without approval.

run-evidence-missing
  Agent made changes without enough evidence/logging.

tooling-gap
  Repetitive work should be automated.

prompt-policy-gap
  AGENTS.md or template instruction was missing/weak.
```

---

## 18. Example end-to-end flow

```text
User asks:
  Add refresh token auth.

obend creates:
  changes/add-refresh-token-auth/

Agent fills:
  proposal.md
  requirements.md
  acceptance.md
  design.md
  tasks.md
  verification.md

obend impact finds:
  src/auth/**
  tests/auth/**
  specs/current/auth/session-model.md
  docs/facts/auth-session-model.fact.md
  contracts/openapi/public-api.yaml
  docs/adr/0002-use-refresh-tokens.md

Agent runs:
  obend run start add-refresh-token-auth

Run writes:
  runs/2026-06-05-add-refresh-token-auth/plan.md
  runs/2026-06-05-add-refresh-token-auth/execution-log.md
  runs/2026-06-05-add-refresh-token-auth/test-results.md

Agent implements:
  refresh token model
  endpoint
  tests
  OpenAPI update
  stable auth spec update

Verification runs:
  lint
  typecheck
  unit tests
  auth integration tests
  contract check
  freshness check

Freshness watcher says:
  auth-session-model.fact.md must be reviewed
  session-model.md updated
  OpenAPI updated

PR includes:
  code diff
  tests
  contract update
  memory updates
  run evidence

After merge:
  change archived
  facts/specs updated
  indexes regenerated

Retro says:
  if agent forgot something, add process patch or check
```

---

## 19. Product shape

OctoBend can start as a repo template plus CLI.

### Initial product

```text
A repository operating system for coding agents.
```

### MVP package contents

```text
1. Repo structure generator
2. Markdown templates
3. AGENTS.md operating protocol
4. Change/run/fact schemas
5. Freshness rules
6. CI checks
7. Context pack generator
8. Retrospective/process patch workflow
9. JIT tool proposal workflow
```

### CLI shape

```bash
obend init
obend change new <name>
obend change validate <name>
obend impact <name>
obend context build <name>
obend run start <name>
obend run log <run-id>
obend verify
obend freshness check
obend memory promote
obend retro create <run-id>
obend tool propose <run-id>
obend reindex
```

### Possible repo positioning

```md
# OctoBend

OctoBend is an AI engineering control loop for agent-assisted software development.

It gives coding agents a repo-native operating system:

- stable project memory;
- versioned change specs;
- controlled execution runs;
- context retrieval;
- verification gates;
- freshness checks;
- process retrospectives;
- JIT automation for repeated agent work.

It is not a coding agent.
It is not a prompt collection.
It is not a replacement for tests, specs or code review.
It is the control layer around them.
```

---

## 20. Core principles

```text
1. Long-term memory without freshness watcher rots quickly.
2. Execution without self-optimization repeats the same mistakes.
3. Self-optimization without evals becomes hallucinated process improvement.
4. Freshness without deterministic rules becomes vibes.
5. Search must not become source of truth.
6. Important knowledge found by search should be promoted into curated memory.
7. Agents must not silently rewrite trusted memory.
8. Memory updates should be proposed with evidence.
9. SDD is not only spec → code.
10. The full loop is memory → intent → execution → verification → learning → freshness.
11. Run logs are evidence, not memory.
12. Dynamic markdown artifacts are context packs, not truth.
13. JIT tools must be reviewed before becoming stable.
14. Code/spec/docs/contracts conflicts are drift events, not automatic overwrites.
15. Good AI engineering is boring: artifacts, lifecycle, gates, evidence, ownership.
```

---

## 21. The shortest version

```text
OctoBend = AI Engineering Control Loop.

Stable Knowledge:
  What the project knows.

Change Control:
  What should change.

Execution:
  How the agent performs the change.

Context Access:
  How relevant context is found and packaged.

Evidence & Quality:
  How correctness is proven.

Knowledge Reconciliation:
  How docs/specs/facts stay fresh after code changes.

Process Learning:
  How the workflow improves after failures.

JIT Automation:
  How repeated friction becomes tools/templates/prompts.
```

That is the monster.
