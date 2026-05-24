# Learning Guide: AI Support Escalation Ticket Triage Assistant

## What You Need to Learn

This project requires understanding multi-class text classification, urgency and priority modeling, routing logic, and integration with ticketing platforms. The core skill is writing classification prompts and building systems that produce reliable, consistent decisions on text inputs — a pattern that applies across many enterprise AI use cases.

---

## Phase 1: Text Classification Fundamentals (Week 1-2)

### Core Concepts
- Multi-class text classification: assigning a label from a fixed set
- Zero-shot vs few-shot classification with LLMs
- Urgency and priority signal detection in text
- Confidence scoring for classification outputs

### Resources
- **YouTube**: "Text Classification with Python" by Sentdex — https://www.youtube.com/watch?v=VDg8fCW8LdM
- **YouTube**: "Zero-Shot Classification with Transformers" by Hugging Face — https://www.youtube.com/watch?v=OxO1mF7cPMI
- **Article**: "Zero-Shot Classification Guide" — https://huggingface.co/tasks/zero-shot-classification
- **Course**: DeepLearning.AI NLP Specialization (free audit) — https://www.coursera.org/specializations/natural-language-processing

---

## Phase 2: OpenAI API for Classification (Week 2-3)

### Core Concepts
- Using GPT-4o for structured ticket classification
- JSON mode output for category, urgency, routing
- Few-shot examples for consistent categorization
- Handling ambiguous tickets and low-confidence cases

### Resources
- **YouTube**: "OpenAI API Structured Outputs" — https://www.youtube.com/watch?v=aRFT0PbF08k
- **Docs**: OpenAI Structured Outputs — https://platform.openai.com/docs/guides/structured-outputs
- **Cookbook**: Text classification examples — https://github.com/openai/openai-cookbook/blob/main/examples/Classification_using_embeddings.ipynb

---

## Phase 3: Routing Logic and Rules Engine (Week 3-4)

### Core Concepts
- Building a rules engine on top of LLM classification
- Priority matrix: category × urgency → routing destination
- Escalation thresholds and SLA timer logic
- Handling overlapping categories

### Resources
- **Article**: "Building a Rules Engine in Python" — https://martinfowler.com/bliki/RulesEngine.html
- **Library**: `business-rules` Python library — https://github.com/venmo/business-rules
- **YouTube**: "Decision Tables and Rules Engines" by IBM Technology — https://www.youtube.com/watch?v=oMa1gNi23zU

---

## Phase 4: Draft Response Generation (Week 4-5)

### Core Concepts
- Template-based response generation conditioned on ticket category
- Tone calibration: professional, empathetic, urgent
- Few-shot response examples per category
- Response length and content controls

### Resources
- **YouTube**: "GPT-4 for Customer Service" by AI Jason — https://www.youtube.com/watch?v=sBhK-2K9bUc
- **Docs**: OpenAI best practices for tone — https://platform.openai.com/docs/guides/prompt-engineering
- **Article**: "AI in Customer Service" — https://www.intercom.com/blog/how-ai-is-transforming-customer-service/

---

## Phase 5: Zendesk and Ticketing Platform Integration (Week 5-6)

### Core Concepts
- Zendesk REST API: reading tickets, updating fields, adding comments
- Webhook configuration for real-time ticket processing
- OAuth authentication for CRM integrations
- Salesforce Service Cloud basics

### Resources
- **Docs**: Zendesk API documentation — https://developer.zendesk.com/api-reference/
- **YouTube**: "Zendesk API Tutorial" — https://www.youtube.com/watch?v=eRl5LFy8kKA
- **Docs**: Salesforce REST API Basics — https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/

---

## Phase 6: Streamlit Dashboard and Reporting (Week 6-7)

### Core Concepts
- Ticket volume and category distribution visualization
- SLA compliance tracking
- Agent performance metrics
- Export and reporting

### Resources
- **YouTube**: "Streamlit Data Dashboard" by Data Professor — https://www.youtube.com/watch?v=ZZ4B0QUHuNc
- **Docs**: Streamlit charts — https://docs.streamlit.io/library/api-reference/charts

---

## Suggested Learning Sequence

| Week | Focus |
|------|-------|
| 1 | Text classification concepts |
| 2 | OpenAI structured classification |
| 3 | Routing logic and rules engine |
| 4 | Draft response generation |
| 5 | Zendesk API integration |
| 6 | Streamlit dashboard |
| 7 | End-to-end testing and edge cases |

---

## Books and Deeper Resources

- *Speech and Language Processing* by Jurafsky and Martin (free online) — https://web.stanford.edu/~jurafsky/slp3/
- *Designing Machine Learning Systems* by Chip Huyen — real-world ML deployment patterns
- Hugging Face NLP course (free) — https://huggingface.co/learn/nlp-course/chapter1/1
