# AI Portfolio Technical Strategy

## Objective
Build every portfolio project so it can be delivered to clients and also harvested into reusable technical IP.

## Engineering Model
Use a three-layer architecture in every project:

1. Solution layer: Client-specific workflows, data mapping, and integration rules.
2. Reusable core layer: Shared libraries, utilities, prompts, evaluators, parsers.
3. Interface layer: API, UI, CLI, and MCP tools that call reusable core modules.

Rule: Business logic should live in reusable modules first, and project apps should consume those modules.

## Repository Reuse Blueprint
Suggested shared structure:

- shared/libs: reusable Python libraries and packages.
- shared/scripts: automation scripts for setup, ingestion, evaluation, deployment.
- shared/templates: prompt templates, report templates, proposal/SOW templates.
- shared/mcp: MCP servers and tools built on shared libraries.
- shared/evals: datasets, scoring scripts, benchmark harnesses.
- shared/connectors: adapters for Jira, GitHub, Slack, ServiceNow, ERP, CRM.

## Reuse-First Delivery Workflow
For each project:

1. Define project-specific deliverable.
2. Define extraction target (what reusable asset must be produced).
3. Build MVP against reusable core API, not app-only code.
4. Wrap reusable core in API or MCP endpoint.
5. Add tests and evaluation benchmarks.
6. Publish version and changelog.

## Sprint Exit Criteria
At the end of each sprint, require all of the following:

- One reusable module added or improved in shared/libs.
- One script added in shared/scripts.
- One MCP tool added or updated in shared/mcp.
- Tests added for new reusable logic.
- Usage example and integration note documented.

## MCP Implementation Pattern
Build MCPs as thin adapters over stable library functions.

Example capability groups:

- analyze_test_failures
- detect_flaky_tests
- summarize_incident_logs
- generate_release_risk_report
- retrieve_internal_knowledge
- classify_support_ticket

This keeps maintenance low and allows the same logic to power UI, API, and MCP clients.

## Quality and Reliability Standards
Every reusable asset should define:

- Input and output schema.
- Error behavior and fallback behavior.
- Evaluation metrics and target thresholds.
- Versioning policy.
- Security and privacy constraints.

## Delivery Readiness Checklist
Before client handoff:

- Integration tests pass with client-like sample data.
- Benchmarks recorded (accuracy, latency, confidence behavior).
- Logging, monitoring, and retry behavior enabled.
- Runbook for operations and incident handling delivered.
- Admin docs and user docs delivered.

## Technical KPI Baselines
Track before and after for each project:

- Manual effort time per workflow.
- Cycle time (release, incident triage, support handling, etc.).
- Error rate or false positive rate.
- User adoption and repeat usage.
- SLA impact where relevant.

## 90-Day Build Roadmap

### Month 1
- Stand up shared folder architecture.
- Extract first 2-3 reusable libraries from active projects.
- Ship first MCP server with 3-5 tools.

### Month 2
- Add integration adapters and evaluation harnesses.
- Standardize package versioning and release notes.
- Reuse shared modules across at least 5 project implementations.

### Month 3
- Publish internal asset catalog.
- Add automated regression tests for reusable modules.
- Package top assets into repeatable delivery kits.

## Success Definition
Technical success is achieved when each new client project becomes faster to deliver because existing reusable modules, scripts, and MCP tools reduce custom engineering work while maintaining quality.
