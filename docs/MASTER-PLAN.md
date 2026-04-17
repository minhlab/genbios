# Enterprise GenBI Adaptation Platform Plan

## Summary
Build a **Python-first, strongly opinionated monorepo platform** for consultancies and platform teams to launch **enterprise generative BI assistants** quickly, with the **core product centered on domain adaptation**: building eval frameworks, running evals, fine-tuning models, and using AI assistance to adapt assistants to a new industry/domain and to each client's business context after deployment.

The project should ship:
- a **deployable package** that runs as-is with opinionated defaults for the full GenBI application, ready to be customized

The platform should support:
- project/domain-specific branding, policies, connectors, and KPI vocabulary
- multiple workspaces as the primary organizational unit
- AI-assisted adaptation to a new domain, warehouse, KPI vocabulary, and client business context
- eval framework creation, dataset/rubric authoring, and recurring eval execution
- model and fine-tune refinement workflows driven by eval results
- executive KPI Q&A, narrative summaries, anomaly/explanation workflows
- governed warehouse-native SQL access
- governed Python execution for platform state inspection and controlled mutation
- strong auditability, ABAC-based authorization, and regression testing

## Features
### 1. Monorepo shape and package boundaries
Use a monorepo with explicit service boundaries:

- apps/operator-cli: Typer-based operator CLI for deployment, adaptation, evaluation, and rollout workflows
- apps/service-api: FastAPI chat/session API and orchestration entrypoint
- apps/service-context: FastAPI service for domain metadata, KPI glossary, prompt context, schema snapshots
- apps/service-sql-execution: FastAPI service for SQL planning, policy validation, execution, and result shaping
- apps/service-python-execution: FastAPI service providing a guarded Python execution environment for reading and changing platform state. Supports two execution modes: `read-only` for inspection and analysis, and `edit` for mutations. `edit` mode must be authorized and regulated by policy before code can change configs, metadata, dashboards, semantic models, or other managed platform resources
- apps/service-quality: FastAPI + Celery workers for eval runs, scoring jobs, regression jobs, fine-tune jobs, adaptation workflows, and quality reporting
- apps/service-control: FastAPI service for config versions, onboarding, rollout, policy bundles, and authorization management
- apps/frontend: Next.js admin/chat/eval console
- packages/platform-core: shared Pydantic models, DSPy program contracts, authorization model, policy schema, provider interfaces, tools for agentic AI
- packages/platform-connectors: warehouse, auth, audit, metadata
- mlflow: store traces of all conversations for later analysis, registering 
and managing DSPy models

Default runtime should remain **microservice-based**, with separate deployables for:
- API/orchestrator
- context/metadata service
- execution service
- quality/ops service
- control plane

### 2. Authorization model and policy enforcement
Make authorization a first-class architectural concern using **ABAC with Open Policy Agent**.
Support:
- a **shared control plane**
- **project/domain-specific runtime configuration**
- connector credentials and warehouse targets resolved by environment/project
- multiple workspaces, where each workspace encapsulates data access configuration, semantic models, dashboards, and conversation artifacts
- authentication, authorization, and policy enforcement that remain global across all workspaces
- policy packs, prompt profiles, KPI glossaries, branding, eval suites, and adaptation artifacts
- audit streams and usage accounting

Define ABAC enforcement at these layers:
- request access and auth context
- config and secret resolution
- prompt/context assembly
- tool and SQL policy enforcement
- Python execution policy enforcement
- workspace access and membership evaluation
- warehouse/database/schema/table/column permissions
- logging, traces, eval artifacts, and admin visibility

OPA policies should evaluate:
- subject attributes such as user id, groups, role, business unit
- resource attributes such as workspace_id, dataset, schema, report, prompt pack, config version
- action attributes such as read, execute_sql, execute_python_readonly, execute_python_edit, promote_config, launch_eval, start_finetune
- environment attributes such as deployment stage, time window, approval state, request context

Workspace membership and workspace roles must be modeled as subject/resource attributes and evaluated inside global ABAC policies rather than through a separate per-workspace authorization system.

Runtime flow should be authorization-aware end to end:
1. authenticate request and resolve subject context
2. resolve or prompt for active workspace context at login, and allow switching workspaces during an authorized session
3. load project profile, policy pack, workspace context, connector bindings, and model routing
4. assemble domain/business context and adaptation artifacts generated from the deployed system's active configuration for the selected workspace
5. evaluate OPA policy for requested action, resource scope, workspace context, and environment
6. decide answer, clarification, refusal, or escalation path
7. generate constrained SQL, Python, or non-execution response path within the selected workspace boundary
8. validate against OPA decision, platform policy, and SQL/Python execution policy
9. execute using environment-scoped credentials and ABAC-filtered data access; Python execution must run inside `read-only` or policy-approved `edit` mode
10. produce executive-grade response plus optional chart/table payload
11. log traces, evidence, SQL/Python execution records, policy decisions, active workspace, and scores

### 3. Deployable package and adaptation workflow
The system should ship as a deployable package:
- deployable as-is with all core services, reference UI, local orchestration, and opinionated defaults
- no template expansion or project scaffolding required to get a working system
- adaptation performed after deployment through guided workflows in the control plane and operator CLI
- each new client = new deployment + adaptation

The deployed system should support an AI-assisted adaptation workflow that produces:
- project/domain metadata and branding updates
- warehouse and connector configuration updates
- workspace-scoped semantic models, KPI definitions, and business vocabulary drafts
- KPI glossary and business vocabulary drafts
- SQL governance rules and OPA policy inputs
- Python tool and mutation-policy rules for platform state changes
- executive response style and escalation rules
- seeded eval cases for domain-specific metrics and terminology
- domain-ingestion and business-context prompts for AI-assisted adaptation
- fine-tune / optimization dataset definitions and promotion thresholds

### 4. Dashboarding, forecasting, and conversational interface

- The main features of a deployment include: dashboarding functionalities,
automated forecasting and otherwise prediction, and conversational interface
- Dashboards: simple dashboards display KPIs on a grid in the form of line charts, overlaid accompanied by forecast and trend indicators, 
complex dashboards include other types of charts and custom layouts.
- Dashboarding functionalities: creating, managing, and sharing dashboards
- Forecasting: the platform automatically define forecasting models on discovered
KPI metrics and improve the forecasting models in the background
- Prediction: upon request from user, the platform generate a prediction 
pipeline automatically and improve it in the background
- Conversational interface: the conversational interface is the main way
to interact with the platform. Admins and users can use it to do almost everything:
creating and managing dashboards and KPIs, customizing the platform's behaviors,
managing semantic models, etc.
- Users must be able to choose a workspace at login when they have access to more than one workspace, and switch the active workspace later without leaving the application
- A workspace encapsulates data access configurations, semantic models, dashboards, and conversation history/artifacts
- Semantic models, dashboards, saved queries, and conversational context are all scoped to the currently active workspace
- Conversational interface streams answer chunks as they become available, with
support for interruption and resend
- Conversations are multi-turn, can be stored, can be labelled, tagged, and searched over
- User can provide feedback on each individual answer
- User can mention entities (tagging with "at" sign `@`) in the platform to bring them into context, frontend search for the entity as the user types and display a resolved entity if user chooses one
- The platform provides tools that the agentic workflow can use flexibly, for example: run_sql, execute_python, fetch_dashboards, current_date, etc. Tools are regulated by ABAC system.
- `execute_python` must support `read-only` and `edit` modes. `read-only` mode is for inspecting state and deriving artifacts. `edit` mode is for changing managed platform state and must be explicitly governed by policy, with auditable diffs and mutation scope controls

### 6. Quality, guardrails, and refinement workflow
Quality tooling remains a first-class surface, with policy awareness.
Include:
- AI-assisted workflow to bootstrap eval suites, rubrics, and datasets for a new domain after deployment
- AI-assisted workflow to adapt prompts, KPI glossary, semantic context, and business rules for each client after deployment
- offline eval runner with domain/project/workspace-tagged datasets and rubrics
- regression suites at both global-platform and project/domain levels
- synthetic test generation for KPI, glossary, temporal, ambiguity, and cross-workspace context-switch scenarios
- fine-tuning and optimization workflows triggered by eval gaps
- ABAC policy test suites for subject/action/resource/environment combinations
- SQL safety checks for denied tables, row limits, expensive patterns, and prompt injection artifacts
- Python execution safety checks for forbidden imports, network/FS boundaries, mutation scope, and policy-regulated edit operations
- answer quality rubrics for factuality, executive usefulness, traceability, and tone
- review console showing policy decisions, traces, SQL, tool calls, scores, and failure clusters
- comparison workflows for config versions, model/provider variants, fine-tune revisions, prompt revisions, and OPA policy revisions

Refinement workflow should support:
- using eval failures to recommend changes to glossary, prompts, model routing, or fine-tuning data
- changing global defaults without breaking project-specific overrides
- testing policy changes in isolation
- promoting a project profile version once its suites pass thresholds
- rolling out shared improvements selectively across projects

## Public APIs / Interfaces / Types
Define stable contracts early for:
- `ProjectProfile`: branding, persona, supported tasks, output style, escalation rules, connector bindings
- `WorkspaceContext`: workspace identity, bound data access configuration, semantic-model scope, and active-status metadata
- `SubjectContext`: resolved subject identity, auth claims, groups, role, policy scope
- `ResourceContext`: workspace-scoped or global resource attributes for data assets, config artifacts, eval suites, and admin resources
- `PolicyInput`: subject, resource, action, and environment attributes passed to OPA
- `PolicyPack`: OPA policies, SQL/tool/data-access rules, refusal criteria, evidence requirements, override precedence
- `MetadataProvider`: schema, KPI glossary, business definitions, synonyms, metric lineage, all resolved within a selected workspace
- `ExecutionPlan` and `ExecutionResult`: validated SQL plan, active workspace, execution metadata, result payloads, chart suggestions
- `PythonExecutionRequest` and `PythonExecutionResult`: execution mode, sandbox constraints, requested resources, mutation summary, diff/evidence payloads, and policy decision references
- `EvalCase`, `EvalSuite`: project/domain/workspace scope, expected behaviors, rubric results, trace references
- `AdaptationWorkflow`: domain ingestion inputs, generated artifacts, review state, approvals
- `FineTuneJob`: training dataset refs, base model, hyperparameters, eval gates, promoted artifact refs
- `AuditEvent`: subject id, request, active workspace, tool calls, SQL, OPA policy decisions, outputs, evaluator results

These contracts should live in `platform-core` and be shared across all services and runtime-managed domain configurations.

## Test Plan
The initial implementation should be accepted only if it covers:
- deployment of runnable platform from a clean install without scaffolding
- post-deploy activation of adaptation workflow without code changes to core services
- AI-assisted generation of initial domain/context adaptation assets for a new project
- AI-assisted generation of an initial eval suite, dataset stubs, and rubric for a new domain
- local boot of all services and reference UI with default deployable configuration
- users with multiple workspace grants being able to select a workspace at login and switch workspaces in-session
- same user question producing different grounded behavior for different domain KPI definitions and policies
- semantic models, dashboards, and conversation artifacts being isolated to the selected workspace boundary
- ABAC policy decisions preventing unauthorized config, tool, or data access
- unsafe SQL request being refused according to the active OPA/SQL policy with traceable reasoning
- Python read-only execution being allowed only within declared sandbox boundaries, and Python edit execution being refused or allowed according to active policy with traceable reasoning
- provider swap test proving the multi-provider abstraction works while preserving project config boundaries
- regression suite reporting pass/fail per project/domain and at the global-platform level
- eval failures producing actionable recommendations for prompt/context/model updates
- fine-tune or optimization candidate evaluated against project/domain suite before promotion
- policy bundle rollout and rollback without breaking unrelated workflows
- connector failure, warehouse timeout, and misconfigured policy scenarios with visible operator diagnostics

## Assumptions and Defaults
- v1 is optimized for **enterprise domain adaptation**.
- The package is **deployable as-is**, and adaptation happens after deployment through AI-assisted workflows rather than scaffolding.
- Core differentiator is **adaptation workflow quality**: fast eval creation, eval-driven refinement, and AI assistance for domain/client onboarding.
- Authorization is enforced with **OPA-based ABAC** across APIs, tools, data access, and admin operations.
- The primary data plane is still **warehouse-native SQL**, with workspace-scoped metadata layered on top.
- Workspaces are the primary organizational boundary, while authentication and policy evaluation remain global.
- Quality tooling must work at both the **global** and **project** levels so shared changes can be validated safely before rollout.
