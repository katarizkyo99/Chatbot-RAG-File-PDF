# Chatbot RAG - Football Player Stats

A small Retrieval-Augmented Generation (RAG) chatbot that answers football/player-statistics questions using PDF documents as the knowledge source. This project is a minimal Flask web application that:

- Loads PDFs from the repository DB/ folder.
- Splits document text into chunks, computes embeddings, and indexes them in a FAISS vector store.
- Uses Groq's ChatGroq (Llama3-8b-8192 as configured) as the LLM via the langchain_groq integration and answers queries using RetrievalQA.

This repository is a functional prototype demonstrating a local RAG pipeline (PDF -> embeddings -> FAISS -> LLM) with a tiny web UI.

## Features

- Load multiple PDF files from DB/ and create a retrieval index.
- Use Hugging Face sentence-transformers (all-MiniLM-L6-v2) for embeddings.
- Use FAISS in-memory vector store for nearest-neighbor search.
- Query an LLM (ChatGroq) via a RetrievalQA chain.
- Minimal browser UI to chat with the system.

## System Architecture

Components and their roles:
- Frontend: templates/chat.html — a small HTML/JS page that posts user messages to the backend.
- Backend: main.py (Flask) — loads PDFs, builds embeddings and FAISS index, exposes chat API endpoints, and calls the LLM via langchain.
- Document store: local PDF files in DB/ — the corpus used for retrieval.
- Vector store: FAISS — in-memory index built at app start.
- Embeddings: HuggingFaceEmbeddings with sentence-transformers/all-MiniLM-L6-v2.
- LLM: Groq ChatGroq (configured with model_name="Llama3-8b-8192") — requires GROQ_API_KEY.

Flow:
1. App starts and loads PDFs from DB/.
2. Documents are split into chunks and embeddings are computed.
3. FAISS index is constructed and used as a retriever.
4. User message → POST /chat → RetrievalQA uses retriever + ChatGroq to produce an answer → response returned to client.

## Tech Stack

- Frontend
  - Plain HTML + JavaScript (templates/chat.html)
- Backend
  - Python (Flask)
  - python-dotenv
- AI / LLM
  - langchain_groq (ChatGroq)
  - LLM model configured in code: "Llama3-8b-8192"
  - langchain RetrievalQA and prompt templates
- Embeddings
  - HuggingFaceEmbeddings (sentence-transformers/all-MiniLM-L6-v2)
- Vector Store
  - FAISS (faiss-cpu)
- Document loaders
  - PyPDFDirectoryLoader (langchain_community)
- Tools / Libraries
  - langchain, langchain_core, langchain_community, sentence-transformers, transformers, huggingface-hub, dotenv, Flask

## Project Structure

project root
├── DB/                           # PDF(s) used as knowledge base
│   └── DB_Football Player Stats.pdf
├── templates/
│   └── chat.html                 # Minimal web UI
├── .env                          # Environment variables (GROQ_API_KEY)
├── main.py                       # Flask app and RAG pipeline (entry point)
├── README.md                     # This file (recommended to replace existing)
└── .gitignore

Important files:
- main.py — application entry point. Responsible for loading PDFs, splitting, embedding, creating vectorstore, initializing ChatGroq, and running Flask routes.
- templates/chat.html — simple client UI.
- .env — must contain GROQ_API_KEY.

## Requirements

The repository uses these Python packages (as implied by imports in main.py). Install the matching package names below — exact version pins are not included in the repo.

Minimum prerequisites:
- Python (recommended 3.8+)
- pip

Python packages (install with pip):
- flask
- python-dotenv
- langchain
- langchain-groq
- langchain-core
- langchain-community
- sentence-transformers
- transformers
- huggingface-hub
- faiss-cpu
- (Optional) torch (may be required by sentence-transformers depending on environment)

Environment variables:
- GROQ_API_KEY — required. Set in .env or environment prior to running.

Notes:
- FAISS: if your environment does not support faiss-cpu, install faiss appropriate to your platform or use an alternative vectorstore.
- Hugging Face models are downloaded at runtime by sentence-transformers if not present locally.

## Installation

1. Clone the repository:
   git clone https://github.com/katarizkyo99/Chatbot-RAG-File-PDF.git
   cd Chatbot-RAG-File-PDF

2. Create and activate virtual environment (recommended):
   python -m venv .venv
   source .venv/bin/activate   # Linux/macOS
   .venv\Scripts\activate      # Windows (PowerShell or CMD)

3. Install dependencies:
   pip install flask python-dotenv langchain langchain-groq langchain-core langchain-community sentence-transformers transformers huggingface-hub faiss-cpu

(If faiss-cpu is not available for your platform, see FAISS installation instructions from the FAISS project and install the appropriate wheel.)

## Configuration

Create or edit .env in repository root with your GROQ API key:

GROQ_API_KEY=your_groq_api_key_here

Do NOT commit secrets to the repository.

## Running the Project

Development (local):
1. Ensure .env contains GROQ_API_KEY.
2. Start the app:
   python main.py
3. Visit the URL printed by Flask (by default http://127.0.0.1:5000/) and open the chat UI.

Notes:
- On startup the application loads documents in DB/, splits up to the first 20 documents (per current code), computes embeddings, and builds an in-memory FAISS index. This can take time depending on the PDFs and your environment.
- The app currently runs with Flask debug=True in main.py.

## API endpoints
| Method | Endpoint | Description |
| ------ | -------- | ----------- |
| GET | / | Serve chat UI (templates/chat.html) |
| POST | /chat | Accept JSON { "message": "<text>" } and return { "response": "<LLM answer>" } |

Example curl:
curl -X POST http://127.0.0.1:5000/chat -H "Content-Type: application/json" -d '{"message":"Who is the top scorer?"}'

## Example Conversation (format)
Below is an example of the request/response format. The "response" shown is illustrative — actual answers are produced by the configured LLM using your PDF documents as context.

Request (JSON):
{
  "message": "Who is the top scorer in the DB?"
}

Response (JSON example):
{
  "response": "Based on the provided documents, the top scorer is Player A with 24 goals in the season (source: DB_Football Player Stats.pdf)."
}

Note: The textual content of the response will vary depending on the indexed PDFs and the LLM output.

## Usage

- Use the web UI at / to enter queries.
- Backend returns a text response assembled by the RetrievalQA chain calling the configured LLM with retrieved document context.

## Database / Data

- Document source: the DB/ folder with PDF files (DB_Football Player Stats.pdf included).
- Vector index: created at runtime using FAISS from document chunk embeddings.
- The current code selects the first 20 loaded documents and splits them. There is no persistent index file — index is built in memory on each start.

## Docker / Deployment

- There is no Dockerfile or docker-compose shipped in this repository. If you want containerization, create a Dockerfile that installs the Python environment, system packages for FAISS, and copies the repository files. Be mindful of large downloads (LLM interactions or sentence-transformers models).

## Troubleshooting

- Missing GROQ API key: app will raise ValueError("API Key is missing. Set GROQ_API_KEY in the .env file.")
- FAISS import or binary issues: install faiss-cpu matching your platform or use another vectorstore backend.
- Large PDFs or many documents: initial indexing may be slow and memory intensive.
- LLM or API quota errors: ensure GROQ_API_KEY is valid and you have capacity/quotas.

## Security Notes

- Do not commit API keys or other secrets to the repository. Use .env (which is present in repo but should not contain a real key) or a secrets manager.
- The project calls external model APIs; protect API keys and monitor usage.
- The app runs in debug mode by default in main.py — do not run debug=True in a public production environment.

## Development

- To extend:
  - Add persistent storage for the FAISS index (persist index to disk).
  - Add a requirements.txt or pyproject.toml for reproducible installs.
  - Add logging, error handling, and configurable chunk sizes.
  - Add endpoint authentication if exposing the API.

## Contributing

- Open issues or pull requests with clear descriptions.
- Add tests for pipeline steps if expanding the project.
- Please follow code style and include dependency pinning when adding features.

## License

No license file detected in this repository. If you want a license, add a LICENSE file with the selected license.

## Author

Repository owner: katarizkyo99 — https://github.com/katarizkyo99
