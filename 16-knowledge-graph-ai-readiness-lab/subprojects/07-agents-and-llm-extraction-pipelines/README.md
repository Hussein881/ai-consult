# Subproject 07: Agents and LLM Extraction Pipelines

## Capability Goal

Develop AI agents and LLM pipelines for knowledge extraction that are safe, traceable, and useful in real workflows.

## Learn First

- extraction pipeline design (chunking, prompts, schema output)
- tool calling and workflow orchestration
- evals, guardrails, and human-in-the-loop controls

## Implement

1. Build extraction pipeline from unstructured content to graph-ready output.
2. Add validation and confidence scoring for extracted facts.
3. Build agent workflow for multi-step extraction and enrichment.
4. Add evaluation harness for accuracy, latency, and failure mode tracking.

## Deliverables

- extraction pipeline architecture
- agent workflow definition
- eval report and guardrail policy

## Exit Criteria

- extracted entities/relations meet target quality metrics
- low-confidence outputs are escalated safely
- workflow is observable and debuggable end to end

## Starter Resources

- OpenAI docs: https://platform.openai.com/docs
- Anthropic agent engineering guide: https://www.anthropic.com/engineering/building-effective-agents
- LangGraph docs: https://langchain-ai.github.io/langgraph/
- ReAct paper: https://arxiv.org/pdf/2210.03629.pdf
