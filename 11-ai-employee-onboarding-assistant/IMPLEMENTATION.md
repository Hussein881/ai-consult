# Implementation Guide: AI Employee Onboarding Assistant

## Prerequisites

- Python 3.11+
- OpenAI API key
- Sample HR documents: employee handbook, IT setup guide, benefits FAQ, org chart (use public templates)
- ChromaDB (local)

---

## Architecture Overview

```
HR Docs / Onboarding Checklists
          │
          ▼
   [ChromaDB Vector Store]  + metadata: department, role, doc_type
          │
          ▼
   [FastAPI Backend]
          │
    ┌─────┴────────────────┐
    │ Role-based RAG Chain │  → personalized Q&A per role
    └─────┬────────────────┘
          │
    ┌─────┴────────────────────┐
    │ Checklist Generator      │  → auto-generate 30/60/90 day plan
    └─────┬────────────────────┘
          │
    ┌─────┴──────────────┐
    │ Progress Tracker   │  → store completed items per employee
    └─────┬──────────────┘
          │
   [Streamlit Employee Portal]
```

---

## Phase 1: Project Setup (Day 1)

```bash
mkdir employee-onboarding-assistant
cd employee-onboarding-assistant
python -m venv venv && source venv/bin/activate
mkdir -p src/{ingestion,qa,checklist,progress,database} data/hr_docs tests
touch app.py ui.py .env
pip install fastapi uvicorn openai langchain langchain-openai langchain-community \
            openai chromadb pypdf python-dotenv streamlit pydantic sqlalchemy \
            python-multipart
pip freeze > requirements.txt
```

---

## Phase 2: HR Document Ingestion (Days 2-3)

### Step 2: Ingestion with role and department tags

Reuse the loader pattern from Project 03 with additional metadata:

```python
# src/ingestion/ingest_hr_docs.py
from dotenv import load_dotenv
load_dotenv()

from langchain_community.document_loaders import DirectoryLoader, PyPDFLoader, UnstructuredMarkdownLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import Chroma
from pathlib import Path
import os

METADATA_MAP = {
    "handbook.md": {"doc_type": "policy", "audience": "all", "department": "hr"},
    "it_setup.md": {"doc_type": "setup_guide", "audience": "all", "department": "it"},
    "benefits.md": {"doc_type": "benefits", "audience": "all", "department": "hr"},
    "engineering_onboarding.md": {"doc_type": "onboarding", "audience": "engineering", "department": "engineering"},
}

def ingest():
    docs_dir = "data/hr_docs"
    all_docs = []
    
    for file in Path(docs_dir).rglob("*"):
        if file.suffix in (".md", ".txt"):
            from langchain_community.document_loaders import TextLoader
            loader = TextLoader(str(file))
            docs = loader.load()
        elif file.suffix == ".pdf":
            loader = PyPDFLoader(str(file))
            docs = loader.load()
        else:
            continue
        
        meta = METADATA_MAP.get(file.name, {"doc_type": "general", "audience": "all", "department": "all"})
        for doc in docs:
            doc.metadata.update(meta)
            doc.metadata["source"] = str(file)
        all_docs.extend(docs)
    
    splitter = RecursiveCharacterTextSplitter(chunk_size=600, chunk_overlap=80)
    chunks = splitter.split_documents(all_docs)
    
    embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
    Chroma.from_documents(chunks, embeddings, persist_directory="./chroma_db")
    print(f"Ingested {len(chunks)} chunks from {len(all_docs)} documents")

if __name__ == "__main__":
    ingest()
```

---

## Phase 3: Role-Based Q&A Chain (Days 4-5)

### Step 3: Role-filtered retriever

```python
# src/qa/retriever.py
import os
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import Chroma

def get_onboarding_retriever(role: str = "all", department: str = "all"):
    embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
    vectorstore = Chroma(persist_directory="./chroma_db", embedding_function=embeddings)
    
    # Show docs targeted at role/dept or docs marked 'all'
    where = {"audience": {"$in": [department, "all"]}}
    return vectorstore.as_retriever(search_kwargs={"k": 5, "filter": where})
```

### Step 4: Onboarding Q&A chain

```python
# src/qa/qa_chain.py
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_core.output_parsers import StrOutputParser

def build_onboarding_chain(retriever, employee_name: str, role: str):
    llm = ChatOpenAI(model=os.getenv("CHAT_MODEL", "gpt-4o-mini"), temperature=0)
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", f"""You are a friendly onboarding assistant for {employee_name}, 
a new {role}. Help them navigate their onboarding journey.
Answer questions using the provided HR documentation.
Be warm, encouraging, and practical. Cite which document your answer comes from.
If the answer is not in the docs, say so and suggest they contact HR."""),
        ("human", "Context:\n{{context}}\n\nQuestion: {{question}}")
    ])
    
    def format_docs(docs):
        return "\n\n".join([f"[{d.metadata.get('source','')}]\n{d.page_content}" for d in docs])
    
    return (
        {"context": retriever | format_docs, "question": RunnablePassthrough()}
        | prompt
        | llm
        | StrOutputParser()
    )
```

---

## Phase 4: Checklist Generator (Days 6-7)

### Step 5: 30/60/90 day checklist

```python
# src/checklist/generator.py
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import JsonOutputParser

def generate_onboarding_checklist(employee_name: str, role: str, department: str) -> dict:
    llm = ChatOpenAI(model=os.getenv("CHAT_MODEL", "gpt-4o-mini"), temperature=0.2)
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", """Generate a realistic employee onboarding checklist. 
Return JSON: {
  "day_30": [{"task": "...", "category": "setup|meeting|learning|admin"}],
  "day_60": [...],
  "day_90": [...]
}
Each period should have 8-12 tasks specific to the role."""),
        ("human", "Employee: {name}, Role: {role}, Department: {department}")
    ])
    
    chain = prompt | llm | JsonOutputParser()
    return chain.invoke({"name": employee_name, "role": role, "department": department})
```

---

## Phase 5: Progress Tracking (Day 8)

### Step 6: Task completion tracker

```python
# src/progress/tracker.py
from sqlalchemy import Column, Integer, String, Boolean, DateTime, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from datetime import datetime
import os

Base = declarative_base()

class OnboardingTask(Base):
    __tablename__ = "onboarding_tasks"
    id = Column(Integer, primary_key=True)
    employee_id = Column(String, index=True)
    task = Column(String)
    category = Column(String)
    milestone = Column(String)  # "day_30", "day_60", "day_90"
    is_complete = Column(Boolean, default=False)
    completed_at = Column(DateTime, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)

engine = create_engine(os.getenv("DATABASE_URL", "sqlite:///./onboarding.db"))
Base.metadata.create_all(engine)
SessionLocal = sessionmaker(bind=engine)

def get_progress(employee_id: str, db) -> dict:
    tasks = db.query(OnboardingTask).filter(OnboardingTask.employee_id == employee_id).all()
    total = len(tasks)
    done = sum(1 for t in tasks if t.is_complete)
    return {"total": total, "completed": done, "percent": round(done / total * 100 if total else 0)}
```

---

## Phase 6: FastAPI and Streamlit (Days 9-11)

### Step 7: FastAPI endpoints

```python
# app.py
from fastapi import FastAPI
from pydantic import BaseModel
from src.qa.retriever import get_onboarding_retriever
from src.qa.qa_chain import build_onboarding_chain
from src.checklist.generator import generate_onboarding_checklist
from src.progress.tracker import SessionLocal, OnboardingTask, get_progress

app = FastAPI(title="Employee Onboarding API")

class OnboardingQuery(BaseModel):
    employee_name: str
    role: str
    department: str
    question: str

class TaskCompletion(BaseModel):
    employee_id: str
    task_id: int

@app.post("/ask")
def ask(query: OnboardingQuery):
    retriever = get_onboarding_retriever(role=query.role, department=query.department)
    chain = build_onboarding_chain(retriever, query.employee_name, query.role)
    answer = chain.invoke(query.question)
    return {"answer": answer}

@app.post("/checklist/generate")
def generate_checklist(employee_id: str, employee_name: str, role: str, department: str):
    checklist = generate_onboarding_checklist(employee_name, role, department)
    db = SessionLocal()
    for milestone, tasks in checklist.items():
        for task in tasks:
            db.add(OnboardingTask(
                employee_id=employee_id,
                task=task["task"],
                category=task.get("category", "general"),
                milestone=milestone
            ))
    db.commit()
    db.close()
    return checklist

@app.get("/progress/{employee_id}")
def progress(employee_id: str):
    db = SessionLocal()
    result = get_progress(employee_id, db)
    db.close()
    return result
```

---

## Acceptance Criteria

- [ ] RAG Q&A answers basic HR questions (vacation policy, IT setup steps) correctly
- [ ] Checklist generator returns tasks in three milestone groups
- [ ] Tasks are stored per employee and track completion status
- [ ] Progress endpoint returns correct percentage completion
- [ ] Streamlit employee portal shows checklist with checkboxes
- [ ] "Not in documentation" response is returned for out-of-scope questions
