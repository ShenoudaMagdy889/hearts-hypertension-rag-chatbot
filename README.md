# HEARTS Hypertension RAG Chatbot

A Retrieval-Augmented Generation (RAG) chatbot that answers questions using hypertension-related documents. It retrieves relevant passages from indexed PDFs, checks whether the retrieved information is sufficient, and generates answers grounded in the available context.

This project was developed as part of a hackathon.

> **Disclaimer:** This is an educational project, not a medical diagnostic tool. Its responses may be incomplete or inaccurate and should not replace professional medical advice.

## Features

- **PDF ingestion:** Extracts text and layout blocks from PDF files using PyMuPDF.
- **Section-aware chunking:** Detects likely headings and splits extracted content into manageable chunks with overlap.
- **Semantic search:** Uses Nomic text embeddings and ChromaDB to retrieve relevant document passages.
- **Context-grounded answers:** Uses the Groq API with the `openai/gpt-oss-20b` model to generate answers from retrieved context.
- **Answerability checks:** Applies retrieval-score thresholds and a model-based check before generating an answer.
- **Arabic and English support:** Translates Arabic questions into English for retrieval and instructs the model to respond in the original question's language.
- **Source metadata:** Tracks source filenames, page numbers, section titles, and retrieval scores.
- **Web interface:** Provides a Flask application for asking questions through a browser.
- **Command-line interface:** Supports both interactive questions and individual queries.

## Technology Stack

| Component | Technology |
|---|---|
| Language | Python |
| Web framework | Flask |
| PDF processing | PyMuPDF |
| Embeddings | Sentence Transformers |
| Embedding model | `nomic-ai/nomic-embed-text-v1.5` |
| Vector database | ChromaDB |
| Language model API | Groq |
| Generation model | `openai/gpt-oss-20b` |
| Frontend | HTML, CSS, JavaScript |

## How It Works

1. **Document ingestion:** PDF files are processed, and their text is extracted and cleaned.
2. **Section-aware chunking:** The ingestion pipeline detects likely headings, organizes text into sections, and splits long passages into overlapping chunks.
3. **Embedding and storage:** Each chunk is converted into a vector embedding and stored in a persistent ChromaDB collection with metadata.
4. **Retrieval:** A user's question is embedded and compared with the stored document vectors.
5. **Retrieval threshold check:** The system checks whether the retrieved passages meet its configured score thresholds.
6. **Answerability check:** The language model evaluates whether the retrieved context contains useful information for answering the question.
7. **Answer generation:** If both checks pass, the model generates an answer using the supplied passages and is instructed to cite them.
8. **Response:** The web application returns the answer and available source metadata.

The refusal checks are intended to reduce unsupported answers; they do not guarantee factual accuracy or eliminate hallucinations.

## Project Structure

```text
.
├── app.py                 # Flask web application
├── config.py              # Configuration settings
├── ingest.py              # PDF processing and indexing
├── rag.py                 # Retrieval and answer generation
├── requirements.txt       # Python dependencies
├── .env.example           # Environment variable template
├── README.md
├── .gitignore
├── data/                  # Add permitted source PDFs here
├── test-data/             # Optional sample PDFs for testing
├── static/                # CSS, JavaScript, and other static assets
└── templates/             # HTML templates
```

The local ChromaDB database is stored in `chroma_db_nomic/`. It is excluded from version control and must be generated locally.

The `data/` and `test-data/` directories may be excluded from the published repository to avoid redistributing documents without confirming their permissions.

## Prerequisites

- Python compatible with the dependencies in `requirements.txt`
- Git, if cloning the repository
- A Groq API key
- An internet connection to download the embedding model and access the Groq API

## Installation

### 1. Clone the repository

Replace the placeholders with your actual GitHub repository URL and directory name.

```bash
git clone <YOUR_REPOSITORY_URL>
cd <YOUR_REPOSITORY_FOLDER>
```

Alternatively, download the repository and open the project directory in VS Code.

### 2. Create a virtual environment

On Windows PowerShell:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
```

If PowerShell prevents activation, use Command Prompt:

```bat
.venv\Scripts\activate.bat
```

### 3. Install dependencies

```bash
python -m pip install -r requirements.txt
```

### 4. Configure the API key

Create a `.env` file in the project root using `.env.example` as a template.

Add your own Groq API key:

```dotenv
GROQ_API_KEY=your_groq_api_key_here
```

Obtain an API key through the official [Groq developer platform](https://console.groq.com/).

**Security:** Never commit your actual `.env` file or expose API keys in source code, screenshots, or public repositories.

## Preparing the Document Database

The chatbot requires PDF documents to be indexed before it can answer questions. The ingestion script accepts either a single PDF file or a directory containing PDFs.

### 1. Add your source documents

Place PDF documents you are permitted to use in the `data/` directory. If the directory is not present in your clone, create it.

### 2. Build the vector database

From the project root, run:

```bash
python ingest.py data
```

To process a single PDF:

```bash
python ingest.py "data/example.pdf"
```

The ingestion process extracts text, detects likely section headings, creates chunks, generates embeddings, and stores the results in ChromaDB.

The first run may take longer because the embedding model may need to be downloaded.

### Rebuilding the database

To delete the existing collection and rebuild it from your documents:

```bash
python ingest.py data --reset
```

**Warning:** The `--reset` option deletes the existing ChromaDB collection before ingestion. Use it only when you intend to rebuild the index.

## Running the Web Application

Start the Flask application:

```bash
python app.py
```

Open the local website in your browser:

```text
http://127.0.0.1:5000
```

The application provides a web interface for submitting questions and receiving answers with source metadata.

## Command-Line Usage

The RAG pipeline can also be used directly from the terminal.

### Interactive mode

```bash
python rag.py
```

Enter questions when prompted. Type `exit`, `quit`, or `q` to stop.

### Ask a single question

```bash
python rag.py "How should hypertension implementation be monitored?"
```

### Configure the number of retrieved passages

```bash
python rag.py "Your question here" --top_k 5
```

The default retrieval count is five passages. You can change it with `--top_k`.

To filter results by a source filename:

```bash
python rag.py "Your question here" --source "example.pdf"
```

## Configuration

The primary configuration is defined in `config.py`.

| Setting | Current value |
|---|---|
| Embedding model | `nomic-ai/nomic-embed-text-v1.5` |
| Embedding dimensions | 768 |
| ChromaDB directory | `./chroma_db_nomic` |
| Collection name | `rag_documents` |
| Maximum chunk size | 1,200 characters |
| Chunk overlap | 150 characters |
| Embedding batch size | 100 |
| Default retrieval count | 5 |

The active refusal thresholds are currently defined in `rag.py`. Review both `config.py` and `rag.py` before changing them, because their current values differ.

## Data Sources and Attribution

The project is designed to work with hypertension-related publications, including resources from the World Health Organization and other relevant publishers.

If you redistribute source PDFs, verify the license and reuse permissions for each publication, including any third-party figures, tables, and images.

For documents that cannot be redistributed, obtain them from their official publishers and place permitted copies in the local `data/` directory.

The project is independent and should not be represented as endorsed by the World Health Organization or any other document publisher.

## Team and Contributions

This project was developed in the context of a five-member hackathon team.

**Primary implementation:** [Your Name / GitHub Username]

**Collaborative contributions:** UI assistance, testing, brainstorming, and document filtering.

Update this section with the final contributor names and their actual contributions, subject to the hackathon's attribution rules.

## Limitations

- Responses are limited to the information retrieved from the indexed documents.
- PDF extraction and heading detection may not perfectly preserve the original document structure.
- Retrieval quality depends on the source documents, chunking strategy, and embedding similarity.
- Model-based answerability checks may incorrectly accept or reject a question.
- Arabic retrieval depends on successful question translation.
- The application requires access to the Groq API for model-based operations.
- A fresh installation needs source PDFs and a locally generated ChromaDB index before it can answer questions.
- The project is an educational prototype and has not been presented as a production-hardened medical service.

## License

Choose a license for the original project code after confirming the hackathon's ownership and licensing requirements.

Third-party documents retain their own copyright and licensing conditions. A license applied to the code does not automatically apply to the source PDFs or other third-party materials.
