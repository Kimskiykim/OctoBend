# Agentic Software Engineering Operating Model

**Research note:** найденные концепции, термины, фреймворки и источники, похожие на модель управления AI-кодингом.
**Дата подготовки:** 2026-06-09
**Язык источников:** преимущественно англоязычные академические статьи, engineering blogs, документация инструментов, open-source проекты и enterprise-подходы.

---

## 1. Executive summary

1. **Единого устоявшегося термина уровня `AI coding control model` пока нет.** Ближайшая зонтичная область — **Agentic Software Engineering**, **AI-native SDLC / PDLC**, **AI-assisted software delivery**, **AI coding governance**. Но поле пока распадается на отдельные дисциплины: context engineering, spec-driven development, agent orchestration, memory, verification/evals, governance и human-agent collaboration.

2. **Самый близкий найденный процессный аналог — `Agentic Agile-V / SCOPE-V`.** В статье *Agentic Agile-V: From Vibe Coding to Verified Engineering in Software and Hardware Development* предлагается task-loop **Specify → Constrain → Orchestrate → Prove → Evolve → Verify**. Это почти напрямую ложится на нашу модель: спецификация, ограничения/риск, исполнение, доказательства, улучшение процесса, проверка.

3. **`Context engineering` уже стал узнаваемым термином.** Anthropic, LangChain и Cognition описывают его как дисциплину управления тем, что агент получает в рабочий контекст: инструкции, состояние, память, инструменты, retrieval, сжатие, изоляция и выбор релевантной информации. Это хорошо покрывает блок `Контекст`, но не заменяет всю модель управления AI-разработкой.

4. **`Spec-driven development` быстро оформляется как отдельный слой процесса.** GitHub Spec Kit, OpenSpec и статьи Martin Fowler / Thoughtworks трактуют спецификацию как рабочий контракт между человеком, агентом и кодовой базой. Но уже видны риски: specification rot, excessive ceremony, false confidence и drift между spec/code/tests/docs.

5. **Практические coding-agent платформы уже реализуют части execution-control слоя.** GitHub Copilot cloud agent, OpenAI Codex, Claude Code и Google Jules используют похожий паттерн: задача/issue → план → изолированное окружение или worktree → изменения → тесты → PR → session logs → human review/approval.

6. **Verification/evidence исследованы лучше, чем управленческая эффективность.** SWE-bench и SWE-agent дают язык для benchmark-оценки coding agents, но не закрывают полностью вопросы enterprise proof-of-work, review effort, cognitive load, rework, evidence quality и общей эффективности системы `человек + агент`.

7. **Лучшее позиционирование нашей модели:** не “мы придумали абсолютно новую теорию”, а **практический management / operating framework поверх известных концепций**. Новизна — в системной сборке: context + memory + intent/spec + risk + orchestration + evidence + freshness + human control loop + learning + effectiveness.

---

## 2. Рабочая модель, которую сравниваем

AI-кодинг предлагается описывать не через конкретный инструмент, а через фундаментальные компоненты процесса:

1. **Контекст** — информационная среда агента и то, что реально попало в рабочее окно/сессию.
2. **Память** — память агента, проекта и сессии; authority, lifecycle, freshness.
3. **Намерение / спецификация / критерии готовности** — перевод запроса человека в проверяемое изменение.
4. **Анализ влияния и риска** — impact analysis, ownership, risk-based control budget, governance.
5. **Исполнение и оркестрация** — phases, modes, runtime boundaries, permissions, logs, deviations, audit trail.
6. **Проверка и доказательства** — tests, CI, contract checks, review, inspectable proof of work.
7. **Актуальность знаний** — freshness, drift, reconciliation после изменений.
8. **Объяснение и взаимодействие с человеком** — human-agent interaction, handoff, questions, escalations, status updates.
9. **Обучение и улучшение процесса** — retrospectives, evals, reviewed improvements, no hidden self-modification.
10. **Эффективность** — agent performance, human effectiveness, speed, correctness, review effort, cognitive load, rework, evidence quality.

---

## 3. Таблица найденных концепций

| Найденная концепция | Источник | Как связана с нашей моделью | Что покрывает | Что не покрывает | Насколько близко |
|---|---|---|---|---|---|
| **Agentic Agile-V / SCOPE-V** | [arXiv: Agentic Agile-V](https://arxiv.org/abs/2605.20456) | Самый близкий процессный аналог. SCOPE-V = Specify, Constrain, Orchestrate, Prove, Evolve, Verify. | Intent/spec, constraints, orchestration, proof/evidence, verification, risk-adaptive process, human approval. | Не даёт глубокой отдельной taxonomy для project memory, context authority, knowledge freshness и human-agent productivity metrics. | **Очень близко** |
| **Agentic Software Engineering** | [arXiv survey: From LLMs to LLM-based Agents for Software Engineering](https://arxiv.org/abs/2408.02479), [AI Teammates in SE 3.0](https://arxiv.org/abs/2507.15003) | Зонтичный термин для области: AI-агенты как участники software engineering lifecycle. | Requirements, coding, testing, maintenance, software design, autonomous decision-making. | Не является конкретным management/control framework. | **Близко как umbrella-term** |
| **AI-native SDLC / AI-native PDLC** | [EY.ai PDLC](https://www.ey.com/en_us/services/consulting/ai-native-pdlc-reinventing-software-delivery), [EY launch announcement](https://www.ey.com/en_us/newsroom/2026/03/ernst-young-llp-and-8090-launch-ey-ai-pdlc) | Executive-friendly framing: AI меняет весь product/software delivery lifecycle, а не только coding. | Full lifecycle, intent-driven delivery, AI orchestration, code/tests/docs/infrastructure. | Часто vendor/consulting narrative; меньше инженерной строгости по context authority, evidence bundle и drift reconciliation. | **Близко для презентаций IT-лидерам** |
| **Context engineering** | [Anthropic: Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents), [LangChain: Context Engineering](https://www.langchain.com/blog/context-engineering-for-agents), [Cognition: Don’t Build Multi-Agents](https://cognition.ai/blog/dont-build-multi-agents) | Почти прямой аналог нашего блока `Контекст`. | Context selection, compression, retrieval, isolation, scratchpads, tool state, agent state, long-running context. | Не покрывает acceptance criteria, risk governance, verification, process learning, effectiveness. | **Очень близко к одному блоку** |
| **Spec-driven development / SDD** | [GitHub Spec Kit](https://github.com/github/spec-kit), [GitHub blog on Spec Kit](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/), [Martin Fowler / Thoughtworks on SDD](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html), [arXiv: Spec-Driven Development: From Code to Contract](https://arxiv.org/abs/2602.00180) | Покрывает слой `намерение → спецификация → проверяемое изменение`. | Specs, plans, tasks, acceptance criteria, spec as source/contract, executable or living specs. | Не покрывает полностью memory, context retrieval, approvals, evidence, governance, productivity. | **Близко, но частично** |
| **OpenSpec** | [Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec/) | Практический lightweight spec layer для AI coding assistants. | Agreement before code, proposal/spec/design/tasks, change folders. | Не является полной моделью управления агентом, рисками, evidence, freshness и learning loop. | **Близко к spec layer** |
| **Agent harness / harness engineering** | [OpenAI: Harness engineering with Codex](https://openai.com/index/harness-engineering/) | Аналог execution-control среды вокруг coding agent. | Tooling, validation harness, repo-local instructions, browser/test harnesses, logs, review loop. | Не вся организационная модель; больше runtime/tooling layer. | **Близко к execution layer** |
| **Coding agent platforms** | [GitHub Copilot cloud agent docs](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent), [GitHub Copilot coding agent blog](https://github.blog/ai-and-ml/github-copilot/assigning-and-completing-issues-with-coding-agent-in-github-copilot/), [Google Jules](https://jules.google/), [Google Jules blog](https://blog.google/innovation-and-ai/models-and-research/google-labs/jules/) | Практические реализации task → plan → execute → test → PR → review. | Planning, branch/worktree/cloud env, tests, PRs, comments, session logs, human review. | Нет единой vendor-neutral концептуальной модели; разные продукты реализуют разные границы и гарантии. | **Практически близко** |
| **Permissions / runtime boundaries** | [Claude Code permission modes](https://code.claude.com/docs/en/permission-modes), [Claude Code permissions](https://code.claude.com/docs/en/permissions) | Покрывает `runtime boundaries`, `permissions`, `approvals`. | Plan mode, accept-edits, fine-grained permissions, managed policies, control over actions. | Не покрывает всю цепочку spec/evidence/freshness/effectiveness. | **Близко к control/permissions layer** |
| **AI coding governance / agent governance** | [GitHub Well-Architected: Governing agents](https://wellarchitected.github.com/library/governance/recommendations/governing-agents/), [Microsoft Agent Governance Toolkit](https://github.com/microsoft/agent-governance-toolkit), [OWASP Agentic Skills Top 10](https://owasp.org/www-project-agentic-skills-top-10/), [OWASP Agentic AI threats and mitigations](https://genai.owasp.org/resource/agentic-ai-threats-and-mitigations/) | Покрывает risk, approvals, audit, policy, tool restrictions, security boundaries. | Trust boundaries, security controls, audit logs, policy gates, permissioning, isolated environments. | Обычно не покрывает spec quality, project memory, knowledge reconciliation и developer effectiveness. | **Близко к governance/risk layer** |
| **Agent memory / project memory / AGENTS.md / skills** | [AGENTS.md](https://agents.md/), [OpenAI Codex AGENTS.md guide](https://developers.openai.com/codex/guides/agents-md), [Cloudflare Agent Memory](https://blog.cloudflare.com/introducing-agent-memory/), [OpenAI Agent Skills](https://developers.openai.com/codex/skills), [Claude Agent Skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) | Покрывает идею persistent/project/procedural memory и repo-local knowledge. | Project instructions, setup commands, coding conventions, skills, scripts, persistent memory, retrieval into context. | Слабо формализованы authority levels, lifecycle, freshness, reconciliation and conflict resolution. | **Средне-близко / быстро развивается** |
| **RAG for code / repository-level retrieval** | [arXiv: Retrieval-Augmented Code Generation survey](https://arxiv.org/abs/2510.04905) | Технический фундамент для блока `контекст: извлечение, навигация, сжатие`. | Repo-level retrieval, cross-file reasoning, long-range dependencies, code generation context. | Не покрывает управление процессом, approvals, evidence, governance. | **Близко к context/retrieval layer** |
| **Coding agent evaluation / SWE-bench / SWE-agent** | [SWE-bench](https://www.swebench.com/), [SWE-bench overview](https://www.swebench.com/SWE-bench/), [SWE-agent](https://github.com/swe-agent/swe-agent), [mini-SWE-agent](https://github.com/SWE-agent/mini-swe-agent) | Покрывает часть verification/evaluation. | Benchmarks, issue-fixing tasks, resolved rate, reproducible environments, agent evaluation. | Не равно enterprise proof-of-work; мало про maintainability, review burden, cognitive load, risk evidence. | **Средне-близко** |
| **Human-in-the-loop software development agents** | [HULA paper](https://arxiv.org/abs/2411.12924), [Atlassian HULA blog](https://www.atlassian.com/blog/atlassian-engineering/hula-blog-autodev-paper-human-in-the-loop-software-development-agents), [Martin Fowler: Humans and Agents in Software Engineering Loops](https://martinfowler.com/articles/exploring-gen-ai/humans-and-agents.html) | Прямо связано с human-agent interaction и control loop. | Human guidance, plan refinement, handoff, steerability, review, control points. | Не покрывает весь стек context/memory/governance/evidence/freshness. | **Близко к human-control layer** |
| **Knowledge freshness / specification rot / documentation drift** | [Spec-Driven Development: From Code to Contract](https://arxiv.org/abs/2602.00180), [Martin Fowler / Thoughtworks on SDD](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) | Прямо связано с блоком `актуальность знаний`. | Specification rot, drift between docs/specs/code/tests, risks of outdated artifacts. | Пока мало практических стандартов для systematic reconciliation после каждого изменения. | **Близко, но мало формализовано** |
| **Developer productivity / human-agent effectiveness** | [METR productivity study](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/), [arXiv: Impact of AI on Developer Productivity](https://arxiv.org/abs/2302.06590), [GitHub Copilot productivity research](https://github.blog/news-insights/research/research-quantifying-github-copilots-impact-on-developer-productivity-and-happiness/) | Поддерживает наш блок `эффективность`. | Speed, completion time, productivity perception, mental energy, empirical study design. | Нет единой метрики human-agent system effectiveness: review effort, rework, evidence quality, trust, cognitive load. | **Близко к metrics layer** |

---

## 4. Карта соответствия нашей модели существующим терминам

| Наш термин | Возможные англоязычные аналоги | Комментарий |
|---|---|---|
| **Контекст** | Context engineering, context management, working context, context window, context retrieval, context compression, context isolation, agent state | `Context engineering` — самый узнаваемый термин. Для нашей модели важно добавить `authority`: контекст не равен истине. |
| **Память** | Agent memory, project memory, persistent memory, working memory, session memory, episodic memory, semantic memory, procedural memory, repo instructions, AGENTS.md, Agent Skills | Лучше говорить **Project & Agent Memory Layer**. Важно различать memory as guidance и source of truth. |
| **Намерение / спецификация / критерии готовности** | Intent capture, specification-driven development, spec-driven development, executable specification, acceptance criteria, definition of done, conversation-to-contract gate | Сильная формула: **Intent-to-Spec Layer** или **Conversation-to-Contract Gate**. |
| **Анализ влияния и риска** | Impact analysis, blast-radius assessment, ownership, risk-based controls, approval gates, policy gates, control budget, governance model | `Risk-based control budget` звучит интересно, но для enterprise лучше: **Risk & Impact Control Layer**. |
| **Исполнение и оркестрация** | Agent orchestration, agent harness, execution harness, workflow orchestration, plan mode, permission modes, sandboxing, worktree isolation, runtime boundaries, session logs | `Harness` — среда исполнения; `orchestration` — управление фазами, ролями и переходами. Нужны оба термина. |
| **Проверка и доказательства** | Verification, validation, evidence bundle, proof of work, CI evidence, test evidence, contract checks, validation gates, review artifacts, traceability | Очень сильный термин: **Evidence Bundle**. Он превращает “агент что-то сделал” в проверяемый артефакт. |
| **Актуальность знаний** | Knowledge freshness, documentation drift, specification rot, code-doc drift, spec-code-test drift, reconciliation, living documentation, living specs | Один из самых ценных и менее формализованных блоков. Хорошее название: **Knowledge Freshness & Reconciliation Layer**. |
| **Объяснение и взаимодействие с человеком** | Human-in-the-loop, human-on-the-loop, human-agent collaboration, steerability, task alignment, handoff, escalation, explainability, status updates | Лучше говорить **Human Control Loop**, а не только HITL. HITL часто звучит слишком узко. |
| **Обучение и улучшение процесса** | Process learning, retrospectives, eval loop, agentic flywheel, harness improvement, reviewed improvements, continuous improvement loop | Важная фраза: **reviewed process learning, not hidden self-modification**. |
| **Эффективность** | Agent performance, developer productivity, human-agent system effectiveness, review effort, cognitive load, rework rate, time-to-merge, evidence quality, resolution rate | Лучше говорить **Human-Agent System Effectiveness**, потому что производительность агента сама по себе не равна пользе для команды. |

---

## 5. Какие части модели уже хорошо исследованы

### 5.1. Хорошо исследовано / быстро формализуется

| Область | Статус | Комментарий |
|---|---|---|
| **Context engineering** | Хорошо оформляется | Есть сильные инженерные тексты Anthropic, LangChain, Cognition. Термин уже узнаваем. |
| **Spec-driven development** | Быстро формализуется | GitHub Spec Kit, OpenSpec, Kiro/Tessl-дискуссия, Martin Fowler / Thoughtworks и новые academic papers дают основу. |
| **Coding-agent execution platforms** | Хорошо реализовано в продуктах | GitHub Copilot agent, Jules, Claude Code, Codex показывают похожий pattern: task → plan → isolated execution → tests → PR → review. |
| **Verification / benchmarks** | Хорошо исследовано как benchmark-проблема | SWE-bench стал узнаваемым benchmark-стандартом для repository-level issue resolution. |
| **Governance / security** | Быстро взрослеет | GitHub Well-Architected, Microsoft Agent Governance Toolkit, OWASP Agentic AI / Agentic Skills описывают controls, policies, security risks. |

### 5.2. Менее формализовано / потенциально сильная зона для нашей модели

| Область | Почему это важно |
|---|---|
| **Authority of context** | Многие обсуждают context engineering, но меньше говорят о том, какие куски контекста авторитетны, устарели или конфликтуют. |
| **Project memory lifecycle** | AGENTS.md, skills и persistent memory есть, но нет общего зрелого подхода к authority, ownership, freshness, deprecation, conflict resolution. |
| **Knowledge reconciliation after changes** | Drift между code/docs/specs/contracts — реальная проблема. Но системный процесс reconciliation после agent-generated changes пока слабо стандартизирован. |
| **Evidence bundle as acceptance artifact** | Tests и CI есть давно, но “пакет доказательств” для agent work — кто что проверил, какие assumptions, какие риски, какие логи — пока не выглядит как стандарт. |
| **Human-agent effectiveness metrics** | Существуют productivity studies, но нет простой управленческой модели, которая связывает скорость, review effort, cognitive load, rework и качество доказательств. |
| **Risk-based control budget** | Enterprise governance говорит про policies and approvals, но мало практических моделей для выбора глубины контроля в зависимости от риска задачи. |

---

## 6. Компании, исследователи и сообщества, которые описывают похожие идеи

| Кто | Что полезного у них взять | Источники |
|---|---|---|
| **Anthropic** | Context engineering, Claude Code permissions, Skills, long-running agent patterns. | [Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents), [Claude permissions](https://code.claude.com/docs/en/permissions), [Claude Skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) |
| **OpenAI** | Codex, harness engineering, AGENTS.md, Skills. | [Harness engineering](https://openai.com/index/harness-engineering/), [AGENTS.md guide](https://developers.openai.com/codex/guides/agents-md), [Codex Skills](https://developers.openai.com/codex/skills) |
| **GitHub / Microsoft** | Spec Kit, Copilot cloud agent, GitHub-native agent governance. | [Spec Kit](https://github.com/github/spec-kit), [Copilot cloud agent docs](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent), [Governing agents](https://wellarchitected.github.com/library/governance/recommendations/governing-agents/) |
| **Google** | Jules as async coding agent, API for SDLC automation. | [Jules](https://jules.google/), [Google Jules blog](https://blog.google/innovation-and-ai/models-and-research/google-labs/jules/), [Jules API](https://developers.googleblog.com/en/level-up-your-dev-game-the-jules-api-is-here/) |
| **Cognition** | Context engineering как ядро надёжности long-running agents; осторожность с multi-agent overengineering. | [Don’t Build Multi-Agents](https://cognition.ai/blog/dont-build-multi-agents) |
| **LangChain** | Практическая taxonomy context engineering: write/select/compress/isolate. | [Context Engineering](https://www.langchain.com/blog/context-engineering-for-agents), [Context engineering docs](https://docs.langchain.com/oss/python/langchain/context-engineering) |
| **Cloudflare** | Persistent Agent Memory как отдельный managed слой. | [Agent Memory](https://blog.cloudflare.com/introducing-agent-memory/), [Cloudflare memory docs](https://developers.cloudflare.com/agents/concepts/memory/) |
| **OWASP** | Security risks for agentic apps and skills. | [OWASP Agentic Skills Top 10](https://owasp.org/www-project-agentic-skills-top-10/), [Agentic AI threats and mitigations](https://genai.owasp.org/resource/agentic-ai-threats-and-mitigations/) |
| **Princeton / SWE-bench community** | Benchmarks and evaluation of software engineering agents. | [SWE-bench](https://www.swebench.com/), [SWE-agent](https://github.com/swe-agent/swe-agent) |
| **Atlassian** | Human-in-the-loop software development agents. | [HULA paper](https://arxiv.org/abs/2411.12924), [HULA blog](https://www.atlassian.com/blog/atlassian-engineering/hula-blog-autodev-paper-human-in-the-loop-software-development-agents) |
| **Thoughtworks / Martin Fowler** | Conceptual analysis of spec-driven development and human/agent loops. | [Understanding SDD](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html), [Humans and Agents in Software Engineering Loops](https://martinfowler.com/articles/exploring-gen-ai/humans-and-agents.html) |
| **METR** | Empirical productivity study showing that AI productivity effects are heterogeneous and not automatically positive. | [METR study](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/) |
| **EY / 8090** | Enterprise positioning of AI-native PDLC. | [EY.ai PDLC](https://www.ey.com/en_us/services/consulting/ai-native-pdlc-reinventing-software-delivery) |

---

## 7. 15 наиболее полезных источников

1. **Agentic Agile-V: From Vibe Coding to Verified Engineering in Software and Hardware Development**
   https://arxiv.org/abs/2605.20456
   Самый близкий найденный процессный аналог. Важны SCOPE-V loop, evidence bundle, conversation-to-contract gate.

2. **From LLMs to LLM-based Agents for Software Engineering: A Survey**
   https://arxiv.org/abs/2408.02479
   Хороший академический обзор LLM-based agents в software engineering.

3. **The Rise of AI Teammates in Software Engineering (SE 3.0)**
   https://arxiv.org/abs/2507.15003
   Полезно для framing “AI agents as teammates” и исследования реальных PR от агентов.

4. **Anthropic — Effective context engineering for AI agents**
   https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
   Базовый источник по context engineering.

5. **LangChain — Context Engineering**
   https://www.langchain.com/blog/context-engineering-for-agents
   Практическая taxonomy: write, select, compress, isolate.

6. **Cognition — Don’t Build Multi-Agents**
   https://cognition.ai/blog/dont-build-multi-agents
   Сильный текст про reliability long-running agents и context engineering.

7. **GitHub Spec Kit**
   https://github.com/github/spec-kit
   Важный open-source проект для spec-driven development.

8. **OpenSpec**
   https://github.com/Fission-AI/OpenSpec/
   Lightweight spec layer для согласования требований до кода.

9. **Martin Fowler / Thoughtworks — Understanding Spec-Driven Development**
   https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html
   Хороший обзор того, что SDD сейчас значит и где есть неопределённость.

10. **Spec-Driven Development: From Code to Contract in the Age of AI Coding Assistants**
    https://arxiv.org/abs/2602.00180
    Полезно для spec-as-source, spec-first, specification rot и workflow patterns.

11. **OpenAI — Harness engineering with Codex**
    https://openai.com/index/harness-engineering/
    Источник для слоя execution harness и validation loop.

12. **GitHub Copilot cloud agent / coding agent**
    https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent
    https://github.blog/ai-and-ml/github-copilot/assigning-and-completing-issues-with-coding-agent-in-github-copilot/
    Практический пример агентного execution flow: plan, branch, tests, PR, review.

13. **SWE-bench and SWE-agent**
    https://www.swebench.com/
    https://github.com/swe-agent/swe-agent
    Ключевые источники по evaluation и issue-resolution agents.

14. **GitHub Well-Architected — Governing agents in GitHub Enterprise**
    https://wellarchitected.github.com/library/governance/recommendations/governing-agents/
    Хороший enterprise governance источник.

15. **METR — Measuring the Impact of Early-2025 AI on Experienced Open-Source Developer Productivity**
    https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/
    Важный counterweight против наивного “AI always improves productivity”.

---

## 8. Вывод: как позиционировать нашу модель

Не стоит позиционировать модель как “полностью новую науку”. Почти каждый отдельный блок уже существует в виде термина, статьи, инструмента или enterprise-подхода.

Сильнее позиционирование:

> **A practical management framework for controlled agentic software engineering.**

Или:

> **Agentic Software Engineering Operating Model** — a control framework for context, memory, specification, risk, orchestration, evidence, knowledge freshness, human oversight, process learning, and effectiveness.

На русском:

> **Практический фреймворк управления AI-assisted разработкой, который переводит AI-кодинг из prompt-driven execution в управляемый, проверяемый engineering-процесс.**

### В чём реальная ценность модели

Отдельные блоки уже известны:

- context engineering говорит, **как агент получает информацию**;
- SDD говорит, **что именно нужно построить**;
- governance говорит, **что агенту можно делать**;
- harness/orchestration говорит, **как агент исполняет работу**;
- verification/evidence говорит, **как доказать, что работа сделана**;
- memory/freshness говорит, **как не потерять знания и не жить в устаревшей картине проекта**;
- human-agent effectiveness говорит, **как измерять не только агента, а всю систему человек + агент**.

Новизна — в **сборке этих слоёв в единую operating model**.

---

## 9. Рекомендации по терминологии для презентации IT-лидерам

### Лучшее основное название

```text
Agentic Software Engineering Operating Model
```

### Сильный подзаголовок

```text
A control framework for context, specification, risk, orchestration, evidence, knowledge freshness, and human oversight.
```

### Если нужен governance/control акцент

```text
AI Coding Control Framework
```

### Если нужно звучать enterprise-friendly

```text
Human-Controlled Agentic SDLC Framework
```

### Если нужно покрыть весь delivery/product lifecycle

```text
AI-Native SDLC / PDLC Control Model
```

### Что я бы избегал

- **Context Engineering Framework** — слишком узко для всей модели.
- **AI SDLC** — слишком широко и размыто.
- **AI Coding Framework** — звучит как очередной tool/framework для кода, а не management model.
- **Autonomous Software Engineering Framework** — может пугать enterprise-аудиторию, потому что звучит как removal of control.

---

## 10. Рекомендуемая структура слоёв для модели

| Слой | Название для презентации | Смысл |
|---|---|---|
| 1 | **Context Engineering Layer** | Управляет тем, что агент видит, как извлекает, сжимает и проверяет релевантность информации. |
| 2 | **Project & Agent Memory Layer** | Хранит проектные знания, инструкции, навыки, прошлые решения, но с authority/freshness metadata. |
| 3 | **Intent-to-Spec Layer** | Переводит человеческий запрос в проверяемую спецификацию, scope, non-goals и acceptance criteria. |
| 4 | **Risk & Impact Control Layer** | Оценивает blast radius, ownership, approvals, risk budget, security boundaries. |
| 5 | **Agent Orchestration / Harness Layer** | Управляет фазами исполнения, ролями, permissions, sandbox/worktree, run logs, deviations. |
| 6 | **Verification & Evidence Layer** | Собирает tests, CI, review notes, contract checks, screenshots, logs, proof of work. |
| 7 | **Knowledge Freshness & Reconciliation Layer** | Следит за drift между code/docs/specs/tests/contracts и обновляет knowledge base. |
| 8 | **Human Control Loop** | Обеспечивает status updates, questions, escalations, handoff, review and approval. |
| 9 | **Process Learning Loop** | Превращает review findings and retrospectives в улучшения процесса, skills, docs, checks. |
| 10 | **Human-Agent System Effectiveness Layer** | Меряет не только speed, но и correctness, review effort, cognitive load, rework, evidence quality. |

---

## 11. Возможная финальная формулировка для первого слайда

```text
Agentic Software Engineering Operating Model

A practical control framework for turning AI coding from prompt-driven execution
into a governed, evidence-based engineering process.

Core loop:
Intent → Context → Spec → Risk → Execution → Evidence → Review → Reconciliation → Learning
```

В более жёсткой версии:

```text
From vibe coding to controlled agentic engineering:
a human-controlled loop for specification, execution, verification, and knowledge freshness.
```

---

## 12. Что можно развить дальше в отдельный фреймворк

### 12.1. Минимальные артефакты на каждый тип задачи

Например:

| Тип изменения | Минимальные входные артефакты | Минимальные evidence artifacts |
|---|---|---|
| Small bug fix | issue, reproduction, expected behavior, affected area | diff, test output, explanation, risk note |
| Feature | intent, scope, non-goals, acceptance criteria, affected APIs, UX/contract constraints | tests, docs update, contract check, PR summary, migration notes |
| Refactoring | target module, invariants, non-goals, rollback plan | before/after tests, static checks, behavior invariants, risk assessment |
| Security-sensitive change | threat model, owner approval, secret/data boundaries, permissions | security review notes, audit trail, tests, dependency checks |
| Documentation/spec update | source of truth, affected specs/docs, freshness reason | updated docs, references to code/spec, drift note |

### 12.2. Risk-based control budget

Идея: не все задачи требуют одинакового контроля.

| Risk level | Agent autonomy | Required human control | Required evidence |
|---|---|---|---|
| Low | High | Review after PR | tests/lint + summary |
| Medium | Moderate | Approve plan before execution | tests + affected files + risk note |
| High | Low | Approve spec and plan before execution | tests + CI + design note + owner review |
| Critical | Very low | Human-led, agent-assisted only | formal review + audit trail + rollback plan |

### 12.3. Authority model для контекста и памяти

Контекст должен иметь не только content, но и authority:

| Authority level | Пример | Как использовать |
|---|---|---|
| Source of truth | code, tests, schema, contracts, ADR | Максимальный приоритет |
| Approved project memory | AGENTS.md, architecture docs, accepted specs | Использовать как устойчивые правила |
| Recent session memory | текущий план, обсуждение, временные assumptions | Проверять перед применением |
| Retrieved context | найденные файлы, snippets, search results | Валидировать через code/tests |
| Agent inference | вывод агента | Никогда не считать истиной без проверки |

---

## 13. Набор поисковых запросов для повторного research

```text
"AI coding control model" software engineering agents
"agentic software engineering" context memory verification
"AI SDLC" coding agents governance verification
"AI PDLC" software development agents
"context engineering" AI coding agents
"agent memory" software engineering agents
"specification driven development" AI coding
"spec-driven development" coding agents
"human in the loop" coding agents software engineering
"AI pair programming" developer productivity cognitive load
"coding agent evaluation" verification evidence
"SWE-bench" agent evaluation software engineering
"software engineering agents" orchestration verification
"AI coding governance" enterprise software development
"retrieval augmented generation" code agents context
"agentic coding workflow" specification verification
"AI coding assistant productivity" human effectiveness
"knowledge freshness" software documentation code drift
"documentation drift" AI coding agents
"human agent collaboration" software development
"agent harness engineering" coding agents validation logs
"conversation to contract" AI coding agents
"evidence bundle" AI software engineering agents
"risk based governance" AI coding agents
"AGENTS.md" coding agents repository instructions
"Agent Skills" coding agents repository workflows
"specification rot" AI coding assistants
"human-agent system effectiveness" software engineering
```

---

## 14. Короткий итог

Самая сильная версия идеи:

> Мы не строим ещё один coding-agent framework.
> Мы описываем **operating model** для управляемой разработки с AI-агентами.

Эта operating model отвечает на вопросы, которые отдельные инструменты обычно закрывают фрагментарно:

- откуда агент берёт знания;
- каким знаниям можно доверять;
- как человеческий intent превращается в проверяемую спецификацию;
- как оценивать риск и blast radius;
- как ограничивать и наблюдать исполнение;
- как принимать работу не по “агент сказал готово”, а по evidence bundle;
- как обновлять знания после изменений;
- где человек остаётся в loop;
- как процесс учится и улучшается;
- как измерять эффективность всей системы человек + агент.

Рекомендуемая финальная рамка:

```text
Agentic Software Engineering Operating Model
with an AI Coding Control Loop
```
