# Project Context

## Project Name
GenBIOS

## One-Line Summary
GenBIOS is a platform to create real-world GenBI systems that adapt to your specific
business challenge. Included: workflow to adapt and finetune GenBI application, 
tools for running enterprise generative BI assistants with strong governance, authorization, observability, and evaluation. Core product focus: deploy package as-is, then use AI assistance to build eval framework, run evals, finetune model, and adapt fast to new domain and each client's business context.
The platform must support multiple workspaces as the primary organizational unit, with an active workspace chosen at login or switched later during a session.

## Problem
Consultancies and internal platform teams need repeatable way to launch domain-specific BI assistants fast. Hardest part is not basic deployment; it is adapting assistant to new domain, building eval framework, refining prompts/models, and grounding behavior in each client's KPI vocabulary and business context after deployment. Existing approach makes onboarding slow, weakens governance, and makes regression testing across projects hard.

## Target Users
- Consultancies building GenBIOS assistants for many clients
- Internal platform teams serving many business units
- Admins and operators managing branding, policies, prompts, connectors, and rollouts
- Operators reviewing traces, quality, failures, and rollout/policy state
- Business users asking KPI, summary, anomaly, and explanation questions

## Core Value
- Fast deployment of GenBIOS as-is
- AI-assisted adaptation to new domain and client business context
- Built-in workflow to create eval suites, datasets, and rubrics
- Eval-driven refinement for prompts, routing, and model/fine-tune choices
- Fast post-deploy onboarding of new project/domain without core service edits
- Multiple workspace support with explicit active-workspace context per session
- ABAC authorization across config, auth, policy, data access, traces, and evals
- Governed warehouse-native SQL for trusted BI answers
- Guarded Python execution for platform state inspection and controlled mutation
- Built-in evaluation, rollback, and regression workflow

## Primary Deliverables
- Deployable package with all core services and UI wired by default
- Operator CLI for adaptation, evaluation, and rollout workflows
- Shared platform libraries with stable contracts
- Multi-service runtime for chat/orchestration, context, execution, quality, and control plane
- Guarded Python execution service with `read-only` and policy-regulated `edit` modes
- Reference UI for chat, admin, trace review, and eval workflows
- Adaptation workflow for post-deploy domain/client onboarding
- Fine-tune and optimization workflow gated by eval results

## Product Scope
### In Scope
- Shared control plane with project/domain-scoped runtime resolution
- Multiple workspaces per deployment, with user-selectable active workspace context
- Project/domain-specific branding, prompts, policies, KPI glossary, connectors, and eval suites
- Deployable system that runs as-is before any client/domain adaptation
- AI-assisted generation of domain adaptation assets, KPI glossary drafts, and client-context packs
- Eval framework generation, dataset/rubric authoring, and recurring eval execution
- Fine-tuning / optimization loop driven by eval results
- Executive KPI Q&A, narrative summaries, anomaly/explanation workflows
- Warehouse-native SQL execution with central policy validation
- Guarded Python execution for reading platform state and performing approved mutations
- OPA-based ABAC authorization across APIs, tools, and data access
- Versioned configs, policy bundles, rollouts, and rollback
- Project/domain-aware offline evals, regression suites, and optimization workflows
- Workspace-scoped semantic models, dashboards, conversation history/artifacts, and saved queries

### Out of Scope For V1
- GraphQL API layer
- LangGraph-based orchestration
- Best-effort unsafe SQL fallback behavior

## Architecture Shape
### Monorepo Apps
- `apps/operator-cli`
- `apps/service-api`
- `apps/service-context`
- `apps/service-sql-execution`
- `apps/service-python-execution`
- `apps/service-quality`
- `apps/service-control`
- `apps/frontend`

### Shared Packages
- `packages/platform-core`
- `packages/platform-connectors`

## Runtime Flow
1. Authenticate request and resolve subject context.
2. Resolve or ask for the active workspace at login, and allow authorized switching later in the session.
3. Load project/domain profile, policies, workspace context, connectors, and model routing.
4. Gather domain KPI, schema, business context, and adaptation artifacts from active deployed configuration for the selected workspace.
5. Classify request intent and detect missing domain/client or workspace context.
6. Choose answer, clarification, refusal, or escalation path.
7. Generate constrained SQL, Python, or non-execution response plan within the selected workspace boundary.
8. Validate against OPA ABAC policy plus SQL/Python execution policy.
9. Execute with scoped credentials and ABAC-filtered access; Python runs in `read-only` or policy-approved `edit` mode.
10. Return executive response with evidence and optional chart/table payload.
11. Emit audit, telemetry, SQL/Python execution, scoring artifacts, and active-workspace context.

## Technical Direction
### Backend
- Python 3.12
- FastAPI + Uvicorn + Pydantic v2
- SQLAlchemy 2.x + Alembic
- PostgreSQL
- Celery + Redis
- Open Policy Agent for ABAC authorization

### LLM Layer
- DSPy for orchestration/program composition
- LiteLLM behind shared `ModelAdapter`
- OpenAI as default reference provider
- Anthropic and Gemini as supported v1 providers
- Fine-tuning / optimization candidates promoted only after eval gates pass

### Data Plane
- Warehouse-native SQL
- Initial adapters: Snowflake, BigQuery, Postgres, Databricks SQL
- Metadata/context service for KPI glossary, schema hints, approved joins, synonyms, and semantic guidance
- Workspaces encapsulate data access configurations such as database connections, plus semantic models, dashboards, and conversation artifacts
- Guarded Python execution service for platform-state access, with sandboxed `read-only` inspection and policy-regulated `edit` mutations

### UI
- Next.js 15
- TypeScript + React
- Tailwind CSS + shadcn/ui

### Observability / Quality
- OpenTelemetry
- Prometheus + Grafana
- Structured JSON logs
- MLflow for eval and DSPy optimization tracking
- Eval-driven adaptation and fine-tune review workflow

### Tooling
- Typer operator CLI
- `uv` for Python
- `pnpm` for frontend
- Docker Compose for local orchestration
- Pytest, Playwright, Vitest

## Key Domain Models
- `ProjectProfile`
- `WorkspaceContext`
- `SubjectContext`
- `ResourceContext`
- `PolicyInput`
- `PolicyPack`
- `MetadataProvider`
- `ModelAdapter`
- `ExecutionPlan`
- `ExecutionResult`
- `PythonExecutionRequest`
- `PythonExecutionResult`
- `EvalCase`
- `EvalSuite`
- `EvalScore`
- `AdaptationWorkflow`
- `FineTuneJob`
- `AuditEvent`

## Authorization Rules
- Shared control plane, project/domain-scoped runtime behavior
- Config bundles in versioned YAML
- Secret references and connector credentials resolved by environment/project
- Active workspace selection must be authorized and carried through routing, metadata, query planning, semantic lookup, and audit records
- OPA enforces ABAC using subject, resource, action, and environment attributes
- Workspace membership and workspace roles are modeled as attributes evaluated inside global policy, not in a separate per-workspace auth system
- Authorization must cover routing, config loading, tool access, SQL/data access, Python execution, eval artifacts, logs, traces, and admin views
- Config is only customization path in runtime
- Adaptation artifacts, eval datasets, and fine-tune lineage stay project/domain-scoped
- Adaptation is applied to deployed system through versioned config changes, not template regeneration
- Python execution has two modes: `read-only` for inspecting managed state and `edit` for mutating managed state. `edit` mode requires explicit policy approval, scope controls, and auditable mutation records

## Acceptance Signals
- Deploy runnable platform from clean install
- Start post-deploy adaptation workflow without core service code changes
- Generate initial domain/client adaptation assets with AI assistance
- Generate initial eval suite, dataset stubs, and scoring rubric for new domain/project
- Boot local stack with default package configuration
- User can choose among authorized workspaces at login and switch active workspace later
- Same prompt yields different grounded behavior by domain glossary/policy
- Semantic models, dashboards, and conversation artifacts stay isolated within the selected workspace
- OPA ABAC blocks unauthorized config, tool, and data access
- Unsafe SQL refused with traceable reasoning
- Python `read-only` mode stays sandboxed and Python `edit` mode is allowed only when active policy permits the requested mutation scope
- Provider swap works without breaking project config boundaries
- Eval results reported at project/domain and global levels
- Eval failures recommend prompt/context/model or fine-tune changes
- Fine-tune or optimization candidate gated by project/domain eval suite before promotion
- Policy rollout/rollback isolated and traceable
- Connector failure or timeout remains visible with useful diagnostics

## Constraints
- Strongly opinionated monorepo, not generic framework
- Microservice-based default runtime
- Config-first customization only
- Package must be deployable as-is
- Warehouse-native SQL remains primary data plane
- Multi-workspace support must preserve strict workspace boundaries around configuration, semantic modeling, dashboards, and conversation artifacts
- Python execution must be sandboxed and policy-governed; mutation is never implicit
- Shared changes must be testable without breaking project/domain overrides
- Core product must optimize for adaptation speed and eval quality, not only initial deployment speed

## Risks / Open Edges
- V1 scope large: CLI, control plane, 5 services, UI, eval system
- Adaptation workflow can become too manual or too opaque if AI-generated artifacts not reviewable
- Fine-tune workflow needs clear gating, lineage, rollback, and cost controls
- Need strict config schema/versioning to keep post-deploy adaptation changes compatible
- Need clear secret-management and admin-boundary design to make enterprise isolation credible
- Need project-safe MLflow naming/access strategy
- Need OPA policy testing, explainability, and rollout safety
- Need strong sandboxing and diff/audit guarantees for Python edit mode so platform-state mutations remain reviewable and reversible
- Need clear UX and authorization rules for workspace selection/switching so cross-workspace leakage does not occur

## Immediate Build Focus
1. Define adaptation, eval, and fine-tune contracts in `platform-core`.
2. Add OPA ABAC contracts and policy evaluation path in `platform-core`.
3. Make package deployable as-is with default working configuration.
4. Build AI-assisted workflow to create domain/client context updates and eval suites after deployment.
5. Stand up local runtime and prove context adaptation changes behavior.
6. Prove policy validation and warehouse execution boundaries.
7. Add eval/regression/fine-tune loop and basic operator UI.
