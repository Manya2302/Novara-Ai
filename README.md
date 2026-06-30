# Novara AI: Zero-Knowledge Document Intelligence & Enterprise Governance Platform

Novara AI (also known as Avora AI / SecureVault) is an enterprise-grade, zero-knowledge document intelligence platform and organizational memory system. It transforms raw business documents (contracts, invoices, receipts, and compliance filings) into an active, secure knowledge base. Users can securely upload documents, search using semantic natural language, query an AI Copilot under strict privacy constraints, and visualize relationships via an automatically generated Enterprise Knowledge Graph.

---

## ⚡ Core Features & Capabilities

### Zero-Knowledge Security & Storage
- **Cryptographic Isolation:** Documents are split into chunks, compressed using **Zstandard**, and encrypted using **AES-256-GCM** (with unique IV and authentication tags per chunk).
- **Zero-Knowledge Key Wrapping:** A random 256-bit AES key is generated for each document, which is then wrapped (encrypted) using **RSA-4096** with the tenant/owner's public key.
- **Obfuscated Object Storage:** Encrypted chunks are stored in **MinIO Object Storage** under obfuscated keys, ensuring that even storage administrators cannot read document content.

### Deep Document Intelligence
- **High-Fidelity OCR & Cleaning:** Integrated text extraction pipeline leveraging **PyMuPDF** and **pdfplumber** to clean raw document layouts.
- **Automated Metadata Extraction:** Leverages AI/ML models to extract document categories, departments, fiscal years, keywords, and page counts.
- **Auto-Summarization:** Generates high-level executive summaries upon document ingestion to provide instant context in search results.

### Compliance & Contract Intelligence
- **Contract Clause Extraction:** Classifies contract clauses into specialized categories (Termination, SLA, Liability, Confidentiality, Renewal, Force Majeure).
- **Risk Severity Scoring:** Scores contracts on a severity scale (Low, Medium, High, Critical) based on clauses and highlights deviations from standard terms.
- **Audit Readiness Index:** Aggregates document metadata to evaluate compliance profiles, calculating an overall compliance score and identifying missing documents.
- **Renewal Tracker & Expiry Alerts:** Proactively calculates notice deadlines and notifies admins about contract expirations at 90/60/30-day intervals.

### RAG Copilot & Enterprise Memory
- **Multi-Mode RAG Copilot:** Enterprise-wide chat interface offering five specialized operational modes:
  - **Document Mode:** Direct queries against specific documents.
  - **Compliance Mode:** Verification against regulatory rules and internal standards.
  - **Audit Mode:** Evidence collection and audit preparedness.
  - **Knowledge Mode:** Multi-document reasoning and organizational memory.
  - **Risk Mode:** Analysis of financial and legal liabilities.
- **Hallucination Detection & Verification:** Validates generated responses by checking facts, amounts, and dates against retrieved document sources. Flags low-confidence outputs for manual admin review.
- **AI Governance Dashboard:** Provides administrative analytics for prompt template usage, system latency, user queries, and AI accuracy metrics.

### Enterprise Knowledge Graph
- **NetworkX Graph Engine:** Automatically maps relationships between parsed document nodes (`Vendor ── Contract ── Invoice ── Payment ── Audit Record`).
- **Entity Linking & Visual Explorer:** Creates visual pathways of enterprise document links to identify hidden gaps, vendors, and audit trails.

---

## 🛠️ Technology Stack

| Layer | Technologies Used |
|---|---|
| **Frontend** | React, Next.js (App Router), TypeScript, Tailwind CSS, Zustand, Radix UI, Lucide Icons |
| **Backend** | Python, Django, Django REST Framework (DRF), Celery, Redis |
| **Databases** | SQLite (Development) / PostgreSQL (Production), Qdrant (Vector Database) |
| **Storage** | MinIO (S3-compatible Object Storage) |
| **AI / RAG** | Ollama (Llama 3, Llama 3.2, Nomic-Embed-Text), Groq API (Llama-3.3-70B), Cohere API (Rerank-v3) |
| **Graphing** | NetworkX (Graph theory library for node/relation mapping) |
| **Document Processing** | PyMuPDF, pdfplumber, python-docx, openpyxl, Zstandard, Cryptography (AES/RSA) |
| **Deployment** | Docker, Docker Compose |

---

## ⚙️ RAG Processing Pipeline

```
              User Query
                  ↓
       [Multi-Query Expansion] (Generates 3-4 query variations using LLM)
                  ↓
       [Vector Embedding Search] (Search Qdrant for all variations, owner-scoped)
                  ↓
         [Document Chunking] (Extracts OCR text, chunks into paragraph segments)
                  ↓
         [Cohere Rerank API] (Reranks chunks by relevance, filters out noise)
                  ↓
        [Context Compilation] (Builds prompt template with source citations)
                  ↓
         [LLM Synthesis] (Synthesizes final answer via Groq or local Ollama)
                  ↓
      [Hallucination Check] (Validates numbers/facts vs source context)
                  ↓
   Response Output + Citations + Confidence Score + Governance Logs
```

- **Guardrail Enforcements:** 
  - Grounded answers: Asserts "I could not find this in the documents" instead of hallucinating.
  - Ownership scopes: Tenancy filters are applied directly at Qdrant search-time to prevent cross-user data leakage.
  - Citations: Returns original document names, matching excerpt snippets, and relevance scores with every answer.

---

## 📂 Backend Database Schema (Key Models)

### Core Models (`apps/users` & `apps/documents`)
* **User:** Custom user model supporting OTP authentication, email verification, and roles (`admin`, `user`, `viewer`).
* **UserProfile:** Profile storing company details, industry type, plan details, and storage quota.
* **Document:** Stores primary document state (encrypted size, compression ratio, processing status, and logical tree hierarchies).
* **DocumentChunk:** Tracks individual AES-256-GCM encrypted chunks, storing IVs, authentication tags, and MinIO addresses.
* **DocumentEncryptionKey:** Stores the AES-256 document key encrypted with the owner's RSA-4096 public key.
* **DocumentMetadata:** General document summary, categories, tags, department routing, and page counts.

### Analysis & Governance Models (`apps/contracts` & `apps/compliance`)
* **ContractAnalysis:** Key metadata parsed from contracts (parties, value, currency, risk score, and renewal notice periods).
* **ContractClause:** Stores clause breakdowns with risk flags (Green, Yellow, Red) and deviation reasons.
* **ContractRisk:** Flags specific contract risks with recommendations and resolution statuses.
* **ComplianceProfile:** Overall compliance health index and audit readiness score.
* **ExpiryAlert:** Tracks upcoming contract expirations and generates alerts at 90/60/30 days.
* **AuditPackage:** Generates bundled compliance documents, scores readiness, and highlights documentation gaps.

### RAG & Graph Models (`apps/copilot` & `apps/knowledge`)
* **CopilotConversation:** Conversation session tracking modes, pinned states, and message histories.
* **CopilotMessage:** Single message exchanges including thinking logs, token usage, latency, and confidence.
* **DocumentReference:** Links specific messages to source document citations.
* **ReasoningLog:** Captures the intermediate thinking steps, prompt template version, and hallucination flags.
* **KnowledgeNode:** Vertices of the knowledge graph representing vendors, departments, invoices, and documents.
* **KnowledgeRelationship:** Directed edges mapping relations between vertices (e.g., `belongs_to`, `linked_to`).

---

## ⚡ API Endpoint Directory

### Copilot & Chat Endpoints
* `POST /api/copilot/query/` — Main multi-mode RAG query interface. Returns answer + sources + reasoning logs.
* `GET /api/copilot/conversations/` — Retrieves conversation list with pin and search filters.
* `POST /api/copilot/reports/generate/` — Synthesizes automated reports (e.g., Vendor Compliance, Gap Analyses).
* `GET /api/copilot/recommendations/` — Proactive advice based on compliance gaps and renewal timelines.

### Knowledge Graph Endpoints
* `GET /api/knowledge/graph/` — Fetches complete NetworkX graph structure (nodes & links) for UI visualizations.
* `POST /api/knowledge/build/` — Rebuilds the graph from current database entities.
* `GET /api/knowledge/vendor/<name>/` — Resolves comprehensive profiles for vendors, linking all contracts/invoices.

### Admin & Governance Endpoints
* `GET /api/admin-panel/ai-governance/` — Aggregated metrics on average latency, confidence levels, and model usage.
* `GET /api/admin-panel/flagged-responses/` — Accesses the low-confidence queue for manual review.

---

## 🛠️ Quick Start & Setup

Follow these steps to spin up the local development environment using Docker Compose:

### 1. Configure Environment Variables
Copy the templates and configure your local environments:
```bash
cp .env.example .env
cp backend/.env.example backend/.env
cp frontend/.env.local.example frontend/.env.local
```

### 2. Launch Services
Start the database, vector store (Qdrant), object storage (MinIO), and application services:
```bash
docker-compose up -d --build
```

### 3. Initialize Databases & Admin Users
Run migrations and create an administrator account:
```bash
docker-compose exec backend python manage.py migrate
docker-compose exec backend python manage.py createsuperuser
```

### 4. Setup Ollama Models
Pull the required open-source models inside the Ollama container:
```bash
docker-compose exec ollama ollama pull nomic-embed-text
docker-compose exec ollama ollama pull llama3.2:1b
```

### 5. Seed Copilot Prompts
Load the built-in system prompts for the five RAG modes:
```bash
docker-compose exec backend python manage.py shell -c "
from apps.copilot.services.prompt_library import seed_builtin_prompts
print(seed_builtin_prompts(), 'prompts seeded')
"
```

### 6. Access the Application
* **Frontend UI:** `http://localhost:3000`
* **Admin Dashboard:** `http://localhost:3000/securevault-admin`
* **API Documentation (Swagger):** `http://localhost:8000/swagger/`
* **MinIO Console:** `http://localhost:9001`
