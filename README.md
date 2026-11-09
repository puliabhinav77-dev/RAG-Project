# RAG Project (DocuMind)

A beginner-friendly **Retrieval-Augmented Generation (RAG)** pipeline that answers questions about PDF documents using a **fully local** stack: no paid APIs, no data leaving your machine.

Currently it is set up to answer questions over CV/resume PDFs. You ask a question in plain English, the system finds the most relevant chunks of the PDFs, and a local LLM answers using only that context.

> **Status:** working prototype in a Jupyter notebook. Packaging the code into the `src/` module and adding a persistent vector store are the next steps (see [Roadmap](#roadmap)).

---

## How it works

```
PDF files ──► Load ──► Clean text ──► Split into chunks ──► Embed ──► Vector store
                                                                          │
User question ──► Embed question ──► Top-3 similarity search ◄────────────┘
                                              │
                                              ▼
                              Prompt = retrieved context + question
                                              │
                                              ▼
                                  Local LLM (Mistral via Ollama)
                                              │
                                              ▼
                                           Answer
```

| Stage | What the notebook does |
|---|---|
| **Load** | `DirectoryLoader` + `PyMuPDFLoader` read every PDF under `data/pdf_files/` |
| **Clean** | Regex collapses extra spaces and newlines into clean single-line text |
| **Split** | `RecursiveCharacterTextSplitter` with `chunk_size=250`, `chunk_overlap=50` |
| **Tag** | Each chunk gets an `owner` metadata field and an `"cv of <name> :"` prefix so retrieval knows whose document it came from |
| **Embed** | `sentence-transformers/all-MiniLM-L6-v2` via `langchain-huggingface` |
| **Store** | LangChain `InMemoryVectorStore` |
| **Retrieve** | Similarity search, top `k=3` chunks |
| **Generate** | `ChatOllama` running `mistral` at `temperature=0`; the prompt tells the model to answer **only** from the context, or reply that it has no information |

---

## Tech stack

- **Python** 3.14+
- **LangChain** (`langchain`, `langchain-core`, `langchain-community`, `langchain-huggingface`, `langchain-ollama`)
- **PyMuPDF / pypdf** for PDF parsing
- **sentence-transformers** for embeddings
- **Ollama** + **Mistral** for the local LLM
- **uv** for environment and dependency management
- ChromaDB and FAISS are installed for upcoming persistent-store experiments

---

## Project structure

```
RAG-Project/
├── notebooks/
│   └── document.ipynb      # Full RAG pipeline prototype
├── src/
│   └── rag_project/
│       └── __init__.py     # Package entry point (placeholder for now)
├── data/                   # Your PDFs (git-ignored, create it yourself)
│   └── pdf_files/
├── pyproject.toml          # Project metadata and dependencies
├── requirements.txt        # Core dependencies (pip alternative)
├── uv.lock                 # Locked dependency versions
├── .python-version
└── README.md
```

---

## Getting started

### 1. Prerequisites

- [Python 3.14+](https://www.python.org/downloads/)
- [uv](https://docs.astral.sh/uv/) (or use pip, see below)
- [Ollama](https://ollama.com/) installed and running

### 2. Clone and install

```bash
git clone https://github.com/puliabhinav77-dev/RAG-Project.git
cd RAG-Project
uv sync
```

Using pip instead:

```bash
python -m venv .venv
.venv\Scripts\activate          # Windows
pip install -r requirements.txt
pip install langchain-huggingface langchain-ollama
```

### 3. Pull the local model

```bash
ollama pull mistral
```

### 4. Add your documents

The `data/` folder is git-ignored, so create it yourself and add your PDFs:

```
data/pdf_files/your-document.pdf
```

### 5. Run

Open `notebooks/document.ipynb` in VS Code or Jupyter, select the project's `.venv` as the kernel, and run the cells in order.

> **Note:** the notebook currently maps file names to owner names in a small dictionary (the `owners` cell). If you use your own PDFs, update that dictionary to match your file names.

---

## Example

```python
question = "brazilian e-commerce sales analysis project"
print(answer_question(question))
```

The model answers using only the retrieved chunks. If nothing relevant is found, it replies that it has no information on the question.

---



## Author

**Puli Abhinav**: B.Tech CSE (Data Science)

---

## License

No license specified yet. Add one (for example MIT) if you want others to reuse the code.
