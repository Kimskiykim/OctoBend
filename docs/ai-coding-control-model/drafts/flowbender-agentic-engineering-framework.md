# Flowbender / Agentic Engineering Framework

Core idea: a modular capability framework for governed, inspectable, self-improving AI-assisted software engineering.

---

## 1. What are we building?

We are **not** building “just another SDD framework”.

We are designing a **capability architecture for AI-assisted engineering**: a modular system of concepts, artifacts, policies, operations, and adapters that allows humans and AI coding agents to build software in a controlled, repeatable, inspectable way.

The goal is to move from:

```text
vibe coding with agents
```

to:

```text
governed agentic engineering
```

The framework should not be tied to one specific tool such as Codex, Claude Code, Cursor, Windsurf, Kiro, Cline, or GitHub Copilot. These tools can act as **adapters / executors**. The framework describes the capabilities around them.

---

## 2. Problem statement

Modern coding agents can write code, search files, run commands, create patches, and sometimes manage whole tasks. But raw agentic coding has recurring problems:

```text
- Intent lives in chat history and disappears.
- Project knowledge is scattered across README, code, docs, issues, and human memory.
- Agents lose context during long tasks.
- Specs, docs, ADRs, and code drift apart.
- Search is confused with source of truth.
- Verification is often shallow: “tests passed” instead of “the change is correct”.
- Agent runs do not reliably produce reusable knowledge.
- Mistakes repeat because the process does not learn.
- Agents may overreach without explicit boundaries.
- Temporary prompts/scripts/checklists are created ad hoc and then lost.
```

The framework should answer:

```text
How do we organize development with AI agents as an engineering system?
```

Not:

```text
What is the one perfect workflow?
```

---

## 3. Core shift: from fixed flow to capabilities

Earlier framing:

```text
Memory → Intent → Execution → Retrieval → Verification → Freshness → Learning
```

This is useful, but too flow-oriented.

Better framing:

```text
The framework defines composable capabilities.
Specific workflows are recipes built from those capabilities.
```

Different tasks need different recipes:

```text
feature development
bugfix
refactor
migration
security patch
incident response
documentation update
architecture decision
```

But they reuse the same core capabilities:

```text
intent management
memory management
context retrieval
impact analysis
execution control
verification
freshness watching
repo learning
self-optimization
dynamic scaffolding
JIT automation
governance
provenance
tool routing
release feedback
```

---

## 4. Core vocabulary

### Capability

A **capability** is a reusable engineering function.

It has:

```yaml
name: capability-name
purpose: why this capability exists
inputs: what it consumes
outputs: what it produces
operations: what it can do
artifacts: files or records it creates
policies: rules and constraints
adapters: tools that can implement it
quality_criteria: how we know it worked well
```

### Recipe

A **recipe** is a task-specific composition of capabilities.

Example:

```text
Feature development recipe:
  Intent Management
  → Context Retrieval
  → Impact Analysis
  → Execution Control
  → Verification
  → Freshness Watching
  → Repo Learning
```

### Adapter

An **adapter** maps a capability to a concrete tool.

Examples:

```text
Codex
Claude Code
Cursor
Cline / Roo
Kiro
GitHub Actions
ripgrep
ast-grep
Sourcegraph
Repomix
RepoPrompt
custom scripts
MCP servers
```

### Artifact

An **artifact** is a durable or temporary file produced by the process.

Examples:

```text
change spec
memory fact
context pack
execution run log
verification report
freshness report
lesson
retrospective
dynamic prompt
scratch tool
codemod
```

---

## 5. Main principle

```text
Memory ≠ Search
Search ≠ Spec
Spec ≠ Execution
Execution ≠ Verification
Verification ≠ Freshness
Freshness ≠ Learning
Learning ≠ Silent Mutation
```

The system must keep these boundaries explicit.

---

# 6. Capability map

## 6.1 Intent Management

### Purpose

Manage intentions and changes.

This capability turns vague ideas into explicit, versioned, reviewable change artifacts.

It answers:

```text
What do we want to change?
Why?
What are the requirements?
What is in scope and out of scope?
How will we know it is done?
What are the affected areas?
What open questions remain?
```

### Typical operations

```text
capture_intent
clarify_scope
define_requirements
define_acceptance_criteria
identify_affected_areas
record_open_questions
create_change_artifact
request_approval
```

### Inputs

```text
raw user request
project memory
existing specs
ADRs
code context
issue / ticket / PR
```

### Outputs

```text
intent.md
requirements.md
acceptance.md
affected-areas.md
open-questions.md
change-spec.md
```

### Example structure

```text
changes/
  add-refresh-token-auth/
    intent.md
    requirements.md
    design.md
    tasks.md
    acceptance.md
    affected-areas.md
    open-questions.md
```

### Related tools / approaches

```text
GitHub Spec Kit
OpenSpec
Kiro Specs
BDD / Gherkin
OpenAPI / AsyncAPI
ADR-per-change
```

### Important principle

```text
Intent must not live only in chat.
Intent must become a versioned artifact.
```

---

## 6.2 Knowledge / Memory Management

### Purpose

Maintain long-term project knowledge.

This is the curated source of truth for project knowledge.

It answers:

```text
What does the project know?
What are the domain concepts?
What architecture decisions are active?
What constraints must not be violated?
What patterns and anti-patterns exist?
How do we run and verify the system?
What facts are active, deprecated, or superseded?
```

### Typical operations

```text
record_fact
classify_knowledge
update_fact
deprecate_fact
supersede_fact
link_fact_to_sources
summarize_domain
promote_lesson_to_memory
```

### Artifacts

```text
AGENTS.md
CLAUDE.md
docs/context/product.md
docs/context/domain.md
docs/context/architecture.md
docs/context/tech.md
docs/context/operations.md
docs/facts/*.fact.md
docs/patterns/*.md
docs/anti-patterns/*.md
docs/adr/*.md
docs/runbooks/*.md
```

### Knowledge types

```text
fact
decision
constraint
assumption
risk
open-question
pattern
anti-pattern
run-lesson
deprecated-fact
temporary-note
```

### Important principle

```text
Memory is curated knowledge, not search results.
```

If a useful thing is found through search, it should be promoted into curated memory only after validation.

---

## 6.3 Context Retrieval

### Purpose

Find relevant context quickly.

This capability does not decide truth. It retrieves candidates and evidence.

It answers:

```text
Where is this feature implemented?
Which docs/specs/ADRs mention it?
Which tests cover it?
Which previous runs touched it?
What evidence should be packed for the agent?
What is missing or uncertain?
```

### Typical operations

```text
search_code
search_docs
search_specs
search_ADRs
search_tests
search_previous_runs
rank_results
build_context_pack
cite_sources
expose_gaps
```

### Inputs

```text
query
scope
current change
codebase
project memory
specs
ADRs
runs
issues / PRs
```

### Outputs

```text
context-pack.md
retrieval-report.md
evidence-list.yaml
gaps.md
```

### Implementation levels

#### Simple

```text
ripgrep
find
file paths
markdown frontmatter
```

#### Medium

```text
ripgrep + ast-grep
symbol search
module map
test map
```

#### Advanced

```text
Sourcegraph
vector search
GraphRAG
code graph
MCP retrieval servers
RepoPrompt
Repomix
```

### Important principle

```text
Retrieval accelerates access to truth.
It must not become the source of truth.
```

---

## 6.4 Impact Analysis

### Purpose

Understand consequences of a change.

It answers:

```text
If this module changes, what else is affected?
Which specs are touched?
Which tests should run?
Which docs may drift?
Which owners should review?
Which risks are introduced?
Which downstream APIs may break?
```

### Typical operations

```text
map_affected_files
map_specs
map_tests
map_owners
map_dependencies
map_runtime_risks
select_verification_scope
```

### Artifacts

```text
impact-report.md
affected-areas.yaml
test-selection.yaml
spec-to-code-map.yaml
module-map.yaml
ownership.yaml
service-dependencies.yaml
```

### Example

```yaml
module: auth
source_files:
  - src/auth/**
specs:
  - specs/auth/**
tests:
  - tests/auth/**
owners:
  - platform
depends_on:
  - user
  - permissions
freshness_policy:
  - check_auth_docs_on_auth_diff
```

### Important distinction

```text
Retrieval finds related information.
Impact analysis maps consequences.
```

---

## 6.5 Execution Control

### Purpose

Control how an agent performs work.

It does not necessarily write code itself. It manages the execution process around the coding agent.

It answers:

```text
What phase are we in?
What should the agent do next?
When should it stop?
When should it verify?
When should it escalate?
What is allowed?
What has changed?
What evidence must be produced before completion?
```

### Typical operations

```text
create_execution_plan
split_into_steps
assign_steps
checkpoint
pause
escalate
track_diff
record_commands
summarize_progress
complete_run
```

### Inputs

```text
approved change
project memory
execution policy
context pack
impact report
verification requirements
allowed tools
```

### Outputs

```text
execution-plan.md
run-log.md
checkpoints.md
changed-files.md
commands-run.md
completion-report.md
unresolved-issues.md
```

### Related tools / approaches

```text
GSD / GSD Core
gstack
BMAD Method
Codex task execution
Claude Code
Cline / Roo
worktree-per-agent
review gates
```

### Runtime boundaries as sublayer

Sandboxing is not a top-level framework if tools like Codex already implement it.

But the framework still needs an explicit **Runtime & Permission Boundary** sublayer:

```text
where the agent works
which files it can edit
whether network is allowed
whether secrets are accessible
what commands are forbidden
what requires approval
what must be logged
```

This is a policy/contract around tool-provided mechanisms.

---

## 6.6 Verification

### Purpose

Prove that the result is correct enough.

It answers:

```text
Did the change satisfy the spec?
Did tests pass?
Did we run the right tests?
Did public contracts change?
Did docs/specs drift?
Did security posture change?
Can this be shipped safely?
```

### Typical operations

```text
select_relevant_checks
run_tests
run_lint
run_typecheck
run_contract_checks
run_security_checks
compare_with_spec
check_docs_drift
produce_verification_report
```

### Artifacts

```text
verification-report.md
test-results.md
spec-compliance.md
security-review.md
docs-drift-report.md
contract-check-report.md
```

### Example

```yaml
verification:
  change: add-refresh-token-auth
  status: failed
  checks:
    unit_tests: passed
    typecheck: passed
    lint: passed
    auth_contract_tests: failed
    docs_updated: missing
  required_actions:
    - fix refresh token expiration contract
    - update specs/auth/session-management.md
```

### Important principle

```text
Verification is not just “tests passed”.
Verification means spec compliance + relevant checks + evidence.
```

---

## 6.7 Policy / Governance

### Purpose

Define permissions, boundaries, approvals, and safety constraints.

It answers:

```text
What may the agent do without approval?
What requires human review?
What is forbidden?
Which paths are protected?
Which changes require ADR?
Which changes require security review?
Who can approve trusted memory updates?
```

### Typical operations

```text
allow
deny
require_approval
enforce_ownership
protect_paths
classify_risk
```

### Artifacts

```text
policy.yaml
approval-rules.yaml
protected-paths.yaml
tool-permissions.yaml
CODEOWNERS
docs-owners.yaml
spec-owners.yaml
memory-owners.yaml
```

### Example

```yaml
agent_policy:
  may:
    - read_code
    - edit_tests
    - create_draft_specs
    - propose_patches
  must_not:
    - edit_secrets
    - delete_migrations
    - modify_production_config_without_approval
    - rewrite_trusted_memory_directly
  approval_required:
    - auth_changes
    - security_changes
    - infra_changes
    - public_api_changes
```

### Important principle

```text
Execution tells the agent how to work.
Governance tells the agent what is allowed.
```

---

## 6.8 Evidence / Provenance

### Purpose

Make decisions and outputs traceable.

It answers:

```text
Why did the agent decide this?
Which files supported the conclusion?
Which spec required it?
Which ADR allowed it?
Which tests verified it?
Which memory facts were used?
Which tool produced this result?
```

### Typical operations

```text
collect_evidence
link_decision_to_sources
cite_files
cite_specs
cite_ADRs
cite_tests
build_decision_trace
audit_run
```

### Artifacts

```text
evidence.yaml
decision-trace.md
source-map.yaml
provenance-report.md
```

### Example

```yaml
decision: use refresh tokens
evidence:
  specs:
    - specs/auth/session-management.md
  adr:
    - docs/adr/0002-use-refresh-tokens.md
  code:
    - src/auth/token-service.ts
    - src/auth/session-store.ts
  tests:
    - tests/auth/refresh-token.test.ts
  constraints:
    - checks/security-baseline.md
```

### Important distinction

```text
Execution log says what happened.
Provenance says why the decision was justified.
```

---

## 6.9 Freshness Watching

### Purpose

Keep project memory, docs, specs, and indexes current.

It answers:

```text
Code changed — which docs/specs/ADRs are stale?
API changed — was OpenAPI updated?
Spec closed — should stable capability specs be updated?
README says one thing — code does another?
Which memory facts are outdated?
Which indexes must be rebuilt?
```

### Typical operations

```text
detect_change
map_impact
check_freshness
find_stale_docs
find_conflicting_facts
reindex
propose_update
archive_or_invalidate
```

### Artifacts

```text
freshness-report.md
stale-facts.yaml
memory-update.patch.md
index-update-report.md
```

### Watcher lifecycle

```text
Change merged
  ↓
Watcher detects affected areas
  ↓
Retrieval finds related memory/specs/docs/tests
  ↓
Freshness check detects stale or missing knowledge
  ↓
Proposed PR updates memory/checks/templates
  ↓
Verification runs
  ↓
Memory archived / updated / invalidated
```

### Important principle

```text
Retrieval searches.
Watcher compares reality with memory.
```

---

## 6.10 Repo Learning / Knowledge Sharing

### Purpose

Allow agents to learn through the repository and share knowledge with future agents and humans.

This is not model weight training.

It is:

```text
reviewed, versioned, evidence-backed learning through repo artifacts
```

It answers:

```text
What did the agent learn during this task?
Is this knowledge reusable?
Is it a project fact, process lesson, anti-pattern, check, recipe, or ADR?
How should it be validated?
How will future agents discover it?
```

### Types of learning

```text
Task learning
Project learning
Process learning
Failure learning
Cross-agent learning
```

### Typical operations

```text
record_observation
write_retrospective
extract_lesson_candidate
classify_lesson
attach_evidence
propose_memory_update
propose_check_update
propose_recipe_update
approve_or_reject_lesson
publish_to_memory
```

### Artifacts

```text
learning/lessons/*.lesson.md
learning/retrospectives/*.retrospective.md
learning/proposed-updates/*.patch.md
learning/accepted/*.md
learning/rejected/*.md
docs/facts/*.fact.md
checks/*.md
recipes/*.md
```

### Dream process

```text
Agent works on task
  ↓
Agent records observations
  ↓
Agent writes retrospective
  ↓
Agent extracts reusable lessons
  ↓
Agent classifies lessons
  ↓
Agent proposes memory/rule/check/spec updates
  ↓
Verification validates proposed learning
  ↓
Human/CI approves trusted updates
  ↓
Knowledge is published into repo memory
  ↓
Future agents retrieve and use it
```

### Important principle

```text
Agents do not learn by silently changing themselves.
They learn by producing reviewed, versioned, evidence-backed artifacts.
```

---

## 6.11 Self-Optimization

### Purpose

Improve the AI-development process itself.

This is different from project learning.

It answers:

```text
Which task types does the agent repeatedly fail?
Which prompts or rules are weak?
Which specs are too vague?
Which checks should be added?
Which workflow step causes waste?
Which memory entries cause drift?
```

### Typical operations

```text
collect_traces
analyze_failures
mine_retrospectives
generate_process_improvement
run_eval
compare_process_versions
propose_policy_update
propose_template_update
```

### Artifacts

```text
retrospective.md
process-improvement.md
eval-results.md
prompt-experiment.md
policy-update.patch.md
template-update.patch.md
```

### Related tools / approaches

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
custom eval harnesses
```

### Important distinction

```text
Repo Learning:
  What did the project learn?

Self-Optimization:
  How should the agentic engineering process improve?
```

---

## 6.12 Tool / Model Routing

### Purpose

Select the right tool, model, or executor for each operation.

It answers:

```text
Which model should plan?
Which model should code?
Which model should review?
When should we use grep?
When should we use AST search?
When should we use semantic search?
When should we run e2e?
When should we delegate to a subagent?
```

### Typical operations

```text
choose_model
choose_search_tool
choose_executor
choose_verifier
choose_runtime
choose_context_strategy
```

### Artifacts

```text
routing-policy.yaml
tool-registry.yaml
model-policy.yaml
adapter-registry.yaml
```

### Example

```yaml
routing:
  planning:
    model: strong-reasoning
  implementation:
    executor: codex-or-claude-code
  code_search:
    tools:
      - ripgrep
      - ast-grep
  security_review:
    model: strong-reviewer
    requires_human_approval: true
```

---

## 6.13 Dynamic Scaffolding & JIT Automation

### Purpose

Allow agents to instrument their own work.

Agents should not only execute tasks. They should create task-specific markdown artifacts, prompts, checklists, scripts, codemods, and verification tools when useful.

It answers:

```text
What temporary artifacts are needed for this task?
Should we generate a context pack?
Should we create a dynamic prompt?
Is there repeated work?
Can this be automated?
Should a scratch tool be promoted to a reusable tool?
```

### Typical operations

```text
generate_task_scaffold
generate_context_pack
generate_dynamic_prompt
generate_verification_checklist
detect_repetition
create_scratch_tool
create_codemod
create_audit_script
validate_generated_tool
promote_tool
archive_or_delete_temp_artifacts
```

### Dynamic markdown artifacts

```text
runs/<run-id>/
  task-brief.md
  context-pack.md
  dynamic-prompt.md
  implementation-plan.md
  verification-checklist.md
  reviewer-brief.md
  handoff.md
```

### Prompt-as-artifact

```text
runs/<run-id>/prompts/
  implement-token-rotation.prompt.md
  review-auth-security.prompt.md
  verify-openapi-drift.prompt.md
```

### Task-specific tools

```text
tools/scratch/
tools/codemods/
tools/audits/
tools/checks/
tools/migration-helpers/
tools/test-generators/
tools/report-generators/
```

### When to create a tool

Create a tool if manual work:

```text
- repeats 3+ times
- touches 10+ files
- requires high precision
- should be reproducible
- can become a future check
- is needed for migration/refactor/search/verification
```

Do not create a tool if:

```text
- the task is simple and one-off
- the tool is more complex than the work
- there is no way to verify the tool
- it could damage code/data without review
- the task requires human judgment
```

### Important distinction

```text
Self-Optimization improves the process.
JIT Automation creates concrete tools/artifacts to perform work better.
```

---

## 6.14 Release / Runtime Feedback

### Purpose

Connect development work to deployment and production behavior.

It answers:

```text
How is the change released?
Is there a feature flag?
How do we rollback?
Which metrics should we observe?
Did production behavior improve or degrade?
Did incidents occur?
What should be learned from runtime feedback?
```

### Typical operations

```text
create_release_plan
create_rollback_plan
define_rollout_metrics
observe_runtime
collect_incidents
create_postmortem
update_memory_from_runtime
add_checks_from_incidents
```

### Artifacts

```text
release-plan.md
rollback-plan.md
migration-plan.md
feature-flag.md
post-release-checklist.md
runtime-feedback.md
postmortem.md
```

### Important principle

```text
Verification proves the change before release.
Runtime feedback checks what happened after release.
```

---

# 7. Supporting concepts

## 7.1 Artifact Lifecycle

Artifacts need lifecycle states.

Without lifecycle, the repo becomes a graveyard of stale files.

### Example lifecycles

#### Change

```text
proposed → planned → approved → executing → verifying → completed → archived
```

#### Memory fact

```text
proposed → active → stale → superseded → deprecated → rejected
```

#### ADR

```text
proposed → accepted → active → superseded → deprecated
```

#### Dynamic prompt

```text
draft → used → evaluated → promoted / archived
```

#### Scratch tool

```text
scratch → validated → promoted → maintained / deleted
```

---

## 7.2 Conflict Resolution

The system must distinguish:

```text
actual truth      = how the system currently works
intended truth    = how the system should work
historical truth  = why it became this way
```

Potential conflict examples:

```text
docs say one thing, code does another
old ADR conflicts with new spec
tests preserve behavior that spec now forbids
two agents propose conflicting changes
```

A source hierarchy may depend on the question.

For actual behavior:

```text
current production behavior
current code + tests
runtime logs
docs
chat history
```

For intended behavior:

```text
approved spec
accepted ADR
product decision
tests to be updated
current code
```

Important:

```text
Do not let LLM vibes decide freshness or conflict resolution.
Use metadata, ownership, lifecycle, source priority, and evidence.
```

---

## 7.3 Ownership / Responsibility

Every important artifact should have an owner.

It answers:

```text
Who owns this fact?
Who can approve this spec?
Who reviews this module?
Who can accept memory changes?
Who is responsible for freshness?
```

Artifacts:

```text
CODEOWNERS
docs-owners.yaml
spec-owners.yaml
memory-owners.yaml
approval-policy.yaml
```

---

## 7.4 Security / Threat Modeling

AI-agent workflows introduce extra risks:

```text
prompt injection from docs/issues
tool poisoning
secret leakage
unsafe shell commands
supply-chain attacks
agent over-permission
malicious instructions in repo content
```

Security artifacts:

```text
threat-model.md
security-review.md
secrets-policy.md
tool-permissions.yaml
dependency-risk.md
data-classification.md
```

Security-sensitive changes should require explicit review and stronger verification.

---

## 7.5 Simulation / What-if Analysis

Before coding, the system may simulate consequences:

```text
What if we choose this design?
What modules will be affected?
What edge cases appear?
What migration is required?
What rollout risks exist?
```

Artifacts:

```text
what-if-analysis.md
design-alternatives.md
risk-matrix.md
migration-simulation.md
```

---

## 7.6 Benchmark / Regression Task Suite

The framework itself needs tests.

Example benchmark tasks:

```text
agent must add endpoint without breaking OpenAPI
agent must update docs when public API changes
agent must refuse unsafe command
agent must detect stale ADR
agent must run relevant tests only
agent must preserve backward compatibility
agent must create lesson after failure
agent must promote repeated manual work into a check
```

This supports Self-Optimization.

---

# 8. Proposed repository structure

```text
/
  README.md

  AGENTS.md
  CLAUDE.md

  capabilities/
    intent-management.md
    memory-management.md
    context-retrieval.md
    impact-analysis.md
    execution-control.md
    verification.md
    governance.md
    provenance.md
    freshness-watching.md
    repo-learning.md
    self-optimization.md
    tool-routing.md
    dynamic-scaffolding-jit-automation.md
    release-runtime-feedback.md

  schemas/
    capability.schema.yaml
    change.schema.yaml
    memory-fact.schema.yaml
    retrieval-result.schema.yaml
    impact-report.schema.yaml
    execution-run.schema.yaml
    verification-report.schema.yaml
    freshness-report.schema.yaml
    lesson.schema.yaml
    retrospective.schema.yaml
    dynamic-prompt.schema.yaml
    tool-promotion.schema.yaml

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

    patterns/
      api-error-handling.md
      auth-testing.md

    anti-patterns/
      direct-db-token-access.md

    adr/
      0001-use-postgres.md
      0002-use-refresh-tokens.md

    runbooks/
      local-dev.md
      auth-tests.md

  specs/
    auth/
    billing/
    onboarding/

  changes/
    add-refresh-token-auth/
      intent.md
      requirements.md
      design.md
      tasks.md
      acceptance.md
      affected-areas.md
      verification.md
      open-questions.md

  runs/
    2026-06-05-add-refresh-token-auth/
      task-brief.md
      context-pack.md
      dynamic-prompt.md
      execution-plan.md
      run-log.md
      changed-files.md
      commands-run.md
      test-results.md
      verification-report.md
      reviewer-brief.md
      retrospective.md
      handoff.md
      scratch/
        inspect-auth-usage.ts

  checks/
    docs-updated.md
    spec-compliance.md
    no-breaking-public-api.md
    no-architecture-change-without-adr.md
    memory-freshness.md
    security-baseline.md
    auth-contract-tests-required.md

  tools/
    scratch/
      README.md

    audits/
      find-unhashed-refresh-tokens.ts

    codemods/
      rename-session-id-to-auth-session-id.ts

    checks/
      verify-openapi-drift.ts
      verify-auth-token-storage.ts

    report-generators/
      generate-verification-report.ts

  prompts/
    implementation-agent.prompt.md
    security-review.prompt.md
    verification-agent.prompt.md

  recipes/
    feature-development.md
    bugfix.md
    refactor.md
    migration.md
    security-patch.md
    docs-update.md
    incident-response.md

  adapters/
    codex.md
    claude-code.md
    cursor.md
    github-actions.md
    ripgrep.md
    ast-grep.md
    sourcegraph.md
    repomix.md
    repoprompt.md
    mcp.md

  learning/
    lessons/
      2026-06-05-auth-refresh-token-tests.lesson.md

    retrospectives/
      2026-06-05-add-refresh-token-auth.retrospective.md

    proposed-updates/
      update-auth-facts.patch.md
      update-agent-rules.patch.md
      add-auth-verification-check.patch.md

    accepted/
      auth-token-patterns.md
      auth-test-selection.md

    rejected/
      avoid-global-memory-for-temporary-debugging.md

  memory-index/
    manifest.json
    ownership.json
    freshness-rules.yaml
    source-priority.yaml

  examples/
    add-refresh-token-auth/
    fix-payment-webhook/
    migrate-api-client/
```

---

# 9. Schemas

## 9.1 Capability schema

```yaml
name: context-retrieval
purpose: Find relevant context for a task.

inputs:
  - query
  - scope
  - current_change
  - project_memory
  - codebase

outputs:
  - context_pack
  - evidence_list
  - gaps

operations:
  - search_code
  - search_docs
  - search_specs
  - rank_results
  - build_context_pack

policies:
  - cite_sources
  - prefer_current_code_over_old_docs
  - mark_uncertain_results
  - do_not_treat_search_as_truth

adapters:
  simple:
    - ripgrep
  medium:
    - ripgrep
    - ast-grep
  advanced:
    - Sourcegraph
    - vector_index
    - graph_index

quality_criteria:
  - includes_relevant_code
  - includes_relevant_specs
  - includes_tests
  - exposes_uncertainty
  - avoids_stale_docs
```

---

## 9.2 Memory fact schema

```yaml
type: fact
id: auth-refresh-tokens-hashed
status: active
scope: auth
owner: platform
summary: Refresh tokens are stored hashed, never plaintext.

evidence:
  code:
    - src/auth/token-store.ts
  tests:
    - tests/auth/token-store.test.ts
  specs:
    - specs/auth/session-management.md

source_of_truth:
  - code
  - accepted_spec

freshness:
  check_on_diff:
    - src/auth/**
    - specs/auth/**
  last_checked: 2026-06-05

lifecycle:
  created_at: 2026-06-05
  valid_from: 2026-06-05
  superseded_by: null
  deprecated_at: null
```

---

## 9.3 Change schema

```yaml
type: change
id: add-refresh-token-auth
status: approved
owner: platform

intent:
  summary: Add refresh-token based session renewal.
  reason: Users should remain logged in without re-authenticating.

scope:
  include:
    - src/auth/**
    - tests/auth/**
    - specs/auth/**
  exclude:
    - infra/**
    - production-config/**

requirements:
  - Access tokens expire quickly.
  - Refresh tokens are rotated.
  - Refresh tokens are stored hashed.
  - Logout revokes refresh tokens.

acceptance_criteria:
  - User can refresh an expired access token.
  - Reused refresh token is rejected.
  - Logout invalidates active refresh token.
  - Auth contract tests pass.

risks:
  - token leakage
  - session fixation
  - backwards compatibility

verification:
  required:
    - unit_tests
    - auth_contract_tests
    - typecheck
    - security_review
    - docs_updated
```

---

## 9.4 Execution run schema

```yaml
type: execution_run
id: run-2026-06-05-add-refresh-token-auth
change: changes/add-refresh-token-auth
executor: codex
status: verifying

inputs:
  - change_spec
  - context_pack
  - policy
  - verification_requirements

steps:
  - id: 1
    title: inspect existing auth flow
    status: completed
  - id: 2
    title: implement refresh token store
    status: completed
  - id: 3
    title: add contract tests
    status: failed

commands_run:
  - pnpm test auth
  - pnpm typecheck

changed_files:
  - src/auth/token-store.ts
  - tests/auth/refresh-token.test.ts

checkpoints:
  - after_context_retrieval
  - after_implementation
  - before_completion

unresolved_issues:
  - auth contract test failing for token reuse behavior
```

---

## 9.5 Verification report schema

```yaml
type: verification_report
change: add-refresh-token-auth
status: failed

checks:
  unit_tests:
    status: passed
    command: pnpm test auth
  typecheck:
    status: passed
    command: pnpm typecheck
  auth_contract_tests:
    status: failed
    command: pnpm test contracts/auth
  docs_updated:
    status: missing
  security_review:
    status: pending

spec_compliance:
  refresh_token_rotation: passed
  token_reuse_rejection: failed
  logout_revocation: unknown

required_actions:
  - fix token reuse behavior
  - update specs/auth/session-management.md
  - complete security review
```

---

## 9.6 Lesson schema

```yaml
type: lesson
id: lesson-auth-refresh-token-tests-2026-06-05
scope: auth
source_run: runs/2026-06-05-add-refresh-token-auth
status: proposed
confidence: high

learned:
  summary: Auth refresh-token changes require contract tests, not only unit tests.
  details: Unit tests passed, but contract tests caught token expiration mismatch.

evidence:
  changed_files:
    - src/auth/token-service.ts
    - src/auth/session.ts
  failed_checks:
    - tests/contracts/auth-refresh-token.test.ts
  related_specs:
    - specs/auth/session-management.md

recommended_updates:
  memory:
    - docs/facts/auth-verification.fact.md
  checks:
    - checks/auth-contract-tests-required.md
  agent_rules:
    - AGENTS.md

freshness:
  recheck_on:
    - src/auth/**
    - specs/auth/**
```

---

## 9.7 Dynamic prompt schema

```yaml
type: dynamic_prompt
id: implement-token-rotation
run: runs/2026-06-05-add-refresh-token-auth
role: implementation-agent

goal: Implement refresh token rotation.

scope:
  include:
    - src/auth/**
    - tests/auth/**
  exclude:
    - infra/**
    - production-config/**

context:
  - runs/2026-06-05-add-refresh-token-auth/context-pack.md
  - specs/auth/session-management.md
  - docs/facts/auth-refresh-tokens-hashed.fact.md

constraints:
  - do not store refresh tokens in plaintext
  - do not change public API without updating OpenAPI
  - do not edit production config

required_checks:
  - pnpm test auth
  - pnpm test contracts/auth
  - pnpm typecheck

expected_outputs:
  - code diff
  - verification report
  - unresolved risks
```

---

# 10. Recipes

## 10.1 Feature development

```text
Intent Management
  → Context Retrieval
  → Impact Analysis
  → Execution Control
  → Dynamic Scaffolding
  → Verification
  → Freshness Watching
  → Repo Learning
```

## 10.2 Bugfix

```text
Context Retrieval
  → Impact Analysis
  → Intent Management
  → Execution Control
  → Verification
  → Repo Learning
```

## 10.3 Refactor

```text
Impact Analysis
  → Context Retrieval
  → Intent Management
  → Execution Control
  → JIT Automation / Codemod
  → Verification
  → Provenance
  → Freshness Watching
```

## 10.4 Migration

```text
Intent Management
  → Impact Analysis
  → Simulation / What-if
  → Dynamic Scaffolding
  → JIT Automation
  → Execution Control
  → Verification
  → Release / Rollback
  → Runtime Feedback
```

## 10.5 Security patch

```text
Intent Management
  → Governance
  → Threat / Impact Analysis
  → Execution Control
  → Security Verification
  → Human Approval
  → Freshness Watching
  → Repo Learning
```

## 10.6 Documentation update

```text
Freshness Watching
  → Context Retrieval
  → Memory Management
  → Verification
  → Provenance
```

## 10.7 Incident response

```text
Runtime Feedback
  → Context Retrieval
  → Impact Analysis
  → Intent Management
  → Execution Control
  → Verification
  → Postmortem
  → Repo Learning
  → Self-Optimization
```

---

# 11. MVP version

Do not implement everything at once.

## MVP for solo developer

### Keep

```text
AGENTS.md
docs/context/*
docs/adr/*
changes/*
runs/*
checks/*
recipes/*
learning/lessons/*
```

### Capabilities to implement first

```text
1. Intent Management
2. Context Retrieval
3. Execution Control
4. Verification
5. Repo Learning
6. Freshness Watching
7. Dynamic Scaffolding & JIT Automation
```

### Simple tooling

```text
markdown
git
ripgrep
shell scripts
GitHub Actions
basic templates
```

### Avoid initially

```text
GraphRAG
complex vector DB
multi-agent orchestration
heavy eval platform
enterprise governance
```

---

## MVP repository structure

```text
/
  README.md
  AGENTS.md

  capabilities/
    intent-management.md
    context-retrieval.md
    execution-control.md
    verification.md
    repo-learning.md
    freshness-watching.md
    dynamic-scaffolding-jit-automation.md

  templates/
    change.md
    run.md
    verification-report.md
    lesson.md
    memory-fact.md
    dynamic-prompt.md

  docs/
    context/
      product.md
      architecture.md
      tech.md
      operations.md
    adr/

  changes/
  runs/
  checks/
  recipes/
  learning/
  tools/
```

---

## MVP command ideas

```text
flowbender init
flowbender new change add-refresh-token-auth
flowbender pack-context changes/add-refresh-token-auth
flowbender start-run changes/add-refresh-token-auth
flowbender verify runs/<run-id>
flowbender lesson runs/<run-id>
flowbender freshness --diff main...HEAD
flowbender promote-tool tools/scratch/foo.ts
```

These can initially be shell scripts or documented manual commands.

---

# 12. Roadmap

## Phase 0: Concept repo

Goal: document the architecture.

Deliverables:

```text
README.md
capability docs
schemas
templates
example project
```

## Phase 1: Markdown-first framework

Goal: make it usable without custom software.

Deliverables:

```text
templates
recipes
manual workflows
AGENTS.md example
sample checks
sample learning loop
```

## Phase 2: Lightweight CLI

Goal: reduce friction.

Deliverables:

```text
init command
new change command
new run command
context pack generator
verification report generator
freshness diff checker
lesson extractor
```

## Phase 3: CI integration

Goal: enforce selected rules.

Deliverables:

```text
docs updated check
spec compliance check
memory freshness warning
protected path policy
public API drift check
```

## Phase 4: Agent adapters

Goal: integrate with coding agents.

Deliverables:

```text
Codex adapter guide
Claude Code adapter guide
Cursor adapter guide
Cline adapter guide
GitHub Actions adapter
MCP adapter
```

## Phase 5: Learning and optimization

Goal: make each run improve future runs.

Deliverables:

```text
retrospective miner
lesson promotion workflow
process improvement proposals
benchmark task suite
prompt/template evals
```

## Phase 6: Advanced retrieval and graph

Goal: support larger repos.

Deliverables:

```text
code graph
semantic index
spec-to-code map
test selection map
GraphRAG option
temporal memory option
```

---

# 13. One-sentence framing

```text
Flowbender is a modular capability framework for governed, inspectable, self-improving AI-assisted software engineering.
```

---

# 14. Strong principles for README

```text
1. The repository is not only a codebase.
   It is the learning substrate for agents.

2. Agents should not only execute tasks.
   They should instrument their own work.

3. Intent must not live only in chat.
   It must become a versioned artifact.

4. Memory without freshness watching becomes stale.

5. Search is not source of truth.
   It only accelerates access to sources.

6. Self-optimization without evals becomes hallucinated process improvement.

7. Learning is not silent mutation.
   It is reviewed, versioned, evidence-backed knowledge update.

8. Useful temporary artifacts should be promoted.
   Useless ones should be archived or deleted.

9. Coding agents are executors.
   The framework defines the engineering system around them.

10. SDD is not the whole system.
    SDD is one capability: Intent / Change Management.
```

---

# 15. Open design questions

```text
1. How strict should the schemas be at MVP stage?
2. Should the project start as documentation-only or include a CLI immediately?
3. Should AGENTS.md be the primary entrypoint or generated from capability policies?
4. How should trusted memory approval work in solo mode?
5. How to prevent dynamic markdown artifacts from polluting the repo?
6. How to decide when a scratch tool should be promoted?
7. How to define source priority for actual truth vs intended truth?
8. How to benchmark the framework itself?
9. How tool-specific should adapters be?
10. How much should be enforced by CI vs left as agent guidance?
```

---

# 16. Minimal first milestone

The first real milestone should be:

```text
A repo that explains Flowbender and includes enough templates to use it manually.
```

Minimum files:

```text
README.md
AGENTS.md
capabilities/*.md
templates/change.md
templates/run.md
templates/verification-report.md
templates/lesson.md
templates/dynamic-prompt.md
recipes/feature-development.md
recipes/bugfix.md
examples/add-refresh-token-auth/*
```

This is enough to make the idea concrete without prematurely building tooling.

---

# 17. One-sentence definition

```text
Flowbender is a modular capability framework for turning AI coding agents from ad hoc code generators into governed, inspectable, self-improving participants in a real software engineering process.
```
