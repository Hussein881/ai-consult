# Implementation Guide: AI Technical Documentation Generator

## Prerequisites

- Python 3.11+
- OpenAI API key
- A sample Python codebase or OpenAPI spec to test against
- `python-docx` and `jinja2` installed

---

## Architecture Overview

```
Code Files / OpenAPI Specs / Runbooks
           │
           ▼
   [Source Parser]  → extract function signatures, docstrings, endpoints
           │
           ▼
  [Prompt Builder]  → construct structured documentation prompt per item
           │
           ▼
   [LLM Generator]  → generate documentation section
           │
           ▼
 [Template Assembler]  → Jinja2 → full Markdown or DOCX output
           │
           ▼
  [Review Interface]  → human review, edit, approve
           │
           ▼
   [Export / Publish]  → Markdown, DOCX, PDF, or Confluence
```

---

## Phase 1: Project Setup (Day 1)

### Step 1: Create structure

```bash
mkdir doc-generator
cd doc-generator
python -m venv venv && source venv/bin/activate

mkdir -p src/{parser,generator,templates,exporter} tests data/sample_code
touch app.py ui.py .env
```

### Step 2: Install dependencies

```bash
pip install fastapi uvicorn openai langchain langchain-openai python-dotenv \
            streamlit pydantic jinja2 python-docx pypdf python-multipart
pip freeze > requirements.txt
```

---

## Phase 2: Code Parser (Days 2-3)

### Step 3: Python AST parser

```python
# src/parser/python_parser.py
import ast
from typing import List, Optional
from pydantic import BaseModel

class FunctionInfo(BaseModel):
    name: str
    args: List[str]
    returns: Optional[str]
    docstring: Optional[str]
    body_summary: str  # first line or brief description

def extract_functions(source_code: str) -> List[FunctionInfo]:
    """Extract function signatures and docstrings from Python source."""
    tree = ast.parse(source_code)
    functions = []
    
    for node in ast.walk(tree):
        if isinstance(node, (ast.FunctionDef, ast.AsyncFunctionDef)):
            args = [a.arg for a in node.args.args]
            docstring = ast.get_docstring(node)
            returns = ast.unparse(node.returns) if node.returns else None
            
            # Get first statement for body context
            body_first = ""
            if node.body and len(node.body) > 1:
                try:
                    body_first = ast.unparse(node.body[1])[:100]
                except Exception:
                    pass
            
            functions.append(FunctionInfo(
                name=node.name,
                args=args,
                returns=returns,
                docstring=docstring,
                body_summary=body_first
            ))
    
    return functions
```

### Step 4: OpenAPI spec parser

```python
# src/parser/openapi_parser.py
import json
import yaml
from typing import List
from pydantic import BaseModel

class EndpointInfo(BaseModel):
    method: str
    path: str
    summary: Optional[str]
    description: Optional[str]
    parameters: List[dict]
    request_body: Optional[dict]
    responses: dict

def parse_openapi_spec(content: str) -> List[EndpointInfo]:
    """Parse OpenAPI JSON or YAML spec into endpoint list."""
    try:
        spec = json.loads(content)
    except json.JSONDecodeError:
        spec = yaml.safe_load(content)
    
    endpoints = []
    for path, path_item in spec.get("paths", {}).items():
        for method, operation in path_item.items():
            if method in ("get", "post", "put", "delete", "patch"):
                endpoints.append(EndpointInfo(
                    method=method.upper(),
                    path=path,
                    summary=operation.get("summary"),
                    description=operation.get("description"),
                    parameters=operation.get("parameters", []),
                    request_body=operation.get("requestBody"),
                    responses=operation.get("responses", {})
                ))
    return endpoints
```

---

## Phase 3: LLM Documentation Generator (Days 4-5)

### Step 5: Function documentation generator

```python
# src/generator/doc_generator.py
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

llm = ChatOpenAI(model=os.getenv("CHAT_MODEL", "gpt-4o-mini"), temperature=0)

FUNCTION_PROMPT = ChatPromptTemplate.from_messages([
    ("system", """You are a technical documentation writer. Generate clear, accurate documentation 
     for the given function. Format as Markdown with sections: Description, Parameters, Returns, Example.
     Be concise. Do not invent behavior not shown in the function signature."""),
    ("human", """Function: {name}
Arguments: {args}
Returns: {returns}
Existing docstring: {docstring}
Body hint: {body_summary}

Generate documentation:""")
])

def document_function(func_info) -> str:
    chain = FUNCTION_PROMPT | llm
    result = chain.invoke({
        "name": func_info.name,
        "args": ", ".join(func_info.args),
        "returns": func_info.returns or "None",
        "docstring": func_info.docstring or "(none)",
        "body_summary": func_info.body_summary
    })
    return result.content

API_PROMPT = ChatPromptTemplate.from_messages([
    ("system", """Generate clear REST API documentation for the endpoint. 
     Include: Description, Parameters, Request Body, Response, Example curl command."""),
    ("human", """Endpoint: {method} {path}
Summary: {summary}
Parameters: {parameters}
Responses: {responses}

Generate API documentation:""")
])

def document_endpoint(endpoint) -> str:
    chain = API_PROMPT | llm
    import json
    result = chain.invoke({
        "method": endpoint.method,
        "path": endpoint.path,
        "summary": endpoint.summary or "(none)",
        "parameters": json.dumps(endpoint.parameters[:5], indent=2),
        "responses": json.dumps(list(endpoint.responses.keys()))
    })
    return result.content
```

---

## Phase 4: Template Assembly and Export (Days 6-7)

### Step 6: Jinja2 template

Create `src/templates/api_docs.md.j2`:
```
# API Documentation

Generated: {{ generated_at }}

## Overview

{{ overview }}

{% for endpoint in endpoints %}
---

### {{ endpoint.method }} {{ endpoint.path }}

{{ endpoint.documentation }}

{% endfor %}
```

### Step 7: Document assembler

```python
# src/exporter/assembler.py
from jinja2 import Environment, FileSystemLoader
from datetime import datetime
from pathlib import Path
from docx import Document

def assemble_markdown(endpoints_with_docs: list, overview: str = "") -> str:
    env = Environment(loader=FileSystemLoader("src/templates"))
    template = env.get_template("api_docs.md.j2")
    return template.render(
        generated_at=datetime.now().strftime("%Y-%m-%d %H:%M"),
        overview=overview,
        endpoints=endpoints_with_docs
    )

def export_docx(markdown_content: str, output_path: str):
    doc = Document()
    doc.add_heading("API Documentation", 0)
    
    for line in markdown_content.splitlines():
        if line.startswith("# "):
            doc.add_heading(line[2:], level=1)
        elif line.startswith("## "):
            doc.add_heading(line[3:], level=2)
        elif line.startswith("### "):
            doc.add_heading(line[4:], level=3)
        elif line.strip():
            doc.add_paragraph(line)
    
    doc.save(output_path)
```

---

## Phase 5: FastAPI and Streamlit (Days 8-10)

### Step 8: FastAPI endpoint

```python
# app.py
from fastapi import FastAPI, UploadFile, File
from fastapi.responses import FileResponse
from src.parser.python_parser import extract_functions
from src.parser.openapi_parser import parse_openapi_spec
from src.generator.doc_generator import document_function, document_endpoint
from src.exporter.assembler import assemble_markdown, export_docx
import tempfile, os

app = FastAPI(title="Documentation Generator API")

@app.post("/generate/python")
async def generate_python_docs(file: UploadFile = File(...)):
    content = (await file.read()).decode("utf-8")
    functions = extract_functions(content)
    results = []
    for func in functions[:20]:  # limit for demo
        doc = document_function(func)
        results.append({"name": func.name, "documentation": doc})
    return {"functions": results, "count": len(results)}

@app.post("/generate/api")
async def generate_api_docs(file: UploadFile = File(...)):
    content = (await file.read()).decode("utf-8")
    endpoints = parse_openapi_spec(content)
    results = []
    for ep in endpoints[:10]:
        doc = document_endpoint(ep)
        results.append({"method": ep.method, "path": ep.path, "documentation": doc})
    return {"endpoints": results, "count": len(results)}
```

---

## Acceptance Criteria

- [ ] Python parser extracts function names, args, and return types correctly
- [ ] OpenAPI parser handles both JSON and YAML spec formats
- [ ] LLM generates structured documentation with correct section headers
- [ ] DOCX export is well-formatted and readable
- [ ] End-to-end generation for a 20-function Python file completes in under 60 seconds
- [ ] Streamlit review interface allows editing before export
