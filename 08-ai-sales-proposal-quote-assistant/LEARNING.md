# Learning Guide: AI Sales Proposal and Quote Assistant

## What You Need to Learn

This project blends document generation, template systems, pricing logic, and commercial writing with LLMs. The key skill here is building reliable structured-output generation systems — prompts that produce proposal sections, SOW text, and pricing tables in a consistent format every time. This pattern is directly reusable for any document generation use case.

---

## Phase 1: Commercial Document Structures (Week 1)

### Core Concepts
- Anatomy of a sales proposal: executive summary, problem statement, solution, scope, timeline, pricing, next steps
- Statement of Work (SOW) structure
- Pricing tiers and packaging models
- Common B2B proposal mistakes and how to avoid them

### Resources
- **YouTube**: "How to Write a Business Proposal" by HubSpot — https://www.youtube.com/watch?v=5JBMKUWqtE8
- **Article**: "SOW Template Guide" — https://www.smartsheet.com/statement-of-work-template
- **Article**: "Proposal Best Practices" — https://www.proposify.com/blog/business-proposal

---

## Phase 2: Jinja2 Templating for Document Generation (Week 2)

### Core Concepts
- Jinja2 template syntax: variables, loops, conditionals
- Building modular proposal templates with includes
- Dynamic section generation based on client inputs
- Whitespace control and output formatting

### Resources
- **YouTube**: "Jinja2 Tutorial" by Corey Schafer — https://www.youtube.com/watch?v=bxhXQG1qJPM
- **Docs**: Jinja2 — https://jinja.palletsprojects.com/en/3.1.x/
- **Article**: "Jinja2 Templates for Document Generation" — https://realpython.com/primer-on-jinja-templating/

---

## Phase 3: LLM-Based Section Drafting (Week 2-3)

### Core Concepts
- Generating proposal sections from client input and templates
- Few-shot examples for consistent commercial tone
- Controlling verbosity and avoiding filler text
- Pricing suggestion logic based on scope inputs

### Resources
- **Course**: DeepLearning.AI ChatGPT Prompt Engineering — https://learn.deeplearning.ai/chatgpt-prompt-eng
- **YouTube**: "LangChain Document Generation" by Sam Witteveen — https://www.youtube.com/watch?v=kmbS6FDQh7c
- **Cookbook**: OpenAI text generation examples — https://github.com/openai/openai-cookbook

---

## Phase 4: python-docx and PDF Export (Week 3-4)

### Core Concepts
- Creating DOCX files from Python with styled headings, tables, and lists
- Converting Markdown or HTML to PDF with `WeasyPrint` or `reportlab`
- Preserving formatting across export formats
- Adding cover pages and branding

### Resources
- **YouTube**: "Create Word Docs with Python" by NeuralNine — https://www.youtube.com/watch?v=G08UPMSzf1M
- **Docs**: python-docx — https://python-docx.readthedocs.io/en/latest/
- **Docs**: WeasyPrint — https://doc.courtbouillon.org/weasyprint/stable/
- **Docs**: reportlab — https://www.reportlab.com/docs/reportlab-userguide.pdf

---

## Phase 5: Proposal Management and History (Week 4-5)

### Core Concepts
- Storing proposals in a database (PostgreSQL or SQLite)
- Version history and comparison
- Linking proposals to clients and win/loss outcomes
- Simple search and retrieval interface

### Resources
- **YouTube**: "PostgreSQL Tutorial" by Amigoscode — https://www.youtube.com/watch?v=qw--VYLpxG4
- **Docs**: SQLAlchemy — https://docs.sqlalchemy.org/en/20/
- **YouTube**: "FastAPI with PostgreSQL" — https://www.youtube.com/watch?v=sSIVoR52TOM

---

## Phase 6: Streamlit Interface (Week 5-6)

### Core Concepts
- Client input form for proposal generation
- Preview and edit interface
- Export buttons for DOCX and PDF
- Proposal history browser

### Resources
- **Docs**: Streamlit forms — https://docs.streamlit.io/library/api-reference/control-flow/st.form
- **YouTube**: "Streamlit Forms Tutorial" by Data Professor — https://www.youtube.com/watch?v=ZZ4B0QUHuNc

---

## Suggested Learning Sequence

| Week | Focus |
|------|-------|
| 1 | Proposal structure and commercial writing |
| 2 | Jinja2 templating |
| 3 | LLM section drafting |
| 4 | DOCX and PDF export |
| 5 | Database storage and history |
| 6 | Streamlit UI |
| 7 | Integration and user testing |

---

## Books and Deeper Resources

- *Writing Business Bids and Proposals for Dummies* by Neil Cobb — commercial writing context
- *SPIN Selling* by Neil Rackham — helps you understand what sales teams need from proposals
- Proposify Blog — https://www.proposify.com/blog
