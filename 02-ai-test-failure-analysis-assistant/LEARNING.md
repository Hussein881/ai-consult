# Learning Guide: AI Test Failure Analysis Assistant

## What You Need to Learn

This project requires understanding CI/CD workflows, test result formats, statistical pattern detection (for flakiness), and LLM-based classification. The unique skill here is building a system that reasons about pass/fail patterns over time, not just single-run analysis.

---

## Phase 1: Understanding CI/CD Pipelines and Test Frameworks (Week 1-2)

### Core Concepts
- How CI/CD pipelines work: GitHub Actions, Jenkins, GitLab CI
- JUnit XML report format and schema
- Test result structure: test suites, test cases, pass/fail/skip/error
- Understanding flaky tests vs regression failures

### Resources
- **YouTube**: "GitHub Actions Tutorial for Beginners" by TechWorld with Nana — https://www.youtube.com/watch?v=R8_veQiYBjI
- **YouTube**: "What is CI/CD?" by IBM Technology — https://www.youtube.com/watch?v=42UP1fxi2SY
- **Docs**: JUnit 5 User Guide — https://junit.org/junit5/docs/current/user-guide/
- **Article**: "Understanding JUnit XML format" — https://llg.cubic.org/docs/junit/

---

## Phase 2: Python for Test Data Processing (Week 2-3)

### Core Concepts
- Parsing XML with Python `xml.etree.ElementTree`
- Working with JSON and CSV test outputs
- Pandas for test data aggregation and historical comparison
- Calculating failure rates and flakiness scores

### Resources
- **YouTube**: "Python XML Parsing Tutorial" by Corey Schafer — https://www.youtube.com/watch?v=j0GD8VaECNM
- **YouTube**: "Pandas Tutorial for Beginners" by Corey Schafer — https://www.youtube.com/watch?v=ZyhVh-qRZPA
- **Docs**: Python `xml.etree.ElementTree` — https://docs.python.org/3/library/xml.etree.elementtree.html
- **Docs**: Pandas documentation — https://pandas.pydata.org/docs/

---

## Phase 3: OpenAI API and Classification Prompts (Week 3-4)

### Core Concepts
- LLM-based failure classification (regression, flake, environment, expected)
- Chain-of-thought prompting for reasoning about test context
- Structured output with JSON mode
- Confidence scoring in LLM outputs

### Resources
- **YouTube**: "OpenAI JSON Mode Tutorial" — https://www.youtube.com/watch?v=aRFT0PbF08k
- **Course**: DeepLearning.AI "Building Systems with ChatGPT API" — https://learn.deeplearning.ai/building-systems-with-chatgpt
- **Docs**: OpenAI Function Calling — https://platform.openai.com/docs/guides/function-calling
- **Cookbook**: OpenAI classification examples — https://github.com/openai/openai-cookbook

---

## Phase 4: Statistical Flakiness Detection (Week 4-5)

### Core Concepts
- What makes a test flaky: environment sensitivity, timing, data dependency
- Historical pass/fail rate calculation
- Threshold-based flakiness classification
- Trend detection over multiple runs

### Resources
- **Article**: "Flaky Tests: Detection and Prevention" (Google Testing Blog) — https://testing.googleblog.com/2020/12/test-flakiness-one-of-main-challenges.html
- **Article**: "Detecting Flaky Tests" by Martin Fowler — https://martinfowler.com/articles/nonDeterminism.html
- **YouTube**: "Statistics for Data Science" by StatQuest — https://www.youtube.com/watch?v=qBigTkBLU6g

---

## Phase 5: FastAPI and Database Storage (Week 5-6)

### Core Concepts
- FastAPI endpoints for test result ingestion
- SQLite or PostgreSQL for storing historical test runs
- SQLAlchemy ORM basics
- Query patterns for historical comparison

### Resources
- **YouTube**: "FastAPI with SQLAlchemy" by Amigoscode — https://www.youtube.com/watch?v=sSIVoR52TOM
- **Docs**: SQLAlchemy ORM — https://docs.sqlalchemy.org/en/20/orm/
- **Docs**: FastAPI SQL databases — https://fastapi.tiangolo.com/tutorial/sql-databases/

---

## Phase 6: Report Generation and UI (Week 6-7)

### Core Concepts
- Generating Markdown and HTML reports from Python
- Streamlit for release-readiness dashboard
- Color-coded confidence display
- Export to CSV and PDF

### Resources
- **YouTube**: "Streamlit Dashboard Tutorial" by Nicholas Renotte — https://www.youtube.com/watch?v=Sb0A9i6d320
- **Docs**: Streamlit charts and tables — https://docs.streamlit.io/library/api-reference/charts
- **Library**: `reportlab` for PDF generation — https://www.reportlab.com/

---

## Suggested Learning Sequence

| Week | Focus |
|------|-------|
| 1 | CI/CD and JUnit format |
| 2 | Python XML and Pandas |
| 3 | OpenAI classification prompts |
| 4 | Flakiness detection algorithms |
| 5 | FastAPI and database storage |
| 6 | Streamlit report dashboard |
| 7 | Integration, edge cases, and testing |

---

## Key GitHub Repos to Study

- https://github.com/pytest-dev/pytest
- https://github.com/langchain-ai/langchain
- https://github.com/tiangolo/fastapi
