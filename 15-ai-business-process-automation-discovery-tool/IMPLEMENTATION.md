# Implementation Guide: AI Business Process Automation Discovery Tool

## Prerequisites

- Python 3.11+
- OpenAI API key
- `python-docx` and `jinja2` for report generation
- `python-pptx` (optional) for slide deck export

---

## Architecture Overview

```
Guided Interview (multi-step form)
          │  role, process description, frequency, pain points
          ▼
   [FastAPI Backend]
          │
    ┌─────┴──────────────────────┐
    │ Process Analyzer           │  → categorize automation potential
    └─────┬──────────────────────┘
          │
    ┌─────┴──────────────────────┐
    │ ROI Modeler                │  → estimate effort saved, cost reduction
    └─────┬──────────────────────┘
          │
    ┌─────┴──────────────────────┐
    │ Recommendation Engine      │  → priority ranking of automation opportunities
    └─────┴──────────────────────┘
          │
    ┌─────┴──────────────────────┐
    │ Report Generator           │  → Jinja2 → DOCX executive report
    └─────┬──────────────────────┘
          │
   [Streamlit Multi-Step Wizard]
```

---

## Phase 1: Project Setup (Day 1)

```bash
mkdir automation-discovery-tool
cd automation-discovery-tool
python -m venv venv && source venv/bin/activate
mkdir -p src/{interview,analyzer,roi,reporter} data/templates tests
touch app.py ui.py .env
pip install fastapi uvicorn openai langchain langchain-openai python-dotenv \
            streamlit pydantic jinja2 python-docx python-multipart
pip freeze > requirements.txt
```

---

## Phase 2: Interview Schema and Capture (Days 2-3)

### Step 2: Interview data schema

```python
# src/interview/schema.py
from pydantic import BaseModel
from typing import List, Optional, Literal

class ProcessEntry(BaseModel):
    name: str  # "Vendor Invoice Approval"
    description: str  # "We receive 50 PDFs per week, manually enter into ERP"
    role_owner: str  # "Finance team - 3 people"
    frequency: str  # "daily", "weekly", "monthly", "ad-hoc"
    volume: str  # "50/week", "200/month"
    avg_time_minutes: int  # per occurrence
    pain_points: List[str]  # ["Manual data entry", "Errors", "Delays"]
    tools_used: Optional[str]  # "Email, Excel, SAP"
    current_workarounds: Optional[str]

class DiscoverySession(BaseModel):
    company_name: str
    contact_name: str
    industry: str
    company_size: str  # "10-50", "50-200", "200-1000", "1000+"
    primary_goal: Literal["cost_reduction", "speed", "accuracy", "scalability", "compliance"]
    processes: List[ProcessEntry]
    additional_context: Optional[str]
```

---

## Phase 3: Process Analyzer (Days 4-5)

### Step 3: Automation potential classifier

```python
# src/analyzer/process_analyzer.py
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import JsonOutputParser
from src.interview.schema import ProcessEntry

def analyze_automation_potential(process: ProcessEntry) -> dict:
    llm = ChatOpenAI(model=os.getenv("CHAT_MODEL", "gpt-4o-mini"), temperature=0)
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", """Analyze a business process for automation potential.
Return JSON:
{
  "automation_type": "RPA|AI|WORKFLOW|HYBRID|MANUAL_ONLY",
  "potential_score": 1-10,
  "complexity": "LOW|MEDIUM|HIGH",
  "key_benefits": ["list of main automation benefits"],
  "risks": ["list of implementation risks"],
  "recommended_approach": "1-2 sentence approach description",
  "example_solution": "specific tool or technology name"
}

Scoring: 10 = highly automatable (repetitive, rule-based, high volume), 1 = requires human judgment"""),
        ("human", """Process: {name}
Description: {description}
Frequency: {frequency}, Volume: {volume}
Time: {time} min per occurrence
Pain points: {pain_points}
Tools: {tools}""")
    ])
    
    chain = prompt | llm | JsonOutputParser()
    return chain.invoke({
        "name": process.name,
        "description": process.description,
        "frequency": process.frequency,
        "volume": process.volume,
        "time": process.avg_time_minutes,
        "pain_points": "; ".join(process.pain_points),
        "tools": process.tools_used or "Unknown"
    })
```

---

## Phase 4: ROI Modeler (Days 6-7)

### Step 4: ROI calculator

```python
# src/roi/roi_modeler.py
from src.interview.schema import ProcessEntry, DiscoverySession

FULLY_LOADED_RATE_PER_HOUR = 75  # default fully loaded employee cost per hour
AUTOMATION_EFFICIENCY_FACTOR = 0.85  # 85% of manual time eliminated after automation

def calculate_process_roi(process: ProcessEntry, potential_score: int) -> dict:
    """Calculate annual ROI estimate for automating a process."""
    if potential_score < 4:
        return {"automatable": False, "annual_hours_saved": 0, "annual_cost_saved": 0}
    
    # Calculate annual occurrence volume
    volume_per_week = _parse_volume(process.volume)
    annual_occurrences = volume_per_week * 52
    
    total_hours_per_year = (annual_occurrences * process.avg_time_minutes) / 60
    hours_saved = total_hours_per_year * AUTOMATION_EFFICIENCY_FACTOR * (potential_score / 10)
    cost_saved = hours_saved * FULLY_LOADED_RATE_PER_HOUR
    
    # Rough implementation cost estimate
    impl_cost = 25000 if potential_score >= 7 else 15000
    payback_months = round((impl_cost / (cost_saved / 12)) if cost_saved > 0 else 999)
    
    return {
        "automatable": True,
        "annual_occurrences": int(annual_occurrences),
        "total_annual_hours": round(total_hours_per_year),
        "annual_hours_saved": round(hours_saved),
        "annual_cost_saved": round(cost_saved),
        "estimated_implementation_cost": impl_cost,
        "payback_months": payback_months,
        "three_year_roi": round(cost_saved * 3 - impl_cost)
    }

def _parse_volume(volume_str: str) -> float:
    """Convert volume strings like '50/week', '200/month' to weekly number."""
    import re
    numbers = re.findall(r'\d+', volume_str)
    if not numbers:
        return 10  # default
    n = float(numbers[0])
    if "day" in volume_str.lower():
        return n * 5
    if "month" in volume_str.lower():
        return n / 4.3
    return n  # default: per week
```

---

## Phase 5: Executive Report Generator (Days 8-9)

### Step 5: Report template

Create `data/templates/discovery_report.md.j2`:
```
# Automation Discovery Report
**Company:** {{ company }}  |  **Date:** {{ date }}  |  **Prepared by:** AI Consulting

## Executive Summary

{{ executive_summary }}

## Opportunity Prioritization

| Process | Score | Annual Hours Saved | Annual Savings | Payback |
|---------|-------|--------------------|---------------|---------|
{% for opp in opportunities %}
| {{ opp.name }} | {{ opp.score }}/10 | {{ opp.hours_saved }} hrs | ${{ opp.cost_saved | int }} | {{ opp.payback_months }} mo |
{% endfor %}

## Top Recommendations

{% for opp in top_3 %}
### {{ loop.index }}. {{ opp.name }}

**Approach:** {{ opp.recommended_approach }}

**Estimated 3-Year ROI:** ${{ opp.three_year_roi | int }}

---
{% endfor %}

## Investment Summary

| Phase | Scope | Investment |
|-------|-------|-----------|
| Discovery | Detailed requirements | $10,000 |
| Pilot | Top 1-2 processes | $40,000 - $75,000 |
| Rollout | All identified processes | $100,000 - $200,000 |
```

### Step 6: Report assembler

```python
# src/reporter/report_assembler.py
from jinja2 import Environment, FileSystemLoader
from datetime import datetime
from docx import Document
import os

def generate_report_markdown(session, opportunities: list, executive_summary: str) -> str:
    env = Environment(loader=FileSystemLoader("data/templates"))
    template = env.get_template("discovery_report.md.j2")
    return template.render(
        company=session.company_name,
        date=datetime.now().strftime("%B %d, %Y"),
        executive_summary=executive_summary,
        opportunities=opportunities,
        top_3=opportunities[:3]
    )
```

---

## Phase 6: Executive Summary Generator

### Step 7: LLM executive summary

```python
# src/analyzer/summary_generator.py
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

def generate_executive_summary(session, opportunities: list) -> str:
    llm = ChatOpenAI(model=os.getenv("CHAT_MODEL", "gpt-4o-mini"), temperature=0.3)
    
    top_processes = [f"{o['name']} (score: {o['score']}/10, savings: ${o['cost_saved']:,}/yr)" 
                     for o in opportunities[:3]]
    total_savings = sum(o["cost_saved"] for o in opportunities if o.get("automatable"))
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", "Write a 3-paragraph executive summary for a process automation discovery report. Professional, results-focused tone."),
        ("human", """Company: {company}, Industry: {industry}
Total annual savings potential: ${total_savings:,}
Top opportunities: {top_processes}
Primary business goal: {goal}""")
    ])
    
    chain = prompt | llm
    return chain.invoke({
        "company": session.company_name,
        "industry": session.industry,
        "total_savings": total_savings,
        "top_processes": "; ".join(top_processes),
        "goal": session.primary_goal
    }).content
```

---

## Phase 7: FastAPI and Streamlit Wizard (Days 10-12)

### Step 8: FastAPI endpoint

```python
# app.py
from fastapi import FastAPI
from fastapi.responses import FileResponse
from src.interview.schema import DiscoverySession
from src.analyzer.process_analyzer import analyze_automation_potential
from src.roi.roi_modeler import calculate_process_roi
from src.analyzer.summary_generator import generate_executive_summary
from src.reporter.report_assembler import generate_report_markdown
import tempfile

app = FastAPI(title="Automation Discovery API")

@app.post("/analyze")
def analyze(session: DiscoverySession):
    opportunities = []
    for process in session.processes:
        analysis = analyze_automation_potential(process)
        roi = calculate_process_roi(process, analysis.get("potential_score", 5))
        opportunities.append({
            "name": process.name,
            "score": analysis.get("potential_score"),
            "automation_type": analysis.get("automation_type"),
            "recommended_approach": analysis.get("recommended_approach"),
            "key_benefits": analysis.get("key_benefits", []),
            **roi
        })
    
    # Sort by ROI descending
    opportunities.sort(key=lambda x: x.get("three_year_roi", 0), reverse=True)
    
    exec_summary = generate_executive_summary(session, opportunities)
    report_md = generate_report_markdown(session, opportunities, exec_summary)
    
    return {
        "opportunities": opportunities,
        "executive_summary": exec_summary,
        "report_markdown": report_md,
        "total_annual_savings": sum(o.get("annual_cost_saved", 0) for o in opportunities)
    }
```

---

## Acceptance Criteria

- [ ] Process analyzer returns automation_type from the defined enum
- [ ] ROI calculator handles "200/month" and "50/week" volume formats correctly
- [ ] Opportunities are sorted by 3-year ROI descending
- [ ] Executive summary includes company name and total savings figure
- [ ] Jinja2 report renders correctly for 3-5 processes
- [ ] DOCX export is formatted and ready to send to a client
- [ ] Streamlit wizard collects all required fields across multiple steps
