# Implementation Guide: AI Customer Support Triage Assistant

## Prerequisites

- Python 3.11+
- OpenAI API key
- Sample customer support data (create 30-50 synthetic tickets)
- Zendesk sandbox or Freshdesk free tier for integration testing

---

## Architecture Overview

```
Customer Message (email / form / chat)
          │
          ▼
   [Ingestion Layer]  → normalize multi-channel inputs
          │
          ▼
   [FastAPI Backend]
          │
    ┌─────┴─────┐
    │Classifier │  → category, urgency, sentiment
    └─────┬─────┘
          │
    ┌─────┴──────────┐
    │ Response Drafter│  → tone-matched draft response
    └─────┬──────────┘
          │
   [Streamlit Dashboard]  → review, edit, export
```

---

## Phase 1: Project Setup (Day 1)

### Step 1: Create structure

```bash
mkdir customer-support-triage
cd customer-support-triage
python -m venv venv && source venv/bin/activate
mkdir -p src/{ingestion,classifier,responder,database} tests data
touch app.py ui.py .env
pip install fastapi uvicorn openai langchain langchain-openai python-dotenv \
            streamlit pydantic sqlalchemy pandas python-multipart vaderSentiment
pip freeze > requirements.txt
```

---

## Phase 2: Multi-Channel Input Normalizer (Days 2-3)

### Step 2: Unified ticket schema

```python
# src/ingestion/schema.py
from pydantic import BaseModel
from typing import Literal, Optional
from datetime import datetime

class NormalizedTicket(BaseModel):
    channel: Literal["email", "form", "chat", "unknown"]
    subject: Optional[str]
    body: str
    customer_email: Optional[str]
    received_at: datetime = None
    raw_data: dict = {}
```

### Step 3: Input normalizer

```python
# src/ingestion/normalizer.py
from datetime import datetime
from .schema import NormalizedTicket

def from_email(email_data: dict) -> NormalizedTicket:
    return NormalizedTicket(
        channel="email",
        subject=email_data.get("subject", ""),
        body=email_data.get("body", ""),
        customer_email=email_data.get("from", ""),
        received_at=datetime.utcnow(),
        raw_data=email_data
    )

def from_form(form_data: dict) -> NormalizedTicket:
    return NormalizedTicket(
        channel="form",
        subject=form_data.get("subject", "Support Request"),
        body=form_data.get("message", ""),
        customer_email=form_data.get("email", ""),
        received_at=datetime.utcnow(),
        raw_data=form_data
    )

def from_raw_text(text: str) -> NormalizedTicket:
    return NormalizedTicket(
        channel="unknown",
        subject=None,
        body=text,
        received_at=datetime.utcnow(),
        raw_data={}
    )
```

---

## Phase 3: Classifier with Sentiment (Days 4-5)

### Step 4: Sentiment analyzer

```python
# src/classifier/sentiment.py
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer

analyzer = SentimentIntensityAnalyzer()

def get_sentiment(text: str) -> str:
    scores = analyzer.polarity_scores(text)
    compound = scores["compound"]
    if compound <= -0.5:
        return "frustrated"
    elif compound <= -0.1:
        return "negative"
    elif compound >= 0.1:
        return "positive"
    return "neutral"
```

### Step 5: LLM classifier (same pattern as Project 05)

```python
# src/classifier/llm_classifier.py
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import JsonOutputParser

CUSTOMER_CATEGORIES = [
    "billing_payment", "technical_issue", "account_access",
    "product_question", "delivery_shipping", "refund_return",
    "complaint", "general_inquiry"
]

def classify_customer_ticket(subject: str, body: str, sentiment: str) -> dict:
    llm = ChatOpenAI(model=os.getenv("CHAT_MODEL", "gpt-4o-mini"), temperature=0)
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", f"""Classify customer support tickets. Categories: {CUSTOMER_CATEGORIES}
Urgency: critical (business stopped), high (major issue), medium (degraded), low (question)
Return JSON: {{"category":"...","urgency":"...","routing":"...","confidence":"HIGH|MEDIUM|LOW"}}"""),
        ("human", "Sentiment: {sentiment}\nSubject: {subject}\nMessage: {body}")
    ])
    
    chain = prompt | llm | JsonOutputParser()
    return chain.invoke({"sentiment": sentiment, "subject": subject, "body": body[:2000]})
```

---

## Phase 4: Response Drafter (Days 6-7)

### Step 6: Tone-calibrated response generator

```python
# src/responder/response_generator.py
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

def draft_customer_response(body: str, category: str, urgency: str, sentiment: str) -> str:
    llm = ChatOpenAI(model=os.getenv("CHAT_MODEL", "gpt-4o-mini"), temperature=0.4)
    
    tone_map = {
        "frustrated": "empathetic and reassuring",
        "negative": "professional and helpful",
        "positive": "friendly and efficient",
        "neutral": "clear and professional"
    }
    tone = tone_map.get(sentiment, "professional")
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", f"""Write a {tone} customer support response.
Be direct and specific. Avoid generic phrases like 'I hope this helps'.
Do not make promises about specific resolution timelines.
Under 150 words."""),
        ("human", "Issue type: {category} (urgency: {urgency})\nCustomer message: {body}")
    ])
    
    chain = prompt | llm
    result = chain.invoke({"category": category, "urgency": urgency, "body": body[:1500]})
    return result.content
```

---

## Phase 5: FastAPI and Database (Days 8-9)

### Step 7: Full pipeline endpoint

```python
# app.py
from fastapi import FastAPI
from pydantic import BaseModel
from src.classifier.sentiment import get_sentiment
from src.classifier.llm_classifier import classify_customer_ticket
from src.responder.response_generator import draft_customer_response
from src.ingestion.normalizer import from_raw_text

app = FastAPI(title="Customer Support Triage API")

class TicketInput(BaseModel):
    subject: str = ""
    body: str

@app.post("/triage")
def triage(ticket: TicketInput):
    normalized = from_raw_text(ticket.body)
    sentiment = get_sentiment(ticket.body)
    classification = classify_customer_ticket(ticket.subject, ticket.body, sentiment)
    draft = draft_customer_response(
        ticket.body, classification["category"],
        classification["urgency"], sentiment
    )
    return {
        "sentiment": sentiment,
        "classification": classification,
        "draft_response": draft
    }

@app.get("/health")
def health():
    return {"status": "ok"}
```

---

## Phase 6: Streamlit Dashboard (Days 10-11)

### Step 8: Dashboard with batch processing

```python
# ui.py
import streamlit as st, requests, pandas as pd, json

st.set_page_config(page_title="Support Triage Dashboard", layout="wide")
st.title("AI Customer Support Triage")

tab1, tab2 = st.tabs(["Single Ticket", "Batch Upload"])

with tab1:
    subject = st.text_input("Subject (optional)")
    body = st.text_area("Customer message", height=150)
    if st.button("Triage Ticket"):
        with st.spinner("Analyzing..."):
            result = requests.post(
                "http://localhost:8000/triage",
                json={"subject": subject, "body": body}
            ).json()
        col1, col2, col3 = st.columns(3)
        col1.metric("Category", result["classification"]["category"])
        col2.metric("Urgency", result["classification"]["urgency"])
        col3.metric("Sentiment", result["sentiment"])
        st.subheader("Draft Response")
        st.text_area("", value=result["draft_response"], height=200)

with tab2:
    file = st.file_uploader("Upload CSV with 'subject' and 'body' columns")
    if file:
        df = pd.read_csv(file)
        results = []
        progress = st.progress(0)
        for i, row in df.iterrows():
            result = requests.post("http://localhost:8000/triage",
                                   json={"subject": str(row.get("subject", "")),
                                         "body": str(row["body"])}).json()
            results.append({
                "body_preview": str(row["body"])[:50],
                "category": result["classification"]["category"],
                "urgency": result["classification"]["urgency"],
                "sentiment": result["sentiment"]
            })
            progress.progress((i + 1) / len(df))
        st.dataframe(pd.DataFrame(results))
```

---

## Acceptance Criteria

- [ ] Classifier correctly identifies billing, technical, and account categories on test data
- [ ] Sentiment detection returns "frustrated" for clearly angry messages
- [ ] Draft responses are under 150 words and free of hallucinated details
- [ ] Batch CSV upload processes 20 tickets and displays results
- [ ] API responds in under 5 seconds per ticket
