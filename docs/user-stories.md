# User Stories

## Deployment and Setup

1. As a platform operator, I want to deploy GenBIOS as-is from a clean install so that I can start from a working baseline without custom code generation.
2. As a platform operator, I want the package to boot all core services with opinionated defaults so that I can validate the system before domain adaptation.
3. As a platform operator, I want a guided operator CLI so that I can manage deployment, adaptation, evaluation, and rollout tasks from one interface.
4. As a platform operator, I want environment configuration validated before startup so that deployment failures are caught early.
5. As a platform operator, I want health checks across API, context, execution, quality, and control-plane services so that I can verify deployment readiness quickly.

## AI-Assisted Domain and Client Adaptation

6. As an operator, I want to launch an AI-assisted adaptation workflow after deployment so that I can tailor the application to a specific domain without editing code.
7. As an operator, I want to upload client documentation, KPI definitions, and business terminology so that the system can infer domain context automatically.
8. As an operator, I want the system to draft a KPI glossary from source materials so that I can review and approve business terms faster.
9. As an operator, I want the system to suggest synonyms, metric relationships, and business definitions so that user questions are interpreted in client language.
10. As an operator, I want the system to propose prompt/context updates for a new client so that responses reflect client-specific expectations and tone.
11. As an operator, I want the system to recommend connector and warehouse configuration updates during adaptation so that data access aligns with the new environment.
12. As an operator, I want the system to produce SQL governance suggestions from discovered schemas so that I can review safe data access rules.
13. As an operator, I want the system to propose OPA policy inputs during adaptation so that authorization stays aligned with the client’s access model.
14. As an operator, I want every adaptation artifact to remain reviewable and editable so that AI assistance does not become opaque or uncontrollable.
15. As an operator, I want to replay adaptation workflows for a new client using prior patterns so that onboarding becomes faster over time.

## Conversational BI Experience

16. As a business user, I want to ask KPI questions in natural language so that I can get answers without writing SQL.
17. As a business user, I want answers grounded in warehouse data with supporting evidence so that I can trust the output.
18. As a business user, I want the system to ask clarifying questions when my request is ambiguous so that I get the right answer instead of a guess.
19. As a business user, I want narrative summaries of KPI performance so that I can understand trends without reading raw tables.
20. As a business user, I want anomaly explanations for unusual metric behavior so that I can investigate business changes faster.
21. As a business user, I want responses to follow client-specific terminology and business context so that output matches how my organization talks.
22. As a business user, I want refusal behavior for unsafe or unauthorized requests so that the system does not leak or invent data.
23. As a business user, I want chart and table payloads returned with answers so that I can inspect results visually.
24. As an admin, I want to use the conversational interface to manage KPIs, semantic context, and platform behavior so that operational tasks happen in one place.
25. As a business user, I want consistent answers across repeated questions after adaptation updates so that behavior feels stable and trustworthy.

## Dashboards and Forecasting

26. As a business user, I want simple KPI dashboards with trend and forecast indicators so that I can monitor performance at a glance.
27. As a business user, I want complex dashboards with multiple chart types and custom layouts so that I can track different business views in one place.
28. As a business user, I want to create and manage dashboards through guided workflows so that dashboard setup is accessible to non-technical users.
29. As a business user, I want to share dashboards with authorized teammates so that teams can collaborate around the same metrics.
30. As a business user, I want the platform to automatically define forecasting models on discovered KPIs so that I get predictive insight without manual model setup.
31. As a business user, I want forecasting models to improve in the background as more data and evaluations become available so that predictions stay useful.
32. As a business user, I want to request a new prediction pipeline conversationally so that advanced analysis is easier to initiate.
33. As an operator, I want forecast and prediction outputs evaluated against actuals over time so that model quality can be measured and improved.

## Evaluation Framework

34. As an operator, I want the system to bootstrap an eval framework for a new domain so that quality measurement starts immediately after adaptation.
35. As an operator, I want AI assistance to generate initial eval datasets and rubrics so that I can avoid building evals from scratch.
36. As an operator, I want evals to cover KPI interpretation, glossary usage, temporal reasoning, ambiguity handling, and refusal behavior so that domain adaptation is tested comprehensively.
37. As an operator, I want regression suites at both global and project levels so that shared changes can be validated without breaking client-specific behavior.
38. As an operator, I want to run targeted evals on prompt, policy, and model changes so that I can isolate the effect of each change.
39. As an operator, I want evaluation results linked to traces, SQL, and artifacts so that I can diagnose failures efficiently.
40. As an operator, I want the system to recommend prompt, glossary, routing, or fine-tune changes from eval failures so that iteration is guided by evidence.

## Fine-Tuning and Optimization

41. As an operator, I want the platform to suggest when fine-tuning or optimization is justified so that I do not incur unnecessary model cost or complexity.
42. As an operator, I want fine-tune datasets and training inputs versioned and traceable so that promoted variants have clear lineage.
43. As an operator, I want fine-tune candidates evaluated against project-specific gates before promotion so that weaker variants do not reach production.
44. As an operator, I want rollback to a prior approved model/program variant so that bad promotions can be reversed quickly.
45. As an operator, I want MLflow-backed comparisons across prompts, providers, and fine-tuned variants so that I can choose improvements with evidence.

## Authorization, Governance, and Operations

46. As a security administrator, I want OPA ABAC policies enforced across APIs, tools, and data access so that permissions reflect subject, resource, action, and environment attributes.
47. As a security administrator, I want policy simulation and testing before rollout so that authorization changes do not break production workflows.
48. As a data steward, I want row, column, table, and action-level restrictions enforced during SQL execution so that governed access is maintained end to end.
49. As an operator, I want audit logs for requests, SQL, policy decisions, eval runs, and promotions so that platform behavior is traceable.
50. As a platform owner, I want rollout and rollback of config, policy, adaptation, and model changes so that I can improve the system safely over time.
