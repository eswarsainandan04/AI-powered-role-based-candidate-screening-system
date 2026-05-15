# PGAGI — AI-Powered Interview Assessment System

An end-to-end AI interview platform that parses a candidate's resume, retrieves role-specific knowledge via RAG, conducts an adaptive 20-minute technical interview using Groq LLM, and delivers a detailed performance report.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 14 + Tailwind CSS |
| Backend | Python — FastAPI |
| Database | Supabase PostgreSQL |
| Cloud Storage | Supabase S3 Bucket |
| LLM | Groq API (`llama-3.3-70b-versatile`) |
| Embeddings | Hugging Face Transformers |
| Vector DB | ChromaDB (local) |

---

## Project Structure

```
Project/
├── backend/
│   ├── .env                     # Environment variables
│   ├── main.py                  # FastAPI app entry point
│   ├── db.py                    # PostgreSQL connection & table setup
│   ├── chroma_db/               # ChromaDB persistent vector store
│   ├── extracted_text/          # Pre-extracted knowledge base text files
│   │   ├── ds_book.txt
│   │   └── ml_book.txt
│   ├── knowledge_base/          # Source PDF books for embedding
│   │   ├── AIML_role/
│   │   └── Data_Science_Applied_ML/
│   ├── ingestion/               # Resume upload & parsing pipeline
│   │   ├── base.py              # Main /resume/upload endpoint
│   │   ├── resume_parsing.py    # PDF → text → metadata via Groq
│   │   ├── resume_upload.py     # Supabase S3 upload helper
│   │   └── role_selection.py    # Role & knowledge level detection
│   ├── llm/                     # Interview engine
│   │   ├── interview.py         # Adaptive question generation
│   │   ├── conversation_chat.py # S3 conversation read/write helpers
│   │   ├── query_generation.py  # RAG query list generation
│   │   ├── insights.py          # Post-interview scoring & analysis
│   │   └── sessions.py          # Session list & delete endpoints
│   ├── rag_system/              # Vector search pipeline
│   │   ├── embeddings.py
│   │   └── retrieval.py
│   ├── login/
│   │   └── login.py
│   ├── signup/
│   │   └── signup.py
│   └── scripts/
│       ├── pdf_to_text.py       # Convert knowledge base PDFs to text
│       └── check_db.py
│
└── frontend/
    └── src/app/
        ├── page.js              # Landing / home page
        ├── login/page.js
        ├── signup/page.js
        ├── dashboard/page.js    # Session history dashboard
        ├── Interview/page.js    # Live interview chat UI
        └── insights/page.js     # Results & score breakdown
```

---

## Local Setup

### Prerequisites

Make sure the following are installed on your machine before you begin:

- Python 3.12+
- Node.js 18+ and npm
- Git

---

### Step 1 — Clone the Repository

```bash
git clone https://github.com/your-username/PGAGI.git
cd PGAGI
```

---

### Step 2 — Backend Setup

#### 2.1 Create and activate a virtual environment

```bash
cd backend
python -m venv venv

# Windows
venv\Scripts\activate

# macOS / Linux
source venv/bin/activate
```

#### 2.2 Install Python dependencies

```bash
pip install fastapi uvicorn psycopg2-binary python-dotenv boto3 groq \
            PyPDF2 sentence-transformers chromadb pydantic
```

#### 2.3 Configure environment variables

Create a file named `.env` inside the `backend/` folder:

```env
# ── Groq LLM ──────────────────────────────────────────
GROQ_API_KEY=your_groq_api_key_here

# ── Hugging Face ───────────────────────────────────────
HF_TOKEN=your_huggingface_token_here

# ── Supabase PostgreSQL ────────────────────────────────
POSTGRES_HOST=your_supabase_db_host
POSTGRES_PORT=6543
POSTGRES_DB=postgres
POSTGRES_USER=your_db_user
POSTGRES_PASSWORD=your_db_password

# ── Supabase S3 Storage ────────────────────────────────
SUPABASE_BUCKET=your_bucket_name
Access_key_ID=your_s3_access_key
Secret_access_key=your_s3_secret_key
SUPABASE_S3_ENDPOINT=https://your-project-ref.storage.supabase.co/storage/v1/s3

# ── Frontend origin (for CORS) ─────────────────────────
FRONTEND_ORIGIN=http://localhost:3000
```

> **Where to get these keys:**
> - `GROQ_API_KEY` → [console.groq.com](https://console.groq.com)
> - `HF_TOKEN` → [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens)
> - Supabase credentials → Your Supabase project → Settings → Database / Storage

#### 2.4 Embed the knowledge base into ChromaDB

This step converts the PDF books in `knowledge_base/` to text and indexes them into ChromaDB. **Run this once** before starting the server.

```bash
# If the extracted_text/ files don't exist yet, generate them first:
python scripts/pdf_to_text.py

# Then embed and store in ChromaDB:
python rag_system/embeddings.py
```

> The ChromaDB data is persisted in the `chroma_db/` folder automatically.

#### 2.5 Start the FastAPI backend server

```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

The API will be available at `http://localhost:8000`.  
Interactive API docs: `http://localhost:8000/docs`

---

### Step 3 — Frontend Setup

Open a new terminal window and navigate to the frontend folder:

```bash
cd frontend
```

#### 3.1 Install Node dependencies

```bash
npm install
```

#### 3.2 Configure the frontend environment

Create a `.env.local` file inside the `frontend/` folder:

```env
NEXT_PUBLIC_ENDPOINT=http://localhost:8000
```

#### 3.3 Start the Next.js development server

```bash
npm run dev
```

The frontend will be available at `http://localhost:3000`.

---

### Step 4 — Verify Everything Is Running

Open your browser and check the following:

| Service | URL | Expected |
|---|---|---|
| Backend health | `http://localhost:8000` | `{"status": "ok"}` |
| API docs | `http://localhost:8000/docs` | Swagger UI |
| Frontend | `http://localhost:3000` | Landing page |

---

## API Endpoints Reference

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/signup` | Register a new user |
| `POST` | `/login` | Authenticate and get user info |
| `GET` | `/users/profile?email=` | Fetch user profile by email |
| `POST` | `/resume/upload` | Upload resume PDF, trigger full pipeline |
| `POST` | `/interview/next` | Get next AI question / submit answer |
| `POST` | `/interview/end` | End session and trigger insight generation |
| `POST` | `/insights/generate` | Manually generate insights for a session |
| `GET` | `/insights/analysis?userid=&sessionid=` | Fetch final score and analysis |
| `GET` | `/sessions/list?userid=` | List all sessions for a user |
| `DELETE` | `/sessions/delete` | Delete a session and all its files |

---

## How the Interview Pipeline Works

```
Resume PDF Upload
      │
      ▼
PDF → Text (PyPDF2)
      │
      ▼
Groq LLM → meta_data.json  (skills, experience, role, knowledge level)
      │
      ▼
Groq LLM → query.json       (12–20 semantic search queries)
      │
      ▼
ChromaDB Retrieval → context.json  (relevant knowledge chunks)
      │
      ▼
Live Interview  (Groq adapts questions based on context + conversation history)
      │
      ▼
Session ends at 20 min
      │
      ▼
Groq LLM → analysis.json   (score, feedback, strengths, improvements)
      │
      ▼
Results Dashboard
```

---

## Session File Structure (Supabase S3)

All files for a session are stored under the path `userid/sessionid/` in the S3 bucket:

```
{userid}/{sessionid}/
    ├── your_resume.pdf       ← Original uploaded resume
    ├── meta_data.json        ← Parsed skills, role, knowledge level
    ├── query.json            ← Generated RAG queries
    ├── context.json          ← Retrieved knowledge chunks
    ├── conversation.json     ← Full Q&A transcript with feedback
    └── analysis.json         ← Final score and performance report
```

> `userid` is a UUID assigned at signup. `sessionid` is a UTC timestamp (`YYYYMMDDHHmmss`), ensuring each interview is isolated.

---

## Supported Interview Roles

The system currently supports the following target roles:

- AI/ML Engineer
- Data Analyst
- Data Science Engineer
- Backend Engineer

---

## Common Issues & Fixes

**ChromaDB empty / no context retrieved**  
Run `python rag_system/embeddings.py` to re-index the knowledge base before starting the server.

**`GROQ_API_KEY is not set` error**  
Ensure the `.env` file is in the `backend/` root directory (not the project root) and that the key has no extra spaces or quotes.

**CORS errors in the browser**  
Check that `FRONTEND_ORIGIN` in `.env` exactly matches the URL your frontend runs on (e.g. `http://localhost:3000`).

**Supabase S3 upload fails**  
Verify `SUPABASE_BUCKET` exists in your Supabase project and that the `Access_key_ID` / `Secret_access_key` belong to a service role with storage write permissions.

**PostgreSQL connection refused**  
Confirm `POSTGRES_HOST`, `POSTGRES_PORT`, and credentials are correct. Supabase pooler connections typically use port `6543`.

---

## Screenshots

### Landing Page
![Landing Page](screenshots/img1.jpg)

### Interview Session
![Interview Session](screenshots/img2.jpg)

> Place your screenshots in a `screenshots/` folder at the project root and name them `img1.jpg`, `img2.jpg`, etc.

---

## License

This project is for assessment and educational purposes.