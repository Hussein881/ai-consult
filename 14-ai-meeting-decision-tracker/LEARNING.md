# Learning Guide: AI Meeting and Decision Tracker

## What You Need to Learn

This project is about structured information extraction from conversational text: meeting notes and transcripts. The key skill is extracting decisions, action items, owners, and timelines reliably — a pattern that is directly reusable for any document-to-task-extraction workflow. The secondary skill is building task management integrations.

---

## Phase 1: Meeting Transcript Structure and Formats (Week 1)

### Core Concepts
- Meeting transcript formats: Otter.ai, Zoom transcripts, raw text notes
- Speaker identification and attribution in transcripts
- Handling informal language and incomplete sentences
- Timestamp handling and meeting section detection

### Resources
- **Article**: "Working with Meeting Transcripts" — https://support.otter.ai/hc/en-us
- **Library**: `assemblyai` Python SDK — https://www.assemblyai.com/docs
- **YouTube**: "Meeting Transcription with Python" — https://www.youtube.com/watch?v=i_b8ZRXnpho

---

## Phase 2: Decision and Action Item Extraction (Week 1-2)

### Core Concepts
- Identifying decision statements vs discussion vs action items in text
- Named entity extraction: person names, dates, project names
- Distinguishing committed actions from suggestions
- Confidence thresholds for extraction

### Resources
- **YouTube**: "Information Extraction with LLMs" — https://www.youtube.com/watch?v=hNS_30EfNpY
- **Docs**: OpenAI structured outputs — https://platform.openai.com/docs/guides/structured-outputs
- **Article**: "Action Item Extraction" — https://towardsdatascience.com/extracting-action-items-from-meeting-notes-with-gpt-4-f7e6c2985d46

---

## Phase 3: LangChain Extraction Chains (Week 2-3)

### Core Concepts
- LangChain extraction chain with Pydantic output schemas
- Multi-document extraction for long transcripts
- Map-reduce for meeting series (multiple transcripts)
- Output validation and quality checks

### Resources
- **YouTube**: "LangChain Extraction Tutorial" — https://www.youtube.com/watch?v=hNS_30EfNpY
- **Docs**: LangChain extraction — https://python.langchain.com/docs/tutorials/extraction/
- **Docs**: Pydantic models — https://docs.pydantic.dev/latest/

---

## Phase 4: Decision Log and Searchable History (Week 3-4)

### Core Concepts
- Database schema for decisions: content, date, meeting, owner, status
- Full-text search across decision history
- Tagging and categorization
- Conflict detection: same topic, different decisions

### Resources
- **YouTube**: "PostgreSQL Full-Text Search" — https://www.youtube.com/watch?v=szfUbzsKvtk
- **Docs**: SQLAlchemy full-text search — https://docs.sqlalchemy.org/en/20/
- **Library**: `whoosh` for lightweight full-text search — https://whoosh.readthedocs.io/

---

## Phase 5: Task Management Integration (Week 4-5)

### Core Concepts
- Jira API: creating tasks from extracted action items
- Asana API: task creation with assignees and due dates
- Notion API: adding items to a project database
- Error handling and duplicate detection

### Resources
- **Docs**: Jira REST API — https://developer.atlassian.com/cloud/jira/platform/rest/v3/
- **Docs**: Asana Python client — https://github.com/Asana/python-asana
- **Docs**: Notion API — https://developers.notion.com/
- **YouTube**: "Jira Python API Tutorial" — https://www.youtube.com/watch?v=0V3JkC2MFAI

---

## Phase 6: Follow-Up Email Generation and Streamlit UI (Week 5-6)

### Core Concepts
- Generating professional follow-up email summaries from meeting extracts
- Streamlit interface for upload, review, and export
- Action item tracking UI with status checkboxes
- Export to CSV for project management

### Resources
- **YouTube**: "Build a Meeting Assistant with Python" by AI Jason — https://www.youtube.com/watch?v=sBhK-2K9bUc
- **Docs**: Streamlit data editor — https://docs.streamlit.io/library/api-reference/data/st.data_editor

---

## Suggested Learning Sequence

| Week | Focus |
|------|-------|
| 1 | Transcript formats and decision/action extraction |
| 2 | LangChain extraction chains |
| 3 | Decision log database design |
| 4 | Task management integrations (Jira, Asana) |
| 5 | Follow-up email generation |
| 6 | Streamlit UI |
| 7 | End-to-end integration test |

---

## Books and Deeper Resources

- *Getting Things Done* by David Allen — action item methodology context
- LangChain extraction docs — https://python.langchain.com/docs/tutorials/extraction/
- Notion API documentation — https://developers.notion.com/
- Jira API documentation — https://developer.atlassian.com/cloud/jira/platform/rest/v3/
