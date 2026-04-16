# Multi-Tenant Enterprise GenBI Scaffold Platform Plan

## Summary
Build a **Python-first, strongly opinionated monorepo template platform** for consultancies and platform teams to launch **multi-tenant generative BI assistants** quickly, while preserving enterprise-grade governance, isolation, observability, and evaluation.

The project should ship **both**:
- a **CLI scaffolder** that generates a new GenBI platform instance or a tenant-ready assistant pack, and
- a **shared platform library set** that provides runtime contracts, tenancy-aware governance, connector interfaces, evaluation harnesses, and a reference UI.

The default platform should support:
- multiple tenants in one control plane
- tenant-specific branding, prompts, policies, connectors, and KPI vocabulary
- executive KPI Q&A, narrative summaries, anomaly/explanation workflows
- governed warehouse-native SQL access
- strong auditability, tenant isolation, and regression testing

## Implementation Changes
### 1. Monorepo shape and package boundaries
Use a monorepo with explicit service boundaries:

- apps/scaffold-cli: Typer-based generator for new-platform and new-tenant
- apps/service-api: FastAPI chat/session API and orchestration entrypoint
- apps/service-context: FastAPI service for tenant metadata, KPI glossary, prompt context, schema snapshots
- apps/service-execution: FastAPI service for SQL planning, policy validation, execution, and result shaping
- apps/service-quality: FastAPI + Celery workers for eval runs, scoring jobs, regression jobs, and quality reporting
- apps/service-control-plane: FastAPI service for tenant registry, config versions, onboarding, rollout, and RBAC
- apps/reference-ui: Next.js admin/chat/eval console
- packages/platform-core: shared Pydantic models, DSPy program contracts, tenancy model, policy schema, provider interfaces
- packages/platform-connectors: warehouse, auth, audit, metadata
- templates/platform-enterprise: full platform scaffold
- templates/tenant-pack: tenant-specific scaffold

Default runtime should remain **microservice-based**, with separate deployables for:
- API/orchestrator
- context/metadata service
- execution service
- quality/ops service
- control plane

### 2. Tenancy model and isolation
Make multi-tenancy a first-class architectural concern.
Support:
- a **shared control plane**
- **tenant-specific runtime configuration**
- per-tenant connector credentials and warehouse targets
- per-tenant policy packs, prompt profiles, KPI glossaries, branding, and eval suites
- per-tenant audit streams and usage accounting

Define tenant isolation at these layers:
- request routing and auth context
- config and secret resolution
- prompt/context assembly
- tool and SQL policy enforcement
- warehouse/database/schema permissions
- logging, traces, eval artifacts, and admin visibility

The generated platform should allow one deployment mode:
- **logical multi-tenancy** in one environment for faster delivery

### 3. Scaffolded outputs
The CLI should support two generation flows:
- `new-platform`: creates a full multi-tenant GenBI platform with all services, reference UI, local orchestration, shared contracts, and sample tenants
- `new-tenant`: creates a tenant pack with branding, KPI glossary, policies, connector config, prompt pack, eval suite, and optional extension hooks

Each tenant pack should include:
- tenant metadata and branding
- warehouse and connector configuration schema
- KPI glossary and business vocabulary
- SQL governance rules and tool permissions
- executive response style and escalation rules
- seeded eval cases for tenant-specific metrics and terminology

### 4. Runtime architecture and extension model
Adopt config as the only customization path.
Expose stable public interfaces for:
- `TenantProfile`
- `PolicyPack`
- `MetadataProvider`
- `ModelAdapter`
- `ExecutionPolicyValidator`
- `AnswerPostProcessor`
- `EvalSuite`
- `AuditEventSink`

Runtime flow should be tenant-aware end to end:
1. authenticate request and resolve tenant
2. load tenant profile, policy pack, connector bindings, and model routing
3. assemble tenant-specific KPI/business context
4. classify request intent
5. decide answer, clarification, refusal, or escalation path
6. generate constrained SQL or non-SQL response path
7. validate against tenant and global policy
8. execute using tenant-scoped credentials and data access
9. produce executive-grade response plus optional chart/table payload
10. log tenant-scoped traces, evidence, SQL, policy decisions, and scores

### 5. Control plane and lifecycle management
Add a dedicated control-plane capability for multi-tenant operations:
- tenant registry and lifecycle state
- tenant onboarding workflow
- per-tenant config/version assignment
- rollout controls for prompts, policies, and model settings
- tenant-level feature flags
- tenant-level usage, quality, and error reporting
- tenant admin RBAC and operator RBAC
- safe promotion/rollback of tenant profiles

This should be the main mechanism consultancies use to onboard new clients without cloning whole codebases.

### 6. Quality, guardrails, and refinement workflow
Quality tooling remains a first-class surface, now with tenant awareness.
Include:
- offline eval runner with tenant-tagged datasets and rubrics
- regression suites at both global-platform and per-tenant levels
- synthetic test generation for KPI, glossary, temporal, and ambiguity scenarios per tenant
- SQL safety checks for denied tables, row limits, expensive patterns, and prompt injection artifacts
- answer quality rubrics for factuality, executive usefulness, traceability, and tone
- review console showing tenant-filtered traces, SQL, tool calls, scores, and failure clusters
- comparison workflows for tenant config versions, model/provider variants, and prompt revisions

Refinement workflow should support:
- changing global defaults without breaking tenant overrides
- testing tenant-specific changes in isolation
- promoting a tenant profile version once its suites pass thresholds
- rolling out shared improvements selectively across tenants

## Public APIs / Interfaces / Types
Define stable contracts early for:
- `TenantProfile`: branding, persona, supported tasks, output style, escalation rules, connector bindings
- `TenantContext`: resolved tenant identity, auth claims, policy scope, feature flags, active profile version
- `PolicyPack`: SQL/tool/data-access rules, refusal criteria, evidence requirements, override precedence
- `MetadataProvider`: schema, KPI glossary, business definitions, synonyms, metric lineage
- `ModelAdapter`: prompt invocation, structured output, tool-calling abstraction, fallback handling
- `ExecutionPlan` and `ExecutionResult`: validated SQL plan, execution metadata, result payloads, chart suggestions
- `EvalCase`, `EvalSuite`, and `EvalScore`: tenant scope, expected behaviors, rubric results, trace references
- `AuditEvent`: tenant id, request, tool calls, SQL, policy decisions, outputs, evaluator results

These contracts should live in `platform-core` and be shared across all services and generated tenant packs.

## Test Plan
The initial implementation should be accepted only if it covers:
- CLI generation of a runnable multi-tenant platform from a clean install
- CLI generation of a new tenant pack that can be registered without code changes to core services
- local boot of all services and reference UI with at least two sample tenants
- same user question producing different grounded behavior for different tenant KPI definitions and policies
- tenant-scoped auth and routing preventing cross-tenant config or data access
- unsafe SQL request being refused according to the active tenant policy with traceable reasoning
- provider swap test proving the multi-provider abstraction works while preserving tenant config boundaries
- regression suite reporting pass/fail per tenant and at the global-platform level
- tenant profile rollout and rollback without affecting unrelated tenants
- connector failure, warehouse timeout, and misconfigured tenant scenarios with isolated blast radius and visible operator diagnostics

## Assumptions and Defaults
- v1 is optimized for **multi-tenant delivery**, with one platform serving multiple clients or business units.
- The architecture uses a **shared control plane plus tenant-specific runtime configuration**.
- Multi-tenancy is logical by default, but the same tenant pack can also be deployed in a dedicated environment for stricter clients.
- The runtime remains **microservice-based by default**.
- The primary data plane is still **warehouse-native SQL**, with tenant-specific metadata layered on top.
- Quality tooling must work at both the **global** and **tenant** levels so shared changes can be validated safely before rollout.
