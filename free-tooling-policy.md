# Zero-Cost Tooling Policy

Status: Binding for the 1v1 Agent Backend Track  
Last verified: September 24, 2026  
Applies to: Required implementation, assessment, and graduation work

## 1. Zero-cost definition

The required course path must be completable with:

- No software, API, cloud, or platform payment
- No credit card or billing account
- No trial credit that later converts to paid usage
- No service that can create an accidental bill
- A documented no-cost fallback for every quota-limited hosted service

The student's existing computer, electricity, and internet connection are
prerequisites rather than course-tool fees. The baseline profile targets a
computer with at least 8 GB RAM and 25 GB available disk space. On an 8 GB
machine, the lightweight profile runs the fake or small local model separately
from the Kubernetes and observability exercises. A 16 GB machine is recommended
when the local model, data services, and observability stack run together.

Open-source local tools are preferred. A hosted free tier may be used only as
an optional adapter; it cannot be required to pass a story, gate, or final
assessment.

Every required component must have a no-payment local path. If a hosted quota,
account, or network service is unavailable, the student must still be able to
build, test, demonstrate, and graduate.

## 2. Required zero-cost stack

| Area | Required path | Cost and fallback rule |
|---|---|---|
| Source control and planning | Git, GitHub CLI, private GitHub Free repository, GitHub Issues and Projects | Private code and one GitHub workspace; merge enforcement uses the free fallback when protected private branches are unavailable |
| Editor | VS Code or VSCodium | No paid extension is required |
| Language and packaging | Python, `venv` plus `pip` or `uv` | Lock dependencies; no commercial package index is required |
| API and persistence code | FastAPI, Pydantic, SQLAlchemy, Alembic | Open-source libraries; local execution is canonical |
| Agent workflow | LangGraph and selected LangChain open-source packages | LangSmith and other paid tracing services are not required |
| Required test provider | Deterministic fake model | All automated acceptance and failure tests run without an external API |
| Interactive local model | Ollama with Qwen3 1.7B, or another mentor-approved no-cost model with compatible license | CPU execution is sufficient for the small-model learning path; provider gateway allows replacement |
| Local embeddings | `sentence-transformers/all-MiniLM-L6-v2` | No paid embedding endpoint is required |
| Durable data and vectors | PostgreSQL and pgvector | Run locally in containers |
| Cache and ephemeral state | Valkey using the Redis protocol and client patterns | Teaches Redis-compatible data structures and failure behavior without a managed cache service |
| Messaging | RabbitMQ | Run the open-source broker locally |
| Artifact storage | Versioned local filesystem adapter | S3-compatible cloud storage is an architecture comparison, not a requirement |
| Containers | Docker Engine and Compose; Docker Desktop only under its Personal or Education terms | Colima or Podman is the no-cost fallback when Docker Desktop terms do not apply |
| CI | GitHub Actions in the private repository | Use included hosted minutes with no payment method, then a free self-hosted runner or local checks after quota exhaustion |
| Image delivery | `kind load docker-image`, `k3d image import`, or public GHCR | A paid or private image registry is not required |
| Staging and Kubernetes | kind or k3d, Kubernetes, and Helm | The required staging environment runs locally |
| Observability | OpenTelemetry, Prometheus, and Grafana OSS | Grafana Cloud and commercial Grafana features are not required |
| Tests and performance | pytest, Testcontainers, and Locust | Hosted test and load-generation services are not required |
| Security checks | Trivy, `pip-audit`, and Bandit | Commercial scanners may not replace the required local checks |
| Secrets | Environment variables for local development; SOPS plus age for encrypted examples | No managed secret service is required |
| Diagrams and evidence | Markdown, Mermaid, and OBS Studio | Mock recordings are created locally; no paid recording platform is required |

The tool manifest created in Session 1 records the exact version, license,
download source, minimum hardware, and fallback for every installed component.

## 3. LLM and embedding policy

### Required path

The application must support:

1. A deterministic fake provider for tests and recorded fixtures
2. A local Ollama provider for the interactive agent demo
3. A provider-neutral gateway so an optional hosted adapter can be enabled
   without changing domain or tool code

The fake provider verifies graph routing, typed tools, confirmation, retries,
and failure behavior. The local model verifies real inference and tool-calling
integration. Neither path requires an API payment.

The baseline local model is Qwen3 1.7B because its published model license is
Apache 2.0 and its size is suitable for CPU-based student machines. The mentor
may replace it only after recording the replacement model's license, download
size, memory need, tool-calling behavior, and fallback.

RAG embeddings use `sentence-transformers/all-MiniLM-L6-v2` locally. Hosted
embedding APIs are not part of the required path.

### Optional hosted adapters

Gemini Developer API or GroqCloud may be used for a comparison demo only when:

- The account currently exposes an unpaid quota
- No billing account or automatic paid upgrade is enabled
- Only synthetic restaurant and customer data is sent
- The student verifies the current model, quota, region, age, and data terms
- Rate-limit or provider failure falls back to local inference or the fake
  provider

Google's current unpaid-service terms allow submitted prompts and responses to
be used for product improvement and explicitly prohibit sending personal,
sensitive, or confidential information. Free quotas and model availability are
not guaranteed. Groq states that exact limits are account-specific and visible
on the account limits page. These services therefore cannot be graduation
dependencies.

## 4. Voice policy

Voice is a late, optional extension and must not introduce a paid dependency.
The no-cost path is:

- Browser microphone capture
- Local `whisper.cpp` speech-to-text
- Browser `SpeechSynthesis` for audio output
- Manual text as the universal fallback

A free hosted speech or live-audio API may be demonstrated with synthetic data,
but the extension must remain usable without it. Voice is not required to pass
the 19-session core track.

## 5. CI and package cost controls

GitHub documents standard Actions runners as free for public repositories.
Private GitHub Free repositories currently include a monthly allowance, but
overage can be billable when a payment method exists.

The course therefore requires:

- Private GitHub Free repository as the default
- Standard runners only; never paid larger runners
- No payment method, so Actions stops after the included allowance, or a
  zero-dollar budget with stop-usage enforcement
- Short artifact retention and no large video or model artifacts in Actions
- A self-hosted runner on the student's machine when hosted minutes are
  exhausted; local `make check` remains the final fallback
- Local image import as the canonical deployment path

GitHub Free private repositories do not include the advanced protected-branch
and required-review features documented for GitHub Pro and Team. The required
free workflow therefore uses the
[GitHub Operating Model](github-operating-model.md) merge-guard command.
Students who already have a no-cost Pro, Team, or education entitlement may
enable native branch protection; purchasing a plan is never required.

Public GitHub Packages are currently free, and GitHub states that Container
Registry storage and bandwidth are currently free. Because that policy may
change, graduation cannot depend on GHCR: kind and k3d must also accept a local
image build.

## 6. Deployment and cloud policy

The required deployment target is a local staging environment:

- Docker Compose for the integration environment
- kind or k3d for the Kubernetes staging environment
- GitHub Actions for build and test
- An ephemeral kind deployment in CI for migration, smoke, rollout, and
  rollback validation

AWS is taught as an architecture and migration exercise only. The student maps
the local components to ECS Fargate, RDS, ElastiCache, Amazon MQ, S3, Secrets
Manager, CloudWatch, ECR, and an Application Load Balancer, then explains
networking, reliability, scaling, and cost controls without provisioning them.

Fargate, ElastiCache, Amazon MQ, and the complete managed stack are usage-billed
and cannot be guaranteed free. AWS trial credits, temporary free tiers, or
school credits do not qualify as the required path. A student may deploy to a
cloud account independently, but that work is optional, cannot affect the
grade, and must use a separate budget and teardown plan.

## 7. Tool approval rules

A new required tool is approved only when:

1. Its required course features cost zero.
2. Its license permits the intended educational use.
3. It does not require a credit card.
4. It has a local or open-source fallback.
5. It works on the baseline hardware profile or has a lightweight course mode.
6. It has a documented uninstall and data-cleanup procedure.
7. It does not send code, prompts, recordings, or PII to a third party without
   an explicit course need and disclosure.

Trial-only services, products that silently roll into paid usage, and tools
whose required features exist only in an enterprise edition are rejected.

## 8. Audit checkpoints

The mentor and student audit the tool manifest:

- Before Session 1: installation, license, account, and hardware readiness
- After Session 5: container, database, cache, and CI usage
- Before Session 6: local model and any optional hosted-model terms
- Before Session 9: embedding model and corpus data policy
- Before Session 17: Actions usage, artifact retention, registry, deployment,
  and zero-dollar budget controls
- At Gate F: confirm that no resume claim depends on a paid or trial-only tool

If a required tool becomes paid or unavailable, the affected story pauses until
the documented fallback replaces it. The student is never required to purchase
access to stay on schedule.

## 9. Official references

These terms are time-sensitive and must be rechecked at each cohort kickoff:

- [Gemini API pricing](https://ai.google.dev/gemini-api/docs/pricing)
- [Gemini API rate limits](https://ai.google.dev/gemini-api/docs/rate-limits)
- [Gemini API terms](https://ai.google.dev/gemini-api/terms)
- [Groq rate limits](https://console.groq.com/docs/rate-limits)
- [GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions)
- [GitHub Packages billing](https://docs.github.com/en/billing/concepts/product-billing/github-packages)
- [GitHub plan allowances](https://docs.github.com/en/billing/reference/product-usage-included)
- [GitHub plans and private-repository features](https://docs.github.com/en/get-started/learning-about-github/githubs-plans)
- [GitHub protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
- [Docker Desktop license](https://docs.docker.com/subscription-billing/desktop-license/)
- [AWS Fargate pricing](https://aws.amazon.com/fargate/pricing/)
- [Amazon ElastiCache pricing](https://aws.amazon.com/elasticache/pricing/)
- [Amazon MQ pricing](https://aws.amazon.com/amazon-mq/pricing/)
- [PostgreSQL license](https://www.postgresql.org/about/licence/)
- [Valkey source and license](https://github.com/valkey-io/valkey)
- [RabbitMQ source](https://github.com/rabbitmq/rabbitmq-server)
- [LangGraph source and license](https://github.com/langchain-ai/langgraph)
- [Ollama source and license](https://github.com/ollama/ollama)
- [Qwen3 1.7B model license](https://huggingface.co/Qwen/Qwen3-1.7B/blob/main/LICENSE)
- [all-MiniLM-L6-v2 model card](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2)
