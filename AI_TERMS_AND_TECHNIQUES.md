# AI Terms and Techniques: Practical Consulting Glossary

## Purpose

This reference explains common AI terms and techniques in business-friendly language.

Use it to:

- speak clearly with technical and non-technical stakeholders
- choose the right technique for a client problem
- avoid over-engineering or misusing AI patterns

## Quick Rule

Match technique to business need:

- If the problem is deterministic and rules are stable, use automation first.
- If language understanding, summarization, or reasoning is needed, consider LLM-based approaches.
- If facts must be current and verifiable, combine LLM with retrieval (RAG).
- If workflows require multi-step tool use and decisions, consider agents with guardrails.

## Core Terms

## LLM (Large Language Model)

Definition:

An LLM is a model trained on large text corpora that can generate, summarize, classify, and transform language.

What it is good for:

- summarization
- drafting and rewriting
- classification and extraction from text
- natural language Q&A

Common business use cases:

- support ticket summarization
- policy and procedure drafting
- knowledge-base Q&A
- proposal and report generation

Where it fails:

- strict factual correctness without grounding
- high-stakes decisions without controls
- exact arithmetic or deterministic logic tasks

## Prompt Engineering

Definition:

Designing instructions, examples, constraints, and output format to improve model responses.

What it is good for:

- improving consistency and structure
- reducing ambiguity
- controlling output style and format

Common business use cases:

- standardized email/report drafting
- structured JSON extraction from text
- controlled customer-facing responses

Where it fails:

- cannot fully compensate for missing data
- cannot replace governance and evaluation

## RAG (Retrieval-Augmented Generation)

Definition:

A pattern where a system retrieves relevant data (docs, records, knowledge sources) and provides it to the LLM before generation.

What it is good for:

- grounding outputs in client-specific content
- reducing hallucinations
- enabling source-cited responses

Common business use cases:

- internal policy assistant
- technical documentation assistant
- support knowledge assistant
- legal/compliance document lookup with citations

Where it fails:

- poor source quality or indexing
- weak chunking/retrieval strategy
- stale or conflicting source data

## Embeddings

Definition:

Numerical representations of text (or other content) that capture semantic meaning for similarity search.

What it is good for:

- semantic retrieval
- deduplication support
- clustering related content

Common business use cases:

- finding similar incidents/tickets
- matching support requests to known solutions
- grouping customer feedback themes

Where it fails:

- domain jargon not represented well
- poor chunking and metadata design

## Vector Database

Definition:

A database optimized for storing and searching embeddings using nearest-neighbor similarity.

What it is good for:

- fast semantic search at scale
- retrieval in RAG pipelines

Common business use cases:

- enterprise document search
- related-case retrieval
- recommendation and matching systems

Where it fails:

- weak filtering/metadata strategy
- lack of lifecycle management for stale embeddings

## Re-ranking

Definition:

A second-stage ranking step that improves retrieval quality by reordering candidate results with stronger models or rules.

What it is good for:

- improving precision of top search results
- reducing noisy context passed to LLM

Common business use cases:

- customer support assistant relevance
- compliance or policy lookup accuracy

Where it fails:

- if initial retrieval misses relevant items entirely

## Fine-tuning

Definition:

Training or adapting a base model on task-specific data to improve behavior for a specific domain/task.

What it is good for:

- domain-specific style and terminology
- repetitive structured tasks at scale

Common business use cases:

- specialized classification
- branded writing style adaptation
- domain-specific extraction tasks

Where it fails:

- expensive and unnecessary for many use cases
- cannot solve missing knowledge in source systems

## Agents

Definition:

AI systems that plan and execute multi-step tasks, often using tools (APIs, search, databases, code) and memory.

What it is good for:

- multi-step workflows
- dynamic decision paths
- orchestrating tools and actions

Common business use cases:

- automated triage with tool calls
- incident investigation assistants
- workflow automation across multiple SaaS systems

Where it fails:

- brittle without constraints and fallback logic
- risky for high-impact autonomous actions

## Agentic Workflow (vs Single Prompt)

Definition:

A structured multi-step pipeline where each stage has clear responsibilities (retrieve, classify, reason, act, verify).

What it is good for:

- reliability and observability
- easier debugging than one giant prompt

Common business use cases:

- onboarding and compliance workflows
- multi-source reporting automation
- support escalation routing

Where it fails:

- over-complex design for simple problems

## Tool Calling / Function Calling

Definition:

A model capability to invoke predefined tools/functions and use outputs in the response.

What it is good for:

- connecting LLMs to real systems
- reducing free-text hallucination in transactional tasks

Common business use cases:

- CRM updates
- ticket creation
- account lookup and workflow actions

Where it fails:

- weak tool schemas and poor error handling

## Context Window

Definition:

The amount of text/tokens a model can process in a single request.

What it is good for:

- handling long documents and multi-turn history

Common business use cases:

- contract review drafts
- long-form report synthesis

Where it fails:

- large context can still dilute relevance
- cost and latency can increase significantly

## Hallucination

Definition:

When a model produces plausible but false or unsupported information.

What it is good for:

- none; must be mitigated in production workflows

Mitigation patterns:

- retrieval grounding and citation requirements
- confidence thresholds and human review gates
- output validation rules

## Evaluation (Evals)

Definition:

Systematic testing of AI quality using representative test sets and metrics.

What it is good for:

- measuring performance before release
- preventing regressions after updates

Common business use cases:

- response quality benchmarking
- extraction/classification accuracy checks
- safety and policy compliance checks

Where it fails:

- poor test set design
- metrics not tied to business outcomes

## Guardrails

Definition:

Controls that constrain model behavior and outputs (policies, checks, filters, validators, escalation rules).

What it is good for:

- reducing risk in production
- enforcing compliance and brand constraints

Common business use cases:

- regulated response workflows
- customer-facing assistants with policy controls

Where it fails:

- if controls are not tested against real edge cases

## Human-in-the-Loop (HITL)

Definition:

A workflow pattern where humans review, approve, or correct AI outputs at key risk points.

What it is good for:

- high-impact decisions
- improving trust and adoption

Common business use cases:

- compliance document drafting with reviewer approval
- support actions requiring manager confirmation

Where it fails:

- if review burden is too high and process is not designed well

## Orchestration

Definition:

Coordinating models, tools, retrieval, and business logic across a complete workflow.

What it is good for:

- building reliable end-to-end systems
- observability and troubleshooting

Common business use cases:

- cross-system workflow automation
- AI-assisted business process execution

Where it fails:

- hidden complexity and brittle dependencies

## Latency, Throughput, and Cost

Definition:

Operational metrics for AI systems:

- latency: response time
- throughput: volume handled per unit time
- cost: spend per request/workflow

What it is good for:

- production planning and ROI analysis

Common business use cases:

- deciding model tiers
- balancing quality vs cost in support/reporting workflows

Where it fails:

- optimization done without measuring business impact

## AI Techniques by Problem Type

## 1. Summarization and Drafting

Recommended approach:

- LLM + prompt templates + output schema checks

Typical use cases:

- meeting summaries
- incident/postmortem drafts
- executive report drafting

## 2. Knowledge Q&A

Recommended approach:

- RAG + strong retrieval + citations + evals

Typical use cases:

- internal assistant for docs/policies
- support agent assist

## 3. Classification and Routing

Recommended approach:

- LLM classification or small fine-tuned model + confidence thresholds + fallback rules

Typical use cases:

- ticket triage
- escalation routing
- intent categorization

## 4. Extraction from Unstructured Documents

Recommended approach:

- prompt-guided extraction + schema validation + human review on low confidence

Typical use cases:

- invoice and form processing
- contract clause extraction
- compliance evidence capture

## 5. Multi-step Workflow Automation

Recommended approach:

- agentic workflow with tool calling + explicit state + guardrails

Typical use cases:

- onboarding orchestration
- cross-system operations workflows
- case management support

## 6. Predictive/Forecasting Problems

Recommended approach:

- classical ML or time-series models, optionally with LLM explanation layer

Typical use cases:

- demand forecasting
- churn risk prediction
- anomaly detection

## Technique Selection Checklist (Consulting Use)

Before recommending a technique, confirm:

1. What business KPI is being improved?
2. What is the baseline today?
3. Is data quality sufficient?
4. What error types are acceptable or unacceptable?
5. Where is human approval required?
6. How will we evaluate quality before rollout?
7. What is the fallback when AI confidence is low?
8. How will adoption and ROI be measured?

## Common Mistakes in Client AI Projects

- starting with tooling before defining business outcome
- using agents when deterministic automation would work better
- skipping evals and relying on demo quality
- lacking source grounding for factual tasks
- no governance plan for risk and compliance
- weak change management and user onboarding

## Practical Maturity Path

## Stage 1: Assisted Work

- LLM-assisted drafting, summarization, and internal search

## Stage 2: Controlled Automation

- AI classification/extraction with approvals and guardrails

## Stage 3: Integrated Operations

- AI embedded in core workflows with monitoring and evals

## Stage 4: Scaled AI Advantage

- reusable AI components across departments with clear ROI tracking

## Final Note

Strong AI consulting is not about using the newest model. It is about selecting the right technique for the business problem, delivering safely, and proving measurable outcomes.