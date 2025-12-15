# Anatomy Physiology Exam Bot

An interactive Retrieval-Augmented Generation (RAG) chatbot that answers human anatomy and physiology exam questions using a custom corpus of lecture notes and study material. The app is deployed as a Gradio UI on Hugging Face Spaces and backed by a persisted LlamaIndex vector store.

> Live demo: `https://huggingface.co/spaces/Sidd2005/Anatomy-Physiology-Exam-Bot`

---

## Features

- Conversational Q&A interface for anatomy and physiology exam prep.  
- Retrieval-Augmented Generation over a curated set of PDFs and slides (blood, brain, respiratory system, heart, urinary system, etc.).  
- Vector store persisted to disk so the index is loaded instantly on startup instead of being rebuilt on every run. [web:1228]  
- Low-latency responses using Groq-hosted `llama-3.1-8b-instant`.  
- Clean Gradio `ChatInterface` front-end that works in the browser without any local setup. [web:964]

---

## Architecture

At a high level, the project combines three layers:

1. **Document indexing (offline / one-time)**  
   - Source documents are ingested with `SimpleDirectoryReader` and converted into nodes. [web:1210]  
   - LlamaIndex builds a `VectorStoreIndex` over these nodes using sentence embeddings from `sentence-transformers/all-MiniLM-L6-v2`. [web:1225]  
   - The index metadata, docstore, and vector store are persisted under the `storage/` directory. [web:1228]

2. **Retrieval + generation (online)**  
   - On app startup, the index is loaded from `storage/` via `StorageContext.from_defaults` and `load_index_from_storage`, avoiding re-indexing. [web:991]  
   - A query engine (`index.as_query_engine()`) retrieves the most relevant chunks for each user question and passes them to the LLM. [web:1230]  
   - The Groq LLM (`llama-3.1-8b-instant`) generates grounded answers conditioned on the retrieved context.

3. **UI and deployment**  
   - A simple `ask(message, history)` function wraps the query engine and returns a plain text response suitable for Gradio’s `ChatInterface`. [web:1003]  
   - The app is exposed as a Space with `sdk: gradio` and `app_file: app.py` in the Space metadata. [web:1092]

---

## Tech Stack

- **Language:** Python 3.x  
- **Frameworks:**  
  - [LlamaIndex](https://github.com/run-llama/llama_index) for indexing, retrieval, and storage. [web:998]  
  - [Gradio](https://www.gradio.app/) for the web UI. [web:1003]  
- **Embeddings:** `sentence-transformers/all-MiniLM-L6-v2`.  
- **LLM:** `llama-3.1-8b-instant` served via Groq API.  
- **Deployment:** Hugging Face Spaces (CPU runtime) with persisted storage. [web:1092]

---
## Repository Structure
.
├── app.py # Main Gradio + LlamaIndex + Groq application
├── requirements.txt # Python dependencies
├── storage/ # Persisted LlamaIndex artifacts (vector store, docstore, index store)
├── .gitattributes # LFS/Xet config created during deployment
├── .gitignore # Ignore raw data, env files, and virtualenv
├── README.md # Project description (this file)
└── LICENSE # MIT license

> Note: The original exam PDFs and PPTX files are **not** included in this repository to keep it lightweight and to respect content ownership. The `storage/` index encodes those materials for retrieval without distributing the raw documents. [web:1222][web:1228]

---

## Running Locally

1. **Clone the repo and create a virtual environment:**

git clone https://github.com/sid-42-d/Anatomy-Physiology-Exam-Bot-Deployed-using-Hugging-Face-.git
cd Anatomy-Physiology-Exam-Bot-Deployed-using-Hugging-Face-
python -m venv venv
source venv/bin/activate # On Windows: venv\Scripts\activate


2. **Install dependencies:**

pip install -r requirements.txt


3. **Set environment variables:**

Create a `.env` file with:

GROQ_API_KEY=your_groq_api_key_here


4. **Run the app:**

python app.py


Then open `http://localhost:7860` in your browser.

---

## Lessons Learned

- Handling “binary file” push rejections on Hugging Face and GitHub by separating raw `Data/` from the persisted `storage/` index. [web:1122]  
- Using access tokens instead of passwords for pushing to Hugging Face and GitHub. [web:1082]  
- Debugging Gradio `ChatInterface` return types to avoid `AttributeError: 'tuple' object has no attribute 'get'`. [web:1202]  
- Designing a small, focused RAG system tailored to a specific exam domain (anatomy & physiology) rather than generic web search. [web:1231]

