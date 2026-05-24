# Implementation Guide: AI Support Escalation Ticket Triage Assistant

## Prerequisites

- Python 3.11+
- OpenAI API key
- Sample support ticket data (CSV or JSON — generate synthetic data for development)
- Zendesk sandbox account (free) for integration testing — or mock the API

---

## Architecture Overview

```
Incoming Ticket (text)
        │
        ▼
  [FastAPI Backend]
        │
   ┌────┴────┐
   │Classifier│  → category, urgency, confidence (LLM + JSON mode)
   └────┬────┘
        │
   ┌────┴──────┐
   │ Rules     │  → routing destination based on category × urgency matrix
   │ Engine    │
   └────┬──────┘
        │
   ┌────┴──────────┐
   │Response       │  → draft customer response (LLM)
   │Generator      │
   └────┬──────────┘
        │
  [Streamlit Dashboard]
```

---

## Phase 1: Project Setup (Day 1)

### Step 1: Create structure

```bash
mkdir ticket-triage-assistant
cd ticket-triage-assistant
python -m venv venv && source venv/bin/activate

mkdir -p src/{classifier,router,responder,database} tests data
touch app.py ui.py .env
```

### Step 2: Install dependencies

```bash
pip install fastapi uvicorn openai langchain langchain-openai python-dotenv \
            streamlit pydantic sqlalchemy pandas python-multipart
pip freeze > requirements.txt
```

### Step 3: Environment

```
OPENAI_API_KEY=your_key_here
CHAT_MODEL=gpt-4o-mini
DATABASE_URL=sqlite:///./tickets.db
```

---

## Phase 2: Ticket Classifier (Days 2-4)

### Step 4: Define ticket taxonomy and schema

```python
# src/classifier/schema.py
from pydantic import BaseModel
from typing import Literal

CATEGORIES = [
    "billing", "technical", "account", "product_feature",
    "shipping_delivery", "returns_refunds", "security", "other"
]

URGENCY_LEVELS = ["critical", "high", "medium", "low"]

ROUTING_DESTINATIONS = [
    "tier1_support", "tier2_engineering", "billing_team",
    "security_team", "account_management", "self_service"
]

class TicketClassification(BaseModel):
    category: str
    urgency: str
    routing: str
    confidence: Literal["HIGH", "MEDIUM", "LOW"]
    reasoning: str
```

### Step 5: LLM classifier

```python
# src/classifier/llm_classifier.py
import os, json
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import JsonOutputParser
from .schema import TicketClassification, CATEGORIES, URGENCY_LEVELS, ROUTING_DESTINATIONS

def classify_ticket(subject: str, body: str) -> TicketClassification:
    llm = ChatOpenAI(model=os.getenv("CHAT_MODEL", "gpt-4o-mini"), temperature=0)
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", f"""You are a support triage classifier. Classify the ticket and return JSON.

Categories: {CATEGORIES}
Urgency levels: {URGENCY_LEVELS}  
Routing destinations: {ROUTING_DESTINATIONS}

Urgency rules:
- critical: system down, security breach, data loss
- high: major feature broken, payment failure
- medium: partial functionality, slow response
- low: question, feature request, minor inconvenience

Return: {{"category": "...", "urgency": "...", "routing": "...", "confidence": "HIGH|MEDIUM|LOW", "reasoning": "..."}}"""),
        ("human", "Subject: {subject}\n\nBody: {body}")
    ])
    
    chain = prompt | llm | JsonOutputParser()
    result = chain.invoke({"subject": subject, "body": body[:2000]})
    return TicketClassification(**result)
```

---

## Phase 3: Routing Rules Engine (Day 5)

### Step 6: Build routing matrix

```python
# src/router/routing_engine.py
from src.classifier.schema import TicketClassification

# Routing override rules: (category, urgency) → destination
ROUTING_RULES = {
    ("technical", "critical"): "tier2_engineering",
    ("technical", "high"): "tier2_engineering",
    ("billing", "critical"): "billing_team",
    ("billing", "high"): "billing_team",
    ("security", "critical"): "security_team",
    ("security", "high"): "security_team",
    ("account", "medium"): "account_management",
    ("account", "low"): "self_service",
}

def apply_routing_rules(classification: TicketClassification) -> str:
    """Apply deterministic routing rules on top of LLM classification."""
    key = (classification.category, classification.urgency)
    return ROUTING_RULES.get(key, classification.routing)
```

---

## Phase 4: Draft Response Generator (Days 6-7)

### Step 7: Response templates and LLM generator

```python
# src/responder/response_generator.py
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

# Category-specific tone guidelines
TONE_GUIDELINES = {
    "billing": "empathetic, professional, offer escalation path",
    "technical": "direct, helpful, provide next steps",
    "security": "urgent, professional, escalate immediately",
    "default": "friendly, professional, solution-focused"
}

def generate_draft_response(subject: str, body: str, category: str, urgency: str) -> str:
    llm = ChatOpenAI(model=os.getenv("CHAT_MODEL", "gpt-4o-mini"), temperature=0.3)
    tone = TONE_GUIDELINES.get(category, TONE_GUIDELINES["default"])
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", f"""You are a professional customer support agent.
Write a helpful, {tone} response to this support ticket.
Do not promise specific timelines you cannot guarantee.
Do not share internal information.
Keep it under 200 words."""),
        ("human", "Subject: {subject}\nMessage: {body}\nUrgency: {urgency}")
    ])
    
    chain = prompt | llm
    result = chain.invoke({"subject": subject, "body": body[:1500], "urgency": urgency})
    return result.content
```

---

## Phase 5: Database Storage (Day 8)

### Step 8: Ticket storage model

```python
# src/database/models.py
from sqlalchemy import Column, Integer, String, DateTime, Text, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from datetime import datetime
import os

Base = declarative_base()

class TicketRecord(Base):
    __tablename__ = "tickets"
    id = Column(Integer, primary_key=True)
    subject = Column(String)
    body = Column(Text)
    category = Column(String, index=True)
    urgency = Column(String, index=True)
    routing = Column(String)
    confidence = Column(String)
    draft_response = Column(Text)
    created_at = Column(DateTime, default=datetime.utcnow)

engine = create_engine(os.getenv("DATABASE_URL", "sqlite:///./tickets.db"))
Base.metadata.create_all(engine)
SessionLocal = sessionmaker(bind=engine)
```

---

## Phase 6: FastAPI and Streamlit (Days 9-11)

### Step 9: FastAPI endpoint

```python
# app.py
from fastapi import FastAPI
from pydantic import BaseModel
from src.classifier.llm_classifier import classify_ticket
from src.router.routing_engine import apply_routing_rules
from src.responder.response_generator import generate_draft_response
from src.database.models import SessionLocal, TicketRecord

app = FastAPI(title="Ticket Triage API")

class TicketInput(BaseModel):
    subject: str
    body: str

@app.post("/triage")
def triage(ticket: TicketInput):
    classification = classify_ticket(ticket.subject, ticket.body)
    routing = apply_routing_rules(classification)
    draft = generate_draft_response(
        ticket.subject, ticket.body,
        classification.category, classification.urgency
    )
    
    db = SessionLocal()
    record = TicketRecord(
        subject=ticket.subject, body=ticket.body,
        category=classification.category, urgency=classification.urgency,
        routing=routing, confidence=classification.confidence,
        draft_response=draft
    )
    db.add(record)
    db.commit()
    db.close()
    
    return {
        "classification": classification.dict(),
        "routing": routing,
        "draft_response": draft
    }
```

### Step 10: Test with synthetic data

Create `data/sample_tickets.json` with 20 sample tickets across categories and urgency levels, then write a test script that sends each through the `/triage` endpoint and prints results.

---

## Acceptance Criteria

- [ ] Classifier returns valid category, urgency, and routing for 20 test tickets
- [ ] Routing rules override LLM routing for security and billing critical tickets
- [ ] Draft responses are under 200 words and do not hallucinate promises
- [ ] All tickets are stored in the database with correct metadata
- [ ] Streamlit dashboard shows ticket volume by category and urgency
- [ ] End-to-end triage completes in under 5 seconds per ticket
