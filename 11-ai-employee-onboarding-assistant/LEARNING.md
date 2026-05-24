# Learning Guide: AI Employee Onboarding Assistant

## What You Need to Learn

This is a specialized RAG application focused on HR and onboarding content. The unique additions are role-based content paths, checklist generation, and progress tracking. The skills are directly reusable for any HR-tech or people-operations consulting engagement.

---

## Phase 1: RAG for HR Documents (Week 1-2)

### Core Concepts
- Applying RAG to HR-specific document types: handbooks, policies, role guides
- Chunking strategies for structured policy documents
- Handling FAQ-style and procedural content differently
- Metadata tagging: department, role level, location, topic

### Resources
- **YouTube**: "RAG From Scratch" by LangChain — https://www.youtube.com/playlist?list=PLfaIDFEXuae2LXbO1_PKyVJiQ23ZztA0x
- **Docs**: LangChain metadata filtering — https://python.langchain.com/docs/how_to/vectorstore_retriever/
- **Article**: "HR Document AI Use Cases" — https://www.shrm.org/topics-tools/tools/toolkits/using-ai-hr
- **Course**: DeepLearning.AI RAG course — https://learn.deeplearning.ai/

---

## Phase 2: Role-Based Content Routing (Week 2-3)

### Core Concepts
- Defining role profiles: engineering, sales, HR, operations, etc.
- Metadata-filtered retrieval: return only docs relevant to the user's role
- Dynamic prompt construction based on user profile
- Content access rules: not all employees see all documents

### Resources
- **Docs**: LangChain metadata filters — https://python.langchain.com/docs/how_to/vectorstore_retriever/
- **Docs**: ChromaDB where filters — https://docs.trychroma.com/reference/py-client#where-filters
- **Article**: "Role-Based Document Retrieval" — https://www.pinecone.io/learn/metadata-filtering/

---

## Phase 3: Checklist Generation (Week 3)

### Core Concepts
- Generating structured checklists from onboarding documents with LLMs
- Role-specific task lists: week 1, week 2, 30-60-90 day plans
- Pydantic models for checklist item structure
- Rendering checklists as interactive UI components

### Resources
- **YouTube**: "Structured Output with OpenAI" — https://www.youtube.com/watch?v=aRFT0PbF08k
- **Docs**: OpenAI structured outputs — https://platform.openai.com/docs/guides/structured-outputs
- **Docs**: Streamlit checkboxes — https://docs.streamlit.io/library/api-reference/widgets/st.checkbox

---

## Phase 4: Progress Tracking and Database Design (Week 4)

### Core Concepts
- User session and onboarding progress storage in SQLite or PostgreSQL
- Tracking which checklist items are completed per employee
- Manager view: progress across multiple new hires
- Identifying common stall points from aggregate data

### Resources
- **YouTube**: "SQLite with Python" by Corey Schafer — https://www.youtube.com/watch?v=pd-0G0MigUA
- **Docs**: SQLAlchemy models — https://docs.sqlalchemy.org/en/20/orm/
- **YouTube**: "FastAPI with Database" by Amigoscode — https://www.youtube.com/watch?v=sSIVoR52TOM

---

## Phase 5: Slack Integration for Onboarding Notifications (Week 5)

### Core Concepts
- Sending onboarding reminders and milestone messages via Slack
- Slash commands for new-hire self-service queries
- Scheduling daily or weekly nudges with APScheduler
- Direct message vs channel message patterns

### Resources
- **Docs**: Slack Bolt for Python — https://slack.dev/bolt-python/
- **YouTube**: "Slack Bot with Python" by NeuralNine — https://www.youtube.com/watch?v=KJ5bFv-IRFM
- **Library**: APScheduler — https://apscheduler.readthedocs.io/en/3.x/

---

## Phase 6: Analytics Dashboard for HR (Week 6)

### Core Concepts
- Unanswered question tracking and documentation gap identification
- Average time to complete checklist by role
- Content usage analytics
- Manager-facing reporting view

### Resources
- **YouTube**: "Streamlit Dashboard" by Nicholas Renotte — https://www.youtube.com/watch?v=Sb0A9i6d320
- **Docs**: Streamlit metrics — https://docs.streamlit.io/library/api-reference/data/st.metric

---

## Suggested Learning Sequence

| Week | Focus |
|------|-------|
| 1 | RAG for HR documents |
| 2 | Role-based content routing |
| 3 | Checklist generation |
| 4 | Progress tracking database |
| 5 | Slack integration |
| 6 | Analytics dashboard |
| 7 | End-to-end integration |

---

## Books and Deeper Resources

- *Work Rules!* by Laszlo Bock — Google's HR playbook, context on onboarding
- SHRM (Society for HR Management) resources — https://www.shrm.org/
- Workday developer docs (for integration context) — https://developer.workday.com/
