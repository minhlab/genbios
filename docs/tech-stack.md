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
- **Config format**: YAML for platform/domain/project config, `.env` for local secrets
- **Operator CLI**: Typer
- **Reference UI**: Next.js 15 + TypeScript + React
- **UI component base**: shadcn/ui + Tailwind CSS
- **Auth**: OIDC/OAuth2 via Auth0 or Microsoft Entra ID adapter boundary
- **Authorization**: Open Policy Agent (OPA) with ABAC policies
- **Observability**: OpenTelemetry + Prometheus + Grafana + structured JSON logs
- **Model registry and evaluation tracking**: MLflow
- **Testing**: Pytest on backend, Playwright for UI, Vitest for frontend unit tests
- **Container/dev env**: Docker Compose for local orchestration
- **Packaging / monorepo tooling**: `uv` for Python dependency management, `pnpm` for frontend workspace tooling

### 2. Backend architecture
Use **FastAPI + Pydantic v2** everywhere for consistent service contracts and generated OpenAPI.

Use **DSPy** inside `service-api` as the core programming model for request handling:
1. resolve subject and auth context
2. load active project/domain profile and config version
3. evaluate OPA ABAC policy for requested action and data scope
4. gather KPI/business/schema context from `service-context`
5. route into the appropriate DSPy program for answer, clarification, refusal, or escalation
6. request SQL planning/execution from `service-execution`
7. assemble executive response and evidence
8. emit audit and telemetry events
9. optionally enqueue evaluation artifacts and scoring jobs to `service-quality`

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
- deployment/runtime metadata
- config/version metadata
- eval definitions and run metadata that are operationally useful outside MLflow
- audit/event metadata index
- rollout state and feature flags
- subject, resource, and policy metadata backing ABAC decisions

### 3. LLM layer, DSPy design, and model management
Choose **LiteLLM** as the provider abstraction and wrap it behind a shared `ModelAdapter` interface in `platform-core`.

Reference providers in v1:
- OpenAI
- Anthropic
- Google Gemini

Use **DSPy** rather than LangGraph. Design the platform around:
- reusable DSPy signatures for core BI tasks
- project-configurable DSPy program composition
- optimizer-ready modules for prompt and example tuning
- deterministic non-LLM service boundaries around OPA policy enforcement, SQL policy, execution, and subject resolution

Design rules:
- OpenAI is the default reference path used in examples and starter configs
- provider choice is config-driven
- all model calls must go through the shared adapter for structured outputs, retries, timeouts, routing, and telemetry
- DSPy programs are versioned artifacts tied to config versions
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
- eval score histories across domains, projects, and releases

### 4. Authorization and control plane
Implement authorization with **Open Policy Agent + ABAC** enforced at every service boundary.

Use:
- `SubjectContext`, `ResourceContext`, and `PolicyInput` Pydantic models in `platform-core`
- subject resolution from auth claims, API keys, group membership, and environment context
- project/domain config bundles stored as versioned YAML plus secret references
- connector credentials resolved at runtime from a secret provider abstraction
- project-specific DSPy program configuration, examples, optimization settings, and evaluation baselines
- OPA policy bundles defining ABAC rules over subject, action, resource, and environment attributes

Control plane responsibilities:
- domain/project onboarding after deployment
- config version registration and activation
- feature flags
- model/provider assignment per project
- DSPy program version assignment per project
- policy bundle registration, simulation, and rollout
- audit and usage reporting
- rollout and rollback of prompts, policies, and response profiles

Authorization boundaries must cover:
- request access
- config loading
- policy enforcement
- warehouse credentials
- eval datasets/results
- MLflow experiment and registry permissions strategy
- logs/traces/admin views
- tool access and administrative actions

Deployment model:
- package ships deployable as-is with default services wired together
- first deployment should boot usable baseline application without code generation step
- domain/client adaptation happens through control plane workflows after deployment

### 5. Data and BI access
Keep **warehouse-native SQL** as the primary data plane.

Support initial warehouse adapters for:
- Snowflake
- BigQuery
- Postgres
- Databricks SQL

Use SQLAlchemy-based adapters plus environment-specific connection profiles.
Add a metadata layer in `service-context` for:
- KPI glossary
- business term synonyms
- curated schema descriptions
- approved joins and table allowlists
- semantic hints for executive summaries

SQL generation policy:
- DSPy-generated SQL plans must pass OPA authorization and SQL validation
- denylisted schemas/tables are enforced before execution
- row/scan/timeout guards are enforced centrally
- row/column/table access is filtered by ABAC rules
- unsupported or risky requests return clarification or refusal, not best-effort guessing

### 6. Quality, optimization, and evaluation workflow
Quality tooling should center on **MLflow-backed evaluation and DSPy optimization**.

`service-quality` should provide:
- AI-assisted workflow to analyze deployed system behavior and propose adaptation changes
- offline eval runner with domain/project-tagged datasets and rubrics
- regression suites at both global-platform and project/domain levels
- synthetic case generation for KPI, glossary, temporal, and ambiguity scenarios
- ABAC policy eval cases covering user roles, resource tags, and environment constraints
- SQL safety and refusal-behavior checks
- answer quality rubrics for factuality, executive usefulness, traceability, and tone
- DSPy optimization workflows against approved training/eval corpora
- comparison workflows for config versions, DSPy program variants, OPA policy revisions, and provider/model variants

MLflow usage:
- each optimization or eval run logs parameters, datasets, metrics, and artifacts
- promoted DSPy program variants are registered with stage/state metadata
- project-safe naming and access boundaries are enforced for experiments and registered artifacts
- rollback means reassigning a previously approved registered variant, not mutating artifacts in place

Refinement workflow:
1. run domain/project or global eval suite
2. inspect failures in UI and MLflow
3. adjust config, examples, DSPy module definitions, or OPA policy
4. run optimization or targeted re-evaluation
5. promote a versioned program/config bundle once thresholds pass

### 7. Reference UI
Use **Next.js 15 + TypeScript + Tailwind + shadcn/ui** for the reference UI.

UI surfaces:
- domain-aware chat experience
- operator/admin dashboard
- eval review console
- trace explorer
- config/version viewer
- policy explorer and ABAC decision simulator
- rollout status page
- MLflow-linked experiment and model/program comparison views

The UI should consume FastAPI services over JSON/REST in v1. Do not add GraphQL in the first version.

### 8. Developer experience and operations
Use **Typer** for operator and developer workflows.

CLI and control-plane workflows should use LLM to guide post-deploy adaptation:
- import domain/client source material
- generate KPI glossary drafts
- propose semantic layer and context updates
- generate eval suites and rubrics
- recommend prompt/model/fine-tune changes based on eval outcomes
