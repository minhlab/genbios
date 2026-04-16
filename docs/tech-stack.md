1. Chosen stack

- **Backend services**: Python 3.12
- **API framework**: FastAPI
- **Service runtime**: Uvicorn + Pydantic v2
- **LLM programming/orchestration**: DSPy
- **LLM provider layer**: LiteLLM, with OpenAI as the reference provider
- **Task queue / async jobs**: Celery
- **Message broker / cache**: Redis
- **Primary database**: PostgreSQL
- **ORM / DB access**: SQLAlchemy 2.x + Alembic
- **Warehouse connectivity**: SQLAlchemy dialects plus connector adapters per warehouse
- **Config format**: YAML for tenant/platform config, `.env` for local secrets
- **CLI scaffolder**: Typer
- **Reference UI**: Next.js 15 + TypeScript + React
- **UI component base**: shadcn/ui + Tailwind CSS
- **Auth**: OIDC/OAuth2 via Auth0 or Microsoft Entra ID adapter boundary
- **Observability**: OpenTelemetry + Prometheus + Grafana + structured JSON logs
- **Model registry and evaluation tracking**: MLflow
- **Testing**: Pytest on backend, Playwright for UI, Vitest for frontend unit tests
- **Container/dev env**: Docker Compose for local orchestration
- **Packaging / monorepo tooling**: `uv` for Python dependency management, `pnpm` for frontend workspace tooling

### 2. Backend architecture
Use **FastAPI + Pydantic v2** everywhere for consistent service contracts and generated OpenAPI.

Use **DSPy** inside `service-api` as the core programming model for request handling:
1. resolve tenant and auth context
2. load tenant profile and active config version
3. gather KPI/business/schema context from `service-context`
4. route into the appropriate DSPy program for answer, clarification, refusal, or escalation
5. request SQL planning/execution from `service-execution`
6. assemble executive response and evidence
7. emit audit and telemetry events
8. optionally enqueue evaluation artifacts and scoring jobs to `service-quality`

Use DSPy modules/signatures for:
- intent classification
- clarification decisioning
- SQL planning prompts
- executive narrative generation
- refusal/escalation behaviors
- answer critique and self-check paths where explicitly allowed

Use **Celery + Redis** for:
- background eval runs
- synthetic dataset generation
- offline optimization/tuning jobs
- heavy trace post-processing
- async onboarding validation jobs

Use **PostgreSQL** for:
- tenant registry
- config/version metadata
- eval definitions and run metadata that are operationally useful outside MLflow
- audit/event metadata index
- rollout state and feature flags

### 3. LLM layer, DSPy design, and model management
Choose **LiteLLM** as the provider abstraction and wrap it behind a shared `ModelAdapter` interface in `platform-core`.

Reference providers in v1:
- OpenAI
- Anthropic
- Google Gemini

Use **DSPy** rather than LangGraph. Design the platform around:
- reusable DSPy signatures for core BI tasks
- tenant-configurable DSPy program composition
- optimizer-ready modules for prompt and example tuning
- deterministic non-LLM service boundaries around SQL policy, execution, and tenant resolution

Design rules:
- OpenAI is the default reference path used in examples and starter configs
- provider choice is tenant-configurable
- all model calls must go through the shared adapter for structured outputs, retries, timeouts, routing, and telemetry
- DSPy programs are versioned artifacts tied to tenant config versions
- optimization artifacts and promoted model/program variants are registered in MLflow

Use **MLflow** for:
- model and prompt/program registry
- experiment tracking for DSPy optimization runs
- eval result storage and comparison
- versioned promotion/rollback metadata for approved program variants

MLflow should be the system of record for:
- DSPy program versions
- provider/model combinations under test
- optimization metrics
- eval score histories across tenants and releases

### 4. Multi-tenancy and control plane
Implement multi-tenancy with a **shared control plane + tenant-scoped runtime resolution**.

Use:
- `TenantProfile` and `TenantContext` Pydantic models in `platform-core`
- tenant resolution from auth claims, API key mapping, or explicit tenant slug depending on integration mode
- per-tenant config bundles stored as versioned YAML plus secret references
- per-tenant connector credentials resolved at runtime from a secret provider abstraction
- per-tenant DSPy program configuration, examples, optimization settings, and evaluation baselines

Control plane responsibilities:
- tenant onboarding
- config version registration and activation
- tenant-level feature flags
- model/provider assignment per tenant
- DSPy program version assignment per tenant
- tenant-level audit and usage reporting
- rollout and rollback of prompts, policies, and response profiles

Isolation boundaries must cover:
- request routing
- config loading
- policy enforcement
- warehouse credentials
- eval datasets/results
- MLflow experiment and registry naming/permissions strategy
- logs/traces/admin views

### 5. Data and BI access
Keep **warehouse-native SQL** as the primary data plane.

Support initial warehouse adapters for:
- Snowflake
- BigQuery
- Postgres
- Databricks SQL

Use SQLAlchemy-based adapters plus tenant-specific connection profiles.
Add a metadata layer in `service-context` for:
- KPI glossary
- business term synonyms
- curated schema descriptions
- approved joins and table allowlists
- semantic hints for executive summaries

SQL generation policy:
- DSPy-generated SQL plans must pass tenant and global validation
- denylisted schemas/tables are enforced before execution
- row/scan/timeout guards are enforced centrally
- unsupported or risky requests return clarification or refusal, not best-effort guessing

### 6. Quality, optimization, and evaluation workflow
Quality tooling should center on **MLflow-backed evaluation and DSPy optimization**.

`service-quality` should provide:
- offline eval runner with tenant-tagged datasets and rubrics
- regression suites at both global-platform and per-tenant levels
- synthetic case generation for KPI, glossary, temporal, and ambiguity scenarios
- SQL safety and refusal-behavior checks
- answer quality rubrics for factuality, executive usefulness, traceability, and tone
- DSPy optimization workflows against approved training/eval corpora
- comparison workflows for tenant config versions, DSPy program variants, and provider/model variants

MLflow usage:
- each optimization or eval run logs parameters, datasets, metrics, and artifacts
- promoted DSPy program variants are registered with stage/state metadata
- tenant-safe naming and access boundaries are enforced for experiments and registered artifacts
- rollback means reassigning a previously approved registered variant, not mutating artifacts in place

Refinement workflow:
1. run tenant or global eval suite
2. inspect failures in UI and MLflow
3. adjust config, examples, or DSPy module definitions
4. run optimization or targeted re-evaluation
5. promote a versioned program/config bundle once thresholds pass

### 7. Reference UI
Use **Next.js 15 + TypeScript + Tailwind + shadcn/ui** for the reference UI.

UI surfaces:
- tenant-aware chat experience
- operator/admin dashboard
- eval review console
- trace explorer
- tenant config/version viewer
- rollout status page
- MLflow-linked experiment and model/program comparison views

The UI should consume FastAPI services over JSON/REST in v1. Do not add GraphQL in the first version.

### 8. Developer experience and scaffolding
Use **Typer** for the CLI and render curated templates directly.

CLI should use LLM to create an agentic workflow where data gets imported,
views get created, KPI glossary get written, semantic layer gets created, etc.