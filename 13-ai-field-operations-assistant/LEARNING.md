# Learning Guide: AI Field Operations Assistant

## What You Need to Learn

This project is about processing unstructured technician notes and generating structured operational outputs. The key skill is building robust text extraction pipelines that work reliably on messy, real-world field note inputs. This skill is directly applicable to any industry with unstructured workforce inputs.

---

## Phase 1: Processing Unstructured Field Notes (Week 1-2)

### Core Concepts
- Field notes characteristics: abbreviations, non-standard language, missing context
- Information extraction: entities, actions, parts, locations, times
- Named Entity Recognition (NER) for domain-specific terms
- Normalization: standardizing abbreviations and vendor names

### Resources
- **YouTube**: "NLP Information Extraction with spaCy" — https://www.youtube.com/watch?v=dIUTsFT2MeQ
- **Docs**: spaCy NER — https://spacy.io/usage/linguistic-features#named-entities
- **YouTube**: "spaCy Tutorial" by Explosion AI — https://www.youtube.com/watch?v=THduWAnG97k
- **Library**: spaCy — https://spacy.io/

---

## Phase 2: LLM-Based Structured Extraction (Week 2-3)

### Core Concepts
- Extracting parts, labor hours, actions, and findings from free text
- Structured output with JSON mode for consistent extraction
- Handling missing or ambiguous information gracefully
- Building extraction schemas for different service types

### Resources
- **YouTube**: "Information Extraction with GPT-4" — https://www.youtube.com/watch?v=P7FfGnS9fUE
- **Docs**: OpenAI JSON mode — https://platform.openai.com/docs/guides/structured-outputs
- **Course**: DeepLearning.AI "Functions, Tools and Agents" — https://learn.deeplearning.ai/functions-tools-agents-langchain

---

## Phase 3: Customer Update Generation (Week 3-4)

### Core Concepts
- Generating professional customer-facing summaries from technical field notes
- Tone calibration: non-technical language for customer updates
- Work status messaging: completed, partial, pending follow-up
- Next-step and follow-up visit recommendations

### Resources
- **YouTube**: "Text Generation with GPT-4" by Sam Witteveen — https://www.youtube.com/watch?v=kmbS6FDQh7c
- **Docs**: OpenAI prompt engineering — https://platform.openai.com/docs/guides/prompt-engineering
- **Article**: "Writing Better Technical Communication" — https://ieeexplore.ieee.org/document/1490895

---

## Phase 4: Work Order and Billing Export (Week 4-5)

### Core Concepts
- Work order data schema: job ID, technician, date, parts, labor, status
- Generating CSV and JSON exports for billing and dispatch
- Parts catalog lookup and validation
- Labor hour calculation and formatting

### Resources
- **Docs**: Python `csv` module — https://docs.python.org/3/library/csv.html
- **Library**: Pydantic for work order schema — https://docs.pydantic.dev/latest/
- **YouTube**: "Python Data Export Best Practices" — https://www.youtube.com/watch?v=1_vRahPZ4p4

---

## Phase 5: Recurring Issue Detection (Week 5)

### Core Concepts
- Pattern detection: same issue type appearing multiple times for same asset
- Asset history lookup and comparison
- Flagging recurring failures for escalation
- Simple frequency analysis with Pandas

### Resources
- **YouTube**: "Pandas GroupBy and Aggregation" by Corey Schafer — https://www.youtube.com/watch?v=txMdrV1Ut64
- **Docs**: Pandas groupby — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.groupby.html
- **Article**: "Pattern Recognition in Service Data" — https://towardsdatascience.com/

---

## Phase 6: Mobile-Friendly Streamlit UI (Week 6-7)

### Core Concepts
- Streamlit mobile layout considerations
- Voice-to-text input integration
- Simple note submission forms
- Real-time feedback on submission

### Resources
- **Docs**: Streamlit forms — https://docs.streamlit.io/library/api-reference/control-flow/st.form
- **Library**: OpenAI Whisper for voice input — https://github.com/openai/whisper
- **YouTube**: "OpenAI Whisper Tutorial" — https://www.youtube.com/watch?v=ABFqbY_rmEk

---

## Suggested Learning Sequence

| Week | Focus |
|------|-------|
| 1 | Unstructured text and NER with spaCy |
| 2 | LLM structured extraction |
| 3 | Customer update generation |
| 4 | Work order export and billing |
| 5 | Recurring issue detection |
| 6 | Streamlit UI |
| 7 | Integration and testing with real notes |

---

## Books and Deeper Resources

- *Natural Language Processing with Python* by NLTK authors (free online) — https://www.nltk.org/book/
- ServiceNow Field Service Management docs — https://docs.servicenow.com/
- *Competing Against Luck* by Clayton Christensen — understanding field service customer jobs-to-be-done
