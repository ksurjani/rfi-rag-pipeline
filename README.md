# rfi-rag-pipeline
# Construction RFI RAG Pipeline

An end-to-end Retrieval-Augmented Generation (RAG) pipeline that extracts answers
from mixed construction documents and generates a weekly RFI report with cited,
traceable sources.

## Problem
Construction project managers handle RFIs (Requests for Information) — formal
questions to the design team. Many answers already exist in the project documents
(specifications, drawings, prior RFIs, meeting minutes), but they're hard to find,
so PMs wait days for an answer that's already available. Meanwhile, open and
overdue RFIs delay the schedule and add cost.

## Solution
For each open RFI, the pipeline searches the project documents for the likely
answer and presents it with a citation, so a PM can verify and close instead of
waiting. It also produces a status report on open, overdue, and cost/schedule-
impacting RFIs.

## Pipeline
Ingestion (mixed formats) → Chunking (LangChain RecursiveCharacterTextSplitter)
→ Embedding (local sentence-transformers, MiniLM) → In-memory vector index →
Semantic retrieval (cosine similarity, threshold-gated) → Three-tier report.

- **Ingestion** — PDFs (pdfplumber), Word docs (python-docx, incl. tables), CSV
  (pandas). Each segment tagged with source and location for citations.
- **Chunking** — one general splitter across document types; 800-char chunks
  with overlap to keep complete answers intact.
- **Embedding** — local model for confidentiality; normalized vectors so cosine
  similarity reduces to a dot product.
- **Retrieval** — embed the RFI question, score against all chunks, take the best
  match, apply a similarity threshold to decide "suggested answer" vs "no match."
- **Evaluation** — labeled ground-truth set; precision/recall swept across
  thresholds to tune the cutoff by measurement, not by eye.
- **Report** — three tiers: open/overdue RFIs and cost/schedule implications
  (from structured CSV via pandas), plus RFIs answerable from documents with
  citations (via retrieval).

## Design principles
- Use RAG only where it fits — structured RFI data is queried directly with
  pandas; retrieval is reserved for unstructured documents.
- Capture provenance at ingestion so every answer is traceable to a source.
- Tune parameters (chunk size, threshold) by measurement against a labeled set.
- Every suggestion is advisory — designed for a human to verify before use.

## How to run
1. Runtime → Restart session
2. Runtime → Run all
3. The weekly report is generated as an HTML file (saved to the data folder).

## Data
All documents are synthetic — a fictional project ("Riverside Medical Office
Building") — created to demonstrate the pipeline. No confidential data is used.

## Production path
At scale: live data via a source-system API (e.g. Procore), a persistent vector
database (FAISS/Chroma), incremental re-indexing, hybrid retrieval, an LLM
extraction-and-verification layer, and scheduled delivery.

## Tech stack
Python · pdfplumber · python-docx · pandas · LangChain · sentence-transformers ·
NumPy
