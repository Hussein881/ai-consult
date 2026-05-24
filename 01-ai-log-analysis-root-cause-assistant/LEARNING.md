# Learning Guide: AI Log Analysis and Root Cause Assistant

## What You Need to Learn

This project sits at the intersection of log engineering, NLP, and LLM-based reasoning. You need to understand how logs are structured, how to parse and clean them, and how to use LLMs to summarize, cluster, and generate root-cause hypotheses.

---

## Phase 1: Python Foundations for Log Processing (Week 1-2)

### Core Concepts
- Python file I/O, string manipulation, and regex
- Parsing structured logs (JSON, syslog format, CSV)
- Regular expressions for log pattern matching
- Working with timestamps and datetime libraries

### Resources
- **YouTube**: "Python Regular Expressions Tutorial" by Corey Schafer — https://www.youtube.com/watch?v=K8L6KVGG-7o
- **YouTube**: "Python File Handling" by Tech With Tim — https://www.youtube.com/watch?v=Uh2ebFW8OYM
- **Docs**: Python `re` module — https://docs.python.org/3/library/re.html
- **Docs**: Python `datetime` module — https://docs.python.org/3/library/datetime.html

---

## Phase 2: Log Formats and Parsing (Week 2-3)

### Core Concepts
- Common log formats: syslog, Apache access logs, JSON structured logs, application logs
- Python logging module and log level conventions
- Noise filtering and normalization
- Grouping related log entries by time window

### Resources
- **YouTube**: "Python Logging Tutorial" by Corey Schafer — https://www.youtube.com/watch?v=-ARI4Bi-Id4
- **YouTube**: "Log Analysis Tutorial" by NetworkChuck — https://www.youtube.com/watch?v=1lxKZ0ZJYHo
- **Docs**: Python `logging` module — https://docs.python.org/3/library/logging.html
- **Article**: "A Guide to Structured Logging" — https://betterstack.com/community/guides/logging/structured-logging/

---

## Phase 3: OpenAI API and Prompt Engineering (Week 3-4)

### Core Concepts
- OpenAI Chat Completions API
- Prompt engineering: structured prompts, role prompts, chain-of-thought
- Token limits and chunking strategies
- JSON-structured output from LLMs

### Resources
- **YouTube**: "OpenAI API Tutorial for Beginners" by Andrej Karpathy — https://www.youtube.com/watch?v=OB99E7Y1cMA
- **YouTube**: "Prompt Engineering Guide" by DAIR.AI — https://www.youtube.com/watch?v=dOxUroR57xs
- **Docs**: OpenAI API Reference — https://platform.openai.com/docs/api-reference
- **Course**: DeepLearning.AI ChatGPT Prompt Engineering for Developers (free) — https://learn.deeplearning.ai/chatgpt-prompt-eng

---

## Phase 4: LangChain for Document Processing Pipelines (Week 4-5)

### Core Concepts
- LangChain chains and runnables
- Text splitters and chunking for log data
- LLM-based summarization chains
- Output parsers for structured responses

### Resources
- **YouTube**: "LangChain Crash Course" by James Briggs — https://www.youtube.com/watch?v=aywZrzNaKjs
- **YouTube**: "LangChain Full Course" by FreeCodeCamp — https://www.youtube.com/watch?v=lG7Uxts9SXs
- **Docs**: LangChain Python docs — https://python.langchain.com/docs/introduction/
- **GitHub**: LangChain examples — https://github.com/langchain-ai/langchain/tree/master/cookbook

---

## Phase 5: FastAPI Backend Development (Week 5-6)

### Core Concepts
- FastAPI routing, request/response models
- File upload endpoints
- Background tasks and async processing
- Pydantic data validation

### Resources
- **YouTube**: "FastAPI Full Tutorial" by Tech With Tim — https://www.youtube.com/watch?v=0sOvCWFmrtA
- **Docs**: FastAPI official docs — https://fastapi.tiangolo.com/
- **YouTube**: "FastAPI File Uploads" — https://www.youtube.com/watch?v=s7wmiS2mSXY

---

## Phase 6: Streamlit for the Front-End Interface (Week 6-7)

### Core Concepts
- Streamlit app structure and layout
- File upload components
- Displaying dataframes and markdown
- Session state management

### Resources
- **YouTube**: "Streamlit Tutorial for Beginners" by Data Professor — https://www.youtube.com/watch?v=ZZ4B0QUHuNc
- **Docs**: Streamlit official docs — https://docs.streamlit.io/
- **Gallery**: Streamlit app examples — https://streamlit.io/gallery

---

## Suggested Learning Sequence

| Week | Focus |
|------|-------|
| 1 | Python regex and file I/O |
| 2 | Log parsing and normalization |
| 3 | OpenAI API and prompt engineering |
| 4 | LangChain chains and summarization |
| 5 | FastAPI backend |
| 6 | Streamlit UI |
| 7 | Integrate and test end-to-end |

---

## Books and Deeper Resources

- *Python Crash Course* by Eric Matthes — fundamentals
- *Building LLM Powered Applications* by Valentina Alto — LangChain patterns
- OpenAI Cookbook — https://github.com/openai/openai-cookbook
- *The Site Reliability Workbook* (Google) — context on SRE and incident management

---

## Key GitHub Repos to Study

- https://github.com/langchain-ai/langchain
- https://github.com/openai/openai-python
- https://github.com/tiangolo/fastapi
- https://github.com/streamlit/streamlit
