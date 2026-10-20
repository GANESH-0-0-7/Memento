# Memento 🧠

**Memento** is a private, AI-powered document knowledge workspace. Upload PDF, DOCX, or TXT documents and chat with an AI assistant that retrieves relevant document content and provides answers with source citations.

Memento runs locally on Windows using Ollama for AI inference, PostgreSQL with pgvector for vector storage, Spring Boot for the backend, and React for the frontend.

## ✨ Features

- Upload PDF, DOCX, and TXT documents.
- Chat with an AI assistant about uploaded documents.
- Retrieval-augmented generation (RAG).
- Semantic search using vector embeddings.
- Source citations for document-based answers.
- Local AI inference using Ollama.
- JWT-based authentication.
- Streaming chat responses using Server-Sent Events (SSE).

## 🛠️ Technology Stack

| Component | Technology |
|---|---|
| Backend | Java 21, Spring Boot 3, Spring AI |
| Build tool | Apache Maven |
| Database | PostgreSQL 17 |
| Vector search | pgvector |
| AI runtime | Ollama |
| Chat model | `qwen2.5:3b` |
| Embedding model | `nomic-embed-text` |
| Frontend | React 19, TypeScript, Vite |
| Styling | Tailwind CSS |
| State management | Zustand |
| Authentication | JWT |

## 📋 Prerequisites

Install and configure:

- Java JDK 21
- Apache Maven 3.9+
- Node.js and npm
- PostgreSQL 17
- pgvector extension
- Ollama

**Docker is not required.** The instructions below use Windows PowerShell and keep the project and AI model files on the D: drive.

## 📁 Project Structure

```text
D:\Memento\Memento\
├── backend\
│   ├── src\
│   ├── pom.xml
│   └── .env
├── frontend\
│   ├── src\
│   ├── package.json
│   └── vite.config.ts
├── README.md
├── API_SPEC.md
├── ARCHITECTURE.md
└── FRONTEND.md
```

## 🚀 How to Run Memento Locally

Follow these steps in order. You will use **three separate PowerShell windows** and keep all three open while running the application.

### Step 1: Check PostgreSQL

Ensure the PostgreSQL Windows service is running and that the `memento` database and `memento_user` database user exist.

The pgvector extension must be enabled in the `memento` database:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

Run this SQL in a PostgreSQL client connected to the `memento` database. Do not enter SQL directly into PowerShell.

### Step 2: Configure the backend environment

Make sure `backend\.env` contains the correct database connection and local AI settings. Example:

```dotenv
DB_URL=jdbc:postgresql://localhost:5432/memento
DB_USER=memento_user
DB_PASSWORD=YOUR_POSTGRESQL_PASSWORD

OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_CHAT_MODEL=qwen2.5:3b
OLLAMA_EMBEDDING_MODEL=nomic-embed-text
EMBEDDING_DIMENSIONS=768

CORS_ORIGIN=http://localhost:5173
```

Replace the database password placeholder with your actual password. Also configure the JWT secret and any other variables required by the backend.

Keep `.env` private and do not commit credentials to Git.

### Step 3: Open PowerShell 1 — Ollama AI server

Open your first PowerShell window.

Navigate to the Ollama installation directory only if needed, then start the server:

```powershell
$env:OLLAMA_MODELS = "D:\AIModels\Ollama"
& "D:\Tools\Ollama\ollama.exe" serve
```

If Ollama is already running, do not start another server. Use the existing Ollama server instead.

Open a separate terminal if you need to check that the models are installed:

```powershell
& "D:\Tools\Ollama\ollama.exe" list
```

The required models are:

- `qwen2.5:3b` — generates chat responses.
- `nomic-embed-text` — creates document embeddings.

Keep PowerShell 1 open while Memento is running.

### Step 4: Open PowerShell 2 — Spring Boot backend

Open a **second PowerShell window**. This window runs the Java backend.

Navigate to the project root:

```powershell
cd D:\Memento\Memento
```

Set the database environment variables for this PowerShell session:

```powershell
$env:DB_URL = "jdbc:postgresql://localhost:5432/memento"
$env:DB_USER = "memento_user"
$env:DB_PASSWORD = Read-Host "Enter your PostgreSQL password"
```

Enter your PostgreSQL password when prompted.

Start the Spring Boot backend:

```powershell
mvn -f backend/pom.xml spring-boot:run
```

Wait until the logs indicate that the application has started successfully and Tomcat is running on port `8080`.

Keep PowerShell 2 open.

**Important:** The environment variables above apply only to this PowerShell session. Repeat them if you open a new backend terminal. If your backend loads the values correctly from `backend\.env`, the manual setup may not be necessary.

### Step 5: Open PowerShell 3 — React frontend

Open a **third PowerShell window**.

Navigate to the frontend directory:

```powershell
cd D:\Memento\Memento\frontend
```

Install frontend dependencies if you have not already done so:

```powershell
npm install
```

Start the Vite development server:

```powershell
npm run dev
```

Wait for Vite to display its local URL, normally:

```text
http://localhost:5173/
```

Keep PowerShell 3 open.

You only need to run `npm install` again when frontend dependencies change or after setting up the project on a new machine.

### Step 6: Open Memento in your browser

Open your browser and visit:

**Frontend:** http://localhost:5173

Other useful local addresses:

| Service | URL |
|---|---|
| Memento frontend | http://localhost:5173 |
| Spring Boot backend | http://localhost:8080 |
| Swagger UI | http://localhost:8080/swagger-ui.html |
| Ollama API | http://localhost:11434 |

The Swagger UI address depends on the backend's OpenAPI configuration. If it does not load, check the configured Swagger path.

### Step 7: Use Memento

Once all services are running:

1. Open the frontend.
2. Register or sign in if authentication is enabled.
3. Upload a supported document.
4. Wait for document processing and ingestion to finish.
5. Open a chat session and ask a question about the uploaded document.
6. Review the response and any source citations.

## 🧠 How Memento Works

1. A user uploads a document.
2. The backend extracts the document text.
3. The text is split into smaller chunks.
4. Ollama generates embeddings for the chunks.
5. PostgreSQL with pgvector stores and retrieves vector data.
6. Relevant chunks are retrieved when the user asks a question.
7. The chat model generates an answer using the retrieved context.
8. The application returns the response and source citations.

## 🧪 Validation and Build

Run these commands from the repository root.

### Run backend tests

```powershell
mvn -f backend/pom.xml clean test
```

### Build the frontend

```powershell
npm run build --prefix frontend
```

### Run frontend tests

If the frontend defines a test script:

```powershell
npm test --prefix frontend
```

## 🔌 API

The application uses versioned endpoints under `/api/v1`.

The main API groups include:

- Authentication
- Knowledge spaces
- Documents
- Chat sessions
- Messages

Chat messages are submitted to:

```text
POST /api/v1/sessions/{sessionId}/messages
```

The implementation can stream events such as `token`, `citations`, and `done` using Server-Sent Events (SSE).

See the project documentation:

- [API Specification](API_SPEC.md)
- [Architecture](ARCHITECTURE.md)
- [Frontend Documentation](FRONTEND.md)

## ⚠️ Troubleshooting

### Backend cannot connect to PostgreSQL

- Check that the PostgreSQL service is running.
- Verify the database name, username, password, and port.
- Confirm that the backend is loading the intended environment variables.
- Check the backend terminal for the exact database error.

### Ollama is unavailable

- Check whether the Ollama server is running.
- Confirm that the API responds at `http://localhost:11434`.
- Run `ollama list` to verify that both required models are installed.

### Frontend cannot reach the backend

- Check that Spring Boot is running on port `8080`.
- Check the frontend API base URL.
- Verify CORS settings.
- Inspect the browser developer console and network requests.

### Port already in use

If port `8080`, `5173`, or `11434` is already occupied, check which application is using it before starting another server instance.

### Embedding dimension mismatch

The `nomic-embed-text` model uses 768-dimensional embeddings. Ensure the application's configuration and pgvector schema are compatible. Back up data before changing the vector schema or rebuilding stored embeddings.

## 🔐 Privacy and Security

Memento supports local AI inference through Ollama. Keep database credentials, JWT secrets, and other private configuration in environment variables, and exclude `.env` files from version control.

Local execution reduces reliance on hosted AI services, but overall privacy still depends on application configuration, network access, and operating-system security.
## 📸 Application Screenshots

### Memento Dashboard

![Memento Dashboard](docs/images/memento-dashboard.png)
## 🎯 Project Goal

Memento demonstrates how to build a full-stack, AI-powered document knowledge assistant using Java, Spring Boot, React, PostgreSQL, pgvector, and local language models.
