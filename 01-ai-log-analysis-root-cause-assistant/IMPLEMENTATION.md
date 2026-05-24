# Implementation Guide: AI Log Analysis and Root Cause Assistant

## Prerequisites

Before starting, ensure you have:
- Python 3.11+ installed
- An OpenAI API key (set as `OPENAI_API_KEY` environment variable)
- Basic familiarity with the command line
- `pip` or `uv` for package management

---

## Architecture Overview

```
Log Files (txt/log/json)
        │
        ▼
  [FastAPI Backend]
        │
   ┌────┴────┐
   │ Parser  │  → normalize timestamps, extract severity, group events
   └────┬────┘
        │
   ┌────┴────┐
   │ LangChain│  → summarize, cluster errors, generate root cause hypotheses
   │  Chain  │
   └────┬────┘
        │
   ┌────┴────────┐
   │ Incident    │  → structured JSON report
   │ Report Gen  │
   └────┬────────┘
        │
  [Streamlit UI]   → file upload, analysis display, export
```

---

## Phase 1: Project Setup (Day 1)

### Step 1: Create the project structure

```bash
mkdir log-analysis-assistant
cd log-analysis-assistant
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

mkdir -p src/{parser,analyzer,reporter} tests data/sample_logs
touch src/__init__.py src/parser/__init__.py src/analyzer/__init__.py src/reporter/__init__.py
touch app.py ui.py .env README.md
```

### Step 2: Install dependencies

```bash
pip install fastapi uvicorn langchain langchain-openai openai python-dotenv \
            streamlit pydantic pandas python-multipart
pip freeze > requirements.txt
```

### Step 3: Configure environment

Create `.env`:
```
OPENAI_API_KEY=your_key_here
OPENAI_MODEL=gpt-4o-mini
MAX_LOG_SIZE_MB=50
```

### Step 4: Load environment in code

```python
# src/config.py
from dotenv import load_dotenv
import os

load_dotenv()

OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
OPENAI_MODEL = os.getenv("OPENAI_MODEL", "gpt-4o-mini")
MAX_LOG_SIZE_MB = int(os.getenv("MAX_LOG_SIZE_MB", 50))
```

---

## Phase 2: Log Parser (Days 2-3)

### Step 5: Build the log normalizer

```python
# src/parser/log_parser.py
import re
from datetime import datetime
from typing import List, Optional
from pydantic import BaseModel

class LogEntry(BaseModel):
    timestamp: Optional[str]
    level: str  # ERROR, WARN, INFO, DEBUG, UNKNOWN
    source: Optional[str]
    message: str
    raw_line: str

def parse_log_line(line: str) -> LogEntry:
    """Parse a single log line into a structured entry."""
    # Common patterns: [TIMESTAMP] [LEVEL] message
    patterns = [
        r'(\d{4}-\d{2}-\d{2}[\sT]\d{2}:\d{2}:\d{2}[^\s]*)\s+(\w+)\s+(.*)',
        r'(\w{3}\s+\d+\s+\d{2}:\d{2}:\d{2})\s+(\w+)\s+(.*)',
    ]
    for pattern in patterns:
        match = re.match(pattern, line.strip())
        if match:
            ts, level, msg = match.groups()
            return LogEntry(timestamp=ts, level=level.upper(), message=msg, raw_line=line)
    
    # Fallback: check for severity keywords
    level = "UNKNOWN"
    for keyword in ["ERROR", "WARN", "WARNING", "CRITICAL", "FATAL", "INFO", "DEBUG"]:
        if keyword in line.upper():
            level = keyword
            break
    return LogEntry(timestamp=None, level=level, message=line.strip(), raw_line=line)

def parse_log_file(content: str) -> List[LogEntry]:
    """Parse a full log file content into structured entries."""
    lines = content.splitlines()
    entries = []
    for line in lines:
        if line.strip():
            entries.append(parse_log_line(line))
    return entries
```

### Step 6: Add error grouping

```python
# src/parser/grouper.py
from collections import Counter
from typing import List, Dict
from .log_parser import LogEntry

def group_errors(entries: List[LogEntry]) -> Dict:
    """Group log entries by error type and frequency."""
    errors = [e for e in entries if e.level in ("ERROR", "CRITICAL", "FATAL")]
    
    # Simple frequency grouping by message prefix
    message_patterns = Counter()
    for entry in errors:
        # Take first 60 chars as pattern key
        key = entry.message[:60].strip()
        message_patterns[key] += 1
    
    return {
        "total_errors": len(errors),
        "top_patterns": message_patterns.most_common(10),
        "error_entries": [e.dict() for e in errors[:100]]  # limit for LLM
    }
```

---

## Phase 3: LLM Analysis Chain (Days 4-6)

### Step 7: Build the summarization chain

```python
# src/analyzer/llm_analyzer.py
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import JsonOutputParser
from pydantic import BaseModel
from typing import List
import json

class RootCauseAnalysis(BaseModel):
    summary: str
    key_events: List[str]
    likely_root_causes: List[dict]  # {"hypothesis": str, "confidence": str, "evidence": str}
    recommended_next_steps: List[str]
    severity: str  # CRITICAL, HIGH, MEDIUM, LOW

def analyze_logs(grouped_data: dict, raw_sample: str) -> RootCauseAnalysis:
    llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", """You are an expert SRE and incident analyst. Analyze log data and provide 
         a structured root cause analysis. Return valid JSON matching this schema:
         {{
           "summary": "1-2 sentence incident summary",
           "key_events": ["list of key events in chronological order"],
           "likely_root_causes": [
             {{"hypothesis": "...", "confidence": "HIGH|MEDIUM|LOW", "evidence": "..."}}
           ],
           "recommended_next_steps": ["ordered debugging steps"],
           "severity": "CRITICAL|HIGH|MEDIUM|LOW"
         }}"""),
        ("human", """Analyze these logs:

Error summary:
{error_summary}

Sample log entries (first 50 errors):
{log_sample}

Provide root cause analysis as JSON.""")
    ])
    
    chain = prompt | llm | JsonOutputParser()
    
    result = chain.invoke({
        "error_summary": json.dumps(grouped_data, indent=2),
        "log_sample": raw_sample[:4000]  # stay within token budget
    })
    
    return RootCauseAnalysis(**result)
```

---

## Phase 4: FastAPI Backend (Days 7-8)

### Step 8: Build the API

```python
# app.py
from fastapi import FastAPI, UploadFile, File, HTTPException
from fastapi.middleware.cors import CORSMiddleware
import json

from src.config import MAX_LOG_SIZE_MB
from src.parser.log_parser import parse_log_file
from src.parser.grouper import group_errors
from src.analyzer.llm_analyzer import analyze_logs

app = FastAPI(title="Log Analysis API")

app.add_middleware(CORSMiddleware, allow_origins=["*"], allow_methods=["*"], allow_headers=["*"])

@app.post("/analyze")
async def analyze_log_file(file: UploadFile = File(...)):
    # Validate file size
    content = await file.read()
    if len(content) > MAX_LOG_SIZE_MB * 1024 * 1024:
        raise HTTPException(status_code=413, detail="File too large")
    
    # Validate file type
    if not file.filename.endswith((".log", ".txt", ".json")):
        raise HTTPException(status_code=400, detail="Unsupported file type")
    
    text = content.decode("utf-8", errors="replace")
    entries = parse_log_file(text)
    grouped = group_errors(entries)
    
    # Sample for LLM
    error_lines = "\n".join([e["raw_line"] for e in grouped["error_entries"][:50]])
    analysis = analyze_logs(grouped, error_lines)
    
    return {
        "filename": file.filename,
        "total_lines": len(entries),
        "analysis": analysis.dict()
    }

@app.get("/health")
def health():
    return {"status": "ok"}
```

### Step 9: Run the API

```bash
uvicorn app:app --reload --port 8000
```

---

## Phase 5: Streamlit UI (Days 9-10)

### Step 10: Build the interface

```python
# ui.py
import streamlit as st
import requests
import json

st.set_page_config(page_title="Log Analysis Assistant", layout="wide")
st.title("AI Log Analysis and Root Cause Assistant")

uploaded_file = st.file_uploader("Upload a log file", type=["log", "txt", "json"])

if uploaded_file:
    with st.spinner("Analyzing..."):
        response = requests.post(
            "http://localhost:8000/analyze",
            files={"file": (uploaded_file.name, uploaded_file, "text/plain")}
        )
    
    if response.status_code == 200:
        data = response.json()
        analysis = data["analysis"]
        
        # Header metrics
        col1, col2, col3 = st.columns(3)
        col1.metric("Total Lines", data["total_lines"])
        col2.metric("Severity", analysis["severity"])
        col3.metric("Root Causes Found", len(analysis["likely_root_causes"]))
        
        # Summary
        st.subheader("Incident Summary")
        st.write(analysis["summary"])
        
        # Key Events
        st.subheader("Key Events")
        for event in analysis["key_events"]:
            st.markdown(f"- {event}")
        
        # Root Causes
        st.subheader("Root Cause Hypotheses")
        for rc in analysis["likely_root_causes"]:
            with st.expander(f"[{rc['confidence']}] {rc['hypothesis']}"):
                st.write(f"**Evidence:** {rc['evidence']}")
        
        # Next Steps
        st.subheader("Recommended Next Steps")
        for i, step in enumerate(analysis["recommended_next_steps"], 1):
            st.markdown(f"{i}. {step}")
        
        # Export
        st.download_button(
            "Export as JSON",
            data=json.dumps(data, indent=2),
            file_name="incident_analysis.json",
            mime="application/json"
        )
    else:
        st.error(f"Analysis failed: {response.text}")
```

### Step 11: Run the UI

```bash
streamlit run ui.py
```

---

## Phase 6: Testing (Day 11)

### Step 12: Create sample test logs

```bash
# data/sample_logs/sample_error.log
cat > data/sample_logs/sample_error.log << 'EOF'
2024-01-15 10:23:01 ERROR DatabaseConnectionPool: Connection refused to db-primary:5432 after 3 retries
2024-01-15 10:23:02 ERROR DatabaseConnectionPool: Connection refused to db-primary:5432 after 3 retries
2024-01-15 10:23:03 WARN LoadBalancer: Switching to fallback db-replica:5432
2024-01-15 10:23:04 ERROR AuthService: Failed to validate token - database unavailable
2024-01-15 10:23:05 ERROR UserService: 500 Internal Server Error on /api/users endpoint
EOF
```

### Step 13: Write a basic test

```python
# tests/test_parser.py
from src.parser.log_parser import parse_log_line, parse_log_file

def test_parse_error_line():
    line = "2024-01-15 10:23:01 ERROR DatabasePool: Connection refused"
    entry = parse_log_line(line)
    assert entry.level == "ERROR"
    assert "Connection refused" in entry.message

def test_parse_log_file():
    content = "2024-01-15 10:23:01 ERROR Service: Failed\n2024-01-15 10:23:02 INFO Service: OK"
    entries = parse_log_file(content)
    assert len(entries) == 2
```

---

## Phase 7: Docker Deployment (Day 12)

### Step 14: Create Dockerfile

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Step 15: Build and run

```bash
docker build -t log-analysis-assistant .
docker run -e OPENAI_API_KEY=$OPENAI_API_KEY -p 8000:8000 log-analysis-assistant
```

---

## Acceptance Criteria

Phase 1 complete when:
- [ ] Project structure created and environment loads correctly
- [ ] All dependencies install without error

Phase 2 complete when:
- [ ] Parser correctly extracts ERROR, WARN, INFO levels from common log formats
- [ ] Grouper returns correct error counts and top patterns

Phase 3 complete when:
- [ ] LLM chain returns valid JSON matching the `RootCauseAnalysis` schema
- [ ] Analysis completes for a 500-line log file in under 30 seconds

Phase 4 complete when:
- [ ] `/analyze` endpoint accepts a file upload and returns JSON
- [ ] `/health` returns 200 OK

Phase 5 complete when:
- [ ] UI displays analysis results clearly
- [ ] JSON export downloads correctly

Phase 6 complete when:
- [ ] All unit tests pass
- [ ] End-to-end test with `sample_error.log` returns a valid analysis
