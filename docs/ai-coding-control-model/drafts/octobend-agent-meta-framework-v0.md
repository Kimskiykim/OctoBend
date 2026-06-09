# OctoBend Agent Meta-Framework v0

> Consolidated framework distilled from the chat notes and architecture drafts in this repository.
> Goal: separate project knowledge from agent-specific work, describe agent meta-tasks, keep the model modular and layered, and make the first version usable immediately.

---

## 1. Short Definition

OctoBend is a repo-native control framework for AI-assisted software development.

It does not replace Codex, Claude Code, Cursor, CI, tests, ADRs or specs. It defines the control layer around them:

```text
Project artifacts are durable nouns.
Agent operations are controlled verbs.
Policies define which verbs may act on which nouns, under what evidence and review rules.
```

The practical purpose:

```text
Turn chat-driven agent work into inspectable engineering work:
intent -> context -> impact -> execution -> evidence -> freshness -> learning.
```

But this is not a rigid pipeline. It is a set of capabilities and recipes. Different task types use different subsets.

---

## 2. Non-Negotiable Boundaries

These boundaries prevent most agent workflow confusion:

```text
Memory is not search.
Search is not truth.
Spec is not execution.
Execution is not verification.
Tests passed is not full evidence.
Run logs are not trusted memory.
Change specs are not stable specs.
ADRs are rationale, not always current behavior.
Generated context packs are not source of truth.
Self-optimization is not silent self-mutation.
```

The agent must always know which question it is answering:

| Question | Strongest sources |
|---|---|
| What is implemented now? | runtime evidence, tests, code |
| What is intended/promised? | contracts, stable specs, accepted requirements |
| Why was it designed this way? | ADRs, design docs, reviewed decisions |
| What are we changing now? | active change spec |
| What happened in this run? | run log and evidence |
| What should future agents learn? | reviewed facts, lessons, policies, checks |

Conflict between sources is not an automatic overwrite. It is a drift event that needs resolution.

---

## 3. Layered Model

OctoBend has six layers. Each layer has a different abstraction level and lifecycle.

```text
L0. Principles
    Invariants and forbidden confusions.

L1. Project Knowledge
    Durable nouns: facts, specs, contracts, ADRs, docs, code maps, runbooks.

L2. Change State
    Bridge artifacts: what we want to change now, why, and how completion is judged.

L3. Agent Operations
    Controlled verbs: clarify, retrieve, analyze, implement, verify, reconcile, learn, automate.

L4. Control Policies
    Authority, permissions, risk, lifecycle, freshness, evidence and review gates.

L5. Recipes and Adapters
    Task-specific workflows and concrete tools: Codex, Claude Code, CI, rg, ast-grep, OpenAPI diff, MCP.
```

The main design rule:

```text
Do not encode a task workflow directly into project memory.
Do not encode project truth directly into an agent prompt.
Bind them through explicit policies and artifacts.
```

---

## 4. Project Knowledge Model: The Nouns

Project knowledge answers:

```text
What does the project know, promise, contain, require or prohibit?
```

Starter taxonomy:

| Artifact class | Examples | Purpose | Update rule |
|---|---|---|---|
| Project rules | `AGENTS.md`, `.ai/policy/*` | How humans/agents must work | reviewed change |
| Product/domain | `docs/context/product.md`, `docs/context/domain.md` | Business and domain context | reviewed change |
| Architecture | `docs/context/architecture.md`, `docs/adr/*` | System shape and rationale | ADR/update/waiver |
| Technical policy | `docs/context/tech.md`, dependency policy | Stack, commands, conventions | reviewed change |
| Stable specs | `specs/current/**/*` | Current intended behavior | reviewed change |
| Contracts | `contracts/openapi/*`, protobuf, DB schemas | Public/internal promises | contract review |
| Atomic facts | `docs/facts/*.fact.md` | Small reusable knowledge units | owner + evidence + freshness |
| Code maps | `docs/code-map.md`, ownership maps | Conceptual code responsibility | curated, not symbol dump |
| Runbooks | `docs/runbooks/*` | Operational procedures | reviewed change |
| Derived indexes | `memory-index/generated/*` | Access acceleration | regenerated, never truth |

Minimal fact rule:

```text
No fact without owner.
No fact without evidence.
No fact without freshness trigger.
No fact that tries to describe the whole project.
```

---

## 5. Change State Model: The Bridge

Change artifacts answer:

```text
What are we trying to change right now?
```

They are not stable truth. They are a temporary bridge between human intent, agent execution and future stable knowledge.

Minimal structure:

```text
changes/<change-id>/
  proposal.md          why, scope, non-goals
  requirements.md      requirements and open questions
  acceptance.md        checkable acceptance criteria
  verification.md      commands, required checks, evidence
  memory-updates.md    expected docs/specs/facts/contracts updates or waivers
  status.yaml          status, risk, owner, lifecycle
```

For medium/high-risk changes, add:

```text
  design.md
  impact.md
  tasks.md
```

Change lifecycle:

```text
draft -> proposed -> approved -> in_progress -> implemented -> verified -> merged -> archived
```

Rules:

```text
No non-trivial implementation without an active change.
No implementation if acceptance criteria are missing.
No completion claim without verification evidence.
After merge, promote useful knowledge into stable artifacts or archive the change.
```

---

## 6. Agent Operation Model: The Verbs

Agent operations answer:

```text
What is the agent doing with project artifacts?
```

Core operation set:

| Operation | Goal | Reads | Writes | Must not write |
|---|---|---|---|---|
| `clarify-intent` | turn request into scoped change | user request, docs, specs | `changes/<id>/*` | production code |
| `retrieve-context` | find relevant sources | repo, specs, facts, tests | context pack/report | stable truth |
| `analyze-impact` | map consequences and risk | change, code, contracts, specs | `impact.md`, verification scope | production code |
| `plan-execution` | split work into steps | change, impact, context | `runs/<id>/plan.md` | code |
| `implement` | modify code/tests/docs in scope | approved change, context | code, tests, allowed docs | trusted memory silently |
| `verify` | prove result is acceptable | diff, specs, tests, contracts | verification report, evidence | specs/contracts silently |
| `reconcile-knowledge` | detect and propose knowledge updates | diff, specs, facts, docs | freshness report, update proposal | direct trusted overwrite |
| `review` | inspect correctness and risks | diff, evidence, policies | review notes | unrelated code |
| `learn` | extract reusable lessons | run logs, failures, review | lesson/process patch proposal | AGENTS/policy silently |
| `toolsmith` | turn repeated friction into tools | retros, repeated tasks | tool proposal, experimental script | stable tool without review |
| `handoff` | preserve task state | run, plan, evidence | handoff summary | hidden assumptions |

The agent should declare an operation header before non-trivial work:

```yaml
operation:
  mode: analyze-impact
  change: changes/add-refresh-token-auth
  target_artifacts:
    - specs/current/auth/session-model.md
    - contracts/openapi/public-api.yaml
    - docs/facts/auth-session-model.fact.md
  allowed_actions:
    - read
    - search
    - write impact report
  forbidden_actions:
    - edit production code
    - rewrite trusted memory
  evidence_expected:
    - changes/add-refresh-token-auth/impact.md
    - changes/add-refresh-token-auth/verification.md
```

Mode self-classification is not enough. Mode transitions require gates:

```text
clarify-intent -> implement:
  requires acceptance criteria

analyze-impact -> implement:
  requires affected areas and verification scope for medium/high risk

implement -> verify:
  requires diff and expected checks

verify -> complete:
  requires evidence and acceptance checklist

verify -> reconcile:
  required when code/spec/contract/doc/fact drift is possible
```

---

## 7. Control Policy Model: The Binding Layer

Policies answer:

```text
Which agent operations are allowed on which project artifacts?
What evidence is required?
Who reviews it?
When does it become stale?
```

Minimum policy files:

```text
.ai/policy/source-priority.md
.ai/policy/operation-matrix.md
.ai/policy/artifact-registry.md
.ai/policy/evidence-policy.md
.ai/policy/freshness-rules.md
.ai/policy/risk-levels.md
```

### 7.1 Source Priority

Starter priority:

```text
1. Runtime evidence / CI / tests
2. Public/internal contracts
3. Stable specs
4. Code
5. ADRs
6. Docs/context/facts
7. Change specs
8. Run logs/chat/search results
9. Generated indexes/vector DB/GraphRAG
```

Important nuance:

```text
Code is strongest for implemented behavior.
Specs/contracts are strongest for intended/promised behavior.
ADRs are strongest for historical rationale.
```

### 7.2 Risk-Based Control Budget

Do not use the same ceremony for every change.

| Risk | Examples | Required artifacts |
|---|---|---|
| Low | typo, local test fix, small internal refactor | short change note, verification evidence |
| Medium | feature, behavior change, module refactor | change spec, acceptance, impact, verification |
| High | auth, payments, security, DB migration, public API, infra | design, impact, security/contract checks, rollback/freshness, review |

Rule:

```text
Control intensity should match change risk.
```

### 7.3 Evidence Policy

Evidence must be inspectable and tied to acceptance.

Acceptable evidence:

```text
command output
CI link
test result file
contract diff
security scan report
review note
screenshot or trace when UI/runtime behavior matters
manual checklist with reviewer
```

Weak evidence:

```text
"I checked"
"tests pass" with no commands
LLM says it looks fine
diff exists
search found nothing
```

### 7.4 Freshness Policy

Freshness should be deterministic first, LLM second.

Triggers:

```text
API path changed -> contract update or waiver
architecture path changed -> ADR update/proposal or waiver
domain behavior changed -> spec/fact review
test meaning changed -> related spec/fact review
trusted memory changed -> explicit memory update plan
```

The watcher creates findings, not silent rewrites.

---

## 8. Capability Modules

Capabilities are reusable functions. Recipes compose them.

Core capabilities:

```text
intent-management
knowledge-management
context-retrieval
impact-analysis
execution-control
verification
evidence-provenance
knowledge-reconciliation
process-learning
toolsmith-jit-automation
governance
tool-model-routing
release-runtime-feedback
```

Each capability can be described with the same card:

```yaml
name:
purpose:
inputs:
outputs:
operations:
artifacts:
policies:
adapters:
quality_criteria:
```

This keeps the framework modular. A small repo can use only 5 capabilities. A larger org can add governance, routing, observability and evals later.

---

## 9. Recipes

Recipes are task-specific compositions of capabilities.

Feature:

```text
intent-management
-> context-retrieval
-> impact-analysis
-> execution-control
-> verification
-> knowledge-reconciliation
-> process-learning if needed
```

Bugfix:

```text
context-retrieval
-> impact-analysis
-> intent-management
-> execution-control
-> verification
-> process-learning if failure repeated
```

Refactor:

```text
impact-analysis
-> context-retrieval
-> intent-management
-> toolsmith-jit-automation if broad/mechanical
-> execution-control
-> verification
-> provenance
-> knowledge-reconciliation
```

Documentation update:

```text
knowledge-reconciliation
-> context-retrieval
-> knowledge-management
-> verification
-> provenance
```

Security patch:

```text
intent-management
-> governance
-> threat/impact-analysis
-> execution-control
-> security-verification
-> human approval
-> freshness
-> process-learning
```

---

## 10. Instant-Apply Kit

This is the minimal version to add to any repo today.

```text
/
  AGENTS.md

  .ai/
    policy/
      source-priority.md
      operation-matrix.md
      artifact-registry.md
      evidence-policy.md
      freshness-rules.md
      risk-levels.md
    templates/
      change.md
      run.md
      verification-report.md
      freshness-report.md
      fact.md
      lesson.md

  docs/
    context/
      product.md
      architecture.md
      tech.md
    facts/
    adr/

  specs/
    current/

  changes/
  runs/
  checks/
```

### 10.1 Minimal `AGENTS.md`

```md
# Agent Operating Protocol

## Core Rule

Work through project artifacts. Do not rely on chat memory alone.

## Before Non-Trivial Code Changes

1. Find or create an active `changes/<id>`.
2. Read `AGENTS.md`, `.ai/policy/source-priority.md`, and the active change.
3. Declare operation mode, target artifacts, write boundary and expected evidence.
4. Do not implement until acceptance criteria are clear.

## During Execution

1. Work in small steps.
2. Keep evidence in `runs/<run-id>/`.
3. Do not silently expand scope.
4. Do not treat search results as source of truth.
5. Do not rewrite trusted memory unless `memory-updates.md` exists.

## Verification

1. Run the commands listed in the active change.
2. Attach command output or CI links.
3. If a check is skipped, write a waiver.
4. Completion requires acceptance criteria and evidence.

## Freshness

If code behavior, public contracts, architecture, specs or facts may have changed, create a freshness finding or update proposal.
```

### 10.2 Minimal Change Template

````md
# Change: <id>

## Problem

What problem, user need or engineering risk triggers this change?

## Goal

What must become true after this change?

## Non-Goals

What is explicitly out of scope?

## Risk

low | medium | high

## Acceptance Criteria

- [ ] Observable behavior or engineering condition that proves the goal.
- [ ] Required edge case, regression or compatibility condition.

## Impact

- Code:
- Tests:
- Specs:
- Contracts:
- Facts/docs/ADR:

## Verification Plan

Commands:

```bash
# project-specific verification command, for example:
npm test
```

Required evidence:

- command output, CI link, report file or reviewer note

## Memory Updates

- [ ] No stable knowledge update needed
- [ ] Specs/facts/docs/contracts update proposed
- [ ] Waiver written
````

### 10.3 Minimal Run Template

````md
# Run: <run-id>

Change: `changes/<id>`
Agent/tool:
Branch:
Base commit:

## Operation Header

Mode:
Target artifacts:
Allowed writes:
Forbidden writes:
Expected evidence:

## Loaded Context

## Plan

## Actions

## Deviations

## Verification Evidence

## Freshness Findings

## Result
````

### 10.4 Minimal Fact Template

````md
---
id:
title:
status: active
owner:
updated:
source_of_truth:
evidence:
freshness_watches:
superseded_by:
---

# Fact

# Evidence

# Freshness Rule
````

---

## 11. Minimum Checks

Start with three checks. They can be manual, CI scripts, or PR checklist items.

```text
check.change-required
  Non-trivial PR must reference `changes/<id>`.

check.verification-evidence
  Change cannot close without commands/results or explicit waiver.

check.no-silent-trusted-memory-update
  Specs/facts/ADR/contracts/policy updates require a memory update note.
```

Add these next:

```text
check.api-contract
  API route/controller changes require contract update or waiver.

check.architecture-adr
  architecture-sensitive changes require ADR/update or waiver.

check.freshness-review
  watched paths require affected docs/specs/facts review.
```

---

## 12. Failure Taxonomy

Use these labels in retrospectives:

```text
intent-loss
scope-ambiguity
context-selection-failure
knowledge-authority-confusion
impact-blindness
premature-implementation
execution-drift
verification-gap
evidence-gap
contract-drift
doc-drift
memory-harm
unsafe-memory-write
tooling-gap
prompt-policy-gap
process-non-learning
```

Retrospectives should produce one of:

```text
no action
fact/spec/doc update proposal
template update proposal
policy update proposal
new check proposal
new tool proposal
eval/regression task proposal
```

---

## 13. New Ideas To Keep

These are additions to make the framework more immediately usable and less bureaucratic.

### 13.1 Control Budget

Every change gets a control budget based on risk. The framework should spend ceremony only where mistakes are expensive.

### 13.2 Truth Lens

Every conflict should be classified by lens:

```text
actual truth      what currently happens
intended truth    what should happen
historical truth  why it became this way
process truth     how the work was performed
```

This prevents the agent from "fixing docs" when the real issue is broken code or stale requirements.

### 13.3 Mode Gates

The agent may declare its mode, but policies decide whether that mode is allowed. For example, `implement` is blocked when acceptance criteria are absent.

### 13.4 Artifact TTL

Temporary artifacts should have expiry rules:

```text
context packs expire when source files change
changes archive after merge
runs archive after review
facts require review after watched path changes
scratch tools must be promoted or deleted
```

### 13.5 Friction-To-Tool Rule

If a task is repeated 3+ times, touches 10+ files, requires high precision, or should become reproducible, create a tool proposal.

### 13.6 Agent Handoff As First-Class Artifact

Long tasks should not rely on hidden conversation state. A run can create `handoff.md` with:

```text
current status
next step
files changed
commands run
known risks
unresolved questions
```

---

## 14. Adoption Path

Stage 1: artifact discipline

```text
AGENTS.md
change template
run template
evidence policy
source priority
```

Stage 2: verification gates

```text
required change id
required acceptance checklist
required verification evidence
PR template
```

Stage 3: freshness MVP

```text
artifact registry
watched paths
API -> contract rule
architecture -> ADR rule
domain behavior -> fact/spec review
```

Stage 4: retrieval MVP

```text
rg/ast-grep scripts
context pack generator
impact map generator
```

Stage 5: process learning

```text
retrospective template
failure taxonomy
process patch proposals
small eval/regression tasks
```

Stage 6: JIT automation

```text
tool proposals
experimental tools
promotion criteria
stable tool registry
```

Only after these stages should the project invest in vector/graph search, dashboards, complex multi-agent orchestration or org-level policy services.

---

## 15. One-Day Implementation

For immediate use in any repo:

1. Create `AGENTS.md` from the minimal protocol.
2. Create `.ai/policy/source-priority.md`.
3. Create `.ai/policy/operation-matrix.md`.
4. Create `.ai/templates/change.md` and `.ai/templates/run.md`.
5. Start the next non-trivial task under `changes/<id>`.
6. Require acceptance criteria before implementation.
7. Save verification output under `runs/<run-id>`.
8. Before calling the task complete, ask: what specs/contracts/facts/docs may now be stale?

This is enough to start. Everything else is optional growth.

---

## 16. Synthesis Sources

This v0 framework was synthesized from:

```text
ai_agentic_engineering_chat_notes.md
ai_engineering_control_system.md
flowbender_agentic_engineering_framework.md
octobend-architecture.md
```

The strongest idea from the source material is the split between project nouns and agent verbs.

The strongest correction is risk-based minimalism: do not build a beautiful markdown graveyard. Every artifact needs owner, lifecycle, evidence and freshness rule.
