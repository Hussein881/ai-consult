# Learning Guide: Internal Knowledge Assistant for Company Documents

## What You Need to Learn

This is the enterprise-grade version of the RAG project from Project 03, with additional requirements around access control, governance, and multi-department deployment. The key additional skills are RBAC (role-based access control), document governance, and enterprise deployment considerations.

---

## Phase 1: RAG Pipeline Review and Enterprise RAG Considerations (Week 1-2)

### Core Concepts
- Review RAG fundamentals (same as Project 03)
- Enterprise RAG requirements: access control, audit logging, data governance
- Handling PII and sensitive documents in RAG systems
- Document lifecycle: versioning, expiration, deprecation

### Resources
- **YouTube**: "Enterprise RAG Patterns" by LangChain — https://www.youtube.com/watch?v=wd7TZ4w1mSw
- **Article**: "RAG for Enterprise" — https://www.databricks.com/blog/rag-enterprise
- **YouTube**: "LlamaIndex Enterprise RAG" — https://www.youtube.com/watch?v=JN-GNzJRKFI
- **Paper**: "REALM: Retrieval-Augmented Language Model Pre-Training" — https://arxiv.org/abs/2002.08909

---

## Phase 2: Document Sources for Enterprise (Week 2-3)

### Core Concepts
- SharePoint document ingestion
- Google Drive API for document access
- Confluence REST API for wiki ingestion
- PDF, DOCX, and PPTX parsing in Python

### Resources
- **Docs**: Microsoft Graph API for SharePoint — https://learn.microsoft.com/en-us/graph/api/resources/sharepoint
- **Docs**: Google Drive API Python quickstart — https://developers.google.com/drive/api/quickstart/python
- **Docs**: Confluence REST API — https://developer.atlassian.com/cloud/confluence/rest/v2/
- **Library**: `python-docx` — https://python-docx.readthedocs.io/en/latest/
- **Library**: `python-pptx` — https://python-pptx.readthedocs.io/

---

## Phase 3: Access Control and Permissions (Week 3-4)

### Core Concepts
- Role-based access control (RBAC) concepts
- Document-level permissions: who can see what
- JWT authentication in FastAPI
- Filtering retrieved documents by user permissions

### Resources
- **YouTube**: "JWT Authentication in FastAPI" by Amigoscode — https://www.youtube.com/watch?v=5GxQ1rLTwaU
- **YouTube**: "RBAC Explained" by IBM Technology — https://www.youtube.com/watch?v=4Uya_I_Oxjk
- **Docs**: FastAPI Security — https://fastapi.tiangolo.com/tutorial/security/
- **Article**: "Implementing RBAC in Python" — https://auth0.com/blog/role-based-access-control-rbac/

---

## Phase 4: Unanswered Question Tracking and Analytics (Week 4-5)

### Core Concepts
- Logging queries that return low-confidence answers
- Identifying documentation gaps from question patterns
- Admin dashboard for content improvement feedback
- Confidence threshold calibration

### Resources
- **Library**: Python `logging` for structured event logging
- **YouTube**: "Building Admin Dashboards with Streamlit" — https://www.youtube.com/watch?v=Sb0A9i6d320
- **Article**: "Monitoring RAG Systems in Production" — https://www.pinecone.io/learn/monitoring-rag/

---

## Phase 5: Slack and Microsoft Teams Integration (Week 5-6)

### Core Concepts
- Slack Bolt SDK for Python: building slash commands and bots
- Microsoft Teams bot framework
- Handling user context in chat integrations
- Rate limiting and error handling in production bots

### Resources
- **Docs**: Slack Bolt for Python — https://slack.dev/bolt-python/
- **YouTube**: "Build a Slack Bot with Python" by NeuralNine — https://www.youtube.com/watch?v=KJ5bFv-IRFM
- **Docs**: Microsoft Teams Bot Framework — https://learn.microsoft.com/en-us/microsoftteams/platform/bots/what-are-bots

---

## Phase 6: Production Deployment (Week 6-7)

### Core Concepts
- Docker containerization
- Environment variables for API key management
- PostgreSQL for metadata and logging storage
- Cron job for scheduled re-indexing

### Resources
- **YouTube**: "Docker for Beginners" by TechWorld with Nana — https://www.youtube.com/watch?v=3c-iBn73dDE
- **Docs**: Docker official documentation — https://docs.docker.com/

---

## Suggested Learning Sequence

| Week | Focus |
|------|-------|
| 1 | Enterprise RAG concepts and governance |
| 2 | SharePoint, Confluence, Google Drive ingestion |
| 3 | JWT auth and RBAC |
| 4 | Unanswered question tracking |
| 5 | Slack/Teams integration |
| 6 | Docker and production deployment |
| 7 | End-to-end integration testing |

---

## Books and Deeper Resources

- *Building Secure APIs* by Corey Ball — API security patterns
- Auth0 RBAC guide — https://auth0.com/docs/manage-users/access-control
- LangChain enterprise patterns — https://python.langchain.com/docs/use_cases/enterprise_rag/
