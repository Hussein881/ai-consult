# Implementation Guide: AI Meeting and Decision Tracker

## Prerequisites

- Python 3.11+
- OpenAI API key
- Sample meeting transcripts (generate 5-10 synthetic transcripts)
- `SQLAlchemy` for decision and action item persistence

---

## Architecture Overview

```
Meeting Transcript (text paste or .txt upload)
          │
          ▼
   [FastAPI Backend]
          │
    ┌─────┴───────────────────────────────┐
    │ LangChain Extraction Chain          │
    │  → decisions, action items, owners  │
    └─────┬───────────────────────────────┘
          │
    ┌─────┴───────────────────┐
    │ Meeting Summary Generator│  → 3-5 bullet summary
    └─────┬───────────────────┘
          │
    ┌─────┴──────────────────┐
    │ Database Storage        │  → meetings, decisions, action items
    └─────┬──────────────────┘
          │
   [Streamlit UI]  → paste transcript, review, export, view history
```

---

## Phase 1: Project Setup (Day 1)

```bash
mkdir meeting-decision-tracker
cd meeting-decision-tracker
python -m venv venv && source venv/bin/activate
mkdir -p src/{extractor,summarizer,database} tests data/transcripts
touch app.py ui.py .env
pip install fastapi uvicorn openai langchain langchain-openai python-dotenv \
            streamlit pydantic sqlalchemy python-multipart pandas
pip freeze > requirements.txt
```

---

## Phase 2: Synthetic Transcript Data (Day 1)

### Step 2: Create test transcript

Create `data/transcripts/sample_meeting.txt`:
```
Meeting: Q1 Product Planning
Date: January 15, 2024
Attendees: Sarah (PM), James (Engineering Lead), Maria (Design), Tom (CEO)

Tom: Alright, let's get started. We need to finalize the Q1 roadmap today.

Sarah: The three priorities we discussed are the search feature, mobile app, and the reporting dashboard.

James: Engineering can commit to the search feature by end of February. The mobile app will take until March.

Tom: Good. We need to decide on the reporting dashboard. Sarah, can you get stakeholder sign-off this week?

Sarah: Yes, I'll send the survey by Friday.

Maria: I'll have mockups for the search feature ready by next Tuesday.

James: I'll need the API spec from Sarah before I can start the mobile backend. Can we schedule that for Thursday?

Sarah: Thursday works. James, after I send the spec, you'll need two weeks to finish the backend, right?

James: Correct, two weeks from Thursday.

Tom: Let's go with that. Also, we decided to postpone the AI feature to Q2 to focus on stability. James, please document that decision in Confluence.
```

---

## Phase 3: LLM Extraction Chain (Days 2-4)

### Step 3: Extraction schema

```python
# src/extractor/schema.py
from pydantic import BaseModel
from typing import List, Optional

class ActionItem(BaseModel):
    description: str
    owner: Optional[str]
    due_date: Optional[str]  # e.g., "Friday", "January 19", "next Tuesday"
    priority: str = "medium"  # high, medium, low

class Decision(BaseModel):
    description: str
    made_by: Optional[str]
    rationale: Optional[str]

class MeetingExtraction(BaseModel):
    meeting_title: Optional[str]
    meeting_date: Optional[str]
    attendees: List[str]
    decisions: List[Decision]
    action_items: List[ActionItem]
    key_topics: List[str]
    follow_up_required: bool
```

### Step 4: Extraction chain

```python
# src/extractor/llm_extractor.py
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import JsonOutputParser
from .schema import MeetingExtraction

def extract_meeting_content(transcript: str) -> MeetingExtraction:
    llm = ChatOpenAI(model=os.getenv("CHAT_MODEL", "gpt-4o-mini"), temperature=0)
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", """Extract structured information from a meeting transcript.
Return JSON with:
- meeting_title, meeting_date (from transcript or null)
- attendees: list of names mentioned
- decisions: what was decided (description, who made it, brief rationale)
- action_items: tasks assigned (description, owner, due_date if mentioned, priority)
- key_topics: 3-5 main subjects discussed
- follow_up_required: true if unresolved items remain

Be precise. Only extract what is explicitly in the transcript."""),
        ("human", "Meeting transcript:\n{transcript}")
    ])
    
    chain = prompt | llm | JsonOutputParser()
    result = chain.invoke({"transcript": transcript[:6000]})
    return MeetingExtraction(**result)
```

---

## Phase 4: Meeting Summary Generator (Days 5-6)

### Step 5: Executive summary

```python
# src/summarizer/meeting_summarizer.py
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

def generate_meeting_summary(transcript: str, extraction) -> str:
    llm = ChatOpenAI(model=os.getenv("CHAT_MODEL", "gpt-4o-mini"), temperature=0.2)
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", """Write a concise meeting summary for a busy executive.
Format as 3-5 bullet points. Start with the most important decision or outcome.
Include: what was decided, key action items and owners, and any blockers or risks.
No filler language."""),
        ("human", """Transcript excerpt:
{transcript}

Decisions made: {decisions}
Action items: {action_items}""")
    ])
    
    chain = prompt | llm
    result = chain.invoke({
        "transcript": transcript[:2000],
        "decisions": "; ".join([d.description for d in extraction.decisions]),
        "action_items": "; ".join([f"{ai.description} ({ai.owner})" for ai in extraction.action_items])
    })
    return result.content
```

---

## Phase 5: Database Storage (Days 7-8)

### Step 6: Meeting history models

```python
# src/database/models.py
from sqlalchemy import Column, Integer, String, Boolean, DateTime, Text, ForeignKey, create_engine
from sqlalchemy.orm import relationship, sessionmaker
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime
import os

Base = declarative_base()

class Meeting(Base):
    __tablename__ = "meetings"
    id = Column(Integer, primary_key=True)
    title = Column(String)
    meeting_date = Column(String)
    summary = Column(Text)
    attendees = Column(String)  # JSON-serialized list
    created_at = Column(DateTime, default=datetime.utcnow)
    decisions = relationship("DecisionRecord", back_populates="meeting")
    action_items = relationship("ActionItemRecord", back_populates="meeting")

class DecisionRecord(Base):
    __tablename__ = "decisions"
    id = Column(Integer, primary_key=True)
    meeting_id = Column(Integer, ForeignKey("meetings.id"))
    description = Column(Text)
    made_by = Column(String, nullable=True)
    meeting = relationship("Meeting", back_populates="decisions")

class ActionItemRecord(Base):
    __tablename__ = "action_items"
    id = Column(Integer, primary_key=True)
    meeting_id = Column(Integer, ForeignKey("meetings.id"))
    description = Column(Text)
    owner = Column(String, nullable=True)
    due_date = Column(String, nullable=True)
    priority = Column(String, default="medium")
    is_complete = Column(Boolean, default=False)
    meeting = relationship("Meeting", back_populates="action_items")

engine = create_engine(os.getenv("DATABASE_URL", "sqlite:///./meetings.db"))
Base.metadata.create_all(engine)
SessionLocal = sessionmaker(bind=engine)
```

---

## Phase 6: FastAPI and Streamlit (Days 9-11)

### Step 7: FastAPI endpoints

```python
# app.py
from fastapi import FastAPI
from pydantic import BaseModel
from src.extractor.llm_extractor import extract_meeting_content
from src.summarizer.meeting_summarizer import generate_meeting_summary
from src.database.models import SessionLocal, Meeting, DecisionRecord, ActionItemRecord
import json

app = FastAPI(title="Meeting Tracker API")

class TranscriptInput(BaseModel):
    transcript: str

@app.post("/process")
def process(req: TranscriptInput):
    extraction = extract_meeting_content(req.transcript)
    summary = generate_meeting_summary(req.transcript, extraction)
    
    db = SessionLocal()
    meeting = Meeting(
        title=extraction.meeting_title or "Untitled Meeting",
        meeting_date=extraction.meeting_date,
        summary=summary,
        attendees=json.dumps(extraction.attendees)
    )
    db.add(meeting)
    db.flush()
    
    for d in extraction.decisions:
        db.add(DecisionRecord(meeting_id=meeting.id, description=d.description, made_by=d.made_by))
    
    for ai in extraction.action_items:
        db.add(ActionItemRecord(
            meeting_id=meeting.id,
            description=ai.description,
            owner=ai.owner,
            due_date=ai.due_date,
            priority=ai.priority
        ))
    
    db.commit()
    meeting_id = meeting.id
    db.close()
    
    return {
        "meeting_id": meeting_id,
        "summary": summary,
        "extraction": extraction.dict()
    }

@app.get("/action-items")
def get_open_action_items():
    db = SessionLocal()
    items = db.query(ActionItemRecord).filter(ActionItemRecord.is_complete == False).all()
    result = [{"id": i.id, "description": i.description, "owner": i.owner,
               "due_date": i.due_date, "priority": i.priority} for i in items]
    db.close()
    return {"action_items": result}
```

---

## Acceptance Criteria

- [ ] Extraction identifies all 4+ decisions from the sample transcript
- [ ] Action items include correct owners from the transcript
- [ ] All meetings, decisions, and action items persist in the database
- [ ] Open action items endpoint returns only incomplete items
- [ ] Streamlit shows decision history across multiple meetings
- [ ] Processing a 500-word transcript completes in under 10 seconds
