# Implementation Guide: Internal Engineering Knowledge Assistant

## Prerequisites

- Python 3.11+
- OpenAI API key
- A collection of internal documents (Markdown, PDFs) — use public GitHub docs as stand-ins during development
- ChromaDB (no account needed, runs locally)

---

## Architecture Overview

```
Documents (Markdown, PDF, text)
          │
          ▼
   [Ingestion Pipeline]
          │
    ┌─────┴──────┐
    │  Chunker   │  → split docs into semantic chunks
    └─────┬──────┘
          │
    ┌─────┴──────────┐
    │ Embedding Model│  → OpenAI text-embedding-3-small
    └─────┬──────────┘
          │
    ┌─────┴──────────┐
    │  ChromaDB      │  → store vectors + metadata
    └─────┬──────────┘
          │
   [Query Pipeline]
          │
    ┌─────┴──────────┐
    │  Retriever     │  → top-k semantic search with metadata filters
    └─────┬──────────┘
          │
    ┌─────┴──────────┐
    │  LLM + Prompt  │  → generate answer with citations
    └─────┬──────────┘
          │
   [Streamlit Chat UI]
```

---

## Phase 1: Project Setup (Day 1)

### Step 1: Create structure

```bash
mkdir engineering-knowledge-assistant
cd engineering-knowledge-assistant
python -m venv venv && source venv/bin/activate

mkdir -p src/{ingestion,retrieval,generation} data/docs tests
touch app.py ui.py .env
```

### Step 2: Install dependencies

```bash
pip install fastapi uvicorn langchain langchain-openai langchain-community \
            openai chromadb pypdf python-dotenv streamlit pydantic \
            tiktoken python-multipart sentence-transformers
pip freeze > requirements.txt
```

### Step 3: Environment

```
OPENAI_API_KEY=your_key_here
EMBEDDING_MODEL=text-embedding-3-small
CHAT_MODEL=gpt-4o-mini
CHROMA_PERSIST_DIR=./chroma_db
CHUNK_SIZE=800
CHUNK_OVERLAP=100
TOP_K=5
```

---

## Phase 2: Document Ingestion Pipeline (Days 2-4)

### Step 4: Document loader

```python
# src/ingestion/loader.py
from langchain_community.document_loaders import (
    DirectoryLoader, TextLoader, PyPDFLoader, UnstructuredMarkdownLoader
)
from pathlib import Path

def load_documents(directory: str):
    """Load all supported document types from a directory."""
    docs = []
    path = Path(directory)
    
    # Markdown
    for md_file in path.rglob("*.md"):
        loader = UnstructuredMarkdownLoader(str(md_file))
        loaded = loader.load()
        for doc in loaded:
            doc.metadata["source"] = str(md_file)
            doc.metadata["type"] = "markdown"
        docs.extend(loaded)
    
    # PDF
    for pdf_file in path.rglob("*.pdf"):
        loader = PyPDFLoader(str(pdf_file))
        loaded = loader.load()
        for doc in loaded:
            doc.metadata["source"] = str(pdf_file)
            doc.metadata["type"] = "pdf"
        docs.extend(loaded)
    
    # Plain text
    for txt_file in path.rglob("*.txt"):
        loader = TextLoader(str(txt_file))
        loaded = loader.load()
        for doc in loaded:
            doc.metadata["source"] = str(txt_file)
            doc.metadata["type"] = "text"
        docs.extend(loaded)
    
    return docs
```

### Step 5: Chunker

```python
# src/ingestion/chunker.py
import os
from langchain_text_splitters import RecursiveCharacterTextSplitter

def chunk_documents(documents):
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=int(os.getenv("CHUNK_SIZE", 800)),
        chunk_overlap=int(os.getenv("CHUNK_OVERLAP", 100)),
        separators=["\n\n", "\n", ".", " ", ""]
    )
    return splitter.split_documents(documents)
```

### Step 6: Vector store builder

```python
# src/ingestion/vectorstore.py
import os
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import Chroma

def build_vectorstore(chunks):
    embeddings = OpenAIEmbeddings(model=os.getenv("EMBEDDING_MODEL", "text-embedding-3-small"))
    persist_dir = os.getenv("CHROMA_PERSIST_DIR", "./chroma_db")
    
    vectorstore = Chroma.from_documents(
        documents=chunks,
        embedding=embeddings,
        persist_directory=persist_dir
    )
    return vectorstore

def load_vectorstore():
    embeddings = OpenAIEmbeddings(model=os.getenv("EMBEDDING_MODEL", "text-embedding-3-small"))
    persist_dir = os.getenv("CHROMA_PERSIST_DIR", "./chroma_db")
    return Chroma(persist_directory=persist_dir, embedding_function=embeddings)
```

### Step 7: Ingestion script

```python
# ingest.py
from dotenv import load_dotenv
load_dotenv()

from src.ingestion.loader import load_documents
from src.ingestion.chunker import chunk_documents
from src.ingestion.vectorstore import build_vectorstore

docs = load_documents("./data/docs")
print(f"Loaded {len(docs)} documents")
chunks = chunk_documents(docs)
print(f"Created {len(chunks)} chunks")
vectorstore = build_vectorstore(chunks)
print("Vector store built successfully")
```

Run with:
```bash
python ingest.py
```

---

## Phase 3: RAG Query Pipeline (Days 5-6)

### Step 8: Retriever

```python
# src/retrieval/retriever.py
import os
from src.ingestion.vectorstore import load_vectorstore

def get_retriever(filters: dict = None):
    vectorstore = load_vectorstore()
    top_k = int(os.getenv("TOP_K", 5))
    
    if filters:
        return vectorstore.as_retriever(
            search_kwargs={"k": top_k, "filter": filters}
        )
    return vectorstore.as_retriever(search_kwargs={"k": top_k})
```

### Step 9: RAG chain with citations

```python
# src/generation/rag_chain.py
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_core.output_parsers import StrOutputParser

def format_docs_with_sources(docs):
    formatted = []
    for i, doc in enumerate(docs, 1):
        source = doc.metadata.get("source", "unknown")
        formatted.append(f"[{i}] Source: {source}\n{doc.page_content}")
    return "\n\n".join(formatted)

def build_rag_chain(retriever):
    llm = ChatOpenAI(model=os.getenv("CHAT_MODEL", "gpt-4o-mini"), temperature=0)
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", """You are an internal engineering knowledge assistant.
Answer questions using ONLY the provided documentation context.
Always cite which source number ([1], [2], etc.) your answer comes from.
If the answer is not in the context, say "I don't have information about this in the current documentation."
Never make up information."""),
        ("human", "Context:\n{context}\n\nQuestion: {question}")
    ])
    
    chain = (
        {"context": retriever | format_docs_with_sources, "question": RunnablePassthrough()}
        | prompt
        | llm
        | StrOutputParser()
    )
    return chain
```

---

## Phase 4: FastAPI Backend (Day 7)

### Step 10: API endpoints

```python
# app.py
from fastapi import FastAPI
from pydantic import BaseModel
from src.retrieval.retriever import get_retriever
from src.generation.rag_chain import build_rag_chain

app = FastAPI(title="Engineering Knowledge API")

class QueryRequest(BaseModel):
    question: str
    filter_type: str = None  # e.g. "markdown", "pdf"

@app.post("/query")
def query(request: QueryRequest):
    filters = {"type": request.filter_type} if request.filter_type else None
    retriever = get_retriever(filters)
    chain = build_rag_chain(retriever)
    answer = chain.invoke(request.question)
    
    # Also return source docs for citations
    docs = retriever.invoke(request.question)
    sources = list(set([d.metadata.get("source", "") for d in docs]))
    
    return {"answer": answer, "sources": sources}

@app.get("/health")
def health():
    return {"status": "ok"}
```

---

## Phase 5: Streamlit Chat Interface (Days 8-9)

### Step 11: Chat UI

```python
# ui.py
import streamlit as st
import requests

st.set_page_config(page_title="Engineering Knowledge Assistant", layout="wide")
st.title("Engineering Knowledge Assistant")

if "messages" not in st.session_state:
    st.session_state.messages = []

for msg in st.session_state.messages:
    with st.chat_message(msg["role"]):
        st.write(msg["content"])
        if msg.get("sources"):
            with st.expander("Sources"):
                for s in msg["sources"]:
                    st.markdown(f"- `{s}`")

if question := st.chat_input("Ask a question about our engineering docs..."):
    st.session_state.messages.append({"role": "user", "content": question})
    
    with st.chat_message("assistant"):
        with st.spinner("Searching documentation..."):
            response = requests.post(
                "http://localhost:8000/query",
                json={"question": question}
            ).json()
        
        st.write(response["answer"])
        if response.get("sources"):
            with st.expander("Sources used"):
                for s in response["sources"]:
                    st.markdown(f"- `{s}`")
        
        st.session_state.messages.append({
            "role": "assistant",
            "content": response["answer"],
            "sources": response.get("sources", [])
        })
```

---

## Acceptance Criteria

- [ ] Ingestion pipeline processes 10+ documents without errors
- [ ] Retriever returns 5 relevant chunks for test questions
- [ ] Answer always includes source citations
- [ ] Out-of-scope questions return the graceful "not in documentation" message
- [ ] Streamlit chat retains conversation history in session
- [ ] End-to-end latency under 10 seconds for a query
