# Implementation Guide: AI Operations Reporting Assistant

## Prerequisites

- Python 3.11+
- OpenAI API key
- Sample operational data: CSV exports from Jira, spreadsheets, or mock data
- `pandas`, `jinja2`, and `langchain` installed

---

## Architecture Overview

```
Operational Inputs (CSV, JSON, manual form entries)
          │
          ▼
   [Data Aggregation Layer]  → Pandas normalization
          │
          ▼
   [FastAPI Backend]
          │
    ┌─────┴──────────────┐
    │ LangChain           │  → map-reduce summarization
    │ Summarization Chain │
    └─────┬──────────────┘
          │
    ┌─────┴───────────────┐
    │ Report Assembler    │  → Jinja2 → Markdown → HTML email or PDF
    └─────┬───────────────┘
          │
   [Streamlit Dashboard]  + [Email/Slack distribution]
```

---

## Phase 1: Project Setup (Day 1)

```bash
mkdir ops-reporting-assistant
cd ops-reporting-assistant
python -m venv venv && source venv/bin/activate
mkdir -p src/{aggregator,summarizer,reporter,distributor} data/inputs templates tests
touch app.py ui.py .env
pip install fastapi uvicorn openai langchain langchain-openai python-dotenv \
            streamlit pydantic pandas sqlalchemy jinja2 python-multipart openpyxl
pip freeze > requirements.txt
```

---

## Phase 2: Data Aggregation (Days 2-4)

### Step 2: Unified operational input schema

```python
# src/aggregator/schema.py
from pydantic import BaseModel
from typing import List, Optional

class TeamUpdate(BaseModel):
    team_name: str
    period: str  # "Week of 2024-01-15"
    accomplishments: List[str]
    blockers: List[str]
    risks: List[str]
    metrics: dict  # {"metric_name": value}
    next_week_goals: Optional[List[str]]

class AggregatedReport(BaseModel):
    report_period: str
    teams: List[TeamUpdate]
    total_blockers: int
    total_risks: int
```

### Step 3: CSV ingestion for team updates

```python
# src/aggregator/csv_ingester.py
import pandas as pd
from typing import List
from .schema import TeamUpdate

def ingest_team_updates_csv(csv_path: str) -> List[TeamUpdate]:
    """
    Expected CSV columns: team, period, accomplishments (pipe-separated),
    blockers (pipe-separated), risks (pipe-separated)
    """
    df = pd.read_csv(csv_path)
    updates = []
    
    for _, row in df.iterrows():
        updates.append(TeamUpdate(
            team_name=str(row.get("team", "")),
            period=str(row.get("period", "")),
            accomplishments=[a.strip() for a in str(row.get("accomplishments", "")).split("|") if a.strip()],
            blockers=[b.strip() for b in str(row.get("blockers", "")).split("|") if b.strip()],
            risks=[r.strip() for r in str(row.get("risks", "")).split("|") if r.strip()],
            metrics={},
            next_week_goals=[]
        ))
    return updates
```

### Step 4: Create synthetic sample data

```python
# data/create_sample.py
import csv

rows = [
    {"team": "Engineering", "period": "Week of 2024-01-15",
     "accomplishments": "Shipped v2.3 API|Fixed critical auth bug|Onboarded 2 new devs",
     "blockers": "Waiting on design approval for new dashboard|DB migration blocked by infra",
     "risks": "Q1 deadline at risk if design not resolved by Friday"},
    {"team": "Product", "period": "Week of 2024-01-15",
     "accomplishments": "Completed user research for mobile feature|Updated roadmap for Q1",
     "blockers": "Need engineering estimate for search feature",
     "risks": "Competitor launched similar feature last week"},
]

with open("data/inputs/sample_updates.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=rows[0].keys())
    writer.writeheader()
    writer.writerows(rows)
```

---

## Phase 3: LLM Summarization Chain (Days 5-6)

### Step 5: Map-reduce summarization

```python
# src/summarizer/report_summarizer.py
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

llm = ChatOpenAI(model=os.getenv("CHAT_MODEL", "gpt-4o-mini"), temperature=0)

def summarize_team_section(team_update: dict) -> str:
    """Generate a concise narrative for one team's update."""
    prompt = ChatPromptTemplate.from_messages([
        ("system", """Write a concise weekly status summary for a team. 
2-3 sentences per section. Include: accomplishments, blockers requiring escalation, risks.
Professional tone suitable for executive review."""),
        ("human", """Team: {team_name} | Period: {period}
Accomplishments: {accomplishments}
Blockers: {blockers}
Risks: {risks}""")
    ])
    chain = prompt | llm
    return chain.invoke({
        "team_name": team_update["team_name"],
        "period": team_update["period"],
        "accomplishments": "; ".join(team_update["accomplishments"]),
        "blockers": "; ".join(team_update["blockers"]),
        "risks": "; ".join(team_update["risks"])
    }).content

def generate_executive_summary(all_summaries: list, period: str) -> str:
    """Synthesize all team summaries into a 3-5 sentence executive digest."""
    combined = "\n\n".join([f"{s['team']}: {s['summary']}" for s in all_summaries])
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", """Write a 3-5 sentence executive summary of the weekly operations report.
Focus on: overall progress, critical blockers requiring executive attention, and key risks.
Start with the most important item. No filler language."""),
        ("human", "Period: {period}\n\nTeam summaries:\n{combined}")
    ])
    chain = prompt | llm
    return chain.invoke({"period": period, "combined": combined}).content
```

---

## Phase 4: Report Template and Assembly (Days 7-8)

### Step 6: Jinja2 report template

Create `templates/weekly_report.md.j2`:
```
# Weekly Operations Report: {{ period }}

## Executive Summary

{{ executive_summary }}

---

{% for team in teams %}
## {{ team.team_name }}

{{ team.summary }}

**Key Blockers:**
{% for blocker in team.blockers %}
- {{ blocker }}
{% endfor %}

**Risks:**
{% for risk in team.risks %}
- ⚠️ {{ risk }}
{% endfor %}

---
{% endfor %}

*Generated: {{ generated_at }}*
```

### Step 7: Report assembler

```python
# src/reporter/assembler.py
from jinja2 import Environment, FileSystemLoader
from datetime import datetime
from src.summarizer.report_summarizer import summarize_team_section, generate_executive_summary

def assemble_report(aggregated_data) -> str:
    team_summaries = []
    for team in aggregated_data.teams:
        summary = summarize_team_section(team.dict())
        team_summaries.append({
            "team_name": team.team_name,
            "summary": summary,
            "blockers": team.blockers,
            "risks": team.risks
        })
    
    exec_summary = generate_executive_summary(
        [{"team": t["team_name"], "summary": t["summary"]} for t in team_summaries],
        aggregated_data.report_period
    )
    
    env = Environment(loader=FileSystemLoader("templates"))
    template = env.get_template("weekly_report.md.j2")
    
    return template.render(
        period=aggregated_data.report_period,
        executive_summary=exec_summary,
        teams=team_summaries,
        generated_at=datetime.now().strftime("%Y-%m-%d %H:%M UTC")
    )
```

---

## Phase 5: FastAPI and Streamlit (Days 9-11)

### Step 8: FastAPI endpoint

```python
# app.py
from fastapi import FastAPI, UploadFile, File
from src.aggregator.csv_ingester import ingest_team_updates_csv
from src.aggregator.schema import AggregatedReport
from src.reporter.assembler import assemble_report
import tempfile, os

app = FastAPI(title="Operations Reporting API")

@app.post("/generate-report")
async def generate_report(file: UploadFile = File(...), period: str = "This Week"):
    content = await file.read()
    with tempfile.NamedTemporaryFile(suffix=".csv", delete=False, mode="wb") as tmp:
        tmp.write(content)
        tmp_path = tmp.name
    
    teams = ingest_team_updates_csv(tmp_path)
    os.unlink(tmp_path)
    
    aggregated = AggregatedReport(
        report_period=period,
        teams=teams,
        total_blockers=sum(len(t.blockers) for t in teams),
        total_risks=sum(len(t.risks) for t in teams)
    )
    
    report = assemble_report(aggregated)
    return {"period": period, "report": report, "total_blockers": aggregated.total_blockers}
```

---

## Acceptance Criteria

- [ ] CSV ingestion correctly parses team, accomplishments, blockers, risks columns
- [ ] LLM summarization produces coherent team narratives
- [ ] Executive summary highlights the most critical blockers
- [ ] Jinja2 template renders all teams with correct data
- [ ] Report generation for 5 teams completes in under 45 seconds
- [ ] Streamlit UI allows download of generated report as Markdown
