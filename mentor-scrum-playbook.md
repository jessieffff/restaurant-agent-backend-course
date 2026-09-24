# Mentor and Scrum Playbook

Status: Proposed 1v1 operating model  
Parent track: [1v1 Agent Backend Track](1v1-agent-backend-track.md)

## 1. Purpose

This playbook gives the student realistic engineering ownership and delivery
practice inside a 1v1 mentorship. It simulates the decisions, handoffs, reviews,
incidents, and communication patterns of a product team without pretending that
additional employees or a real company exist.

The student should describe the experience accurately:

> I was the Agent Backend Engineer on a mentor-led industry project. My mentor
> acted as Tech Lead and Product Owner. We used sprint planning, design reviews,
> pull-request reviews, demos, retrospectives, and incident drills.

The student must not claim a five-person team, production customers, paid
employment, or an internship unless those facts are real and verifiable.

All planning, implementation review, CI/CD evidence, and durable documentation
follow the [GitHub Operating Model](github-operating-model.md). GitHub is the
single operational workspace; Jira, Linear, and a separate Wiki are not used.

## 2. Operating model

### Real roles

| Role | Owner | Responsibilities |
|---|---|---|
| Agent Backend Engineer | Student | Design, implementation, testing, deployment, operations, documentation, and demos |
| Tech Lead | Mentor | Architecture guidance, design approval, code review, reliability and security standards |
| Product Owner | Mentor | Product priorities, acceptance criteria, scope changes, and demo acceptance |
| Interviewer | Mentor | Technical deep dives, system design, debugging, behavioral feedback, and graduation decision |

### Simulated stakeholder perspectives

These are perspectives used to create realistic inputs; they are not fictional
people the student should list as teammates.

| Perspective | How it appears in the project | Student response |
|---|---|---|
| Restaurant operator | Business rules, hours, menu, cancellation policy | Clarify ambiguity and encode accepted rules |
| Frontend engineer | API contract request, error-handling need, schema change | Negotiate and version the contract |
| QA/SDET | Reproduction steps, boundary cases, regression report | Reproduce, test, fix, and document |
| SRE/Cloud engineer | Alert, capacity constraint, dependency outage | Triage, mitigate, write runbook and incident report |
| Security/privacy reviewer | Threat-model finding, PII or tool-risk question | Assess severity and implement a verified control |
| Data/ML platform reviewer | Evaluation drift, provider change, cost limit | Diagnose model or retrieval behavior using evidence |

## 3. Responsibility map

| Work item | Student | Mentor as Tech Lead | Mentor as Product Owner |
|---|---|---|---|
| Product requirement | Consulted | Consulted | Accountable |
| Story design and estimates | Responsible | Reviews | Accepts scope |
| Architecture decision | Proposes | Accountable for approval | Informed |
| API and event contract | Responsible | Reviews | Consulted |
| Feature implementation | Responsible | Does not implement for student | Informed |
| Tests and evaluation | Responsible | Reviews coverage and rigor | Accepts business behavior |
| Pull-request approval | Responds to feedback | Accountable | Informed |
| Staging release | Responsible | Reviews risk | Accepts demo |
| Incident response | Incident commander | Coaches and injects evidence | Informed |
| Sprint acceptance | Demonstrates | Reviews engineering quality | Accepts or rejects outcome |
| Career evidence | Drafts from artifacts | Verifies accuracy | Not applicable |

The mentor may pair on diagnosis or demonstrate a small isolated technique, but
the student remains responsible for the final design and implementation.

## 4. Nineteen-session sprint cadence

The track uses exactly 19 scheduled 1v1 sessions. Sprints are bounded by session
numbers rather than calendar weeks:

| Sprint | Sessions | Ceremony pattern |
|---|---:|---|
| Sprint 0 | 1 | Kickoff, baseline, role assignment, and plan acceptance |
| Sprint 1 | 2-3 | Foundation build followed by Gate A |
| Sprint 2 | 4-5 | Reservation build followed by Gate B |
| Sprint 3 | 6-8 | Agent design, tool review, and integrated demo |
| Sprint 4 | 9-10 | RAG build followed by Gate C |
| Sprint 5 | 11-13 | Order and messaging build followed by Gate D |
| Sprint 6 | 14-16 | Hardening, incident work, and Gate E |
| Sprint 7 | 17-19 | Deployment, capstone rehearsal, and Gate F |

### Sprint opening

The student opens the GitHub Project and prepares:

- Previous retrospective actions
- Proposed sprint goal
- Prioritized stories with dependencies
- Risk list and open questions
- Capacity estimate

The mentor and student agree on one sprint goal and no more than three primary
deliverables. Stretch work is explicitly labeled and never required to pass the
sprint.

### Build sessions

- Planning and requirement clarification
- Design note or ADR before high-risk code
- Thin vertical slice before broad implementation
- First pull request opened early
- Mid-sprint design and risk checkpoint

### Gate session

- Integration, failure handling, and observability
- Test and evaluation completion
- Pull-request review and revision
- Demo in the staging-like environment
- Sprint acceptance and retrospective

### Each 1v1 session, 60 minutes

| Time | Activity |
|---|---|
| 5 minutes | Progress, blockers, and workload health |
| 5 minutes | Sprint goal, metrics, evidence, and risk review |
| 15 minutes | Artifact acceptance, architecture, or design defense |
| 20 minutes | Pull-request, debugging, implementation, or incident deep dive |
| 10 minutes | Interview drill tied to current work |
| 5 minutes | Decisions, owner, required artifact, and next acceptance check |

Gate and capstone sessions may move time from implementation review to the demo
or mock interview, but every session still records an accepted artifact,
feedback, and the next assignment.

The mentor sends decisions and action items after the session. The student
records technical decisions in the repository, not only in private notes.

### Async work

- The student completes 6-10 hours of project work between sessions.
- The student posts at least one structured update between sessions:
  completed, next, blocked, and evidence.
- The mentor targets a two-business-day turnaround for review.
- The mentor reviews the ready pull request and linked Issue directly; a
  separate status document is not required.
- The mentor releases the Session 10 mock packet after Session 8 and the
  Session 16 mock packet after Session 13.
- Mock packages are due at least 48 hours before the live defense, and mentor
  review is timeboxed to 45 minutes per package, including the gate evidence
  package.
- Sessions 10 and 16 are mock weeks: the live session is a formal mock defense
  plus gate evidence acceptance, with no regular lesson. The mentor's week is
  bounded to two hours: at most 45 minutes of combined async review, 10
  minutes of preparation, 60 minutes live, and about 5 minutes to record the
  rubric.
- A late package is marked `NOT_ASSESSED` rather than creating an emergency
  review obligation; it follows the documented remediation process.
- A blocker lasting more than one student work session is escalated with logs,
  reproduction steps, attempted hypotheses, and a specific question.

## 5. Backlog hierarchy

One GitHub Project uses `Backlog`, `Sprint`, `In Review`, and `Done`. Every
implementation task is a GitHub Issue on that board and follows the transitions
defined in the [GitHub Operating Model](github-operating-model.md).

Use this hierarchy:

```text
Outcome
  Epic
    Story
      Engineering task
      Test or evaluation task
      Documentation or operations task
```

Every story includes:

- User or operator value
- Scope and non-goals
- API, event, or data contract impact
- Acceptance criteria
- Failure and authorization behavior
- Observability requirement
- Test evidence
- Rollout or migration note

The backlog should use stable IDs such as:

- `API-*` for public and staff contracts
- `RES-*` for reservations
- `ORD-*` for takeout ordering
- `AGT-*` for agent workflow and tools
- `RAG-*` for knowledge retrieval
- `EVT-*` for events and workers
- `PLT-*` for platform and deployment
- `OPS-*` for observability and incidents
- `SEC-*` for security and privacy

## 6. Definition of ready

A story is ready when:

- The customer or operator outcome is clear
- Acceptance criteria are testable
- Dependencies and ownership are known
- Data, API, event, and security impacts are identified
- Unknowns are bounded enough for the sprint
- A smaller vertical slice is not more appropriate

The mentor rejects a story that is only a technology task, such as "add
Valkey," without a product or reliability outcome.

## 7. Definition of done

A story is done only when:

- The implemented behavior satisfies every acceptance criterion
- Types, lint, tests, and compatibility checks pass
- Authorization and failure paths are covered
- Logs, metrics, and traces make the path diagnosable
- API, event, schema, and runbook documentation are current
- Database changes include migration and rollback or forward-fix strategy
- Review comments are resolved
- The focused pull request is approved, all required checks are green, and the
  squash merge is complete
- The GitHub Project item is moved to Done
- The staging-like environment demonstrates the behavior
- The evidence ledger links the story to its artifacts and measurements

"Works on my machine" and "the happy path passes" are not done.

## 8. Pull-request workflow

### Student expectations

- Start from a GitHub Issue accepted into the Sprint column
- Create one short-lived topic branch and one pull request for that task
- Open a draft pull request after the first vertical slice
- Keep the change focused; split unrelated refactors
- Include problem, approach, alternatives, risk, test plan, and screenshots or
  traces when useful
- Mark generated files and migration effects
- Self-review before requesting mentor review
- Respond to each comment with a code change or a reasoned explanation
- Merge only through the approved merge-guard or native protected-branch path,
  use squash merge, and delete the topic branch

### Mentor review order

The mentor reviews in this order:

1. Requirement and contract correctness
2. Data consistency and authorization
3. Failure behavior and retry safety
4. Test quality and observability
5. Maintainability and clarity
6. Performance and operational cost

Style comments do not distract from correctness. A blocking comment explains
the violated invariant and the evidence required to resolve it; it does not
provide the complete implementation.

### Review labels

- `blocking`: correctness, security, data loss, unsafe tool, or broken contract
- `required`: maintainability, test, observability, or operational issue needed
  before merge
- `suggestion`: useful improvement that may be deferred
- `question`: asks the student to explain or defend a decision

## 9. Design-review workflow

A design review is required before:

- Introducing a new durable data model
- Changing a public API or event contract
- Adding an LLM tool with write capability
- Changing transaction or idempotency behavior
- Adding a cache for correctness-sensitive data
- Creating a new queue, retry policy, or dead letter path
- Changing tenant, authentication, or authorization boundaries
- Adding a production dependency or deployment unit

The student submits a short RFC or ADR containing:

- Problem and constraints
- Current behavior
- At least two viable options
- Chosen option and rejected alternatives
- Correctness and failure analysis
- Security and privacy analysis
- Observability and test plan
- Rollout, rollback, and cost

The student leads the review. The mentor asks questions and records the decision.

## 10. Sample stories

### RES-012: Prevent last-table double booking

**Outcome**

Two customers racing for the same final table must not both receive confirmed
reservations.

**Acceptance criteria**

- The search result is advisory and does not guarantee a table
- Hold creation rechecks availability inside a transaction
- The database prevents overlapping active allocation
- Exactly one competing request succeeds
- The losing request receives `SLOT_STALE`
- The result remains correct across multiple API processes
- The concurrency test is repeatable and stores its run parameters

**Evidence**

- Schema and migration
- Transaction sequence diagram
- Integration and concurrency test
- Trace showing winner and loser
- Student explanation of lock and isolation choices

### AGT-008: Require review before agent mutation

**Outcome**

The agent cannot create, modify, cancel, or submit an order based on an
ambiguous conversational acknowledgement.

**Acceptance criteria**

- The graph creates a versioned pending action
- A customer-visible summary is emitted before confirmation is accepted
- Confirmation comes from the same session and a later event
- Pending actions expire and cannot be replayed
- The typed command tool revalidates state and authorization
- "Okay" in an ambiguous context does not execute the command

**Evidence**

- Graph path
- Tool schema
- Deterministic evaluation cases
- Audit trace for prepared, confirmed, expired, and rejected paths

### EVT-006: Recover notifications after broker outage

**Outcome**

Committed reservations and orders retain their notification work while
RabbitMQ is unavailable.

**Acceptance criteria**

- The business transaction and outbox record commit atomically
- The API does not report notification completion before it occurs
- Outbox age and publish failures alert
- Relay retries do not publish an invalid schema
- Duplicate messages are safe for the notification consumer
- The backlog drains after broker recovery

**Evidence**

- Event contract
- Outbox and consumer-deduplication code
- Five-minute outage drill
- Queue and outbox dashboard
- Recovery runbook

## 11. Planned requirement changes

The mentor injects changes only after the student has a stable baseline. Each
change tests adaptation rather than surprise trivia.

| Sprint | Change request | Skill being tested |
|---|---|---|
| 2 | Restaurant closes the patio for an unexpected private event | Time-bound inventory override and contract behavior |
| 3 | Primary LLM provider rate-limits requests | Gateway fallback, deadlines, and degraded experience |
| 4 | A policy changes while the old version remains cached and indexed | Document versioning and atomic activation |
| 5 | A consumer receives the same order event three times | Idempotent side effects |
| 6 | A query becomes slow as seed data grows by two orders of magnitude | Measurement, indexes, and plan analysis |
| 6 | Valkey is unavailable during peak traffic | Graceful degradation and overload protection |
| Capstone | A backward-incompatible field must be introduced without downtime | Contract and migration evolution |

The mentor does not inject more than one major change at a time.

## 12. Incident drills

The student acts as incident commander:

1. Declare impact and current understanding.
2. Preserve evidence and start a timeline.
3. Mitigate customer impact before pursuing the perfect root cause.
4. Communicate status in concise updates.
5. Identify root cause with logs, metrics, traces, and controlled reproduction.
6. Restore service and verify recovery.
7. Write a blameless incident report with corrective actions and owners.

Required drills:

- Last-table race or lock contention
- Model timeout or invalid tool output
- RabbitMQ outage plus outbox backlog
- Poison message and dead letter recovery
- Valkey outage
- Slow database query or pool exhaustion

The mentor scores diagnosis quality, not whether the student guesses the injected
fault immediately.

## 13. Sprint review and retrospective

### Demo

The student demonstrates:

- The user outcome
- A failure or edge case
- Relevant telemetry
- Automated evidence
- Known limitations

Slides cannot replace a running path.

### Acceptance

The mentor records:

- Accepted, conditionally accepted, or rejected
- Blocking gaps
- Deferred debt with an owner and reason
- Evidence added to the ledger
- Readiness for the next sprint

### Retrospective

Use four prompts:

- What accelerated delivery?
- What caused rework or risk?
- Which engineering habit should continue?
- What one process change will be tested next sprint?

Only one or two process actions are carried forward so the retrospective changes
behavior rather than producing a long wish list.

## 14. Mentor scorecard

Each sprint is scored from 1 to 4:

| Dimension | 1 | 2 | 3 | 4 |
|---|---|---|---|---|
| Delivery | Incomplete core path | Happy path only | Accepted outcome | Accepted plus justified improvement |
| Design | Cannot explain choices | Follows pattern without tradeoff | Defends options and constraints | Revises decision using evidence |
| Correctness | Known unsafe behavior | Some edge cases missing | Invariants and failures covered | Strong prevention plus diagnosis |
| Testing | Manual-only evidence | Basic automated tests | Layered tests match risk | Tests expose subtle regressions |
| Operations | No useful telemetry | Logs only | Metrics, traces, runbook | Detects and recovers under drill |
| Communication | Status unclear | Reactive updates | Clear decisions and escalation | Leads review and aligns tradeoffs |

A score of 1 in correctness, security, or data integrity blocks sprint
acceptance regardless of the average.

## 15. Collaboration evidence

Even in a 1v1 setting, the project should preserve:

- Requirement clarifications and accepted scope
- Student-authored design proposals
- Mentor review comments and student responses
- API contract negotiations represented as issue discussions
- Demo notes and Product Owner acceptance
- Incident timelines and post-incident actions
- Retrospective improvements

These artifacts support truthful interview stories about feedback,
disagreement, ownership, and iteration. They must not be presented as
interactions with teammates who did not exist.

## 16. Mentor guardrails

The mentor should:

- Ask for a hypothesis before giving a debugging answer
- Require the student to reproduce failures
- Reject any required tool that violates the
  [Zero-Cost Tooling Policy](free-tooling-policy.md)
- Review the smallest viable design, not encourage speculative infrastructure
- Teach through constraints and evidence
- Separate product acceptance from code quality approval
- Make hidden operational assumptions explicit
- Escalate repeated gaps into a focused remediation task

The mentor should not:

- Write the feature and ask the student to copy it
- Accept architecture vocabulary without runnable evidence
- Add cloud or Kubernetes complexity before local correctness
- Require a credit card, trial credit, or billable cloud resource
- Let LLM-generated code bypass student explanation
- Manufacture production metrics, users, team size, or employment history
- Merge work with unresolved blocking findings

## 17. Completion handoff

At the end of the track, the mentor signs off on:

- Graduation rubric and hard gates
- Final architecture defense
- Project demo and operational drill
- Evidence ledger
- Resume wording and measured claims
- Interview strengths and remaining development areas

The final evaluation process is defined in
[Assessment, Interview, and Resume Evidence](assessment-interview-resume.md).
