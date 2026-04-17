# Policy Model

This document defines how GenBIOS grants and enforces permissions for:

- changing platform-managed entities such as dashboards, users, database bindings, semantic models, KPI definitions, prompts, and policies
- accessing governed data through table-, column-, and row-level controls

The policy model is **ABAC-first** and **deny-by-default**.

## Core Rule

- Only subjects with the role `workspace_admin` may change platform entities.
- Everyone else is denied platform-entity mutation by default.
- Data access is also deny-by-default and must be explicitly allowed by ABAC policy.
- All permissions are evaluated in the context of:
  - subject
  - action
  - resource
  - environment
  - active database

## Policy Architecture

Authorization is enforced at multiple boundaries:

- `apps/service-api` for request admission and action routing
- `apps/service-sql-execution` for SQL/data access enforcement
- `apps/service-python-execution` for guarded platform-state inspection and mutation
- `apps/service-control` for config, rollout, and policy bundle management
- `apps/service-context` for semantic-model and KPI artifact access

OPA is the decision engine. Services must not make local allow/deny decisions outside the policy model except for hard safety checks such as malformed requests or sandbox violations.

## Subject Attributes

Each request should resolve a `SubjectContext` with attributes such as:

- `subject.id`
- `subject.role`
- `subject.groups`
- `subject.business_unit`
- `subject.authenticated`
- `subject.workspace_id`
- `subject.project_id`
- `subject.database_grants`

The primary role model for v1 is intentionally narrow:

- `workspace_admin`
- everyone else treated as non-admin for platform mutation purposes

This means role-based mutation is simple, while data access remains attribute-driven.

## Resource Classes

Policies should distinguish between two major resource classes.

### Platform Entities

Platform entities are managed objects inside GenBIOS. Examples:

- dashboards
- dashboard folders and shares
- users and user-to-workspace assignments
- database connections and database bindings
- semantic models
- KPI definitions
- glossary entries and synonym mappings
- prompt packs and response policies
- config versions
- adaptation artifacts
- eval suites and rollout metadata

### Data Resources

Data resources are governed warehouse objects and query results. Examples:

- database
- schema
- table
- view
- column
- row-set filtered by policy predicate
- result payload derived from protected data

## Action Model

Policies should evaluate explicit actions instead of generic write/read checks.

### Platform-Entity Actions

- `read_platform_entity`
- `create_dashboard`
- `update_dashboard`
- `share_dashboard`
- `delete_dashboard`
- `manage_user`
- `manage_database_binding`
- `manage_semantic_model`
- `manage_kpi`
- `manage_glossary`
- `manage_prompt_policy`
- `promote_config`
- `rollback_config`
- `execute_python_readonly`
- `execute_python_edit`

### Data Actions

- `read_metadata`
- `select_table`
- `select_column`
- `select_rowset`
- `execute_sql`
- `export_result`

## Default Policy Posture

The system default is:

- deny platform mutation unless `subject.role == "workspace_admin"`
- deny SQL/data access unless the policy explicitly allows the requested database/schema/table/column/row scope
- deny database switching unless the target database is in the subject's allowed database set
- deny `service-python-execution` `edit` mode unless the requested mutation scope is explicitly allowed

## Platform Entity Policy

Platform entity changes must follow these rules.

### Read Access

- Platform entities may be readable by non-admin users only if explicitly allowed by policy.
- Sensitive entities such as users, policy bundles, credentials, and rollout state should default to admin-only read access.

### Mutation Access

- Only `workspace_admin` may create, update, delete, share, promote, or roll back platform-managed entities.
- Mutation requests must include:
  - target entity type
  - target entity id or creation scope
  - active database if the entity is database-scoped
  - requested action
  - requested diff or mutation summary

### Database-Scoped Entities

The following entities are scoped to the active database:

- semantic models
- KPI definitions
- glossary mappings tied to data semantics
- saved queries
- database-specific dashboards where widgets resolve against one database

Policy must reject attempts to:

- mutate a database-scoped entity without an active database
- read or modify an entity from database `A` while the request is authorized only for database `B`
- reuse a semantic model or KPI definition across databases unless an explicit copy/promotion workflow exists

## Data Access Policy

Data access should be enforced in layers.

### Database Selection

- User chooses an active database at login or switches later in-session.
- Policy must verify the subject is allowed to activate the target database.
- Active database becomes part of every downstream policy decision and audit event.

### Table-Level Access

Policy can allow or deny access by:

- workspace
- project
- role
- business unit
- database
- schema
- table tags such as `finance`, `pii`, `restricted`

Typical rule shape:

- allow `execute_sql` only if every referenced table is allowed for the subject in the active database

### Column-Level Access

Column policy is evaluated after table policy and can further restrict access.

Examples:

- allow revenue metrics but deny raw salary columns
- allow customer dimension keys but deny email and phone columns
- allow aggregated access but deny direct identifier columns

If a query references any denied column, execution must be refused or rewritten by a governed planning layer before execution.

### Row-Level Access

Row-level access is enforced through policy predicates derived from subject and environment attributes.

Examples:

- `region in subject.allowed_regions`
- `business_unit == subject.business_unit`
- `country != "sanctioned"`
- `account_owner_id == subject.id`

These predicates must be applied before execution reaches the warehouse result visible to the user.

### Result-Level Access

Even if a query executes, policy may still restrict:

- number of rows returned
- export permission
- drill-through visibility
- whether raw detail rows are allowed versus aggregate-only output

## `service-python-execution` Policy

`apps/service-python-execution` is the controlled path for querying or changing platform state.

### `read-only` Mode

`read-only` mode is used to inspect platform entities and derive proposed changes.

Rules:

- requires `execute_python_readonly`
- may read platform state only within the requested and authorized scope
- must not mutate state
- must run in a sandbox with restricted filesystem, network, and side effects

### `edit` Mode

`edit` mode is the mutation path for platform-managed entities.

Rules:

- requires `execute_python_edit`
- only `workspace_admin` may be granted this action
- request must declare mutation scope before execution
- execution must be constrained to approved services/resources
- mutation must produce structured diff/audit output
- direct secret exfiltration, unrestricted network access, and undeclared side effects are denied

`edit` mode should be used for:

- updating dashboards
- changing semantic models
- editing KPI definitions
- updating glossary artifacts
- modifying database bindings
- promoting or rolling back config versions

It should not be used as a general bypass around service-level authorization.

## Recommended ABAC Shape

OPA inputs should follow this structure:

```json
{
  "subject": {
    "id": "u_123",
    "role": "workspace_admin",
    "groups": ["ops"],
    "business_unit": "finance",
    "workspace_id": "ws_1",
    "project_id": "proj_1",
    "database_grants": ["db_finance", "db_sales"]
  },
  "action": "manage_semantic_model",
  "resource": {
    "type": "semantic_model",
    "id": "sm_42",
    "project_id": "proj_1",
    "database_id": "db_finance",
    "owner_workspace_id": "ws_1",
    "tags": ["semantic"]
  },
  "environment": {
    "deployment_stage": "prod",
    "approval_state": "approved",
    "active_database": "db_finance"
  }
}
```

## Example Decision Rules

### Platform Mutation Allow

Allow when all of the following are true:

- `subject.authenticated == true`
- `subject.role == "workspace_admin"`
- `subject.workspace_id == resource.owner_workspace_id`
- `environment.active_database == resource.database_id` for database-scoped resources
- action is one of the allowed platform mutation actions

Otherwise deny.

### Data Access Allow

Allow data access when all of the following are true:

- subject is authenticated
- active database is in `subject.database_grants`
- requested schema/table is allowed for the subject
- every referenced column is allowed
- row predicate can be derived and enforced
- action is permitted in the current environment

Otherwise deny.

## Audit Requirements

Every allow or deny decision for sensitive operations should be traceable.

Audit records should include:

- subject id
- role
- action
- resource type/id
- active database
- allow/deny result
- policy bundle version
- reason codes
- SQL text or Python mutation summary where applicable
- resulting artifact/version ids

## Operational Guidance

- Keep policy bundles versioned and promotable through `service-control`.
- Test policy changes in isolation before rollout.
- Treat database switching as a privileged context change, not a UI-only preference.
- Never infer mutation permission from read permission.
- Never infer data access from platform-admin status unless policy explicitly allows it.
- Prefer explicit action names over broad `write` grants.

## Initial v1 Policy Recommendation

Start with a narrow, safe model:

- `workspace_admin` can manage platform entities, subject to workspace/project/database scope checks
- non-admin users cannot mutate platform entities
- all data access requires explicit allow rules at database/table/column/row level
- `service-python-execution` `edit` mode is admin-only and fully audited
- semantic models, KPIs, dashboards, and saved queries remain isolated to the active database boundary

This keeps the first implementation understandable, auditable, and consistent with the rest of the GenBIOS architecture.
