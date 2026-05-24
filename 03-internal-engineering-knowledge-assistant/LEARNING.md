# Learning Guide: Internal Engineering Knowledge Assistant

## What You Need to Learn

This is the canonical RAG (Retrieval-Augmented Generation) project. It requires deep understanding of document ingestion, embedding models, vector databases, retrieval strategies, and LLM answer generation. RAG is the most commercially deployed AI architecture in enterprise software today — mastering it unlocks dozens of client use cases.

---

## Phase 1: Understanding RAG Architecture (Week 1)

### Core Concepts
- What RAG is and why it outperforms pure LLM for knowledge questions
- The RAG pipeline: ingest → chunk → embed → store → retrieve → generate
- When to use RAG vs fine-tuning vs pure prompting
- Key failure modes: hallucination, poor retrieval, out-of-scope answers

### Resources
- **YouTube**: "Retrieval Augmented Generation (RAG) Explained" by IBM Technology — https://www.youtube.com/watch?v=T-D1OfcDW1M
- **YouTube**: "RAG From Scratch" by LangChain — https://www.youtube.com/playlist?list=PLfaIDFEXuae2LXbO1_PKyVJiQ23ZztA0x
- **Paper**: "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks" — https://arxiv.org/abs/2005.11401
- **Article**: "RAG vs Fine-Tuning" — https://www.pinecone.io/learn/retrieval-augmented-generation/

---

## Phase 2: Document Ingestion and Chunking (Week 2)

### Core Concepts
- Loading documents: PDF, Markdown, HTML, DOCX
- Text splitting strategies: fixed-size, recursive, semantic
- Why chunking size matters for retrieval quality
- Metadata preservation during chunking

### Resources
- **YouTube**: "LangChain Document Loaders and Text Splitters" — https://www.youtube.com/watch?v=ih9PBGVVOO4
- **Docs**: LangChain text splitters — https://python.langchain.com/docs/modules/data_connection/document_transformers/
- **Docs**: LlamaIndex document loading — https://docs.llamaindex.ai/en/stable/module_guides/loading/
- **Library**: `pypdf` for PDF reading — https://github.com/py-pdf/pypdf

---

## Phase 3: Embeddings and Vector Databases (Week 2-3)

### Core Concepts
- What embeddings are: dense vector representations of text
- Embedding models: OpenAI `text-embedding-3-small`, all-MiniLM-L6-v2
- Vector databases: Chroma (local), Pinecone (cloud), Qdrant, Weaviate
- Similarity search: cosine similarity, dot product

### Resources
- **YouTube**: "Vector Databases Explained" by Fireship — https://www.youtube.com/watch?v=klTvEwg3oJ4
- **YouTube**: "Chroma DB Tutorial" by Sam Witteveen — https://www.youtube.com/watch?v=7EG8OQMX7rQ
- **Docs**: ChromaDB — https://docs.trychroma.com/
- **Docs**: Pinecone quickstart — https://docs.pinecone.io/guides/get-started/quickstart
- **Article**: "Understanding Embedding Models" — https://www.sbert.net/docs/pretrained_models.html

---

## Phase 4: LangChain and LlamaIndex RAG Pipelines (Week 3-4)

### Core Concepts
- LangChain LCEL (LangChain Expression Language) for RAG chains
- LlamaIndex query engine pattern
- Re-ranking retrieved results for quality improvement
- Multi-query retrieval to handle ambiguous questions

### Resources
- **YouTube**: "Build a RAG App with LangChain" by Tech With Tim — https://www.youtube.com/watch?v=tcqEUSNCn8I
- **YouTube**: "LlamaIndex Full Tutorial" by Jerry Liu (creator) — https://www.youtube.com/watch?v=JN-GNzJRKFI
- **Docs**: LangChain RAG tutorial — https://python.langchain.com/docs/tutorials/rag/
- **Docs**: LlamaIndex query engines — https://docs.llamaindex.ai/en/stable/module_guides/querying/

---

## Phase 5: Citations and Answer Quality (Week 4-5)

### Core Concepts
- Returning source document references with answers
- Confidence scoring for retrieved chunks
- Handling out-of-scope questions gracefully
- Evaluating RAG quality: faithfulness and relevance metrics

### Resources
- **YouTube**: "RAG Evaluation with RAGAS" — https://www.youtube.com/watch?v=1gyuRCkqKiY
- **Library**: RAGAS evaluation framework — https://github.com/explodinggradients/ragas
- **Docs**: LangChain QA with sources — https://python.langchain.com/docs/tutorials/qa_chat_history/

---

## Phase 6: FastAPI and Streamlit Integration (Week 5-6)

### Core Concepts
- FastAPI endpoints for document upload and query
- Streamlit chat interface with citation display
- Metadata filtering by document category or team
- Session-based conversation history

### Resources
- **YouTube**: "Build a ChatBot with Streamlit" by Alejandro AO — https://www.youtube.com/watch?v=sBhK-2K9bUc
- **Docs**: Streamlit chat elements — https://docs.streamlit.io/library/api-reference/chat

---

## Suggested Learning Sequence

| Week | Focus |
|------|-------|
| 1 | RAG architecture and concepts |
| 2 | Document loading, chunking, embeddings |
| 3 | Vector DBs and similarity search |
| 4 | LangChain/LlamaIndex RAG pipeline |
| 5 | Citations, evaluation, out-of-scope handling |
| 6 | FastAPI + Streamlit integration |
| 7 | End-to-end testing with real documents |

---

## Books and Deeper Resources

- *LLM Engineer's Handbook* — https://www.llm-engineer.com/
- *Developing Apps with GPT-4 and ChatGPT* by Olivier Caelen — O'Reilly
- OpenAI Cookbook RAG examples — https://github.com/openai/openai-cookbook/tree/main/examples/vector_databases
