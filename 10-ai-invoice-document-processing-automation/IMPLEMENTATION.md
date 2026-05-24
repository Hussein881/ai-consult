# Implementation Guide: AI Invoice and Document Processing Automation

## Prerequisites

- Python 3.11+
- OpenAI API key (GPT-4o for vision-based extraction)
- `pypdf`, `Pillow`, `pytesseract` (optional for scanned docs)
- Sample invoice PDFs (download free templates from internet)

---

## Architecture Overview

```
Invoice / Document (PDF or image)
          │
          ▼
   [Document Ingestion]  → determine digital vs scanned
          │
          ▼
   [Extraction Layer]
    ├── Digital PDF → direct text extract → LLM field extraction
    └── Scanned image → OCR → LLM field extraction
          │
          ▼
   [Validation Engine]  → field checks, business rules, duplicate detection
          │
          ▼
   [Export Layer]  → JSON, CSV, structured output for ERP
          │
          ▼
   [FastAPI + Streamlit UI]
```

---

## Phase 1: Project Setup (Day 1)

```bash
mkdir invoice-processing
cd invoice-processing
python -m venv venv && source venv/bin/activate
mkdir -p src/{extractor,validator,exporter,database} tests data/sample_invoices
touch app.py ui.py .env
pip install fastapi uvicorn openai python-dotenv streamlit pydantic \
            sqlalchemy pypdf pillow python-multipart pandas
pip freeze > requirements.txt
```

---

## Phase 2: Document Text Extraction (Days 2-3)

### Step 2: PDF text extractor

```python
# src/extractor/pdf_extractor.py
import pypdf
from pathlib import Path

def extract_text_from_pdf(file_bytes: bytes) -> str:
    """Extract text from a digital PDF."""
    import io
    reader = pypdf.PdfReader(io.BytesIO(file_bytes))
    pages = []
    for page in reader.pages:
        text = page.extract_text()
        if text:
            pages.append(text)
    return "\n\n".join(pages)

def is_scanned_pdf(file_bytes: bytes) -> bool:
    """Check if PDF appears to be a scanned image (minimal extractable text)."""
    text = extract_text_from_pdf(file_bytes)
    return len(text.strip()) < 100  # less than 100 chars = likely scanned
```

### Step 3: LLM field extractor (GPT-4o)

```python
# src/extractor/llm_extractor.py
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import JsonOutputParser
from pydantic import BaseModel
from typing import List, Optional

class LineItem(BaseModel):
    description: str
    quantity: Optional[float]
    unit_price: Optional[float]
    total: Optional[float]

class InvoiceFields(BaseModel):
    vendor_name: Optional[str]
    vendor_address: Optional[str]
    invoice_number: Optional[str]
    invoice_date: Optional[str]
    due_date: Optional[str]
    subtotal: Optional[float]
    tax_amount: Optional[float]
    total_amount: Optional[float]
    currency: Optional[str]
    payment_terms: Optional[str]
    line_items: List[LineItem] = []
    po_number: Optional[str]
    extraction_confidence: str  # HIGH, MEDIUM, LOW

def extract_invoice_fields(document_text: str) -> InvoiceFields:
    llm = ChatOpenAI(model="gpt-4o", temperature=0)  # Use gpt-4o for best extraction
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", """Extract structured invoice data from the provided document text.
Return valid JSON with these fields (use null if not found):
vendor_name, vendor_address, invoice_number, invoice_date, due_date,
subtotal, tax_amount, total_amount, currency, payment_terms, po_number,
line_items (array of: description, quantity, unit_price, total),
extraction_confidence (HIGH if most fields found, MEDIUM if partial, LOW if few fields).

Return only the JSON object, no markdown."""),
        ("human", "Document text:\n{text}")
    ])
    
    chain = prompt | llm | JsonOutputParser()
    result = chain.invoke({"text": document_text[:6000]})
    return InvoiceFields(**result)
```

---

## Phase 3: Validation Engine (Days 4-5)

### Step 4: Field-level validator

```python
# src/validator/invoice_validator.py
from src.extractor.llm_extractor import InvoiceFields
from typing import List
from pydantic import BaseModel

class ValidationResult(BaseModel):
    is_valid: bool
    missing_fields: List[str]
    warnings: List[str]
    errors: List[str]
    data_quality_score: float  # 0.0 - 1.0

REQUIRED_FIELDS = ["vendor_name", "invoice_number", "invoice_date", "total_amount"]

def validate_invoice(invoice: InvoiceFields) -> ValidationResult:
    missing = []
    warnings = []
    errors = []
    
    for field in REQUIRED_FIELDS:
        if getattr(invoice, field) is None:
            missing.append(field)
    
    # Math validation
    if invoice.subtotal and invoice.tax_amount and invoice.total_amount:
        expected_total = (invoice.subtotal or 0) + (invoice.tax_amount or 0)
        if abs(expected_total - invoice.total_amount) > 0.10:
            errors.append(f"Total mismatch: subtotal+tax={expected_total:.2f}, total={invoice.total_amount:.2f}")
    
    # Reasonable amount check
    if invoice.total_amount and invoice.total_amount > 500_000:
        warnings.append(f"Unusually large amount: {invoice.total_amount:,.2f}")
    
    if invoice.total_amount and invoice.total_amount < 0:
        errors.append("Negative total amount")
    
    # Score
    found_required = len(REQUIRED_FIELDS) - len(missing)
    score = found_required / len(REQUIRED_FIELDS)
    
    return ValidationResult(
        is_valid=len(errors) == 0 and len(missing) == 0,
        missing_fields=missing,
        warnings=warnings,
        errors=errors,
        data_quality_score=round(score, 2)
    )
```

---

## Phase 4: Database and Duplicate Detection (Days 6-7)

### Step 5: Storage and deduplication

```python
# src/database/models.py
from sqlalchemy import Column, Integer, String, Float, DateTime, Text, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from datetime import datetime
import os

Base = declarative_base()

class ProcessedInvoice(Base):
    __tablename__ = "invoices"
    id = Column(Integer, primary_key=True)
    invoice_number = Column(String, index=True)
    vendor_name = Column(String)
    invoice_date = Column(String)
    total_amount = Column(Float)
    currency = Column(String)
    quality_score = Column(Float)
    is_valid = Column(Integer)  # 0 or 1
    raw_json = Column(Text)
    created_at = Column(DateTime, default=datetime.utcnow)

engine = create_engine(os.getenv("DATABASE_URL", "sqlite:///./invoices.db"))
Base.metadata.create_all(engine)
SessionLocal = sessionmaker(bind=engine)

def check_duplicate(invoice_number: str, vendor_name: str, db) -> bool:
    existing = db.query(ProcessedInvoice).filter(
        ProcessedInvoice.invoice_number == invoice_number,
        ProcessedInvoice.vendor_name == vendor_name
    ).first()
    return existing is not None
```

---

## Phase 5: FastAPI and Streamlit (Days 8-11)

### Step 6: FastAPI upload endpoint

```python
# app.py
from fastapi import FastAPI, UploadFile, File, HTTPException
from src.extractor.pdf_extractor import extract_text_from_pdf
from src.extractor.llm_extractor import extract_invoice_fields
from src.validator.invoice_validator import validate_invoice
from src.database.models import SessionLocal, ProcessedInvoice, check_duplicate
import json

app = FastAPI(title="Invoice Processing API")

@app.post("/process")
async def process_invoice(file: UploadFile = File(...)):
    if not file.filename.endswith(".pdf"):
        raise HTTPException(status_code=400, detail="Only PDF files supported")
    
    content = await file.read()
    text = extract_text_from_pdf(content)
    
    if len(text.strip()) < 50:
        raise HTTPException(status_code=422, detail="Could not extract text. Scanned PDFs require OCR.")
    
    invoice = extract_invoice_fields(text)
    validation = validate_invoice(invoice)
    
    db = SessionLocal()
    is_duplicate = False
    if invoice.invoice_number and invoice.vendor_name:
        is_duplicate = check_duplicate(invoice.invoice_number, invoice.vendor_name, db)
    
    if not is_duplicate:
        db.add(ProcessedInvoice(
            invoice_number=invoice.invoice_number,
            vendor_name=invoice.vendor_name,
            invoice_date=invoice.invoice_date,
            total_amount=invoice.total_amount,
            currency=invoice.currency,
            quality_score=validation.data_quality_score,
            is_valid=int(validation.is_valid),
            raw_json=json.dumps(invoice.dict())
        ))
        db.commit()
    db.close()
    
    return {
        "filename": file.filename,
        "extracted_fields": invoice.dict(),
        "validation": validation.dict(),
        "is_duplicate": is_duplicate
    }
```

---

## Acceptance Criteria

- [ ] Digital PDF text extraction returns readable text for test invoices
- [ ] LLM extracts vendor_name, invoice_number, total_amount from 10 test invoices
- [ ] Validation correctly flags missing required fields
- [ ] Math validation catches total/subtotal+tax mismatches > $0.10
- [ ] Duplicate detection prevents re-processing same invoice number+vendor
- [ ] CSV export of processed invoices is formatted for accounting import
- [ ] Streamlit shows extraction confidence score per invoice
