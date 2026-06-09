# Team Contribution Guide

Welcome to Lai! This document lists all available tasks for team members. **Pick any task you want**, comment your name next to it, and raise a PR when done.

## How to Contribute

1. Fork or clone the repo: `git clone https://github.com/mist-ic/Lai.git`
2. Create a branch: `git checkout -b feat/your-task-name`
3. Do your work
4. Push and open a PR against `master`
5. Tag @mist-ic for review

## Setup (read before starting)

### Frontend (for UI tasks)
```bash
cd frontend
npm install
npm run dev
# App runs at http://localhost:5173
```

### Backend (for test/backend tasks)
```bash
cd backend
python -m venv .venv
# Windows: .venv\Scripts\activate
# Mac/Linux: source .venv/bin/activate
pip install -e .
# Copy .env.example to .env and add your Gemini API key
uvicorn app.main:app --reload
# API runs at http://localhost:8000
```

### Database
```bash
docker compose up -d
# Starts PostgreSQL at localhost:5432
```

---

## Available Tasks

### Task 1: End-to-End Testing with Sample Contracts

**What**: Create 3 test contracts (PDF/DOCX) and verify the full pipeline works.

**Details**:
- Create or find 3 sample contracts:
  - A standard NDA (low risk)
  - A SaaS agreement with some problematic clauses (medium-high risk)
  - An employment contract with intentionally risky clauses (high risk, e.g. uncapped indemnity, no termination notice, broad IP assignment)
- Upload each through the UI or API
- Verify clause extraction finds all 7 clause types (indemnity, limitation of liability, governing law, termination, IP ownership, payment terms, confidentiality)
- Verify risk scoring catches the planted problematic clauses
- Verify the executive summary is accurate and readable
- Compare the same clause type across all 3 contracts
- Test the chat feature with at least 5 questions per contract
- Document results in a `tests/e2e_results.md` file with screenshots

**Files to create**:
- `test-contracts/standard_nda.pdf` (or .docx)
- `test-contracts/risky_saas.pdf` (or .docx)
- `test-contracts/employment_contract.docx`
- `tests/e2e_results.md`

**Skills needed**: Basic testing, document creation

---

### Task 2: Unit Tests for Backend Services

**What**: Write pytest unit tests for the risk scoring rules and API endpoints.

**Details**:
- Install pytest: `pip install pytest pytest-asyncio httpx`
- Write tests for the rule-based risk scorer (`backend/app/services/scorer.py`):
  - Test each of the 13 risk patterns triggers correctly
  - Test score clamping (stays within 0-100)
  - Test `compute_overall_risk` with various clause score combinations
  - Test edge cases: empty text, very long text, no patterns match
- Write tests for API endpoints using FastAPI TestClient:
  - POST /api/contracts/upload (valid file, invalid file type, oversized file)
  - GET /api/contracts (empty list, with contracts)
  - GET /api/contracts/{id} (valid, 404)
  - GET /api/health
- Add a `conftest.py` with test database setup (use SQLite in-memory for tests)

**Files to create**:
- `backend/tests/__init__.py`
- `backend/tests/conftest.py`
- `backend/tests/test_scorer.py`
- `backend/tests/test_api.py`

**Run with**: `cd backend && pytest tests/ -v`

**Skills needed**: Python, pytest

---

### Task 3: Frontend Polish - Upload Page and Responsive Design

**What**: Improve the Upload page UX and make the entire app responsive on tablet/mobile.

**Details**:

Upload page improvements:
- Add an upload progress bar (show percentage while uploading)
- Add file type icons (PDF icon vs DOCX icon)
- Add a "recent uploads" section below the drop zone showing last 3 uploads
- Add smooth transition animations when file is selected/removed
- Show estimated analysis time after upload starts

Responsive design (all pages):
- Sidebar should collapse to a hamburger menu on screens < 768px
- Dashboard stats grid: 2 columns on tablet, 1 column on mobile
- Contract page: stack the risk gauge and radar chart vertically on mobile
- Compare page: stack the comparison cards vertically on mobile
- Chat page: full width on mobile with sticky input at bottom
- Tables should horizontally scroll on small screens

**Files to modify**:
- `frontend/src/pages/UploadPage.tsx`
- `frontend/src/components/layout/AppLayout.tsx`
- `frontend/src/pages/DashboardPage.tsx`
- `frontend/src/pages/ContractPage.tsx`
- `frontend/src/pages/ComparePage.tsx`
- `frontend/src/pages/ChatPage.tsx`

**Test by**: Resize browser window or use Chrome DevTools device toolbar

**Skills needed**: React, Tailwind CSS, responsive design

---

### Task 4: Frontend Polish - Loading States, Error Handling, and Animations

**What**: Add proper loading skeletons, error states, toast notifications, and micro-animations.

**Details**:

Loading states:
- Add skeleton loaders for Dashboard contract table (animated placeholder rows)
- Add skeleton for Contract page while analysis results load
- Add a pulsing "analyzing" animation on the contract card in dashboard while processing

Error handling:
- Add a toast notification system (use react-hot-toast or build a simple one)
- Show toast on: upload success, upload failure, analysis complete, analysis failed, delete success
- Add error boundary component that catches React errors and shows a friendly message
- Add retry buttons on failed API calls

Animations:
- Add staggered fade-in animation for clause cards (each card appears 50ms after the previous)
- Add smooth number counting animation for the risk score gauge (count up from 0 to final score)
- Add hover scale effect on stat cards in dashboard
- Add transition on clause card expand/collapse (smooth height animation)

**Files to create/modify**:
- `frontend/src/components/ui/Skeleton.tsx` (create)
- `frontend/src/components/ui/Toast.tsx` (create)
- `frontend/src/components/ui/ErrorBoundary.tsx` (create)
- `frontend/src/pages/DashboardPage.tsx` (modify)
- `frontend/src/pages/ContractPage.tsx` (modify)
- `frontend/src/index.css` (add animation keyframes)

**Skills needed**: React, CSS animations, error handling patterns

---

### Task 5: Documentation and API Docs

**What**: Write comprehensive setup documentation and API reference.

**Details**:
- Create a detailed setup guide covering:
  - System requirements (Node.js, Python, Docker versions)
  - Step-by-step installation for Windows, macOS, and Linux
  - Environment variable reference (what each one does)
  - Common issues and troubleshooting (port conflicts, Docker issues, API key problems)
  - How to run in development vs production mode

- Create API documentation:
  - Document every endpoint with request/response examples
  - Include curl commands for each endpoint
  - Document error codes and their meanings
  - Add Postman collection or similar

- Write a brief architecture overview with a diagram explaining:
  - How the analysis pipeline works (parse -> extract -> score -> summarize)
  - How the frontend communicates with the backend
  - Database schema explanation

**Files to create**:
- `docs/setup-guide.md`
- `docs/api-reference.md`
- `docs/architecture.md`

**Skills needed**: Technical writing, Markdown

---

### Task 6: Baseline Seeding Script and Additional Baselines

**What**: Create a script to seed market-standard baselines into the database, and add a "services agreement" baseline.

**Details**:
- Create a Python script that reads all JSON files from `backend/app/baselines/` and inserts them into the PostgreSQL `baselines` table
- The script should be idempotent (running it twice does not create duplicates)
- Add a new baseline file: `backend/app/baselines/services.json` for general services/consulting agreements with clauses for:
  - Indemnity (standard mutual indemnity)
  - Limitation of liability (capped at contract value)
  - Governing law (standard jurisdiction)
  - Termination (30-day notice with cure period)
  - IP ownership (client owns deliverables, vendor retains pre-existing IP)
  - Payment terms (net-30, milestone-based)
  - Confidentiality (mutual, 3-year survival)
- Add a CLI command or management script: `python -m app.seed_baselines`
- Add a button in the baselines API to trigger re-seeding

**Files to create**:
- `backend/app/seed_baselines.py`
- `backend/app/baselines/services.json`

**Skills needed**: Python, SQL/SQLAlchemy basics

---

## Task Assignment

| Task | Assigned To | Status |
|------|-------------|--------|
| Task 1: E2E Testing | | Not started |
| Task 2: Unit Tests | | Not started |
| Task 3: Responsive Design | | Not started |
| Task 4: Loading/Error/Animations | | Not started |
| Task 5: Documentation | | Not started |
| Task 6: Baselines Script | | Not started |

> There are 6 tasks and 4 team members. Everyone picks at least 1 task. If you finish early, grab another one. Tasks 1-2 are backend-focused, Tasks 3-4 are frontend-focused, Task 5 is docs, Task 6 is a mix.

## Code Style

- **No emdashes** in any code, comments, or documentation. Use hyphens (-) or commas.
- **Python**: Follow PEP 8, use type hints
- **TypeScript**: Use strict mode, prefer `const` over `let`
- **CSS**: Use Tailwind utility classes, follow existing patterns
- **Commits**: Use conventional commits (feat:, fix:, test:, docs:)
- **Branch naming**: `feat/task-name`, `fix/bug-name`, `test/test-name`
