# Implementation Guide: AI Sales Proposal and Quote Assistant

## Prerequisites

- Python 3.11+
- OpenAI API key
- Sample winning proposal documents (2-3 examples to use as few-shot context)
- `python-docx` and `jinja2` for document generation

---

## Architecture Overview

```
Client Input Form (client name, problem, budget, scope)
          │
          ▼
   [FastAPI Backend]
          │
    ┌─────┴──────────┐
    │ Template Engine│  → Jinja2 fills structure with client data
    └─────┬──────────┘
          │
    ┌─────┴───────────┐
    │ LLM Section Gen │  → generates each proposal section
    └─────┬───────────┘
          │
    ┌─────┴──────────┐
    │ Pricing Engine │  → suggests pricing based on scope inputs
    └─────┬──────────┘
          │
    ┌─────┴──────┐
    │ Exporter   │  → DOCX and PDF output
    └─────┬──────┘
          │
   [Streamlit UI]  → form, preview, edit, export
```

---

## Phase 1: Project Setup (Day 1)

```bash
mkdir sales-proposal-assistant
cd sales-proposal-assistant
python -m venv venv && source venv/bin/activate
mkdir -p src/{generator,exporter,templates,database} data/examples tests
touch app.py ui.py .env
pip install fastapi uvicorn openai langchain langchain-openai python-dotenv \
            streamlit pydantic sqlalchemy jinja2 python-docx python-multipart
pip freeze > requirements.txt
```

---

## Phase 2: Client Input Schema (Days 2-3)

### Step 2: Proposal input model

```python
# src/generator/schema.py
from pydantic import BaseModel
from typing import List, Optional, Literal

class ClientInput(BaseModel):
    client_name: str
    client_industry: str
    problem_description: str
    desired_outcome: str
    budget_range: Optional[str]  # e.g. "$50,000-$100,000"
    timeline: Optional[str]  # e.g. "3-4 months"
    key_stakeholders: Optional[str]
    special_requirements: Optional[str]
    engagement_type: Literal["discovery", "pilot", "full_implementation", "retainer"]

class ProposalSection(BaseModel):
    title: str
    content: str

class GeneratedProposal(BaseModel):
    client_name: str
    sections: List[ProposalSection]
    pricing_summary: dict
    next_steps: List[str]
```

---

## Phase 3: Proposal Section Generators (Days 4-6)

### Step 3: Executive summary generator

```python
# src/generator/section_generators.py
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

llm = ChatOpenAI(model=os.getenv("CHAT_MODEL", "gpt-4o-mini"), temperature=0.3)

def generate_executive_summary(client_input: dict) -> str:
    prompt = ChatPromptTemplate.from_messages([
        ("system", """Write a professional executive summary for a consulting proposal.
2-3 paragraphs. Focus on the client's business problem, the proposed solution approach,
and expected business outcomes. Professional and confident tone. No jargon."""),
        ("human", """Client: {client_name} ({client_industry})
Problem: {problem_description}
Desired outcome: {desired_outcome}
Engagement type: {engagement_type}""")
    ])
    chain = prompt | llm
    return chain.invoke(client_input).content

def generate_scope_of_work(client_input: dict) -> str:
    prompt = ChatPromptTemplate.from_messages([
        ("system", """Write a clear Scope of Work section for a consulting proposal.
Include: Work Included, Deliverables, Out of Scope, Assumptions.
Use bullet points. Be specific and concrete."""),
        ("human", """Problem: {problem_description}
Timeline: {timeline}
Special requirements: {special_requirements}
Engagement type: {engagement_type}""")
    ])
    chain = prompt | llm
    return chain.invoke(client_input).content

def generate_approach(client_input: dict) -> str:
    prompt = ChatPromptTemplate.from_messages([
        ("system", """Write a Proposed Approach section for an AI consulting proposal.
Describe the methodology in 3-4 phases. Be specific about AI technologies used.
Make it sound experienced and low-risk for the client."""),
        ("human", """Problem: {problem_description}
Outcome: {desired_outcome}
Timeline: {timeline}""")
    ])
    chain = prompt | llm
    return chain.invoke(client_input).content
```

### Step 4: Pricing suggester

```python
# src/generator/pricing.py
from pydantic import BaseModel

ENGAGEMENT_PRICES = {
    "discovery": {"low": 8000, "mid": 15000, "high": 25000},
    "pilot": {"low": 35000, "mid": 65000, "high": 100000},
    "full_implementation": {"low": 80000, "mid": 150000, "high": 250000},
    "retainer": {"low": 3000, "mid": 6000, "high": 10000},
}

def suggest_pricing(engagement_type: str, budget_range: str = None) -> dict:
    prices = ENGAGEMENT_PRICES.get(engagement_type, ENGAGEMENT_PRICES["pilot"])
    return {
        "engagement_type": engagement_type,
        "recommended_range": f"${prices['low']:,} - ${prices['high']:,}",
        "payment_schedule": "50% upfront, 50% on delivery" if engagement_type != "retainer" else "Monthly invoice",
        "note": "Final pricing depends on scope detail from discovery."
    }
```

---

## Phase 4: Proposal Assembler and Exporter (Days 7-8)

### Step 5: Full proposal generator

```python
# src/generator/proposal_assembler.py
from .section_generators import generate_executive_summary, generate_scope_of_work, generate_approach
from .pricing import suggest_pricing
from .schema import ClientInput, GeneratedProposal, ProposalSection

def generate_proposal(client_input: ClientInput) -> GeneratedProposal:
    input_dict = client_input.dict()
    
    sections = [
        ProposalSection(title="Executive Summary", content=generate_executive_summary(input_dict)),
        ProposalSection(title="Scope of Work", content=generate_scope_of_work(input_dict)),
        ProposalSection(title="Proposed Approach", content=generate_approach(input_dict)),
    ]
    
    pricing = suggest_pricing(client_input.engagement_type, client_input.budget_range)
    
    next_steps = [
        "Review this proposal and share with relevant stakeholders.",
        "Schedule a 30-minute scoping call to refine the approach.",
        "Sign engagement letter to begin the discovery phase."
    ]
    
    return GeneratedProposal(
        client_name=client_input.client_name,
        sections=sections,
        pricing_summary=pricing,
        next_steps=next_steps
    )
```

### Step 6: DOCX exporter

```python
# src/exporter/docx_exporter.py
from docx import Document
from docx.shared import Pt
from src.generator.schema import GeneratedProposal

def export_to_docx(proposal: GeneratedProposal, output_path: str):
    doc = Document()
    
    doc.add_heading(f"Proposal for {proposal.client_name}", 0)
    
    for section in proposal.sections:
        doc.add_heading(section.title, level=1)
        doc.add_paragraph(section.content)
    
    doc.add_heading("Investment", level=1)
    pricing = proposal.pricing_summary
    doc.add_paragraph(f"Engagement Type: {pricing['engagement_type'].replace('_', ' ').title()}")
    doc.add_paragraph(f"Investment Range: {pricing['recommended_range']}")
    doc.add_paragraph(f"Payment: {pricing['payment_schedule']}")
    doc.add_paragraph(f"Note: {pricing['note']}")
    
    doc.add_heading("Next Steps", level=1)
    for i, step in enumerate(proposal.next_steps, 1):
        doc.add_paragraph(f"{i}. {step}")
    
    doc.save(output_path)
```

---

## Phase 5: FastAPI and Streamlit (Days 9-11)

### Step 7: FastAPI

```python
# app.py
from fastapi import FastAPI
from fastapi.responses import FileResponse
from src.generator.schema import ClientInput
from src.generator.proposal_assembler import generate_proposal
from src.exporter.docx_exporter import export_to_docx
import tempfile, os

app = FastAPI(title="Sales Proposal API")

@app.post("/generate")
def generate(client_input: ClientInput):
    proposal = generate_proposal(client_input)
    return proposal.dict()

@app.post("/generate/download")
def generate_and_download(client_input: ClientInput):
    proposal = generate_proposal(client_input)
    tmp = tempfile.NamedTemporaryFile(suffix=".docx", delete=False)
    export_to_docx(proposal, tmp.name)
    return FileResponse(tmp.name, filename=f"proposal_{client_input.client_name}.docx",
                       media_type="application/vnd.openxmlformats-officedocument.wordprocessingml.document")
```

---

## Acceptance Criteria

- [ ] Executive summary uses client name and industry correctly
- [ ] Scope of work includes "Out of Scope" section
- [ ] Pricing suggestion matches the engagement type selected
- [ ] DOCX export is well-formatted and readable
- [ ] Complete proposal generation completes in under 30 seconds
- [ ] Streamlit form captures all required client inputs
