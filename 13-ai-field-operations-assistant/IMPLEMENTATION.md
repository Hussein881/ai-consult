# Implementation Guide: AI Field Operations Assistant

## Prerequisites

- Python 3.11+
- OpenAI API key
- Sample field technician notes (write 20-30 synthetic notes for testing)
- `spaCy` with en_core_web_sm model for entity extraction

---

## Architecture Overview

```
Technician Notes (unstructured free text)
          │
          ▼
   [FastAPI Backend]
          │
    ┌─────┴──────────────────────┐
    │ NER Pre-processor (spaCy)  │  → extract equipment, locations, codes
    └─────┬──────────────────────┘
          │
    ┌─────┴────────────────────┐
    │ LLM Structuring Chain    │  → JSON mode, Pydantic work order schema
    └─────┬────────────────────┘
          │
    ┌─────┴────────────────────┐
    │ Customer Update Generator│  → professional customer summary
    └─────┬────────────────────┘
          │
    ┌─────┴──────────────────┐
    │ Work Order Export      │  → JSON / CSV for field management system
    └─────┬──────────────────┘
          │
   [Streamlit UI]  → paste notes, view structured output, export
```

---

## Phase 1: Project Setup (Day 1)

```bash
mkdir field-ops-assistant
cd field-ops-assistant
python -m venv venv && source venv/bin/activate
mkdir -p src/{nlp,structurer,exporter,database} tests data
touch app.py ui.py .env
pip install fastapi uvicorn openai langchain langchain-openai python-dotenv \
            streamlit pydantic sqlalchemy spacy python-multipart pandas
python -m spacy download en_core_web_sm
pip freeze > requirements.txt
```

---

## Phase 2: Sample Data (Day 1)

### Step 2: Create synthetic technician notes

Create `data/sample_notes.txt`:
```
Note 1: Arrived at 47 Oak Street at 09:15. Customer reported no heat. 
Inspected HVAC unit model Carrier 58STA. Found faulty ignitor P/N 025-33734-001. 
Replaced ignitor, tested unit, heat confirmed working. Unit serial HG2893456. 
Customer satisfied. Approx 2 hours on site.

Note 2: Job at Main Street Warehouse, Unit 3B. Elevator Door not closing. 
Fault code E-47 on panel. Lubricated door tracks and adjusted limit switch. 
Tested 5 cycles, door closes correctly. Parts: WD-40 Pro 2oz. 30 min job.
```

---

## Phase 3: spaCy NER Pre-processor (Days 2-3)

### Step 3: Entity extractor

```python
# src/nlp/entity_extractor.py
import spacy
from typing import List

nlp = spacy.load("en_core_web_sm")

def extract_entities(text: str) -> dict:
    """Extract named entities from field notes as pre-processing context."""
    doc = nlp(text)
    
    entities = {
        "locations": [],
        "organizations": [],
        "dates": [],
        "times": [],
        "equipment_hints": []
    }
    
    for ent in doc.ents:
        if ent.label_ in ("GPE", "LOC", "FAC"):
            entities["locations"].append(ent.text)
        elif ent.label_ == "ORG":
            entities["organizations"].append(ent.text)
        elif ent.label_ == "DATE":
            entities["dates"].append(ent.text)
        elif ent.label_ == "TIME":
            entities["times"].append(ent.text)
        elif ent.label_ in ("PRODUCT", "WORK_OF_ART"):
            entities["equipment_hints"].append(ent.text)
    
    # Custom: find model numbers and fault codes with regex
    import re
    model_numbers = re.findall(r'\b[A-Z]{2,6}[-\s]?\d{3,8}[A-Z0-9\-]*\b', text)
    fault_codes = re.findall(r'\b[Ee]-?\d{2,4}\b', text)
    entities["model_numbers"] = list(set(model_numbers))
    entities["fault_codes"] = list(set(fault_codes))
    
    return entities
```

---

## Phase 4: LLM Structuring Chain (Days 4-5)

### Step 4: Work order schema and extractor

```python
# src/structurer/work_order.py
import os
from pydantic import BaseModel
from typing import List, Optional
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import JsonOutputParser

class Part(BaseModel):
    name: str
    part_number: Optional[str]
    quantity: int = 1

class WorkOrder(BaseModel):
    job_number: Optional[str]
    customer_location: Optional[str]
    equipment_description: Optional[str]
    equipment_model: Optional[str]
    equipment_serial: Optional[str]
    fault_codes: List[str] = []
    work_performed: str
    parts_used: List[Part] = []
    labor_hours: Optional[float]
    job_status: str  # "completed", "follow_up_required", "parts_on_order"
    follow_up_notes: Optional[str]

def extract_work_order(technician_notes: str, entity_hints: dict) -> WorkOrder:
    llm = ChatOpenAI(model=os.getenv("CHAT_MODEL", "gpt-4o-mini"), temperature=0)
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", """Extract a structured work order from technician field notes.
Return valid JSON matching the WorkOrder schema. Be precise; only extract what is explicitly stated.
For labor_hours, convert time descriptions like "2 hours" or "30 min" to decimal hours.
For job_status: use "completed" if work is done, "follow_up_required" if issues remain, "parts_on_order" if waiting on parts."""),
        ("human", """Field Notes:
{notes}

Entity hints from NER (use as additional context):
{hints}

Extract work order as JSON.""")
    ])
    
    chain = prompt | llm | JsonOutputParser()
    result = chain.invoke({
        "notes": technician_notes[:3000],
        "hints": str(entity_hints)
    })
    return WorkOrder(**result)
```

---

## Phase 5: Customer Update Generator (Days 6-7)

### Step 5: Customer-facing message writer

```python
# src/structurer/customer_update.py
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from src.structurer.work_order import WorkOrder

def generate_customer_update(work_order: WorkOrder) -> str:
    llm = ChatOpenAI(model=os.getenv("CHAT_MODEL", "gpt-4o-mini"), temperature=0.3)
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", """Write a professional, friendly customer service message about completed field work.
2-3 sentences. State: what was done, that the issue is resolved (or next steps if not).
Do not include internal part numbers, cost details, or technician criticisms."""),
        ("human", """Work performed: {work_performed}
Equipment: {equipment}
Status: {status}
Follow-up: {follow_up}""")
    ])
    
    chain = prompt | llm
    result = chain.invoke({
        "work_performed": work_order.work_performed,
        "equipment": work_order.equipment_description or "equipment",
        "status": work_order.job_status,
        "follow_up": work_order.follow_up_notes or "None"
    })
    return result.content
```

---

## Phase 6: FastAPI and Streamlit (Days 8-10)

### Step 6: FastAPI endpoint

```python
# app.py
from fastapi import FastAPI
from pydantic import BaseModel
from src.nlp.entity_extractor import extract_entities
from src.structurer.work_order import extract_work_order
from src.structurer.customer_update import generate_customer_update

app = FastAPI(title="Field Operations API")

class NotesInput(BaseModel):
    technician_notes: str

@app.post("/process-notes")
def process(req: NotesInput):
    entities = extract_entities(req.technician_notes)
    work_order = extract_work_order(req.technician_notes, entities)
    customer_message = generate_customer_update(work_order)
    
    return {
        "work_order": work_order.dict(),
        "customer_message": customer_message,
        "entities_found": entities
    }
```

### Step 7: Streamlit UI

```python
# ui.py
import streamlit as st, requests, json, pandas as pd

st.set_page_config(page_title="Field Ops Assistant", layout="wide")
st.title("AI Field Operations Note Processor")

notes = st.text_area("Paste technician field notes:", height=200)

if st.button("Process Notes") and notes:
    with st.spinner("Extracting work order..."):
        result = requests.post("http://localhost:8000/process-notes",
                               json={"technician_notes": notes}).json()
    
    col1, col2 = st.columns(2)
    
    with col1:
        st.subheader("Structured Work Order")
        wo = result["work_order"]
        st.json(wo)
    
    with col2:
        st.subheader("Customer Update Message")
        st.text_area("", value=result["customer_message"], height=150)
        st.subheader("Entities Detected")
        st.write(result["entities_found"])
    
    st.download_button(
        "Export Work Order JSON",
        data=json.dumps(result["work_order"], indent=2),
        file_name="work_order.json"
    )
```

---

## Acceptance Criteria

- [ ] spaCy correctly extracts location, time, and model number from 10 test notes
- [ ] LLM extracts labor_hours as decimal float from "2 hours" and "30 min"
- [ ] Job status is correctly classified as "completed" vs "follow_up_required"
- [ ] Customer message is under 100 words and contains no internal part numbers
- [ ] Work order exports as valid JSON and CSV
- [ ] End-to-end processing of a 200-word note completes in under 8 seconds
