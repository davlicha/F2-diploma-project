# Thesis Proposal

| Атрибут | Значення |
| --- | --- |
| Автор | Лічинський Давид Вікторович |
| Спеціальність | F2 Інженерія програмного забезпечення |
| Рівень | Бакалавр |
| Статус | Подано на рецензію науковому керівнику |

## 1. Попередня робоча назва проєкту

- **UA:** Проєктування та розробка мультиагентної системи декомпозиції та планування задач програмних проєктів з інтеграцією через Model Context Protocol
- **EN:** Design and Development of a Multi-Agent System for Software Project Task Decomposition and Planning via Model Context Protocol

## 2. Актуальність та проблематика

Декомпозиція вимог великих програмних проєктів (епіків, технічних завдань) на атомарні задачі
виконується переважно вручну. Процес потребує значних витрат часу технічного лідера,
призводить до неузгоджених оцінок трудомісткості та пропуску неявних залежностей між задачами,
що виявляються лише на етапі реалізації.

Застосування великих мовних моделей (LLM) у форматі одного запиту не вирішує цю проблему з
таких інженерних причин:

- відсутність верифікації результату: модель генерує план без перевірки на повноту, циклічні
  залежності та узгодженість оцінок;
- відсутність збереження стану: тривалий сеанс планування не відновлюється після збою
  мережі, перевищення ліміту запитів (HTTP 429) або тайм-ауту LLM API;
- обмеження контекстного вікна: оцінювання не враховує історичні дані про завершені задачі
  проєкту;
- нестандартизована інтеграція з трекерами задач: синхронізація виконується
  спеціалізованими скриптами над REST API без гарантій ідемпотентності, що призводить до
  дублювання задач під час повторних запусків.

Складність проблеми полягає в поєднанні недетермінованої поведінки LLM з вимогами
до відтворюваності, контрольованої вартості (кількість токенів) та транзакційної цілісності
даних під час запису в зовнішню систему (GitHub Issues).

## 3. Сформована ідея продукту

Розподілена подієво-орієнтована система з мультиагентним оркестратором на основі графа станів
(LangGraph), що виконує декомпозицію, оцінювання та верифікацію плану задач і синхронізує
затверджений план з GitHub Issues через Model Context Protocol (MCP).

Система складається з контейнеризованих сервісів:

| Контейнер | Призначення |
| --- | --- |
| Web SPA | Завантаження вхідних артефактів, перегляд графа задач, підтвердження плану (human-in-the-loop) |
| API Service | REST API, автентифікація, керування сеансами планування, публікація подій у брокер |
| Agent Orchestrator | Асинхронний worker, що виконує граф агентів Planner → Estimator → Dependency Analyzer → Reviewer |
| GitHub MCP Server | Ізольований шлюз доступу до GitHub API з набором MCP-інструментів (issues, labels, milestones) |
| PostgreSQL + pgvector | Дані сеансів, задач, checkpoint-и графа агентів, векторні представлення історичних задач |
| RabbitMQ | Асинхронна черга сеансів планування та подій синхронізації |
| Redis | Кешування, Rate Limiting, трансляція прогресу сеансу в SPA (Server-Sent Events) |

## 4. Детальний опис функціоналу (Core Features)

1. **Прийом вхідного артефакту.** Завантаження епіку або технічного завдання у форматі Markdown
   або PDF (до 10 МБ), валідація, вилучення тексту та постановка сеансу планування в асинхронну
   чергу.
2. **Декомпозиція вимог (агент Planner).** Формування ієрархії Epic → Story → Task зі
   структурованим виводом, валідованим за JSON Schema (Pydantic); повторний запит у разі
   невалідної відповіді LLM.
3. **Оцінювання трудомісткості (агент Estimator).** Визначення story points для кожної задачі з
   використанням семантичного пошуку схожих завершених задач проєкту (Retrieval-Augmented
   Generation на базі pgvector).
4. **Аналіз залежностей (агент Dependency Analyzer).** Побудова орієнтованого графа залежностей,
   виявлення циклів, топологічне сортування та обчислення критичного шляху.
5. **Циклічна верифікація плану (агент Reviewer).** Перевірка плану на повноту покриття вимог та
   узгодженість оцінок з поверненням на доопрацювання; кількість ітерацій обмежено (N ≤ 3).
   Завершення сеансу потребує підтвердження користувачем (LangGraph interrupt).
6. **Синхронізація з GitHub Issues через MCP.** Створення issues, labels, milestones та
   sub-issues для затвердженого плану. Ідемпотентність забезпечується ключем синхронізації та
   патерном Transactional Outbox; повторний запуск не створює дублікатів.
7. **Спостережуваність та контроль вартості.** Трасування кожного виклику LLM та MCP-інструменту
   (latency, кількість токенів, вартість сеансу), бюджетний ліміт токенів на сеанс з
   примусовим завершенням у разі перевищення.

## 5. Попередній інструментарій та технологічний стек

| Категорія | Технології |
| --- | --- |
| Мови програмування | Python 3.12 (backend, агенти, MCP-сервер); TypeScript 5 (SPA) |
| Архітектурні патерни | Event-Driven Architecture; Clean Architecture (Ports & Adapters); Orchestrator–Worker multi-agent graph; Human-in-the-Loop; Transactional Outbox; Idempotent Consumer; Circuit Breaker, Retry with Exponential Backoff; Repository; Dependency Injection |
| Фреймворки та бібліотеки | FastAPI, Pydantic v2, SQLAlchemy 2.0, Alembic, FastStream; LangGraph, langchain-anthropic; MCP Python SDK; React 19, Vite, TanStack Query |
| LLM | Claude API (Anthropic): Claude Sonnet — агенти планування та верифікації; Claude Haiku — класифікація та допоміжні операції |
| Бази даних та сховища | PostgreSQL 16 з розширенням pgvector; Redis 7 |
| Брокер повідомлень | RabbitMQ 4 (AMQP 0-9-1) |
| Безпека | GitHub OAuth 2.0 + JWT (RS256); RBAC; шифрування токенів доступу GitHub (AES-256-GCM); захист від prompt injection шляхом ізоляції вмісту вхідних документів від системних інструкцій |
| DevOps-інструменти | Docker, Docker Compose; GitHub Actions (CI/CD); Ruff, mypy, pytest; SonarQube (SonarCloud); Trivy; k6 |
| Observability | OpenTelemetry, Prometheus, Grafana; Langfuse (трасування LLM-викликів) |

---

## English Version

### 1. Working Title

Design and Development of a Multi-Agent System for Software Project Task Decomposition and Planning via Model Context Protocol

### 2. Relevance and Problem Statement

Decomposition of large software project requirements (epics, technical specifications) into
atomic tasks is performed predominantly by hand. The process consumes significant tech-lead time,
produces inconsistent effort estimates and omits implicit task dependencies that surface only
during implementation.

Single-prompt LLM usage does not solve the problem for the following engineering reasons:

- no result verification: the model produces a plan without checks for completeness, dependency
  cycles or estimate consistency;
- no state persistence: a long-running planning session cannot recover from network failures,
  rate limiting (HTTP 429) or LLM API timeouts;
- context window limits: estimation ignores historical data on completed project tasks;
- non-standardized tracker integration: synchronization relies on ad-hoc REST scripts without
  idempotency guarantees, which duplicates tasks on repeated runs.

The complexity lies in combining non-deterministic LLM behavior with requirements for
reproducibility, controlled token cost and transactional integrity when writing to an external
system (GitHub Issues).

### 3. Product Concept

A distributed event-driven system with a state-graph multi-agent orchestrator (LangGraph) that
decomposes, estimates and verifies a task plan and synchronizes the approved plan with GitHub
Issues via the Model Context Protocol (MCP). Containers: Web SPA, API Service, Agent Orchestrator,
GitHub MCP Server, PostgreSQL + pgvector, RabbitMQ, Redis.

### 4. Core Features

1. Input artifact ingestion (Markdown/PDF up to 10 MB) and asynchronous session queuing.
2. Requirement decomposition (Planner agent) into an Epic → Story → Task hierarchy with
   JSON Schema validated structured output.
3. Effort estimation (Estimator agent) using semantic search over completed tasks (RAG, pgvector).
4. Dependency analysis (Dependency Analyzer agent): directed graph, cycle detection, topological
   sort, critical path.
5. Iterative plan verification (Reviewer agent) bounded to N ≤ 3 iterations with mandatory
   human-in-the-loop approval (LangGraph interrupt).
6. Idempotent synchronization with GitHub Issues via MCP (issues, labels, milestones, sub-issues)
   using a sync key and the Transactional Outbox pattern.
7. Observability and cost control: tracing of every LLM and MCP tool call (latency, tokens,
   session cost) with a per-session token budget.

### 5. Technology Stack

| Category | Technologies |
| --- | --- |
| Languages | Python 3.12, TypeScript 5 |
| Architectural patterns | Event-Driven Architecture, Clean Architecture, Orchestrator–Worker multi-agent graph, Human-in-the-Loop, Transactional Outbox, Idempotent Consumer, Circuit Breaker, Retry with Exponential Backoff, Repository, Dependency Injection |
| Frameworks | FastAPI, Pydantic v2, SQLAlchemy 2.0, Alembic, FastStream, LangGraph, langchain-anthropic, MCP Python SDK, React 19, Vite, TanStack Query |
| LLM | Claude API (Anthropic): Claude Sonnet, Claude Haiku |
| Databases | PostgreSQL 16 + pgvector, Redis 7 |
| Message broker | RabbitMQ 4 (AMQP 0-9-1) |
| Security | GitHub OAuth 2.0 + JWT (RS256), RBAC, AES-256-GCM token encryption, prompt injection isolation |
| DevOps | Docker, Docker Compose, GitHub Actions, Ruff, mypy, pytest, SonarQube, Trivy, k6 |
| Observability | OpenTelemetry, Prometheus, Grafana, Langfuse |
