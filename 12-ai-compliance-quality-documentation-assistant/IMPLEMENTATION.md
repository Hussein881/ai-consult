# Implementation Guide: AI Compliance and Quality Documentation Assistant

## Prerequisites

- Python 3.11+
- OpenAI API key
- Sample SOP and policy documents (ISO 9001 templates are freely available)
- `sentence-transformers` for semantic gap analysis
- `python-docx` for document generation

---

## Architecture Overview

```
Existing SOPs / Templates
          │
          ▼
   [Document Ingestion + Categorization]
          │
          ▼
   [FastAPI Backend]
          │
    ┌─────┴──────────────────┐
    │ SOP Draft Generator    │  → LLM generates new SOP sections
    └─────┬──────────────────┘
          │
    ┌─────┴──────────────────┐
    │ Gap Analysis Engine    │  → compare doc to standard template
    └─────┬──────────────────┘
          │
    ┌─────┴──────────────────────┐
    │ Audit Readiness Scorer     │  → 0-100 score with explanations
    └─────┬──────────────────────┘
          │
   [Streamlit Review UI]  + DOCX export
```

---

## Phase 1: Project Setup (Day 1)

```bash
mkdir compliance-doc-assistant
cd compliance-doc-assistant
python -m venv venv && source venv/bin/activate
mkdir -p src/{ingestion,generator,gap_analysis,scorer} data/sop_templates tests
touch app.py ui.py .env
pip install fastapi uvicorn openai langchain langchain-openai python-dotenv \
            streamlit pydantic python-docx sentence-transformers pypdf \
            python-multipart numpy
pip freeze > requirements.txt
```

---

## Phase 2: SOP Template Ingestion (Days 2-3)

### Step 2: Standard SOP sections schema

```python
# src/ingestion/schema.py
from pydantic import BaseModel
from typing import List, Optional

# Standard sections required in a compliant SOP
REQUIRED_SOP_SECTIONS = [
    "Purpose",
    "Scope",
    "Responsibilities",
    "Procedure",
    "References",
    "Revision History"
]

class SOPDocument(BaseModel):
    title: str
    doc_number: Optional[str]
    version: Optional[str]
    department: Optional[str]
    sections: dict  # section_name: content
    raw_text: str

class SOPSection(BaseModel):
    section_name: str
    content: str
    is_present: bool
    quality_score: float  # 0.0-1.0
```

### Step 3: Document section extractor

```python
# src/ingestion/section_extractor.py
import re
from .schema import SOPDocument

def extract_sections(text: str, title: str = "Unknown") -> SOPDocument:
    """Extract sections from a text document based on heading patterns."""
    lines = text.splitlines()
    sections = {}
    current_section = "Introduction"
    current_content = []
    
    for line in lines:
        # Match headings: ALL CAPS lines, numbered sections, or lines followed by underlines
        if re.match(r'^[A-Z][A-Z\s]{3,}$', line.strip()) or re.match(r'^\d+\.\s+[A-Z]', line.strip()):
            if current_content:
                sections[current_section] = "\n".join(current_content).strip()
            current_section = line.strip().title()
            current_content = []
        else:
            current_content.append(line)
    
    if current_content:
        sections[current_section] = "\n".join(current_content).strip()
    
    return SOPDocument(
        title=title,
        sections=sections,
        raw_text=text
    )
```

---

## Phase 3: SOP Draft Generator (Days 4-5)

### Step 4: LLM section generator

```python
# src/generator/sop_generator.py
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

llm = ChatOpenAI(model=os.getenv("CHAT_MODEL", "gpt-4o-mini"), temperature=0.2)

SECTION_PROMPTS = {
    "Purpose": "Write a clear Purpose section for an SOP about {topic}. 2-3 sentences stating why this SOP exists.",
    "Scope": "Write a Scope section for an SOP about {topic}. State who this applies to and what it covers.",
    "Responsibilities": "Write a Responsibilities section for an SOP about {topic}. List roles in a bullet table: Role | Responsibility.",
    "Procedure": "Write a step-by-step Procedure section for an SOP about {topic}. Number each step. Include decision points.",
    "References": "Write a References section for an SOP about {topic}. List 3-5 relevant documents or standards.",
}

def generate_sop_section(topic: str, section_name: str, existing_context: str = "") -> str:
    template_prompt = SECTION_PROMPTS.get(section_name, f"Write the {section_name} section for an SOP about {{topic}}.")
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", "You are a compliance documentation expert. Write clear, audit-ready SOP content. Be specific, use active voice."),
        ("human", template_prompt + "\n\nAdditional context: {context}")
    ])
    
    chain = prompt | llm
    return chain.invoke({"topic": topic, "context": existing_context[:500]}).content
```

---

## Phase 4: Gap Analysis Engine (Days 6-7)

### Step 5: Semantic gap analysis using sentence-transformers

```python
# src/gap_analysis/analyzer.py
from sentence_transformers import SentenceTransformer, util
from src.ingestion.schema import SOPDocument, REQUIRED_SOP_SECTIONS, SOPSection
from typing import List

model = SentenceTransformer("all-MiniLM-L6-v2")

def analyze_gaps(document: SOPDocument) -> List[SOPSection]:
    """Assess presence and quality of required SOP sections."""
    results = []
    doc_sections_lower = {k.lower(): v for k, v in document.sections.items()}
    
    for required in REQUIRED_SOP_SECTIONS:
        found_section = None
        found_content = ""
        
        # First: exact match
        if required.lower() in doc_sections_lower:
            found_section = required
            found_content = doc_sections_lower[required.lower()]
        else:
            # Semantic match: find most similar section heading
            if doc_sections_lower:
                required_emb = model.encode(required, convert_to_tensor=True)
                best_score = 0.0
                for section_name, content in doc_sections_lower.items():
                    section_emb = model.encode(section_name, convert_to_tensor=True)
                    score = float(util.cos_sim(required_emb, section_emb))
                    if score > best_score and score > 0.5:
                        best_score = score
                        found_section = section_name
                        found_content = content
        
        is_present = found_section is not None
        quality = assess_section_quality(found_content, required) if is_present else 0.0
        
        results.append(SOPSection(
            section_name=required,
            content=found_content,
            is_present=is_present,
            quality_score=quality
        ))
    
    return results

def assess_section_quality(content: str, section_name: str) -> float:
    """Score content quality: length + presence of key words."""
    if len(content) < 20:
        return 0.1
    elif len(content) < 100:
        return 0.4
    elif len(content) < 300:
        return 0.7
    return 1.0
```

---

## Phase 5: Audit Readiness Scorer (Day 8)

### Step 6: Scoring engine

```python
# src/scorer/audit_scorer.py
from src.gap_analysis.analyzer import SOPSection
from typing import List

def calculate_audit_score(sections: List[SOPSection]) -> dict:
    if not sections:
        return {"score": 0, "grade": "F", "summary": "No sections found."}
    
    present_count = sum(1 for s in sections if s.is_present)
    quality_avg = sum(s.quality_score for s in sections) / len(sections)
    
    # Score: 60% for section presence, 40% for quality
    presence_score = (present_count / len(sections)) * 60
    quality_score = quality_avg * 40
    total = round(presence_score + quality_score)
    
    grade = "A" if total >= 90 else "B" if total >= 80 else "C" if total >= 70 else "D" if total >= 60 else "F"
    
    gaps = [s.section_name for s in sections if not s.is_present]
    weak = [s.section_name for s in sections if s.is_present and s.quality_score < 0.5]
    
    return {
        "score": total,
        "grade": grade,
        "present_sections": present_count,
        "total_required": len(sections),
        "missing_sections": gaps,
        "weak_sections": weak,
        "summary": f"Score {total}/100 ({grade}). Missing: {', '.join(gaps) or 'none'}."
    }
```

---

## Phase 6: FastAPI and Streamlit (Days 9-11)

### Step 7: FastAPI

```python
# app.py
from fastapi import FastAPI, UploadFile, File
from fastapi.responses import FileResponse
from pydantic import BaseModel
from src.ingestion.section_extractor import extract_sections
from src.gap_analysis.analyzer import analyze_gaps
from src.scorer.audit_scorer import calculate_audit_score
from src.generator.sop_generator import generate_sop_section
import tempfile

app = FastAPI(title="Compliance Doc API")

@app.post("/analyze")
async def analyze(file: UploadFile = File(...)):
    content = (await file.read()).decode("utf-8", errors="replace")
    doc = extract_sections(content, title=file.filename)
    sections = analyze_gaps(doc)
    score = calculate_audit_score(sections)
    return {
        "score": score,
        "sections": [s.dict() for s in sections]
    }

class GenerateRequest(BaseModel):
    topic: str
    section_name: str
    context: str = ""

@app.post("/generate-section")
def generate_section(req: GenerateRequest):
    content = generate_sop_section(req.topic, req.section_name, req.context)
    return {"section_name": req.section_name, "content": content}
```

---

## Acceptance Criteria

- [ ] Section extractor correctly identifies sections from a 5-page SOP
- [ ] Semantic matching detects "Overview" as matching "Purpose"
- [ ] Gap analysis identifies all missing required sections
- [ ] Audit score is 90+ for a complete, well-written SOP
- [ ] LLM generates each of the 5 standard sections without hallucination
- [ ] DOCX export includes all generated sections with correct headings
