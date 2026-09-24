# Restaurant AI Agent — 1v1 Agent Backend Course

A mentor-led, project-based SDE track for building and defending a
production-oriented restaurant customer-service agent.

**Format:** 19 one-hour 1v1 sessions  
**Independent work:** 6–10 hours between sessions  
**Primary role:** Student Agent Backend Engineer  
**Mentor role:** Tech Lead, reviewer, interviewer, and simulated stakeholder  
**Cost model:** Required work has a zero-payment local path  
**Repository status:** Curriculum and project-design package; application code
is not included

## 中文概览

这是一个面向 SDE / Agent Backend 方向的 1v1 工业项目课程。学生将在 Mentor
带领下，以主要工程师身份完成一个餐厅 AI 客服后端，覆盖营业信息问答、订座、
外卖下单、RAG、Tool Calling、人工转接、异步消息、可观测性、CI/CD 和
Kubernetes。课程不是连续讲授，而是以 Project Lab、设计评审、PR Review、
故障排查和面试答辩为主。

- 共 19 课时，每课时 60 分钟
- 每课之间安排 6–10 小时独立工程工作
- 学生承担至少 70% 的表达、推理和实际操作
- 所有必修工具均有免费、本地、无需信用卡的使用路径
- GitHub 是 Issue、Board、PR、CI 和项目证据的统一工作台
- AWS 仅做架构映射；不创建影响结课的付费资源
- 最终成果应如实描述为 mentored industry project / practicum，除非确有可验证的
  雇佣或实习关系

## What the student builds

The capstone is a restaurant customer-service backend for the fictional
**Juniper & Stone** restaurant. It supports:

- Trusted answers for hours, location, menu, and policies
- Reservation search, holds, creation, modification, cancellation, and staff
  operations
- Takeout menu browsing, deterministic pricing, order submission, and status
- Grounded policy and FAQ retrieval with citations and abstention
- LangGraph workflows with typed tools and explicit confirmation before writes
- Human handoff when the agent cannot safely complete a request
- Reliable asynchronous work using RabbitMQ and a transactional outbox
- Authentication, authorization, tenant isolation, audit records, and PII
  controls
- Logs, metrics, traces, SLOs, incident drills, load tests, and rollback
- Reproducible local containers, CI, and local Kubernetes staging

The LLM interprets language and coordinates work. Deterministic backend
services remain responsible for prices, permissions, availability, and every
durable state change.

## Learning outcomes

Graduates should be able to:

1. Design versioned APIs and transactional backend services with FastAPI and
   PostgreSQL.
2. Prevent double booking and duplicate effects using constraints, locking,
   idempotency, and explicit state transitions.
3. Build safe LangGraph workflows with provider isolation, typed tool calls,
   confirmation, limits, fallbacks, and traceability.
4. Implement and evaluate RAG with local embeddings, pgvector, citations,
   metadata filters, abstention, and corpus versioning.
5. Apply Valkey caching, RabbitMQ, outbox delivery, retries, dead letters, and
   idempotent consumers.
6. Containerize, test, observe, deploy, operate, and roll back the system.
7. Defend architecture and engineering tradeoffs in SDE, distributed-system,
   LLM-application, and behavioral interviews.
8. Produce evidence-backed, truthful resume bullets without invented scale or
   employment claims.

## Delivery model

Each live session follows a 60-minute working format:

| Time | Activity |
|---:|---|
| 5 min | Standup and blocker check |
| 5 min | Evidence review |
| 15 min | Targeted teaching or design review |
| 20 min | Lab, code review, debugging, or incident work |
| 10 min | Interview drill |
| 5 min | Next task and acceptance criteria |

The course uses a flipped-classroom, project-lab, and Tech Lead mentoring
model. A session is complete only when its required artifact or remediation
evidence is accepted.

## Nineteen-session roadmap

| Sprint | Sessions | Engineering outcome | Gate |
|---|---:|---|---|
| 0 | 1 | Kickoff, role baseline, project plan, and GitHub workspace | Plan accepted |
| 1 | 2–3 | API, PostgreSQL, migrations, containers, and CI | Gate A |
| 2 | 4–5 | Reservation correctness, concurrency, idempotency, and cache | Gate B |
| 3 | 6–8 | Model gateway, LangGraph, tools, confirmation, and handoff | Workflow review |
| 4 | 9–10 | RAG ingestion, retrieval evaluation, and mid-track mock | Gate C |
| 5 | 11–13 | Ordering, RabbitMQ, outbox, retries, and dead letters | Gate D |
| 6 | 14–16 | Security, observability, incidents, and performance | Gate E |
| 7 | 17–19 | CI/CD, Kubernetes, system design, and capstone defense | Gate F |

See the
[complete session-by-session plan](1v1-agent-backend-track.md#10-nineteen-session-delivery-plan)
for required artifacts and acceptance gates.

## Primary technical stack

| Area | Required route |
|---|---|
| Backend | Python, FastAPI, Pydantic, SQLAlchemy, Alembic |
| Agent | LangGraph with selected LangChain integrations |
| Models | Deterministic fake for tests; local Ollama with Qwen3 1.7B for interactive use |
| RAG | `all-MiniLM-L6-v2`, PostgreSQL, pgvector |
| Data | PostgreSQL as source of truth; Valkey for Redis-compatible cache patterns |
| Messaging | RabbitMQ, transactional outbox, idempotent workers, DLQ |
| Testing | pytest, Testcontainers, Locust or k6 |
| Local platform | Docker Compose |
| Staging | kind or k3d with Helm |
| Delivery | GitHub Actions included minutes, self-hosted runner, or local fallback |
| Observability | OpenTelemetry, Prometheus, Grafana OSS |
| Security | Trivy, `pip-audit`, Bandit, SOPS with age |
| Cloud learning | AWS managed-service architecture mapping without required provisioning |

FastAPI is the primary implementation route. A Java-focused learner may select
Spring Boot before implementation begins, but implementing both stacks is not
required.

## Repository guide

### Core curriculum

Read these documents in order:

1. [1v1 Agent Backend Track](1v1-agent-backend-track.md) — goals, scope,
   ownership, session format, and the full 19-session plan.
2. [Technical Learning Architecture](technical-learning-architecture.md) —
   backend, agent, RAG, data, messaging, security, testing, operations, and
   deployment blueprint.
3. [Zero-Cost Tooling Policy](free-tooling-policy.md) — approved tools, hosted
   service restrictions, fallbacks, and audit checkpoints.
4. [GitHub Operating Model](github-operating-model.md) — private repository,
   trunk-based delivery, Issues, Projects, PR review, merge gates, and
   evidence.
5. [Mentor and Scrum Playbook](mentor-scrum-playbook.md) — roles, sprint
   cadence, backlog contracts, reviews, incidents, demos, and retrospectives.
6. [Assessment, Interview, and Resume Evidence](assessment-interview-resume.md)
   — hard gates, scoring, mocks, evidence ledger, interview topics, and
   truthful resume positioning.

### Project baseline

The [`project-baseline/`](project-baseline/) directory makes this repository
self-contained:

- [High-Level Design V1](project-baseline/high-level-design-v1.md)
- [Design Workshop 2](project-baseline/design-workshop-2.md)
- [Mock Restaurant Specification](project-baseline/mock-restaurant-spec.md)
- [Reservation Domain Design](project-baseline/reservation-domain-design.md)
- [Reservation Contracts](project-baseline/reservation-contracts.md)
- [Reservation Acceptance Cases](project-baseline/reservation-acceptance-cases.md)

These documents define the fictional business, domain invariants, API and tool
contracts, concurrency rules, and 43 deterministic reservation scenarios used
as project requirements.

## GitHub operating model

The student implementation should live in a separate private repository:

- `main` is the single releasable trunk.
- One accepted Task Issue maps to one short-lived branch and one pull request.
- The GitHub Project uses `Backlog / Sprint / In Review / Done`.
- The mentor reviews asynchronously between sessions.
- Stable CI checks cover lint, types, unit tests, integration tests,
  migrations, and container builds.
- Changes are squash-merged and the source branch is deleted.
- Architecture decisions, runbooks, incidents, evaluations, performance
  reports, and the evidence ledger live in the implementation repository.

GitHub Free private repositories do not include every native protected-branch
feature. The course therefore uses a GitHub CLI/API merge guard by default.
Native protection is enabled only when an existing no-cost entitlement supports
it; buying a plan is not a course requirement.

## Zero-cost guarantee

Every required graduation outcome has a path that:

- Requires no credit card or billing account
- Does not depend on expiring trial credit
- Can run locally or through an explicitly free fallback
- Uses synthetic restaurant and customer data
- Does not send sensitive data to unpaid hosted model providers

The minimum lightweight target is an 8 GB RAM computer with roughly 25 GB of
free disk; 16 GB RAM is recommended. Hosted LLMs and AWS staging are optional
comparisons and cannot affect grading.

## How to run the course

### Mentor

1. Review the track, architecture, cost policy, operating model, and rubric.
2. Confirm learner prerequisites, target SDE roles, schedule, and hardware.
3. Create the student's private implementation repository and GitHub Project.
4. Seed only the current sprint; keep future requirements in the backlog.
5. Review PRs asynchronously and use live time for reasoning, debugging,
   acceptance, and interview defense.
6. Record Gate A–F decisions and link all evidence.

### Student

1. Read the product baseline and identify unresolved assumptions.
2. Complete the assigned artifact before each session.
3. Open one focused PR per accepted task and respond to review feedback.
4. Maintain ADRs, API and event contracts, runbooks, incident reports,
   evaluation reports, and the evidence ledger.
5. Reproduce every performance or quality claim and preserve its method.
6. Explain and defend every submitted change, including LLM-assisted code.

## Prerequisites

The track assumes working familiarity with:

- Python fundamentals and basic automated testing
- HTTP, REST APIs, and JSON
- SQL and relational data concepts
- Git branches, commits, and pull requests
- Command-line development on macOS, Linux, or Windows with WSL

The mentor assigns short just-in-time remediation when a foundational gap
blocks delivery; this repository is not a beginner programming curriculum.

## Completion criteria

Completion requires all hard gates, a passing capstone, and reproducible
evidence. At minimum:

- No known critical data-integrity, authorization, or unsafe-tool issue remains
  open.
- The student independently explains the architecture and alternatives.
- The staging system can be deployed, observed, exercised, and rolled back.
- Performance and quality claims have preserved measurements.
- Resume wording matches the actual project relationship and measured scope.

Attendance alone does not complete the track.

## Scope and access

This is a private curriculum repository. It contains course design and
synthetic project requirements, not student submissions or real customer data.
No open-source license is included.
