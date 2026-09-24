# GitHub Operating Model

Status: Binding for the 1v1 Agent Backend Track  
Repository model: Private repository, trunk-based development  
System of work: GitHub repository, Projects, pull requests, and Actions

## 1. Purpose

The student and mentor use GitHub as the single operational workspace:

- GitHub Projects shows project status
- Issues hold requirements and acceptance criteria
- Pull requests hold implementation discussion and mentor feedback
- Actions shows build, test, migration, and deployment evidence
- Repository `docs/` holds architecture and operations knowledge
- The evidence ledger links these durable artifacts

The required workflow does not use Jira, Linear, a separate wiki, or a second
task tracker. This minimizes tool switching and makes the repository itself the
student's project portfolio.

## 2. Repository and access model

The default course repository is a private repository on GitHub Free:

- The student owns the repository
- The mentor is invited as a collaborator
- The repository contains only synthetic restaurant and customer data
- No billing method is required
- Actions uses included private-repository minutes and stops when the allowance
  is exhausted; a free self-hosted runner or local CI remains the fallback

GitHub currently documents protected branches and required pull-request
reviewers for private repositories as GitHub Pro or Team features. The course
does not require the student to purchase those plans.

Three enforcement profiles are therefore supported:

| Profile | When to use | Merge enforcement |
|---|---|---|
| Private Free, required default | Every student can use this at no cost | Repository policy plus the Session 3 merge-guard command verifies CI, mentor approval, and review resolution before squash merge |
| Private with existing Pro, Team, or education benefit | The entitlement already exists at no course cost | Native branch protection with required review and status checks |
| Public Free | The student explicitly accepts public source code | Native branch protection and standard public-repository Actions runners |

The student is never required to upgrade a GitHub plan. Switching enforcement
profiles does not change the issue, branch, PR, review, or evidence workflow.

## 3. Trunk-based development rules

`main` is the single trunk and must remain releasable.

- No feature, bug fix, migration, or documentation task is developed directly
  on `main`
- One task Issue maps to one short-lived branch and one pull request
- Branches start from current `main`
- Branch names use `<type>/<issue-id>-<short-slug>`, for example
  `feat/RES-012-prevent-double-booking`
- Supported types are `feat`, `fix`, `refactor`, `test`, `docs`, `ops`, and
  `chore`
- A branch should finish within three student workdays and must not span more
  than one sprint without mentor-approved decomposition
- Unrelated changes are split into separate Issues and pull requests
- The student updates the branch from `main` before final review
- Accepted pull requests use squash merge and delete the source branch
- Force pushes and direct pushes to `main` are prohibited by native protection
  when available and by course policy otherwise

Large features are delivered through independently releasable vertical slices,
not through a long-lived integration branch.

## 4. GitHub Projects board

One GitHub Project provides the course Kanban board. Its Status field contains
exactly:

```text
Backlog -> Sprint -> In Review -> Done
```

The board also uses:

- `Iteration`: Sprint 0 through Sprint 7
- `Type`: Epic, Story, Task, Bug, Incident, or Debt
- `Priority`: P0, P1, P2, or P3
- `Estimate`: 1, 2, 3, 5, or 8
- `Gate`: None, A, B, C, D, E, or F

`blocked` is a label, not a fifth column. A blocked item remains in its current
column, records the blocker and owner, and is discussed at the next async update
or session.

### Board transitions

| Transition | Required condition |
|---|---|
| Create in Backlog | Problem, outcome, and owner are known |
| Backlog to Sprint | Session planning accepts scope, acceptance criteria, estimate, dependencies, and evidence |
| Sprint to In Review | Pull request is ready, self-review is complete, and required CI has run |
| In Review to Sprint | Mentor requests a material design or implementation change |
| In Review to Done | Pull request is approved, all required checks pass, review threads are resolved, squash merge completes, documentation is current, and evidence is linked |

Work-in-progress limits are three items in `Sprint` and two items in
`In Review`. The student finishes or unblocks work before pulling more into the
sprint.

## 5. Issue contract

Every implementation or documentation task has one GitHub Issue containing:

- Stable story ID and concise outcome
- Context and user or operator value
- Scope and non-goals
- Acceptance criteria
- API, event, data, authorization, and failure impact
- Test and observability evidence
- Documentation and migration impact
- Dependencies and estimate
- Zero-cost tooling impact

Epics group Issues but are not used as implementation branches. An Issue is
small enough for one reviewable pull request; otherwise it is decomposed before
work begins.

## 6. Pull-request contract

The student opens a draft pull request after the first vertical slice and links
the Issue with `Closes #<number>`.

Every pull request contains:

- Problem and linked Issue
- Approach and important alternatives
- Risk and rollback or forward-fix plan
- API, schema, event, and migration changes
- Test plan and exact commands
- Observability or screenshots when relevant
- Documentation and evidence links
- Author self-review checklist

The pull request is marked ready only after:

- The change is focused on one Issue
- The student has reviewed the full diff
- Local checks pass
- Hosted Actions has run when allowance is available
- Generated files, migrations, and compatibility effects are identified
- No secret, token, real PII, model file, or large recording is present

## 7. Mentor asynchronous review

The mentor reviews ready pull requests between sessions and targets a
two-business-day first response.

The mentor:

1. Confirms the requirement and acceptance criteria.
2. Reviews consistency, authorization, and retry safety.
3. Reviews tests, failure behavior, and observability.
4. Reviews maintainability and performance.
5. Uses `blocking`, `required`, `suggestion`, and `question` labels or prefixes.
6. Approves only after blocking and required comments are resolved.

The student responds to every comment with a code change, evidence, or reasoned
disagreement. Substantive changes requested after approval require another
mentor review.

One active implementation pull request is preferred. This keeps mentor review
focused and prevents several partially reviewed branches from accumulating.

## 8. CI and merge gates

Session 3 creates `.github/workflows/ci.yml` with stable, unique check names:

- `lint`
- `typecheck`
- `unit-test`
- `integration-test`
- `migration-check`
- `container-build`

Additional agent, RAG, security, and compatibility checks are added in later
sessions. A failed or cancelled required check is not green.

### Native enforcement profile

When the repository plan supports protected private branches, or the repository
is public, protect `main` with:

- Pull request required before merge
- One mentor approval
- Dismiss stale approval after new reviewable commits
- Required status checks
- Required conversation resolution
- Linear history
- Force push and branch deletion disabled
- Administrator bypass disabled when the plan supports it

### Private GitHub Free fallback

When native private-branch protection is unavailable, Session 3 adds a
repository script or command such as `make merge-pr PR=<number>`, implemented
with GitHub CLI and the GitHub API. It must:

1. Query the pull request and all Actions checks with GitHub CLI.
2. Reject draft, closed, conflicting, or out-of-date pull requests.
3. Reject any failed, pending, or missing required check.
4. Verify mentor approval and no outstanding change request.
5. Require review-thread resolution evidence.
6. Perform a squash merge and delete the source branch.
7. Print the PR and Actions URLs for the evidence ledger.

The student agrees not to use the GitHub merge button or push directly to
`main` outside this command. This fallback is process-enforced rather than
platform-enforced, and the student must explain that limitation in interviews.

## 9. CI/CD progression

### Session 3: CI

Every pull request runs linting, type checks, tests, migration validation, and a
container build. Gate A requires a deliberately failing change to be rejected
and the corrected change to pass.

### Session 17: CD

The required zero-cost CD path:

1. Builds and tags an immutable container image.
2. Creates an ephemeral kind cluster in Actions or uses the local fallback.
3. Runs the migration Job.
4. Deploys API, relay, and workers.
5. Runs readiness and synthetic smoke tests.
6. Demonstrates rolling update and rollback.
7. Saves only small text evidence with short retention.

AWS staging is optional only when the institution or student already has
no-cost credits and a strict budget and teardown plan. It cannot affect the
grade. The required Session 17 artifact remains local or ephemeral Kubernetes
staging plus an AWS architecture mapping.

## 10. Repository documentation model

All durable project knowledge is versioned and reviewed with the code:

```text
docs/
  architecture/
  adr/
  api/
  events/
  evaluations/
  performance/
  runbooks/
  incidents/
  evidence/
    ledger.md
```

Conventions:

- ADRs use `docs/adr/ADR-NNNN-short-title.md`
- Runbooks use `docs/runbooks/<system>-<failure>.md`
- Incident reports use `docs/incidents/INC-YYYYMMDD-short-title.md`
- Evaluation and benchmark reports record source revision, environment, data
  version, method, result, and caveat
- The evidence ledger links Issues, pull requests, Actions runs, ADRs,
  dashboards, reports, and demos
- The repository Wiki is not used
- Large recordings and model files are never committed

Documentation changes follow the same Issue, branch, PR, review, and merge
rules as code.

## 11. Session operating rhythm

### Session 1

- Create the private repository and invite the mentor
- Create the GitHub Project and four Status columns
- Seed epics and the first sprint backlog
- Add Issue and PR templates
- Record the selected enforcement profile
- Create the `docs/` structure and evidence ledger

### Every session

- Start from the Project board, not a separate status document
- Review what moved, what is blocked, and what evidence is missing
- Inspect the active Issue, pull request, and Actions run
- End by updating the board and assigning the next accepted Issue

### Every gate

- Link passing Actions runs
- Link merged pull requests and current documentation
- Record mentor acceptance in the Issue
- Add the evidence to `docs/evidence/ledger.md`

## 12. Mapping to company terminology

The student should explain the lightweight workflow in familiar company terms:

| GitHub course artifact | Jira or company equivalent |
|---|---|
| GitHub Project | Team Kanban or Scrum board |
| Backlog column | Product backlog |
| Sprint column plus Iteration field | Sprint backlog |
| Issue | Jira story, task, bug, or incident |
| Labels and custom fields | Issue type, component, priority, and status fields |
| Pull request | Code review and implementation evidence |
| Actions run | CI/CD pipeline result |
| Milestone or Gate field | Release milestone or quality gate |
| `docs/` | Version-controlled engineering knowledge base |

The student does not claim Jira experience from this mapping. The expected
interview claim is that they operated a lightweight Scrum/Kanban workflow in
GitHub and understand how its concepts map to Jira.

## 13. Definition of operating-model compliance

The operating model passes when:

1. Every completed task links one Issue, one focused pull request, passing CI,
   current documentation, and evidence.
2. `main` contains no unreviewed direct feature commits.
3. The Project board matches repository reality.
4. No item moves to Done before merge and evidence completion.
5. Mentor feedback and student responses are preserved in pull requests.
6. The selected merge-enforcement profile is documented and demonstrated.
7. CI and CD stop or fall back locally before creating a charge.
8. A new reviewer can reconstruct the project history from GitHub alone.

## 14. Official GitHub references

- [GitHub plans](https://docs.github.com/en/get-started/learning-about-github/githubs-plans)
- [GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions)
- [About protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
- [About pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)
- [About Projects](https://docs.github.com/en/issues/planning-and-tracking-with-projects/learning-about-projects/about-projects)
