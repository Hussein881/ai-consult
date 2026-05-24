# Learning Guide: AI Customer Support Triage Assistant

## What You Need to Learn

Very similar in skills to Project 05 (ticket triage), but this project focuses on the customer-facing dimension: tone, response drafting, and multi-channel input handling. The unique additions are understanding customer communication patterns and building multi-channel intake systems.

---

## Phase 1: NLP for Customer Communication (Week 1-2)

### Core Concepts
- Sentiment analysis: detecting positive, negative, neutral, frustrated tones
- Intent classification in customer text
- Entity extraction: product names, order IDs, account numbers
- Politeness, urgency, and emotional signal detection

### Resources
- **YouTube**: "NLP with Python - Sentiment Analysis" by Tech With Tim — https://www.youtube.com/watch?v=M7SWr5xObkA
- **Docs**: VADER sentiment analysis (lightweight, no GPU) — https://github.com/cjhutto/vaderSentiment
- **YouTube**: "Text Classification with spaCy" — https://www.youtube.com/watch?v=Nsng2n0BbLs
- **Course**: Hugging Face NLP course (free) — https://huggingface.co/learn/nlp-course/chapter1/1

---

## Phase 2: Multi-Category Classification Design (Week 2-3)

### Core Concepts
- Designing a category taxonomy for support tickets
- Hierarchical classification: top-level category → sub-category
- Handling overlapping categories with soft classification
- Building labeled example datasets from historical tickets

### Resources
- **YouTube**: "Text Classification Best Practices" by sentdex — https://www.youtube.com/watch?v=VDg8fCW8LdM
- **Article**: "Building a Support Taxonomy" — https://www.zendesk.com/blog/ticket-taxonomy/
- **Docs**: OpenAI classification best practices — https://platform.openai.com/docs/guides/prompt-engineering

---

## Phase 3: Response Drafting with LLMs (Week 3-4)

### Core Concepts
- Generating empathetic, professional customer responses
- Tone matching: escalated issues vs routine questions
- Linking self-service resources in responses
- Controlling response length and avoiding hallucination

### Resources
- **YouTube**: "GPT-4 Customer Service Bot" by AI Jason — https://www.youtube.com/watch?v=sBhK-2K9bUc
- **Article**: "Writing Better Customer Service Responses with AI" — https://www.intercom.com/blog/how-ai-is-transforming-customer-service/
- **Course**: DeepLearning.AI "Building LLM Apps" — https://learn.deeplearning.ai/

---

## Phase 4: Multi-Channel Input Handling (Week 4-5)

### Core Concepts
- Parsing email: headers, body, attachments with Python `email` library
- Form submission handling via API
- Chat export parsing (JSON, CSV)
- Normalizing diverse input formats into a standard schema

### Resources
- **Docs**: Python `email` library — https://docs.python.org/3/library/email.html
- **YouTube**: "Python Email Parsing" by NeuralNine — https://www.youtube.com/watch?v=JRCJ6RtE3xU
- **Library**: `mailparser` — https://github.com/SpamScope/mail-parser

---

## Phase 5: Ticketing Platform Integration (Week 5-6)

### Core Concepts
- Zendesk API: creating, updating, reading tickets
- Freshdesk API: ticket management
- Webhook-based real-time processing
- OAuth 2.0 for API authentication

### Resources
- **Docs**: Zendesk API reference — https://developer.zendesk.com/api-reference/
- **Docs**: Freshdesk API — https://developers.freshdesk.com/api/
- **YouTube**: "REST APIs for Beginners" by Web Dev Simplified — https://www.youtube.com/watch?v=SLwpqD8n3d0

---

## Phase 6: Dashboard and Analytics (Week 6-7)

### Core Concepts
- Ticket volume by category and channel
- Average response time tracking
- Agent workload distribution
- Exportable CSV and PDF reports

### Resources
- **YouTube**: "Streamlit Dashboard" by Nicholas Renotte — https://www.youtube.com/watch?v=Sb0A9i6d320
- **Docs**: Streamlit tables and charts — https://docs.streamlit.io/library/api-reference/charts

---

## Suggested Learning Sequence

| Week | Focus |
|------|-------|
| 1 | NLP and sentiment for customer text |
| 2 | Multi-category classification design |
| 3 | Response drafting with LLMs |
| 4 | Multi-channel input parsing |
| 5 | Zendesk / Freshdesk integration |
| 6 | Dashboard and reporting |
| 7 | End-to-end testing |

---

## Books and Deeper Resources

- *The Customer Success Economy* by Nick Mehta — business context for support organizations
- Hugging Face text classification tutorial — https://huggingface.co/docs/transformers/tasks/sequence_classification
- Zendesk Developer Blog — https://developerblog.zendesk.com/
