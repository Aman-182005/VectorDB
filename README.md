# VectorDB — Semantic Search Lab

A local C++ vector-search demo with a browser dashboard and a document RAG workflow powered by Ollama.

## What it includes

- **Demo vector search:** HNSW, KD-tree, and brute-force algorithm choices, cosine/Euclidean/Manhattan distance options, and a latency benchmark. The 20 demo points are hand-crafted 16-dimensional examples designed to illustrate the algorithms.
- **Document RAG:** TXT and selectable-text PDF ingestion, editable extraction preview, 250-word chunks with 30-word overlap, embeddings from `nomic-embed-text`, cosine-distance retrieval, and answers from `llama3.2`.
- **Map:** 20 demo points plus document chunks using their actual Ollama embeddings. The demo vectors and document chunks are projected into separate 2D layers; positions/distances *between* those layers are not comparable.
- **Persistence:** indexed document chunks and their embeddings are stored locally in `documents.vdb` and reloaded when the C++ server starts.
- **Grounded answers:** retrieval excludes chunks above cosine distance `0.45`; the LLM is instructed to answer from retrieved sources only and to abstain if no sufficiently relevant source is found.

## Supported upload types

- `.txt`
- PDFs that contain **selectable text**

Image OCR is intentionally disabled in this version. Scanned/image-only PDFs and PNG/JPG images are not supported; paste extracted text manually instead. PDF.js is loaded from a CDN for browser-side PDF extraction, so internet access is required when extracting PDFs.

## Requirements

- Windows
- MSYS2 UCRT64 with `g++`
- Ollama for Windows
- Models:
  - `nomic-embed-text`
  - `llama3.2`

Verify the models in Windows Command Prompt:

```bat
ollama list
```

If a model is missing:

```bat
ollama pull nomic-embed-text
ollama pull llama3.2
```

Ollama must be running on its local API port `11434`. The C++ app connects to `127.0.0.1:11434`; you do not need to invoke Ollama from the MSYS2 shell if the Windows Ollama app is already running.

## Build and run (MSYS2 UCRT64)

Open MSYS2 UCRT64 in this project folder. Run:

```bash
g++ -std=c++17 -O2 main.cpp -o db.exe -lws2_32
./db.exe
```

Keep the terminal open. The server prints its status and listens on port `8080`.

Open:

- Dashboard: http://localhost:8080
- Server status: http://localhost:8080/status
- Demo vectors: http://localhost:8080/items
- Indexed chunks: http://localhost:8080/doc/list
- Embeddings used by the map: http://localhost:8080/doc/points

**Run `./db.exe` from this directory** so `documents.vdb` is created here. Stop with `Ctrl+C`. Document chunks remain after restart. The 20 demo vectors are loaded in memory from the source on each start.

## Use the RAG workflow

1. Open **Documents**.
2. Choose a TXT file or a PDF with selectable text.
3. Click **Extract text for preview**.
4. Review/correct the extracted text.
5. Click **Index reviewed text**.
6. Open **Ask AI** and ask a question based on the uploaded source.
7. Expand the listed passages to inspect exactly what was sent to the model.

Re-uploading a title that already exists is rejected to avoid accidental duplicate chunks. Delete its existing chunks in the Documents panel first if you need to replace a source.

## Retrieval and score interpretation

The displayed score is **cosine distance** (`1 - cosine similarity`): lower is closer. The RAG retriever currently accepts distances up to `0.45`. This is a practical guardrail, not a perfect relevance classifier; evaluate it against your own test documents. The app deliberately avoids using the demo search results as fallback RAG evidence.

Useful sanity checks:
- Ask a question whose answer appears plainly in a source file.
- Ask a question from an unrelated topic; the app should say it could not find a relevant passage rather than answer from general knowledge.
- Expand the passage chips to inspect the source text.

## Known limitations

- Document storage is local to this project folder and not shared between multiple concurrent server instances.
- PDFs with scanned pages but no selectable text and image files require OCR, which is intentionally excluded here to keep the pipeline smaller and more predictable.
- The demo vector search uses illustrative 16D vectors and a simple text-to-demo-vector heuristic; it is an algorithm visualization, not a production embedding search.
- The RAG distance threshold may need tuning for a different embedding model or document type.

## Troubleshooting

- **`Ollama: OFFLINE`:** open the Ollama app; in Windows CMD run `ollama list`. Confirm `nomic-embed-text` and `llama3.2` are installed.
- **`g++` fails:** ensure the MSYS2 UCRT64 terminal is being used and `httplib.h` is in this folder.
- **PDF extraction fails:** use a selectable-text PDF; scanned PDFs are intentionally unsupported without OCR.
- **A document title is already indexed:** delete its old chunks in the Documents panel, then upload it again.
- **Documents appear missing:** confirm you are starting the server from the same project folder. The persistence file is `documents.vdb`.
