# 1v1 Agent Backend Track

Status: Proposed learning overlay  
Track type: Mentor-led SDE / Agent Backend project  
Delivery format: 19 mentor-led 1v1 sessions, 60 minutes per session  
Independent project work: 6-10 student hours between sessions  
Primary project: Juniper & Stone restaurant customer-service agent

## 1. Purpose

This track turns the existing restaurant-agent design into a production-style
backend engineering project. The student works as the primary Agent Backend
Engineer while a mentor acts as Tech Lead and runs the project using an
industry-style delivery process.

This document defines only the 1v1 track. It does not add a class-course
curriculum or reteach programming fundamentals. Foundational gaps are handled
by short, just-in-time mentor assignments when they block project delivery.

The target role is an entry-level or early-career SDE who can build, explain,
test, deploy, and operate an LLM-enabled backend system.

## 2. Graduate outcome

By the end of the track, the student should be able to:

1. Design and implement versioned REST APIs with FastAPI, PostgreSQL
   transactions, database migrations, validation, authentication, and stable
   error contracts.
2. Build a LangGraph agent workflow that uses typed tools, deterministic
   business services, explicit confirmation, human escalation, and provider
   isolation.
3. Build a grounded RAG pipeline for policy and FAQ documents while keeping
   prices, hours, availability, and order state in structured systems of
   record.
4. Apply Redis-compatible caching with Valkey, rate limiting, expiring state,
   and failure degradation without treating the cache as the source of truth
   for reservations or orders.
5. Use RabbitMQ, an outbox pattern, idempotent consumers, retries, and a dead
   letter queue for reliable asynchronous work.
6. Containerize and deploy the system, establish CI/CD, instrument logs,
   metrics, and traces, and diagnose correctness and performance failures.
7. Defend the design in backend, distributed-system, LLM-system-design, and
   project deep-dive interviews.
8. Describe the experience truthfully using implementation artifacts and
   measured results rather than invented scale or employment claims.

## 3. Scope and non-goals

### Core product scope

- Restaurant information and operating-hours questions
- Reservation search, hold, create, modify, cancel, and staff management
- Takeout menu browsing, cart pricing, order submission, and order status
- FAQ and policy retrieval with citations
- Human handoff when the agent cannot safely complete a request
- Thin customer and staff interfaces backed by the same APIs
- Web voice as a late extension after the text workflow is reliable

### Explicit non-goals

- Training foundation models, deep model research, or fine-tuning
- Building databases, queues, vector stores, or agent frameworks from scratch
- Implementing both a Python and Java backend in the same track
- Premature microservice decomposition
- Processing real customer PII through free model tiers
- Requiring a paid service, credit card, trial credit, or billable cloud
  resource for any graduation outcome
- Claiming this project as paid employment or an internship unless that legal
  relationship actually exists
- Publishing resume metrics before they have been measured and preserved

## 4. Primary technical route

The track uses one coherent implementation path instead of a shallow survey of
multiple stacks.

| Area | Primary choice | Learning purpose |
|---|---|---|
| Engineering workflow | Private GitHub repository, Projects, pull requests, and Actions | Trunk-based delivery, lightweight Scrum, asynchronous review, and portfolio evidence |
| Language and API | Python, FastAPI, Pydantic | Typed REST APIs, async I/O, OpenAPI |
| Persistence | PostgreSQL, SQLAlchemy, Alembic | Transactions, constraints, indexes, migrations |
| Vector retrieval | PostgreSQL with pgvector | Keep relational and vector learning in one operational system |
| Agent framework | LangGraph with selected LangChain integrations | Explicit state, workflow edges, tool calls, checkpoints |
| LLM access | Deterministic fake plus local Ollama/Qwen3 1.7B behind a provider-neutral gateway | Free offline tests and inference, model replacement, timeout, retry, and policy controls |
| Embeddings | Local `all-MiniLM-L6-v2` | RAG ingestion and retrieval without a paid embedding API |
| Cache and ephemeral state | Valkey using Redis-compatible clients and patterns | Cache-aside, TTL, rate limiting, session acceleration |
| Messaging | RabbitMQ | Work queues, routing, retries, dead letters, operational simplicity |
| Testing | pytest, Testcontainers, Locust or k6 | Unit, integration, concurrency, contract, and load testing |
| Local platform | Docker Compose | Reproducible multi-service development |
| Artifact storage | Versioned local filesystem adapter | Document and report lifecycle without paid object storage |
| CI/CD | GitHub Actions included private-repository minutes, a self-hosted runner, and local fallback | Zero-payment quality gates, image builds, migration checks, and ephemeral staged deploys |
| Required staging | kind or k3d plus Helm | Free local Kubernetes deployment, probes, rollout, rollback, and autoscaling concepts |
| Cloud learning | AWS architecture mapping without resource provisioning | ECS, RDS, ElastiCache, Amazon MQ, S3, ECR, and CloudWatch tradeoffs without billing |
| Observability | OpenTelemetry, Prometheus, and Grafana OSS | Correlated logs, metrics, traces, SLOs, and incident diagnosis |

RabbitMQ is the implementation queue because the project primarily needs
reliable task and workflow delivery. Kafka is covered in a design comparison:
the student must explain when retained, replayable, high-throughput event
streams justify Kafka, but does not operate both systems.

FastAPI is the primary route because it keeps the LangGraph integration and
backend service in one typed Python codebase. A Java-focused student may replace
this route with Spring Boot before implementation begins; implementing both
routes is not a graduation requirement.

Every required tool follows the
[Zero-Cost Tooling Policy](free-tooling-policy.md): no payment method, trial
credit, or billable cloud resource is required. Gemini Developer API and
GroqCloud may be used only as optional synthetic-data comparisons. They never
replace the deterministic fake and local model paths used for assessment.

## 5. Architecture learning strategy

The system starts as a modular monolith plus asynchronous workers:

- `api`: customer, staff, and agent-facing REST endpoints
- `domain`: reservation, ordering, restaurant information, and handoff rules
- `agent`: LangGraph workflow, prompts, typed tool adapters, and evaluations
- `rag`: ingestion, retrieval, citation assembly, and retrieval evaluation
- `worker`: notifications, indexing, audit export, and offline evaluation jobs
- `platform`: model gateway, persistence, cache, messaging, observability, and
  security adapters

This structure makes boundaries visible without creating fake distributed
complexity. The mentor may approve a service extraction only after the student
can identify an independent scaling, ownership, reliability, or deployment
need.

The detailed architecture and deployment progression are defined in
[Technical Learning Architecture](technical-learning-architecture.md).

## 6. Student ownership

The student owns the backend implementation end to end:

- Translate product requirements into stories and acceptance criteria
- Maintain the GitHub Project and deliver one focused Issue through one pull
  request
- Write architecture decision records and API contracts
- Implement schema migrations, APIs, domain rules, agent graphs, and workers
- Create unit, integration, concurrency, agent-evaluation, and load tests
- Open small pull requests and respond to review comments
- Operate the staging environment and investigate injected incidents
- Measure performance and preserve evidence for interview and resume use
- Demo each sprint and explain tradeoffs without relying on the mentor

A starter customer UI and staff dashboard may be provided. The student only
owns the thin integration necessary to exercise the backend; UI polish is not a
track objective.

## 7. Mentor ownership

The mentor acts as Tech Lead, not as a feature implementer:

- Establish the initial backlog and define product constraints
- Review the GitHub Project, Issues, pull requests, and Actions evidence
- Review design documents before high-risk implementation starts
- Review pull requests for correctness, reliability, maintainability, and
  security
- Run design defenses and require the student to justify tradeoffs
- Introduce realistic requirement changes, bug reports, incidents, and
  operational constraints
- Prevent overengineering and keep delivery focused on observable outcomes
- Conduct backend, project deep-dive, system-design, and behavioral interviews
- Verify that resume statements match the evidence ledger

The working process and simulated team roles are defined in
[Mentor and Scrum Playbook](mentor-scrum-playbook.md).

## 8. GitHub operating model

GitHub is the single workspace for planning, code, asynchronous review, CI/CD,
documentation, and evidence.

- The default is a private GitHub Free repository with the mentor as a
  collaborator.
- Development is trunk-based around one releasable `main` branch.
- One task Issue maps to one short-lived branch and one focused pull request.
- The GitHub Project uses `Backlog`, `Sprint`, `In Review`, and `Done`.
- The mentor reviews pull requests asynchronously between sessions.
- Session 3 establishes CI and the merge gate; Session 17 extends it to the
  zero-cost CD path.
- ADRs, runbooks, incident reports, evaluations, performance reports, and the
  evidence ledger live under repository `docs/`; the Wiki is not used.
- Every session starts from the Project board and ends with updated ownership,
  status, and evidence.

GitHub Free does not provide the full protected-branch feature set for private
repositories. The required zero-cost profile therefore uses a merge-guard
command that verifies CI and mentor approval before squash merge. Native branch
protection and required checks are enabled when the student already has a
no-cost Pro, Team, or education entitlement. No plan purchase is required.

The complete workflow, board transitions, branch and PR contracts, merge-gate
profiles, documentation layout, and Jira concept mapping are defined in the
[GitHub Operating Model](github-operating-model.md).

## 9. Learning and session format

The track uses a **flipped classroom + project lab + Tech Lead mentoring**
model. It is not a one-hour lecture followed by passive homework, and it is not
a live-coding course in which the mentor implements the feature for the
student.

Across the track, the intended balance is:

- Approximately 20% targeted explanation and design instruction
- Approximately 60% project lab, code review, debugging, and architecture work
- Approximately 20% acceptance, interview practice, and delivery planning

The student should own at least 70% of the speaking, reasoning, and hands-on
activity during a typical session.

### Before each session

The student:

- Reviews the assigned focused reading or reference implementation
- Completes the agreed implementation, test, design, or operations task
- Opens a pull request or submits the required ADR, trace, report, or demo
- Records completed work, next work, blockers, attempted hypotheses, and
  evidence

The main implementation happens between sessions. Live time is reserved for
work that benefits from expert feedback, joint diagnosis, and engineering
judgment.

### Standard 60-minute session

| Time | Activity | Primary owner |
|---:|---|---|
| 5 minutes | Standup: progress, blockers, and delivery risk | Student |
| 5 minutes | Previous artifact and evidence review | Student presents; mentor accepts or rejects |
| 15 minutes | Just-in-time mini lecture or design review | Mentor teaches only the concepts needed for the current work |
| 20 minutes | Project lab: code review, pair debugging, SQL or trace analysis, failure reproduction, or architecture exercise | Student operates; mentor guides and questions |
| 10 minutes | Interview drill or design defense tied to the current feature | Mentor asks; student explains and defends |
| 5 minutes | Next story, acceptance criteria, owner, and required evidence | Mentor and student confirm |

The mentor does not take over the keyboard to complete the student's feature.
The mentor may demonstrate a small isolated technique, but the student must
apply it, finish the change, add tests, and explain the result.

### Session types

The exact balance changes by lesson:

- **Concept plus guided lab:** Sessions 2, 6, 9, 12, and 17 introduce a major
  technology or architecture boundary. They use a focused mini lecture followed
  immediately by design or implementation work.
- **Engineering review and failure lab:** Sessions 5, 8, and 13 focus
  on code review, concurrency, evaluation, dependency failures, incident work,
  and acceptance evidence.
- **Mock week:** Sessions 10 and 16 run a formal mock defense plus gate
  evidence acceptance instead of a regular lesson.
- **System and capstone defense:** Sessions 18 and 19 focus on system design,
  deployment operations, demo, architecture defense, behavioral evidence, and
  resume verification.

### Mock-week format (Sessions 10 and 16)

Sessions 10 and 16 are mock weeks: there is no regular lesson. The live
session is a formal mock defense plus gate evidence acceptance. This keeps
the mentor's week within two hours and gives the student a focused
assessment week with no new feature work assigned.

| Mock | Prompt released | Submission deadline | Live session |
|---|---|---|---|
| Session 10 mid-track deep dive | Immediately after Session 8 | At least 48 hours before Session 10 | 60-minute mock defense |
| Session 16 late-track deep dive | Immediately after Session 13 | At least 48 hours before Session 16 | 60-minute mock defense |

The packet is released early so the student can draft answers across two
intervals; the final interval adds the newest sprint's answers, the
recording, and the gate evidence package.

Each mock package is bounded to:

- 6-8 published questions
- A written response of at most 1,500 English words or 2,500 Chinese characters
- One recorded code and architecture walkthrough of at most 20 minutes
- At most 45 minutes of combined asynchronous mentor review of the mock
  package and the gate evidence package
- One rubric, written feedback record, score, and remediation status

The student submits the Gate C (session 10) or Gate E (session 16) evidence
package against the gate checklist at the same deadline.

The 60-minute live session runs as:

| Time | Activity |
|---:|---|
| 15 minutes | Gate evidence acceptance: the student walks through the checklist evidence; the mentor accepts or rejects |
| 35 minutes | Formal mock defense run as a real interview, based on the submitted package with follow-up questions |
| 10 minutes | Rubric feedback, score, and next assignment |

Mentor time for a mock week is bounded to two hours: at most 45 minutes of
async package review, 10 minutes of preparation, 60 minutes live, and about
5 minutes to record the rubric. No regular artifact review happens that
week; code-level feedback continues through async pull-request comments.

The technical gate and the mock are assessed separately. Gate C and Gate E
measure whether the implemented system satisfies its correctness and operations
criteria. The mock measures technical explanation and interview readiness. A
mock requires a score of at least 70 out of 100. Failing the mock does not
invalidate an otherwise passing technical gate, but every required mock must be
passed before Gate F.

One remediation attempt is allowed. The student submits corrected written
answers or a replacement recording, then defends them in a later session's
10-minute interview slot. Remediation does not add a twentieth session.

A late submission is marked `NOT_ASSESSED`. The live session then reverts to
a regular engineering session (gate evidence review plus lab work), and the
mock follows the remediation process. The mentor is not expected to complete
rushed pre-review before the live session.

Mock artifacts must use synthetic data and must not expose secrets, API keys,
access tokens, real customer PII, or private mentor comments. The evidence
ledger stores the prompt, written response, recording reference, rubric,
feedback, score, and remediation result. Raw recordings are deleted 30 days
after track completion unless the student explicitly chooses to retain their
own copy.

### After each session

The student resolves review findings, completes the accepted story, updates
tests and documentation, and adds evidence to the ledger. The next session
starts from the submitted artifact rather than repeating a general lecture.

## 10. Nineteen-session delivery plan

The track contains exactly 19 mentor-led 1v1 sessions. Each session is planned
for 60 minutes and requires a concrete project artifact before the next
session. The student completes 6-10 hours of implementation, testing, reading,
or documentation between sessions.

A session is not a lecture. It combines artifact acceptance, design review,
code or incident work, an interview drill, and the next assignment. Calendar
pacing may be one or two sessions per week, but dependencies and gates remain
in the order below.

### Sprint map

| Sprint | Sessions | Outcome | Gate |
|---|---:|---|---|
| Sprint 0 | 1 | Kickoff, baseline, role, and product plan | Track plan accepted |
| Sprint 1 | 2-3 | API and engineering foundation | Gate A |
| Sprint 2 | 4-5 | Reservation correctness and data consistency | Gate B |
| Sprint 3 | 6-8 | Agent workflow and typed tool calling | Agent workflow review |
| Sprint 4 | 9-10 | RAG and LLM quality | Gate C |
| Sprint 5 | 11-13 | Ordering and event-driven workflows | Gate D |
| Sprint 6 | 14-16 | Security, operations, and performance | Gate E |
| Sprint 7 | 17-19 | Cloud delivery, Kubernetes, and capstone | Gate F |

### Session-by-session design

| Session | Engineering focus | Live mentor work | Required between-session artifact or gate |
|---:|---|---|---|
| 1 | Project kickoff and SDE baseline | Review the restaurant product, assign Student Engineer and Mentor/Tech Lead roles, assess backend and LLM knowledge, create the private repository and four-column GitHub Project, establish the Sprint 1 backlog and evidence ledger, approve the zero-cost tool manifest, and practice a two-minute project pitch | Project charter, skill baseline, target-role profile, private repo with mentor access, `Backlog / Sprint / In Review / Done` board, Issue and PR templates, prioritized Sprint 1 backlog, `docs/` structure, hardware and environment checklist, versioned zero-cost tool manifest, and accepted 19-session plan |
| 2 | FastAPI and API design | Design resource boundaries, request and response schemas, error taxonomy, versioning, request IDs, health, and readiness; review the first vertical slice | Runnable FastAPI service, OpenAPI baseline, shared result envelope, middleware, unit tests, and API-design ADR |
| 3 | PostgreSQL, migrations, containers, and CI | Trace one request to a committed row; review SQLAlchemy boundaries, Alembic strategy, configuration, Docker Compose, private-repository Actions allowance, self-hosted or local CI fallback, trunk rules, merge-guard or native branch protection, and zero-dollar usage controls | Schema baseline, reversible migration, local stack, stable lint/type/test/migration/image checks, demonstrated rejected failing change, merge-guard or native required checks, squash-merge evidence, clean-start guide, Actions usage control, and **Gate A** |
| 4 | Reservation domain and SQL design | Model restaurant time, tables, combinations, slots, holds, reservations, events, and state transitions; defend indexes and transaction boundaries | Reservation schema and repositories, search and hold contracts, seed data, transaction ADR, and integration-test skeleton |
| 5 | Concurrency, idempotency, and Redis-compatible caching | Reproduce the last-table race; review locks, constraints, optimistic versions, retry semantics, safe cache-aside use with Valkey, rate limits, and cache outage behavior | Working prepare/confirm/modify/cancel flow, repeatable concurrency tests, idempotency records, Valkey degradation test, and **Gate B** |
| 6 | Local model gateway and LangGraph state | Separate deterministic code from model behavior; design fake and Ollama adapters, optional unpaid hosted adapters, deadlines, retries, graph state, nodes, and routing; recheck model terms and hardware | Provider-neutral gateway, deterministic fake, local Qwen3 path, graph skeleton, information and action routes, node tests, and model-boundary ADR |
| 7 | Tool Calling and confirmation safety | Review typed read and command tools, server-injected context, authorization, prepare-review-confirm, prompt injection, and audit requirements | Typed tools calling application services, pending-action protocol, ambiguous-confirmation tests, authorization tests, and tool-contract documentation |
| 8 | Agent reliability, memory, and handoff | Debug invalid tool output and a model timeout; review checkpoints, cancellation, graph limits, fallback, human handoff, traces, and evaluation cases | End-to-end text-agent path, bounded failure handling, handoff flow, trace correlation, initial agent evaluation set, Sprint 3 demo, and release of the Session 10 mock packet |
| 9 | Local RAG ingestion and pgvector | Decide what belongs in RAG; review document versions, metadata, chunking, local MiniLM embeddings, asynchronous ingestion, and atomic activation | Seeded policy corpus, source manifest, local embedding adapter, ingestion worker, versioned chunks and embeddings, pgvector index, and rollback test |
| 10 | Mock week: mid-track deep dive and Gate C | **No regular lesson.** Accept Gate C evidence against the checklist; run the formal mid-track deep-dive mock as a real interview; deliver rubric feedback and score | Gate C evidence package plus the Session 10 mock package (written answers and recorded walkthrough), both submitted at least 48 hours before the session |
| 11 | Takeout ordering and deterministic pricing | Model menu versions, modifiers, carts, totals, order states, and fulfillment; review server-side calculation and confirmation | Menu and ordering schema, price engine, prepare-confirm order API, state-transition tests, and order OpenAPI contract |
| 12 | RabbitMQ, events, and transactional outbox | Design event envelopes, exchanges, routing, producer acknowledgement, outbox atomicity, relay behavior, and schema evolution | Versioned event contracts, outbox table and relay, RabbitMQ topology, correlation and causation IDs, and integration tests |
| 13 | Idempotent workers, retries, and dead letters | Inject duplicate delivery, broker outage, worker crash, and poison message; compare RabbitMQ with Kafka | Notification and indexing workers, consumer deduplication, bounded retries, DLQ and replay command, recovery runbook, release of the Session 16 mock packet, and **Gate D** |
| 14 | Authentication, authorization, and LLM security | Threat-model customer tokens, staff JWT/RBAC, tenant isolation, PII, secrets, rate limits, prompt injection, and unsafe tools | Auth and RBAC implementation, negative authorization suite, audit records, PII-redaction checks, threat model, and security review |
| 15 | Observability, SLOs, and incident response | Trace a request through API, agent, database, outbox, broker, and worker; define service indicators, alerts, and an incident process | OpenTelemetry instrumentation, structured logs, dashboards, SLO proposal, alerts, runbooks, and one mentored incident drill |
| 16 | Mock week: late-track deep dive and Gate E | **No regular lesson.** Accept Gate E evidence against the checklist; run the formal late-track deep-dive mock as a real interview; deliver rubric feedback and score | Gate E evidence package plus the Session 16 mock package (written answers and recorded walkthrough), both submitted at least 48 hours before the session |
| 17 | Zero-cost CI/CD and AWS architecture mapping | Build an immutable image; deploy an ephemeral kind environment in GitHub Actions or the local fallback; run migration, smoke, and rollback checks; map the same system to ECS Fargate, RDS, ElastiCache, Amazon MQ, S3, ECR, Secrets Manager, and CloudWatch without provisioning them; run a bounded debugging interview | Free CI workflow, local or ephemeral staging deployment, migration job, smoke test, rollback evidence, zero-cost audit, AWS architecture and cost-risk diagram, and no billable resource |
| 18 | Kubernetes operations and system design | Operate the same images on kind or k3d; review probes, resources, configuration, secrets, migration jobs, HPA, rollout, rollback, and local-staging-versus-managed-cloud tradeoffs; run the combined system-design mock | Kubernetes manifests or chart, healthy rollout and rollback, pod failure drill, scaling and cloud-mapping explanation, capstone rehearsal, and resolved critical gaps |
| 19 | Capstone, project defense, and career evidence | Run the product demo, architecture defense, project deep dive, behavioral interview, evidence review, and resume verification | Passing hard-gate suite, final reports and runbooks, verified evidence ledger, approved truthful resume bullets, development plan, and **Gate F** |

If a required artifact fails review, remediation happens between scheduled
sessions and is rechecked at the next session. A session is not marked complete
merely because the meeting occurred.

## 11. Parallel interview-readiness lane

Project work supplies the interview material, but the mentor reserves part of
each 1v1 session for explicit SDE preparation:

- Sessions 1-5: API, SQL, transactions, indexes, networking, and debugging
- Sessions 6-10: LLM application design, RAG, tool calling, evaluation, and
  safety
- Sessions 11-13: caching, messaging, consistency, and delivery semantics
- Sessions 14-16: security, reliability, incidents, scaling, and performance
- Sessions 17-19: cloud and Kubernetes tradeoffs, full system design, project
  deep dive, behavioral stories, and resume defense

Algorithm practice may be assigned separately when the target role requires it,
but it does not replace the backend and project gates in this track.

The complete rubric is defined in
[Assessment, Interview, and Resume Evidence](assessment-interview-resume.md).

## 12. Required graduation artifacts

The student must finish with:

- Running source code and reproducible local environment
- GitHub Project history, Issue and pull-request templates, reviewed PRs, and
  demonstrated merge gate
- Versioned OpenAPI contract and database migrations
- Architecture decision records and system diagrams
- Unit, integration, concurrency, contract, agent-evaluation, and load tests
- RAG corpus manifest and evaluation report
- Event contracts, retry policy, dead letter procedure, and replay tool
- CI/CD workflow and deployment manifests
- Versioned tool manifest and final zero-cost audit
- Dashboards, SLOs, alerts, runbooks, and one incident report
- Sprint backlog, review comments, demos, and retrospectives
- Final project presentation and interview question bank
- Evidence ledger and truthful resume bullet set

## 13. Definition of track completion

The track is complete only when:

1. All hard gates in the assessment rubric pass.
2. No known critical data-consistency, authorization, or unsafe-tool issue is
   open.
3. The student independently explains the architecture and its alternatives.
4. The staging system can be deployed, observed, exercised, and rolled back.
5. Performance and quality claims are backed by reproducible measurements.
6. Resume wording accurately identifies the work as a mentored industry
   project or practicum unless a real internship relationship exists.

Passing the track demonstrates production-oriented backend and AI application
engineering. It does not claim expertise in foundation-model research or model
training.
