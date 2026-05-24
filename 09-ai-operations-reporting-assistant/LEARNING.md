# Learning Guide: AI Operations Reporting Assistant

## What You Need to Learn

This project requires data aggregation across multiple sources, structured summarization with LLMs, and report formatting. The key skill is writing prompts that reliably transform operational data into well-structured narrative reports — a pattern that is broadly applicable to management, finance, and operations reporting.

---

## Phase 1: Data Aggregation and Multi-Source Inputs (Week 1-2)

### Core Concepts
- Reading CSV, JSON, and Excel data with Python and Pandas
- Aggregating data across multiple sources
- Handling missing data and inconsistent formats
- Building a normalized input schema

### Resources
- **YouTube**: "Pandas Tutorial for Beginners" by Corey Schafer — https://www.youtube.com/watch?v=ZyhVh-qRZPA
- **YouTube**: "Python Excel with Pandas" by Tech With Tim — https://www.youtube.com/watch?v=vmEHCJofslg
- **Docs**: Pandas documentation — https://pandas.pydata.org/docs/
- **Library**: `openpyxl` for Excel — https://openpyxl.readthedocs.io/

---

## Phase 2: Jira and Project Management Tool APIs (Week 2-3)

### Core Concepts
- Jira REST API: querying issues, sprints, and projects
- Extracting task status, blockers, and owner data
- Asana and Monday.com API basics
- Normalizing PM tool data into a reporting schema

### Resources
- **Docs**: Jira REST API — https://developer.atlassian.com/cloud/jira/platform/rest/v3/
- **YouTube**: "Jira API Tutorial" by Automation Step by Step — https://www.youtube.com/watch?v=0V3JkC2MFAI
- **Docs**: Asana API — https://developers.asana.com/docs
- **Library**: `jira` Python library — https://jira.readthedocs.io/

---

## Phase 3: LLM-Based Report Summarization (Week 3-4)

### Core Concepts
- Structured summarization: accomplishments, blockers, risks, KPIs
- Map-reduce summarization for large data sets
- Section-by-section generation with consistent tone
- Executive summary generation from detailed report data

### Resources
- **YouTube**: "LangChain Summarization Chains" — https://www.youtube.com/watch?v=TsfLm5iiYb4
- **Docs**: LangChain map-reduce summarization — https://python.langchain.com/docs/tutorials/summarization/
- **Article**: "Prompt Patterns for Structured Reports" — https://www.promptingguide.ai/techniques/ape

---

## Phase 4: Report Templates and Markdown Formatting (Week 4)

### Core Concepts
- Building section templates for weekly/monthly reports
- Jinja2 for section assembly
- Converting Markdown to HTML email
- PDF export for executive distribution

### Resources
- **Docs**: Jinja2 — https://jinja.palletsprojects.com/en/3.1.x/
- **Library**: `markdown` Python library — https://pypi.org/project/Markdown/
- **Library**: `premailer` for HTML email styling — https://github.com/peterbe/premailer

---

## Phase 5: Email and Slack Distribution (Week 5)

### Core Concepts
- Sending emails with Python `smtplib` or SendGrid API
- Formatting reports for email and Slack
- Scheduling with `APScheduler` or cron
- HTML email template design

### Resources
- **YouTube**: "Send Emails with Python" by Corey Schafer — https://www.youtube.com/watch?v=JRCJ6RtE3xU
- **Docs**: SendGrid Python library — https://github.com/sendgrid/sendgrid-python
- **Docs**: Slack Python SDK — https://slack.dev/python-slack-sdk/

---

## Phase 6: Historical Reporting and Trends (Week 6-7)

### Core Concepts
- Storing reports in a database for historical comparison
- Week-over-week and month-over-month trend analysis
- Plotly or Matplotlib charts embedded in reports
- SQLAlchemy for report storage

### Resources
- **YouTube**: "Plotly Python Tutorial" by Charming Data — https://www.youtube.com/watch?v=GGL6U0k8WYA
- **Docs**: Plotly Python — https://plotly.com/python/
- **Docs**: SQLAlchemy — https://docs.sqlalchemy.org/en/20/

---

## Suggested Learning Sequence

| Week | Focus |
|------|-------|
| 1 | Pandas and multi-source data aggregation |
| 2 | Jira and PM tool APIs |
| 3 | LLM summarization chains |
| 4 | Report templating and Markdown |
| 5 | Email and Slack distribution |
| 6 | Historical storage and trends |
| 7 | End-to-end testing |

---

## Books and Deeper Resources

- *Storytelling with Data* by Cole Nussbaumer Knaflic — how to make reports executives actually use
- *The Pyramid Principle* by Barbara Minto — structured writing for business communication
- LangChain summarization docs — https://python.langchain.com/docs/tutorials/summarization/
