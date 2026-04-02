# RAG Nutritional Chatbot

![Live Demo](https://img.shields.io/badge/Live_Demo-Available-success?style=for-the-badge&logo=vercel)
![Next.js](https://img.shields.io/badge/Next.js-14-black?style=for-the-badge&logo=next.js)
![Python](https://img.shields.io/badge/Python-3.10+-blue?style=for-the-badge&logo=python)
![Supabase](https://img.shields.io/badge/Supabase-pgvector-green?style=for-the-badge&logo=supabase)
![OpenAI](https://img.shields.io/badge/OpenAI-Embeddings_&_LLM-white?style=for-the-badge&logo=openai)

**Live Prototype:** [RAG Nutrition Bot on Lovable](https://zohaib-rag-nutrition-bot.lovable.app/)

A Production-grade Retrieval-Augmented Generation (RAG) chatbot designed to accurately answer questions based on a massive human nutrition textbook. This project demonstrates end-to-end AI engineering, from data ingestion and semantic chunking to vector search and full-stack integration. 

The UI/UX is inspired by the minimal, sleek design of the [ARC-AGI](https://arcprize.org/arc-agi) platform.

---

## System Architecture & Approaches

This project was built using a robust, highly optimized pipeline to ensure high accuracy and low latency.

### 1. Data Ingestion & Preprocessing (Python)
* **Intelligent Sentence-Based Chunking:** Instead of naive character-splitting, I implemented sentence-boundary chunking using `regex` and `tiktoken`. This preserves semantic meaning and contextual integrity.
* **Overlapping Context:** Configured an overlap of 2 sentences per chunk to prevent context loss at chunk boundaries.
* **Token Throttling:** Built a strictly typed tokenizer pipeline ensuring no chunk exceeds the safety cap of `1300 tokens`, filtering out overly tiny fragments (`< 50 tokens`) to maintain signal-to-noise ratio.
* **Clean Text Normalization:** Built custom regex pipelines to handle PDF artifacts like hyphenation across line breaks, irregular whitespace, and carriage returns via `PyMuPDF (fitz)`.

### 2. Embeddings & Vector Storage
* **OpenAI Embeddings:** Utilized OpenAI's efficient `text-embedding-3-small` model to map text chunks into high-density 1536-dimensional vector space.
* **Supabase & `pgvector`:** Hosted the embeddings in a PostgreSQL database (Supabase) leveraging the `pgvector` extension.
* **Optimized Search Indexing:** Implemented an `ivfflat (vector_cosine_ops)` index over the chunks to ensure lightning-fast Approximate Nearest Neighbor (ANN) search as the dataset scales.

### 3. Advanced Retrieval (SQL)
* **Custom RPC Functions:** Wrote PL/pgSQL stored procedures (`match_documents`) to execute cosine similarity searches (`<=>`) directly at the database layer.
* **Metadata Filtering:** Designed the retrieval to accept dynamic JSONB filters (e.g., exact PDF source matching) before performing vector math, deeply optimizing the search efficiency.

### 4. Full-Stack App & API (Next.js & React)
* **Next.js App Router:** Built a secure `/api/chat` route to act as the orchestrator between the user prompt, Supabase retrieval, and the OpenAI generation step.
* **Interactive Citations:** Engineered a citation system mapping the LLM's response directly to the retrieved chunk index and page number, verifiable in real-time by the user.
* **UI/UX Design:** Built a minimal, ARC-AGI inspired React frontend using modern React hooks, handling asynchronous states smoothly with user-friendly typing indicators.

---

## Repository Structure

```text
├── ingest.py                # Pipeline for PDF parsing, text cleaning, and chunking
├── test_embeddings.py       # Evaluation script for testing vector retrieval accuracy
├── rag-chat/                # Next.js Full Stack Application
│   ├── app/api/chat/route.ts# Core API logic bridging Retrieval and Generation
│   ├── app/page.tsx         # Frontend Chat Interface with citation logic
│   └── ...                  # Tailwind CSS & UI Configurations
└── supabase/                # SQL scripts for pgvector schema and RPCs