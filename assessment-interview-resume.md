# Assessment, Interview, and Resume Evidence

Status: Proposed graduation and career-readiness rubric  
Parent track: [1v1 Agent Backend Track](1v1-agent-backend-track.md)
Delivery workflow: [GitHub Operating Model](github-operating-model.md)

## 1. Assessment principles

The track evaluates demonstrated engineering behavior, not attendance or the
number of technologies mentioned.

Assessment evidence must be:

- Produced by the student
- Reproducible from the repository and documented environment
- Tied to a requirement or engineering decision
- Reviewed by the mentor
- Honest about scope, environment, limitations, and simulated data

The mentor may use LLM-assisted code as a discussion point, but the student must
explain, test, and own every submitted change.

## 2. Graduation hard gates

Every hard gate must pass. A high total score cannot compensate for a failed
gate.

| Gate | Required result |
|---|---|
| Data integrity | No double booking, incorrect total, invalid transition, or lost committed command in required tests |
| Authorization | Customer, staff role, tenant, and location boundaries pass positive and negative tests |
| Agent safety | No write tool executes without validated context, pending action, and later explicit confirmation |
| RAG isolation | No cross-tenant retrieval; structured facts bypass RAG; unsupported answers abstain |
| Retry safety | API retries and message redelivery do not duplicate durable business effects |
| Failure transparency | Dependency failures never return a success-shaped response |
| Reproducibility | A clean environment can build, migrate, test, and start from documentation |
| Deployment | Staging deploy, smoke test, observability check, and rollback are demonstrated |
| Zero-cost compliance | Required workflows pass without a payment method, trial credit, or billable resource; tool manifest and fallbacks are verified |
| Delivery governance | Every completed task maps to an Issue, focused reviewed PR, green CI, approved merge path, current Project status, and evidence |
| Explainability | Student independently defends major architecture and tradeoff decisions |
| Career integrity | Resume and interview statements match the evidence ledger and actual relationship |

## 3. Scored rubric

Passing requires at least 80 of 100 points plus every hard gate.

| Dimension | Points | Full-credit evidence |
|---|---:|---|
| API and backend design | 12 | Versioned contracts, validation, stable errors, pagination, auth, and compatibility |
| Database and concurrency | 15 | Constraints, transactions, locks, idempotency, versions, migrations, and query plans |
| Distributed reliability | 13 | Outbox, delivery semantics, consumer deduplication, retries, dead letters, and recovery |
| Agent workflow and tools | 13 | Explicit graph, typed tools, confirmation, limits, fallback, and traceability |
| RAG and evaluation | 10 | Versioned corpus, filtered retrieval, citations, abstention, isolation, and measured evaluation |
| Testing and security | 10 | Risk-based automated tests, threat model, PII controls, and negative authorization cases |
| Cloud, delivery, and operations | 10 | Zero-cost containers, CI/CD, local Kubernetes staging, AWS architecture mapping, telemetry, runbooks, and rollback |
| Performance engineering | 7 | Reproducible load test, bottleneck proof, justified optimization, and before/after data |
| Product delivery and Agile | 5 | GitHub Project history, accepted Issues, focused reviewed PRs, scope control, demos, retrospectives, and response to change |
| Communication and interview defense | 5 | Clear architecture narrative, tradeoffs, incident story, and concise answers |

## 4. Milestone assessments

### Gate A: Backend foundation, end of Sprint 1 / Session 3

The student must:

- Explain the request lifecycle and module boundaries
- Add and roll back a migration
- Diagnose a deliberately broken configuration
- Defend the API error envelope and versioning policy
- Demonstrate CI failure on an invalid change
- Demonstrate the private-repository trunk workflow from Sprint Issue through
  focused PR, mentor review, green checks, squash merge, board update, and
  evidence link
- Demonstrate the merge guard or native protected-branch profile
- Present the approved zero-cost tool manifest and prove that required CI stops
  instead of billing after free allowance is exhausted

### Gate B: Database correctness, end of Sprint 2 / Session 5

The student must:

- Write SQL for availability and explain the relevant indexes
- Explain the chosen isolation and locking behavior
- Reproduce and prevent a last-table race
- Explain idempotency-key reuse semantics
- Demonstrate correct behavior when Valkey is unavailable

### Gate C: Agent and RAG, end of Sprint 4 / Session 10

The student must:

- Draw the graph and explain every command boundary
- Demonstrate ambiguous-confirmation rejection
- Distinguish retrieval failure from generation failure
- Explain why hours, prices, and availability are not RAG facts
- Run evaluation cases for citations, abstention, and tenant isolation

### Gate D: Distributed systems, end of Sprint 5 / Session 13

The student must:

- Explain at-least-once delivery and its application consequences
- Demonstrate duplicate-consumer safety
- Recover from a broker outage and a poison message
- Explain outbox failure windows
- Compare RabbitMQ and Kafka using this product's requirements

### Gate E: Production readiness, end of Sprint 6 / Session 16

The student must:

- Find a bottleneck using telemetry instead of guessing
- Explain connection pools, backpressure, timeouts, and circuit breakers
- Lead one incident drill
- Demonstrate role and tenant authorization tests
- Present measured performance with environment and caveats

### Gate F: Capstone, end of Sprint 7 / Session 19

The student must:

- Deploy and roll back staging
- Run the final end-to-end and hard-gate suites
- Present a 10-minute product demo and a 15-minute architecture and project
  defense
- Complete the mock interview sequence
- Pass every required mock assessment or its one allowed remediation
- Submit the final zero-cost audit with no required billable dependency
- Submit the verified evidence ledger and resume material

## 5. Interview-readiness plan

### Ongoing format

Each of the 19 mentor sessions includes a short drill tied to the current
engineering work. The student answers first, then receives feedback on
correctness, structure, and depth.

Long-form mocks occur at four points: a mid-track deep dive (session 10), a late-track deep dive (session 16), a system-design mock (session 18), and the capstone loop (session 19). Sessions 10 and 16 are mock weeks: there is no regular lesson. The student submits written answers and a recorded code walkthrough at least 48 hours before the session, the mentor reviews them async (timeboxed to 45 minutes including gate evidence), and the full 60-minute live session is a formal mock defense plus gate evidence acceptance. This keeps formal interview practice rigorous while holding the mentor's week to two hours.

| Session | Mock | Focus |
|---:|---|---|
| 10 | Mid-track deep dive | Backend (REST, SQL, transactions, concurrency, cache) plus AI application (agent workflow, tools, RAG, evaluation, safety) |
| 16 | Late-track deep dive | Distributed systems (messaging, delivery semantics, retries, recovery) plus production (security, incidents, observability, performance) |
| 18 | System-design mock | Backend scale plus LLM application architecture |
| 19 | Capstone loop | Project deep dive, behavioral evidence, and resume defense |

### Mock-week rubric

The Session 10 and Session 16 operating rules, release dates, deadlines, and
artifact limits are defined in
[1v1 Agent Backend Track](1v1-agent-backend-track.md#mock-week-format-sessions-10-and-16).
Each mock is scored independently from its technical gate:

| Dimension | Points | Full-credit evidence |
|---|---:|---|
| Technical correctness | 35 | Accurate explanation of APIs, data, agent, RAG, messaging, security, or operations behavior in scope |
| Tradeoff and failure reasoning | 25 | Alternatives, failure modes, consistency, reliability, and consequences are explained rather than named |
| Evidence from the student's system | 20 | Answers reference the student's code, tests, traces, measurements, incidents, or ADRs |
| Communication and structure | 15 | Direct, organized answers at the expected technical depth |
| Integrity and limitations | 5 | Clearly distinguishes measured facts, assumptions, synthetic data, and unfinished work |

Passing requires at least 70 out of 100. One remediation attempt is allowed.
An unsuccessful or late mock does not reverse a passing Gate C or Gate E, but
it leaves interview readiness incomplete and blocks Gate F until remediated.

### Final interview sequence within Sessions 17-19

1. **Session 17: coding and debugging, 25 minutes**  
   Implement a bounded backend change or diagnose a failing API using tests,
   logs, SQL, and profiler evidence. The rest of the session covers zero-cost
   CI/CD and the AWS architecture mapping.
2. **Session 18: combined system design, 40 minutes**  
   Design a multi-restaurant reservation and takeout platform, then extend it
   with the model gateway, agent workflow, RAG, evaluation, safety, latency, and
   cost controls. The remaining session time records feedback and capstone
   actions.
3. **Session 19: capstone interview, 60 minutes**  
   Allocate 10 minutes to the product demo, 15 minutes to architecture and
   project deep dive, 5 minutes to incident and performance evidence,
   10 minutes to behavioral questions, 10 minutes to resume and evidence
   verification, and 10 minutes to final feedback and development planning.

All interview components are contained within the 19 scheduled sessions; the
plan does not assume additional 1v1 classes.

## 6. SDE topic matrix

### API and backend fundamentals

The student should be able to answer and demonstrate:

- How are resources, commands, and errors represented?
- Which operations are idempotent by method semantics, and which require an
  idempotency key?
- How is backward compatibility preserved?
- How do request cancellation, deadlines, and retries propagate?
- How are customer and staff authorization separated?
- What changes when multiple API processes handle requests?

### PostgreSQL and concurrency

- Which invariants belong in constraints versus application code?
- What anomaly can occur at each relevant isolation level?
- When are row locks, advisory locks, exclusion constraints, or optimistic
  versions appropriate?
- Why can an availability search become stale before a hold?
- How does a query plan change with cardinality and indexes?
- How do pool size and transaction duration affect throughput?

### Valkey and Redis-compatible caching

- Which data can be reconstructed?
- How is cache invalidation triggered?
- How are stale data and stampedes limited?
- What happens when Valkey is slow or unavailable?
- Which Redis concepts and client behavior transfer to Valkey?
- Why is a cache-based distributed lock not the durable reservation guarantee?

### Messaging

- What do producer acknowledgement and consumer acknowledgement guarantee?
- Where can duplicate publication and delivery occur?
- Why is an outbox needed?
- How is a consumer side effect made idempotent?
- When should a message retry, dead-letter, or be rejected immediately?
- When would Kafka be a better fit than RabbitMQ?

### Agent and tool calling

- Why should the model not query the database or calculate order totals?
- How are tool schemas, authorization, and server context enforced?
- How is a graph loop bounded?
- How does the system recover when the model emits invalid arguments?
- Why is confirmation a protocol rather than a prompt sentence?
- Which information is safe to place in model context?

### RAG

- Which content belongs in embeddings?
- How do chunk size, overlap, metadata, and top-k affect retrieval?
- How are document updates made atomic?
- How are retrieval and generation errors separated?
- How are citations verified?
- How do tenant filters and prompt-injection defenses work?
- What metrics are useful, and where can an LLM judge mislead?

### Reliability and performance

- Which SLOs are customer-relevant?
- How are latency budgets split across API, database, retrieval, tools, and
  model?
- What is the bottleneck evidence?
- How does the system apply backpressure?
- Which dependencies can degrade gracefully?
- How are capacity, saturation, and cost estimated?

### Cloud and delivery

- How does the no-cost kind or k3d staging environment preserve meaningful
  deployment and operations learning?
- How do the local components map to ECS Fargate, RDS, ElastiCache, Amazon MQ,
  S3, ECR, Secrets Manager, and CloudWatch?
- Which AWS services create unavoidable usage charges, and why are they not
  graduation dependencies?
- When would managed AWS services or production Kubernetes be justified?
- How are database migrations sequenced for zero-downtime release?
- How are secrets, networking, and image provenance handled?
- What signals trigger rollback?
- How are stateful dependencies restored and tested?

## 7. Debugging exercises

The mentor selects at least four:

1. A reservation race passes unit tests but fails with two API processes.
2. An index exists but PostgreSQL still chooses a sequential scan.
3. A cache key omits location and leaks the wrong operating hours.
4. A request timeout causes a client retry and duplicate side effect.
5. A worker sends two notifications after crashing between side effect and
   acknowledgement.
6. A RAG query retrieves the old policy version.
7. A prompt-injected document attempts to invoke a write tool.
8. A connection pool is exhausted by long transactions.
9. A RabbitMQ poison message creates an infinite retry loop.
10. A deployment succeeds, but readiness fails because migration ordering is
    wrong.

The student must state hypotheses, gather evidence, minimize the reproduction,
fix the root cause, add a regression test, and explain why monitoring did or did
not detect the issue.

## 8. System-design exercises

### Backend system design

Prompt:

> Expand Juniper & Stone from one restaurant to 10,000 locations. Support
> reservations, takeout ordering, staff operations, external providers, and
> regional failover.

Expected discussion:

- Tenant and location partitioning
- Consistency boundary for inventory
- API and event contracts
- Data model, indexing, and archival
- Caching and invalidation
- Queue and worker scaling
- Idempotency and provider reconciliation
- Failure domains and multi-region tradeoffs
- Observability, capacity, security, and migration path

The student should evolve the current design, not replace it with generic
microservices.

### ML/LLM system design

Prompt:

> Design a safe multilingual restaurant agent that answers policies and
> completes reservations and orders while meeting latency, quality, privacy,
> and cost constraints.

Expected discussion:

- Model gateway and model-selection policy
- Agent graph and deterministic boundaries
- Tool schemas, authentication, confirmation, and audit
- Structured lookup versus RAG
- Ingestion, retrieval, citations, and evaluation
- Conversation memory and retention
- Prompt and model versioning
- Online and offline quality signals
- Fallback and human handoff
- Latency and cost budgets
- Privacy, prompt injection, abuse, and incident response

## 9. Project deep-dive question bank

The student should prepare concise and detailed versions of each answer:

- What was the hardest technical decision and what alternatives did you reject?
- Describe the last-table race at the SQL and transaction level.
- Why did you choose a modular monolith?
- Why is Valkey not the source of truth, and which Redis-compatible concepts
  does the design use?
- How do you prevent a model from performing an unauthorized write?
- Explain one RAG miss from query to final answer.
- Where can the outbox flow duplicate work?
- How did you measure and improve one bottleneck?
- Describe an incident, its mitigation, root cause, and prevention.
- What requirement changed and how did your design adapt?
- What would you change for 100 times the traffic?
- What did your mentor disagree with, and how was the decision resolved?
- Which part would you extract first and what evidence would justify it?
- What remains incomplete or risky?

## 10. Behavioral evidence

The student prepares STAR or CARL stories for:

- Owning an ambiguous requirement
- Receiving and applying difficult review feedback
- Disagreeing with the mentor using technical evidence
- Debugging a failure under time pressure
- Cutting scope to protect a release
- Improving a process after a retrospective
- Preventing a security or data-integrity issue
- Learning an unfamiliar technology
- Explaining a technical tradeoff to a non-engineering stakeholder

Each story must identify the student's actual action. The mentor's work and
simulated stakeholder input are not described as the student's implementation.

## 11. Evidence ledger

Maintain one versioned ledger with these fields:

| Field | Meaning |
|---|---|
| Evidence ID | Stable identifier such as `EVID-023` |
| Date and sprint | When the evidence was produced |
| Story or incident | Backlog item or drill |
| Student contribution | Specific design, code, test, or operation owned |
| Claim supported | Potential interview or resume statement |
| Artifact links | Pull request, ADR, test, dashboard, trace, report, or demo |
| Environment | Local, CI, staging, cloud service, data size, and configuration |
| Measurement method | Script, duration, concurrency, repetitions, and tool |
| Result | Actual p50, p95, p99, throughput, error, quality, or recovery result |
| Caveats | Synthetic data, provider variance, known limits, and excluded scope |
| Mentor verification | Date, status, and notes |

### Measurement protocol

Performance evidence records:

- Source revision and image digest
- Dataset and seed version
- Hardware or cloud resource sizes
- Dependency and configuration versions
- Warm-up, run duration, concurrency, and request mix
- Number of runs
- Latency distribution, throughput, and error rate
- Before and after results when claiming improvement
- Cost window when claiming cost impact

A staging load-test result may be described as a staging benchmark. It must not
be presented as production traffic or real user scale.

## 12. Truthful resume positioning

### Recommended heading

```text
PROJECT EXPERIENCE

Agent Backend Engineer - Restaurant Customer Service Agent
Mentored Industry Project | Month Year - Month Year
```

Alternative truthful labels include:

- AI Backend Engineering Practicum
- Mentored Agent Backend Capstone
- Production-Style Backend Engineering Project

Use "Software Engineering Intern" only when a real organization formally
engaged the student in that role and can verify the relationship, dates, and
responsibilities. A mentor-led project should not be relabeled as employment.

### Bullet construction rule

Use:

> Action + engineering problem + implementation + measured result or verified
> system property

Do not use:

- Invented daily active users, revenue, QPS, uptime, or cost savings
- "Production" when only local or staging environments existed
- "Led a team" when the work was 1v1
- Technologies that were explored but not implemented
- Performance percentages without a reproducible before and after

### Evidence-backed bullet templates

Replace every bracket only after the corresponding evidence is verified.

```text
- Designed and implemented a FastAPI and LangGraph restaurant-service backend
  with typed tool calling for reservations, takeout orders, policy questions,
  and human handoff, covering [N] automated agent and contract scenarios.

- Built a transaction-safe PostgreSQL reservation engine using resource locks,
  overlap constraints, optimistic versions, and idempotency keys; prevented
  double booking across [N] concurrent attempts in a reproducible staging test.

- Developed a tenant-filtered RAG pipeline with pgvector, versioned document
  ingestion, citations, and grounded abstention, achieving [measured retrieval
  metric] on a [N]-case evaluation set with zero cross-tenant retrievals.

- Implemented reliable asynchronous workflows with a transactional outbox,
  RabbitMQ, idempotent consumers, retries, and a dead letter queue; recovered
  [N] queued events after a [duration] broker-outage drill without lost
  committed work.

- Profiled PostgreSQL, Valkey, and API behavior under a documented
  [request mix] load test and reduced [measured latency or resource metric]
  from [before] to [after] through [verified optimization].

- Built a zero-cost GitHub Actions delivery workflow and deployed the API and
  workers to local Kubernetes staging with migrations, smoke tests,
  observability, probes, resource controls, and rollback; mapped the design to
  AWS managed services without claiming an unperformed cloud deployment.

- Delivered the project across 19 mentor-led 1v1 sessions with architecture
  reviews, pull-request feedback, demos, retrospectives, and incident drills;
  authored [N] ADRs, [N] runbooks, and a post-incident corrective-action plan.

- Used a trunk-based GitHub workflow with one Issue and focused pull request per
  task, asynchronous mentor review, automated CI gates, and a four-state
  Projects board; preserved implementation and delivery evidence across [N]
  merged pull requests.
```

The final resume should usually select three or four bullets most relevant to
the target role, not include every template.

## 13. Technical skills evidence

Add a skill only after the student can explain where and why it was used.

An expected final structure, subject to actual implementation, is:

```text
Languages: Python, SQL
Frameworks: FastAPI, LangGraph, LangChain, SQLAlchemy, Pydantic, pytest
Data and Messaging: PostgreSQL, pgvector, Valkey (Redis-compatible), RabbitMQ
Infrastructure: Docker, Kubernetes, Helm, GitHub Actions
Observability and Testing: OpenTelemetry, Prometheus, Grafana OSS, Locust
```

Do not list both Kafka and RabbitMQ if Kafka was only discussed. A resume can
mention Kafka in coursework or interests only if that context is explicit.
Likewise, list AWS under skills only if the student actually provisioned and
operated AWS resources. A design-only exercise should appear as an architecture
discussion or project bullet, not as hands-on AWS experience.

## 14. Repository and portfolio evidence

The final repository should include:

- Concise product and architecture overview
- System and sequence diagrams
- Local start and test instructions
- Seed-data and evaluation-data description
- API documentation
- Demo script or short recording
- Architecture decisions
- Benchmark methodology and results
- Security and privacy notes
- Runbooks and incident report
- Known limitations and next steps

Secrets, real customer data, raw access tokens, and private mentor feedback must
not be published.

The student should be able to reproduce the demo from the public instructions
or clearly document which cloud resources require private access.

## 15. Final graduation decision

The mentor records:

- Hard-gate pass or fail
- Rubric score with evidence links
- Strongest demonstrated competencies
- Remaining risk areas
- Interview readiness by category
- Approved resume wording
- Recommended next project or remediation

Graduation means the student can own and explain this production-style system.
It does not imply paid employment, real production users, or expertise beyond
the demonstrated evidence.
