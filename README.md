# RAG-
A Python-based RAG chatbot that lets users query any PDF document using natural language. The system extracts text, builds a semantic vector index, retrieves relevant context, and generates grounded answers via Groq's free LLM API.
# RAG Chatbot — Technical Documentation
### PDF-Based Retrieval-Augmented Generation System
 **LangChain · FAISS · Groq LLM · HuggingFace Embeddings**
## 1. System Overview

A Python-based RAG chatbot that lets users query any PDF document using natural language. The system extracts text, builds a semantic vector index, retrieves relevant context, and generates grounded answers via Groq's free LLM API.

### Technology Stack

| Component | Technology | Purpose |
|---|---|---|
| PDF Parsing | pypdf · PdfReader | Page-by-page text extraction |
| Chunking | LangChain RecursiveCharacterTextSplitter | Segmentation with overlap |
| Embeddings | HuggingFace all-MiniLM-L6-v2 | 384-dim sentence vectors |
| Vector Store | FAISS | Approximate nearest-neighbor search |
| LLM Backend | Groq API · Llama 3.1 8B | Fast, free inference |
| Environment | Google Colab / Python 3.8+ | Execution + Drive storage |
## 2. Data Flow

| Step | Stage | Detail |
|---|---|---|
| 1 | Ingestion | PdfReader extracts raw text from each PDF page |
| 2 | Chunking | Text split into 700-char chunks with 100-char overlap |
| 3 | Embedding | Each chunk encoded into a 384-dim dense vector |
| 4 | Indexing | FAISS indexes all vectors for cosine-similarity search |
| 5 | Retrieval | Query embedded → top-3 matching chunks fetched |
| 6 | Generation | Context + query sent to Groq LLM → grounded answer |
## 3. Module Reference

---

### 3.1 `load_pdf(file_path)`

Reads a PDF and concatenates extractable text from every page.
Pages with no text are skipped silently.

- **file_path** — Absolute path to the PDF file *(string)*
- **Returns** — Full concatenated document text *(string)*

---

### 3.2 `split_text(text)`

Splits document text into overlapping chunks using LangChain's
RecursiveCharacterTextSplitter.

- **chunk_size** = `700` — Balances context richness with retrieval precision
- **chunk_overlap** = `100` — Preserves meaning across chunk boundaries
- **Returns** — List of text chunks *(List[str])*

---

### 3.3 `get_vector_store(pdf_path)`

Manages the FAISS index lifecycle with Google Drive caching.
First run builds and saves the index; subsequent runs load it instantly.

- **Embedding model** — `sentence-transformers/all-MiniLM-L6-v2`
- **Index path** — `/content/drive/MyDrive/faiss_index_db`
- **Cache strategy** — Load from Drive if exists, else build and save

---

### 3.4 `call_groq(system_prompt, user_message)`

Sends a two-message (system + user) request to the Groq API
and returns the model's text response.

- **Model** — `llama-3.1-8b-instant` *(free tier)*
- **Temperature** — `0.7`
- **Max tokens** — `1024`

---

### 3.5 `run_rag(pdf_path)` — Main Pipeline

Orchestrates the full conversation loop.

- Greeting input → bypasses retrieval, uses welcome prompt
- Normal input → retrieves top-3 chunks, calls Groq LLM
- Empty input → silently skipped
- Type `exit` or `quit` to end the session
## 4. Prompt Design

### RAG System Prompt
Used for all document-related questions. Constrains the model
to answer strictly from retrieved context.
## 5. Configuration

| Parameter | Default | Effect |
|---|---|---|
| `chunk_size` | 700 | Larger = more context per chunk |
| `chunk_overlap` | 100 | Higher = less info loss at boundaries |
| `search_k` | 3 | Number of chunks retrieved per query |
| `temperature` | 0.7 | Lower = factual · Higher = creative |
| `max_tokens` | 1024 | Maximum response length |
| `GROQ_API_KEY` | — | Required · set via `os.environ` or Colab Secrets |
| `DB_FAISS_PATH` | `/content/drive/MyDrive/faiss_index_db` | FAISS index storage path |
## 6. Dependencies

| Package | Purpose |
|---|---|
| `pypdf` | PDF reading and text extraction |
| `langchain-text-splitters` | Recursive character text splitting |
| `langchain-huggingface` | HuggingFace embedding integration |
| `langchain-community` | FAISS vector store wrapper |
| `faiss-cpu` | Facebook AI Similarity Search |
| `groq` | Groq API client |
| `sentence-transformers` | Embedding model backend |

### Install Command

```bash
pip install pypdf langchain-text-splitters langchain-huggingface langchain-community faiss-cpu groq
```
## 7. Setup & Usage

### First-Time Setup

1. Get a free Groq API key at **console.groq.com**
2. Set `GROQ_API_KEY` in the script or via Colab Secrets
3. Upload your PDF to Google Drive and update `pdf_path`
4. Run all cells — first run takes **1–3 minutes** to build the index

### Running

```python
run_rag("/content/drive/MyDrive/your_document.pdf")
```

### Example Interactions

| Input | Behaviour |
|---|---|
| `"What is this document about?"` | High-level document summary |
| `"Explain the methodology"` | Targeted retrieval + explanation |
| `"List key points"` | Retrieves and lists relevant content |
| `"hi"` | Greeting response + readiness confirmation |
| `"exit"` / `"quit"` | Ends the session gracefully |
## 8. Security Notes

###  API Key
Never hardcode the key in shared or public notebooks.
Use **Colab Secrets** (key icon in the sidebar) or environment variables.
For production use a secrets manager like AWS Secrets Manager.

###  FAISS Deserialization
`allow_dangerous_deserialization=True` bypasses pickle security checks.
Only load FAISS indexes from sources you trust.
For production consider a managed vector DB — Pinecone, Weaviate, or Qdrant.

###  Data Privacy
- Documents are processed locally inside Colab
- Only query text and retrieved chunks are sent to Groq's API
- Review Groq's data retention policy before using sensitive documents
- For confidential data consider a self-hosted LLM via Ollama or Llama.cpp

---
> *RAG Chatbot · Technical Documentation · 2026*
