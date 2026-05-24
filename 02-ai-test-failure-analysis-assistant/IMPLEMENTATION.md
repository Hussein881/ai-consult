# Implementation Guide: AI Test Failure Analysis Assistant

## Prerequisites

- Python 3.11+
- OpenAI API key
- Sample JUnit XML test results (create synthetic ones if needed)
- SQLite (built-in) or PostgreSQL for historical storage

---

## Architecture Overview

```
JUnit XML / JSON / CSV test results
          │
          ▼
   [FastAPI Backend]
          │
    ┌─────┴─────┐
    │  Parser   │  → extract test cases, durations, errors
    └─────┬─────┘
          │
    ┌─────┴──────────┐
    │ Historical DB  │  → compare to past runs, calculate flakiness
    └─────┬──────────┘
          │
    ┌─────┴─────┐
    │  LLM      │  → classify failure, explain cause, release recommendation
    │ Classifier│
    └─────┬─────┘
          │
   [Streamlit UI]   → upload, review, export release-readiness report
```

---

## Phase 1: Project Setup (Day 1)

### Step 1: Create project structure

```bash
mkdir test-failure-analyzer
cd test-failure-analyzer
python -m venv venv && source venv/bin/activate

mkdir -p src/{parser,classifier,database,reporter} tests data
touch app.py ui.py .env
```

### Step 2: Install dependencies

```bash
pip install fastapi uvicorn langchain langchain-openai openai python-dotenv \
            streamlit pydantic pandas sqlalchemy python-multipart lxml
pip freeze > requirements.txt
```

### Step 3: Configure environment

```
OPENAI_API_KEY=your_key_here
OPENAI_MODEL=gpt-4o-mini
DATABASE_URL=sqlite:///./test_history.db
FLAKINESS_THRESHOLD=0.3
```

---

## Phase 2: JUnit XML Parser (Days 2-3)

### Step 4: Parse JUnit XML

```python
# src/parser/junit_parser.py
import xml.etree.ElementTree as ET
from typing import List, Optional
from pydantic import BaseModel

class TestCase(BaseModel):
    name: str
    classname: Optional[str]
    duration: float
    status: str  # passed, failed, skipped, error
    failure_message: Optional[str]
    failure_type: Optional[str]
    stdout: Optional[str]

class TestSuite(BaseModel):
    name: str
    tests: int
    failures: int
    errors: int
    skipped: int
    duration: float
    test_cases: List[TestCase]

def parse_junit_xml(content: str) -> List[TestSuite]:
    """Parse JUnit XML report into structured test suites."""
    root = ET.fromstring(content)
    suites = []
    
    # Handle both <testsuites> and <testsuite> at root
    suite_elements = root.findall("testsuite") if root.tag == "testsuites" else [root]
    
    for suite_elem in suite_elements:
        test_cases = []
        for tc in suite_elem.findall("testcase"):
            failure = tc.find("failure")
            error = tc.find("error")
            skipped = tc.find("skipped")
            
            status = "passed"
            failure_msg = None
            failure_type = None
            
            if failure is not None:
                status = "failed"
                failure_msg = failure.text or failure.get("message", "")
                failure_type = failure.get("type", "")
            elif error is not None:
                status = "error"
                failure_msg = error.text or error.get("message", "")
                failure_type = error.get("type", "")
            elif skipped is not None:
                status = "skipped"
            
            stdout_elem = tc.find("system-out")
            
            test_cases.append(TestCase(
                name=tc.get("name", ""),
                classname=tc.get("classname"),
                duration=float(tc.get("time", 0)),
                status=status,
                failure_message=failure_msg,
                failure_type=failure_type,
                stdout=stdout_elem.text if stdout_elem is not None else None
            ))
        
        suites.append(TestSuite(
            name=suite_elem.get("name", ""),
            tests=int(suite_elem.get("tests", 0)),
            failures=int(suite_elem.get("failures", 0)),
            errors=int(suite_elem.get("errors", 0)),
            skipped=int(suite_elem.get("skipped", 0)),
            duration=float(suite_elem.get("time", 0)),
            test_cases=test_cases
        ))
    
    return suites
```

---

## Phase 3: Historical Storage and Flakiness (Days 4-5)

### Step 5: Database models

```python
# src/database/models.py
from sqlalchemy import Column, String, Float, Integer, Boolean, DateTime, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
from datetime import datetime

Base = declarative_base()

class TestRunRecord(Base):
    __tablename__ = "test_runs"
    id = Column(Integer, primary_key=True)
    run_id = Column(String, index=True)
    test_name = Column(String, index=True)
    classname = Column(String)
    status = Column(String)
    duration = Column(Float)
    failure_message = Column(String, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)

engine = create_engine(os.getenv("DATABASE_URL", "sqlite:///./test_history.db"))
Base.metadata.create_all(engine)
SessionLocal = sessionmaker(bind=engine)
```

### Step 6: Flakiness calculator

```python
# src/classifier/flakiness.py
from sqlalchemy.orm import Session
from src.database.models import TestRunRecord

def calculate_flakiness_score(test_name: str, db: Session, last_n: int = 10) -> float:
    """Calculate flakiness score: ratio of mixed results in last N runs."""
    recent = db.query(TestRunRecord).filter(
        TestRunRecord.test_name == test_name
    ).order_by(TestRunRecord.created_at.desc()).limit(last_n).all()
    
    if len(recent) < 3:
        return 0.0  # not enough history
    
    statuses = [r.status for r in recent]
    failures = statuses.count("failed") + statuses.count("error")
    passes = statuses.count("passed")
    
    if failures == 0 or passes == 0:
        return 0.0  # consistent either way
    
    return failures / len(statuses)  # 0.0 = stable, 1.0 = always failing
```

---

## Phase 4: LLM Failure Classifier (Days 6-7)

### Step 7: Classification chain

```python
# src/classifier/llm_classifier.py
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import JsonOutputParser

def classify_failure(test_name: str, failure_message: str, flakiness_score: float) -> dict:
    llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", """Classify test failures as one of:
         - REGRESSION: new code broke this test
         - FLAKY: intermittent, likely non-deterministic
         - ENVIRONMENT: infrastructure or config issue  
         - EXPECTED: test is known broken
         
         Return JSON: {{"classification": "...", "confidence": "HIGH|MEDIUM|LOW", "explanation": "..."}}"""),
        ("human", """Test: {test_name}
Failure: {failure_message}
Historical flakiness score: {flakiness_score:.1%} (0% = always passes, 100% = always fails)

Classify this failure.""")
    ])
    
    chain = prompt | llm | JsonOutputParser()
    return chain.invoke({
        "test_name": test_name,
        "failure_message": failure_message[:1000],
        "flakiness_score": flakiness_score
    })
```

---

## Phase 5: Release Readiness Report (Day 8)

### Step 8: Report generator

```python
# src/reporter/report_generator.py
from typing import List

def generate_release_report(suites, classifications: list) -> dict:
    total = sum(s.tests for s in suites)
    failed = sum(s.failures + s.errors for s in suites)
    regressions = [c for c in classifications if c["classification"] == "REGRESSION"]
    
    recommendation = "GO" if len(regressions) == 0 else ("HOLD" if len(regressions) >= 3 else "REVIEW")
    
    return {
        "recommendation": recommendation,
        "total_tests": total,
        "failed_tests": failed,
        "pass_rate": f"{((total - failed) / total * 100):.1f}%" if total > 0 else "0%",
        "regressions": len(regressions),
        "flaky_tests": len([c for c in classifications if c["classification"] == "FLAKY"]),
        "classifications": classifications,
        "summary": f"{recommendation}: {len(regressions)} regressions found."
    }
```

---

## Phase 6: FastAPI and Streamlit (Days 9-11)

### Step 9: FastAPI endpoint

```python
# app.py
from fastapi import FastAPI, UploadFile, File
from src.parser.junit_parser import parse_junit_xml
from src.classifier.llm_classifier import classify_failure
from src.classifier.flakiness import calculate_flakiness_score
from src.reporter.report_generator import generate_release_report
from src.database.models import SessionLocal, TestRunRecord
import uuid
from datetime import datetime

app = FastAPI(title="Test Failure Analyzer")

@app.post("/analyze")
async def analyze(file: UploadFile = File(...)):
    content = (await file.read()).decode("utf-8")
    suites = parse_junit_xml(content)
    run_id = str(uuid.uuid4())
    
    db = SessionLocal()
    classifications = []
    
    for suite in suites:
        for tc in suite.test_cases:
            # Store in history
            db.add(TestRunRecord(run_id=run_id, test_name=tc.name,
                                 classname=tc.classname, status=tc.status,
                                 duration=tc.duration, failure_message=tc.failure_message,
                                 created_at=datetime.utcnow()))
            
            if tc.status in ("failed", "error") and tc.failure_message:
                flakiness = calculate_flakiness_score(tc.name, db)
                classification = classify_failure(tc.name, tc.failure_message, flakiness)
                classification["test_name"] = tc.name
                classifications.append(classification)
    
    db.commit()
    db.close()
    
    return generate_release_report(suites, classifications)
```

---

## Acceptance Criteria

- [ ] JUnit XML parser correctly handles `<testsuites>` and `<testsuite>` root elements
- [ ] Flakiness score returns 0.0 for tests with fewer than 3 historical runs
- [ ] LLM classifier returns one of REGRESSION, FLAKY, ENVIRONMENT, EXPECTED
- [ ] Release recommendation is GO / HOLD / REVIEW based on regression count
- [ ] All test data persists in database for historical comparison
- [ ] Streamlit UI displays classification results with color coding
