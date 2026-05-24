# Implementation Guide: Internal Knowledge Assistant for Company Documents

## Prerequisites

- Python 3.11+
- OpenAI API key
- Sample company documents (use public policy templates as stand-ins)
- ChromaDB (local) or Pinecone (cloud)

This project extends Project 03 with access control, admin management, and enterprise document sources. If you have already built Project 03, approximately 60% of the code is reusable.

---

## Architecture Overview

```
Company Documents (PDF, DOCX, SharePoint, Google Drive)
          │
          ▼
   [Ingestion + Metadata Tagging]
          │ department, type, access_level tags
          ▼
   [ChromaDB Vector Store]
          │
   [JWT Auth Middleware]  → verify user role
          │
          ▼
   [Filtered Retriever]  → return only docs matching user permissions
          │
          ▼
   [RAG Chain with Citations]
          │
          ▼
   [Employee Chat UI]  + [Admin Dashboard]
```

---

## Phase 1: Project Setup (Day 1)

### Step 1: Create structure

```bash
mkdir company-knowledge-assistant
cd company-knowledge-assistant
python -m venv venv && source venv/bin/activate
mkdir -p src/{ingestion,retrieval,generation,auth,admin} data/docs tests
touch app.py ui.py admin_ui.py .env
pip install fastapi uvicorn langchain langchain-openai langchain-community \
            openai chromadb pypdf python-docx python-jose[cryptography] \
            passlib bcrypt python-dotenv streamlit pydantic python-multipart
pip freeze > requirements.txt
```

---

## Phase 2: Document Ingestion with Access Metadata (Days 2-4)

### Step 2: Document schema with access control

```python
# src/ingestion/schema.py
from pydantic import BaseModel
from typing import Literal

class DocumentMetadata(BaseModel):
    source: str
    filename: str
    department: str  # "hr", "engineering", "finance", "all"
    access_level: Literal["public", "internal", "restricted"]
    doc_type: str  # "policy", "sop", "handbook", "guide"
    version: str = "1.0"
```

### Step 3: Multi-format loader (reuse from Project 03, add DOCX)

```python
# src/ingestion/loader.py
from langchain_community.document_loaders import PyPDFLoader, UnstructuredMarkdownLoader
from docx import Document as DocxDocument
from langchain_core.documents import Document
from pathlib import Path
from .schema import DocumentMetadata

def load_docx(file_path: str, metadata: DocumentMetadata):
    doc = DocxDocument(file_path)
    content = "\n".join([p.text for p in doc.paragraphs if p.text.strip()])
    return [Document(page_content=content, metadata=metadata.dict())]

def load_with_metadata(file_path: str, metadata: DocumentMetadata):
    path = Path(file_path)
    if path.suffix == ".pdf":
        loader = PyPDFLoader(file_path)
        docs = loader.load()
    elif path.suffix in (".docx", ".doc"):
        docs = load_docx(file_path, metadata)
    elif path.suffix == ".md":
        loader = UnstructuredMarkdownLoader(file_path)
        docs = loader.load()
    else:
        raise ValueError(f"Unsupported file type: {path.suffix}")
    
    for doc in docs:
        doc.metadata.update(metadata.dict())
    return docs
```

---

## Phase 3: JWT Authentication (Days 5-6)

### Step 4: Auth system

```python
# src/auth/auth.py
import os
from datetime import datetime, timedelta
from typing import Optional
from jose import JWTError, jwt
from passlib.context import CryptContext
from fastapi import HTTPException, Depends
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

SECRET_KEY = os.getenv("JWT_SECRET_KEY", "change-me-in-production")
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 480

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
bearer_scheme = HTTPBearer()

def create_access_token(data: dict) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def decode_token(token: str) -> dict:
    try:
        return jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")

def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(bearer_scheme)):
    return decode_token(credentials.credentials)
```

---

## Phase 4: Permission-Filtered Retriever (Days 7-8)

### Step 5: Build filtered retriever

```python
# src/retrieval/retriever.py
import os
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import Chroma

def get_retriever_for_user(user: dict, top_k: int = 5):
    """Build a retriever that only returns docs the user is allowed to see."""
    embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
    vectorstore = Chroma(
        persist_directory=os.getenv("CHROMA_PERSIST_DIR", "./chroma_db"),
        embedding_function=embeddings
    )
    
    # Build filter based on user role
    user_dept = user.get("department", "all")
    user_access = user.get("access_level", "public")
    
    # Map access levels to what user can see
    visible_access = ["public"]
    if user_access in ("internal", "restricted"):
        visible_access.append("internal")
    if user_access == "restricted":
        visible_access.append("restricted")
    
    # ChromaDB $or filter
    where_filter = {
        "$and": [
            {"access_level": {"$in": visible_access}},
            {"department": {"$in": [user_dept, "all"]}}
        ]
    }
    
    return vectorstore.as_retriever(
        search_kwargs={"k": top_k, "filter": where_filter}
    )
```

---

## Phase 5: RAG Chain (Day 9)

Reuse the RAG chain from Project 03 — it is identical. The only change is passing the permission-filtered retriever.

---

## Phase 6: FastAPI with Auth and Unanswered Question Logging (Days 10-11)

### Step 6: Protected API with logging

```python
# app.py
from fastapi import FastAPI, Depends
from pydantic import BaseModel
from src.auth.auth import get_current_user, create_access_token
from src.retrieval.retriever import get_retriever_for_user
from src.generation.rag_chain import build_rag_chain
from src.database.models import SessionLocal, QueryLog

app = FastAPI(title="Company Knowledge API")

class QueryRequest(BaseModel):
    question: str

@app.post("/query")
def query(request: QueryRequest, user: dict = Depends(get_current_user)):
    retriever = get_retriever_for_user(user)
    chain = build_rag_chain(retriever)
    answer = chain.invoke(request.question)
    
    # Log unanswered questions
    is_unanswered = "don't have information" in answer.lower()
    db = SessionLocal()
    db.add(QueryLog(
        question=request.question,
        answer=answer,
        user_email=user.get("email", ""),
        department=user.get("department", ""),
        was_answered=not is_unanswered
    ))
    db.commit()
    db.close()
    
    return {"answer": answer, "is_unanswered": is_unanswered}

@app.post("/token")
def get_token(email: str, department: str, access_level: str = "internal"):
    """Demo endpoint - in production, validate against user directory."""
    return create_access_token({
        "email": email,
        "department": department,
        "access_level": access_level
    })
```

---

## Phase 7: Admin Dashboard (Day 12)

Build a Streamlit admin dashboard (`admin_ui.py`) that shows:
- Total queries this week
- Top 10 unanswered questions (sorted by frequency)
- Department usage breakdown
- Upload interface for new documents with metadata form

---

## Acceptance Criteria

- [ ] JWT token is required for all `/query` requests
- [ ] User with "public" access cannot retrieve "internal" or "restricted" documents
- [ ] Unanswered questions are logged to database
- [ ] Admin dashboard shows unanswered question log
- [ ] DOCX files are loaded and searchable alongside PDFs
- [ ] All retrieval responses include source citations
