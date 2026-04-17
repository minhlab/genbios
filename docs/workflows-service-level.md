# Workflows

This document maps user stories from [user-stories.md](/Users/nhiquach/Minh/genbios/_bmad-output/user-stories.md) to service-level workflows.

## Shared Service Roles

- `apps/frontend`: chat/admin UI, renders response, tables, charts, controls
- `apps/service-api`: main orchestration entrypoint, authn/authz check, DSPy program routing
- `apps/service-context`: domain metadata, KPI glossary, business vocabulary, schema hints, semantic context
- `apps/service-sql-execution`: SQL planning validation, OPA-aware execution gating, result shaping
- `apps/service-python-execution`: regulated environment to execute Python code
- `apps/service-quality`: async eval hooks, trace scoring, regression triggers, drift detection
- `apps/service-control`: active config/version lookup, policy bundles, rollout state, adaptation artifacts
- `packages/platform-connectors`: warehouse/auth/audit/metadata adapters
- `OPA`: ABAC decision engine consulted by API and execution paths
- `mlflow`: store traces, store DSPy models

## Story 16: Ask KPI Questions in Natural Language

### Goal
User asks KPI question. System answers without user writing SQL.

### Workflow
1. `frontend` sends chat message to `service-api`.
2. `service-api` authenticates subject, builds `SubjectContext`, asks `OPA` if subject may `read` and `execute_sql` for requested resources.
3. `service-api` loads active project config from `service-control`.
4. `service-api` requests glossary, KPI definitions, schema hints, and semantic context from `service-context`.
5. `service-api` runs DSPy intent + query-planning path.
6. If SQL needed, `service-api` calls `service-execution` with structured plan request.
7. `service-execution` validates SQL against platform rules and OPA-filtered resource scope.
8. `service-execution` runs query through warehouse connector.
9. `service-execution` shapes result payload, returns table/chart suggestions.
10. `service-api` composes natural-language answer and returns to `frontend`.

### Output
- answer text
- evidence summary
- optional SQL trace id
- optional chart/table payload

## Story 17: Grounded Answers with Supporting Evidence

### Goal
Answer must show evidence, not unsupported claims.

### Workflow
1. `frontend` sends question to `service-api`.
2. `service-api` loads active response policy from `service-control`.
3. `service-api` gets metric lineage, glossary, and schema references from `service-context`.
4. `service-api` calls `service-execution` for SQL/result retrieval when data access needed.
5. `service-execution` returns result set plus execution metadata:
   - query id
   - source tables/views
   - row/time limits
   - result timestamp
6. `service-api` runs answer-generation program with evidence requirements enabled.
7. `service-api` attaches citations/evidence blocks to response.

### Output
- grounded answer
- evidence block with source/metric references
- traceable execution metadata

## Story 18: Clarifying Questions for Ambiguous Requests

### Goal
System asks for clarification before unsafe or low-confidence execution.

### Workflow
1. `frontend` sends user request to `service-api`.
2. `service-api` gets domain glossary and ambiguity-sensitive business terms from `service-context`.
3. DSPy classifier in `service-api` scores ambiguity:
   - metric ambiguity
   - time-range ambiguity
   - dimension ambiguity
   - action ambiguity
4. `service-api` checks policy from `service-control` for clarification thresholds.
5. If clarification required, `service-api` returns clarification prompt to `frontend` instead of calling `service-execution`.
6. `frontend` captures user reply and resubmits with prior context.
7. `service-api` resumes normal Q&A workflow with clarified parameters.

### Output
- clarification question
- preserved conversation state
- resumed answer after clarification

## Story 19: Narrative Summaries of KPI Performance

### Goal
Translate numeric KPI data into executive narrative.

### Workflow
1. `frontend` sends summary request to `service-api`.
2. `service-api` loads narrative style config from `service-control`.
3. `service-context` returns KPI definitions, business terminology, and comparison baselines.
4. `service-execution` runs KPI retrieval queries for current value, prior period, target, and trend windows.
5. `service-api` invokes narrative-generation program with:
   - KPI values
   - trend deltas
   - targets
   - business vocabulary
6. `service-api` generates short executive summary plus key drivers.
7. `frontend` renders summary with expandable supporting table/chart.

### Output
- executive summary
- KPI deltas and trend explanation
- optional chart/table payload

## Story 20: Anomaly Explanations

### Goal
Explain unusual KPI movement fast.

### Workflow
1. User asks anomaly question in `frontend`.
2. `service-api` classifies intent as anomaly/explanation.
3. `service-context` returns anomaly thresholds, seasonality hints, KPI dependencies, and business calendars if configured.
4. `service-execution` retrieves recent history, comparison windows, slices, and potential contributing dimensions.
5. `service-api` runs anomaly-explanation program:
   - detect abnormal movement
   - compare versus baseline
   - identify likely contributing segments
6. If needed, `service-api` issues follow-up slice queries via `service-execution`.
7. `service-api` returns explanation with confidence and evidence.

### Output
- anomaly summary
- likely drivers
- supporting breakdowns
- optional follow-up questions

## Story 21: Client-Specific Terminology and Business Context

### Goal
Responses use client vocabulary and context from adaptation workflow.

### Workflow
1. `frontend` sends message to `service-api`.
2. `service-api` loads active project version from `service-control`.
3. `service-context` returns adaptation artifacts:
   - approved glossary
   - synonyms
   - semantic mappings
   - response style profile
4. `service-api` injects client vocabulary into classification, planning, and answer-generation steps.
5. If SQL needed, `service-execution` uses mapped KPI/resource definitions from `service-context`.
6. `service-api` generates answer using client terminology, units, and naming rules.

### Output
- answer in client language
- consistent KPI naming
- business-context-aware explanation

## Story 22: Refusal for Unsafe or Unauthorized Requests

### Goal
Block leaking, dangerous, or unauthorized responses.

### Workflow
1. `frontend` sends request to `service-api`.
2. `service-api` builds `PolicyInput` and queries `OPA`.
3. `service-context` may supply resource tags or sensitivity metadata.
4. If `OPA` denies action, `service-api` returns refusal with safe explanation.
5. If request passes `OPA` but SQL risk exists, `service-api` calls `service-execution`.
6. `service-execution` validates:
   - denied tables/schemas
   - row/scan limits
   - risky patterns
   - prompt injection indicators
7. On validation failure, `service-execution` returns structured refusal reason.
8. `service-api` formats refusal for user and logs policy/validation decision.

### Output
- refusal message
- policy-safe explanation
- audit trace of denial reason

## Story 23: Chart and Table Payloads with Answers

### Goal
Return visual-ready data with conversational answer.

### Workflow
1. `frontend` requests KPI answer.
2. `service-api` determines whether visual payload useful from intent and result shape.
3. `service-context` returns preferred chart conventions if configured.
4. `service-execution` returns normalized result set and chart suggestions:
   - time series
   - categorical comparison
   - stacked/grouped shape
5. `service-api` packages:
   - natural-language answer
   - table payload
   - chart spec payload
6. `frontend` renders answer inline with interactive chart/table components.

### Output
- answer text
- structured table payload
- structured chart spec

## Story 24: Admin Uses Conversation to Manage KPIs, Semantic Context, and Platform Behavior

### Goal
Conversation acts as control surface for admin tasks.

### Workflow
1. Admin sends management request through `frontend`.
2. `service-api` authenticates admin, checks `OPA` for admin action such as:
   - `update_kpi`
   - `update_glossary`
   - `promote_config`
   - `modify_prompt_policy`
3. `service-api` classifies request as operational command, not end-user Q&A.
4. `service-api` calls `service-python-execution` in `read-only` mode to inspect the current platform state needed for the task:
   - active config from `service-control`
   - semantic context and KPI state from `service-context`
   - any related rollout or dependency state
5. `service-python-execution` returns a normalized current-state snapshot to `service-api`.
6. `service-api` produces proposed change set and sends for confirmation.
7. After confirmation, `service-api` checks `OPA` for `execute_python_edit` on the requested mutation scope and calls `service-python-execution` in `edit` mode.
8. `service-python-execution` applies the approved change by invoking the appropriate service boundaries, such as:
   - `service-control` to persist versioned config/policy changes
   - `service-context` to update semantic context or refresh derived metadata
9. `service-python-execution` returns mutation summary, diff, and affected artifact/version references.
10. `frontend` shows diff, rollout status, and follow-up actions.

### Output
- proposed config/context change
- confirmation step
- versioned update + rollout/eval status

## Story 25: Consistent Answers Across Repeated Questions After Adaptation Updates

### Goal
Adaptation changes improve behavior without unstable regressions.

### Workflow
1. Operator updates adaptation artifacts through `service-control`.
2. `service-control` creates new config version, marks previous version as baseline.
3. `service-quality` automatically runs:
   - regression suite
   - consistency checks on repeated query sets
   - terminology adherence checks
4. `service-api` in staging can answer same prompt against old and new config versions.
5. Results, traces, SQL, and answer diffs stored by `service-quality`.
6. Operator reviews comparison in `frontend`.
7. If thresholds pass, `service-control` promotes new config.
8. Live `service-api` loads promoted version for future requests.
9. Audit event recorded for promotion.

### Output
- version diff
- consistency score
- promotion/rollback decision

## Cross-Cutting Notes

- Every conversational workflow starts with authn in `service-api` and authz in `OPA`.
- `service-context` is source of domain vocabulary, KPI semantics, and adaptation artifacts.
- `service-execution` is only service allowed to validate and run warehouse queries.
- `service-quality` runs async. It should not block primary answer path except for explicit gated workflows.
- `service-control` owns active version selection, rollout state, policy bundles, and adaptation history.
- `service-api` sends progress update for each step it takes through SSE so that 
the user can observe the behavior of the agent before the answer is ready
