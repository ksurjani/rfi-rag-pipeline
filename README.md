# Construction RFI RAG Pipeline

An end-to-end Retrieval-Augmented Generation (RAG) system that extracts answers
from heterogeneous documents and generates insight reports with cited, traceable
sources. Built to demonstrate production-minded RAG design — from mixed-format
ingestion through evaluation-driven retrieval — using a construction RFI use case.

## What this project demonstrates
- Designing a complete RAG pipeline end to end, not just calling an LLM
- Extracting and unifying data from mixed document types into a common schema
- Deliberate, defensible design decisions at every stage, with tradeoffs
- Evaluation-driven tuning — measuring retrieval quality, not guessing
- Traceability and human-in-the-loop verification for trustworthy outputs
- A clear path from proof-of-concept to production

## The use case
Construction teams handle RFIs (Requests for Information) — formal questions to
the design team. Many answers already exist in the project documents (specs,
drawings, prior RFIs, meeting minutes) but are hard to find. This system retrieves
the likely answer for each open RFI and presents it with a citation, so it can be
verified and closed instead of waited on. The same architecture generalizes to any
domain where answers are buried across large, mixed document sets — legal,
compliance, technical support, or internal knowledge bases.

## Architecture
Ingestion (mixed formats) → Chunking (LangChain) → Embedding (local sentence-
transformers) → In-memory vector index → Semantic retrieval (cosine similarity,
threshold-gated) → Insight report.

- **Ingestion** — PDFs (pdfplumber), Word docs (python-docx, incl. tables), CSV
  (pandas), normalized into a uniform schema with source/location provenance for
  citations.
- **Chunking** — LangChain RecursiveCharacterTextSplitter; chunk size tuned to
  keep complete answers intact, with overlap to preserve boundary-spanning content.
- **Embedding** — local model (MiniLM) chosen for data confidentiality; normalized
  vectors so cosine similarity reduces to a fast dot product.
- **Retrieval** — dense semantic search: embed the query, score against all chunks,
  take the best match, and apply a similarity threshold to decide "answer found"
  vs "no match" (so the system declines rather than fabricates).
- **Evaluation** — labeled ground-truth set with a precision/recall sweep to tune
  the retrieval threshold empirically.
- **Report** — combines structured data (queried directly with pandas) with
  retrieval-backed suggestions, each citing its source for human verification.

## Design principles
- **Apply RAG only where it fits** — structured data is queried directly; retrieval
  is reserved for unstructured content where the answer's location is unknown.
- **Capture provenance early** — source and location are tagged at ingestion so
  every answer is traceable back to a real document.
- **Tune by measurement** — chunk size and retrieval threshold are set against a
  labeled evaluation set, not by intuition.
- **Keep a human in the loop** — every suggestion is advisory and verifiable,
  designed for trust in high-stakes domains.
- **Decouple pipeline from data source** — the core operates on internal data
  structures, so swapping the source (file → API) is an integration change, not a
  rewrite.

## Production path
At scale: live data via a source-system API, a persistent vector database
(FAISS/Chroma/pgvector), incremental re-indexing as documents change, hybrid
retrieval (dense + keyword) for exact-term matching, an LLM extraction-and-
verification layer for complete generated answers, and scheduled automated delivery.

## How to run
1. Runtime → Restart session
2. Runtime → Run all
3. The insight report is generated as an HTML file.

## Data
All documents are synthetic — a fictional project created to demonstrate the
pipeline. No confidential or proprietary data is used.

## Tech stack
Python · pdfplumber · python-docx · pandas · LangChain · sentence-transformers ·
NumPy
