Security/ops controls are described at a policy level, not an implementation level
The plan mentions per-tenant credentials, audit streams, RBAC, and isolation layers (docs/MASTER-PLAN.md (line 45), docs/MASTER-PLAN.md (line 49), docs/MASTER-PLAN.md (line 100)), but it does not define the operational mechanisms that actually make those claims credible: secret storage and rotation, tenant identity model, admin scoping, audit retention, and noisy-neighbor controls. For an enterprise multi-tenant platform, those are not implementation details; they are part of the product contract.

The test plan validates correctness, but not tenancy pressure/failure behavior
The current acceptance tests cover routing, policy refusal, provider swap, and failures like timeout/misconfiguration (docs/MASTER-PLAN.md (line 147)). What is missing is proof that one tenant cannot degrade or observe another under load: concurrency limits, queue isolation, per-tenant rate limits, trace visibility boundaries, and cost attribution. Those are common failure modes in shared AI/data platforms and should be explicit acceptance criteria.

