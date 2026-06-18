# LAI Setup & Installation Guide

Welcome to the Legal Document Intelligence (LAI) platform setup guide. This document provides step-by-step instructions to install, configure, and run the LAI system on your local machine for development and prepare it for production environments.

---

## 💻 System Requirements

Before getting started, ensure your system meets the following specifications:

| Requirement | Supported Version(s) | Notes |
| :--- | :--- | :--- |
| **Node.js** | `>= 18.x` (LTS v20 recommended) | Required to run and compile the Vite + React frontend |
| **Python** | `>= 3.10` (Python 3.11 recommended) | Required for the FastAPI backend and AI processing pipeline |
| **Docker** | `Docker Desktop >= 4.x` | Required for hosting the local PostgreSQL 16 database |
| **RAM** | Minimum `8 GB` (Recommended `16 GB`) | Docling parser uses local parsing engines which can be CPU/RAM intensive |
| **Network** | Broadband Internet Connection | Required for first-run model weight downloads (Docling) and Gemini API calls |

---

## 🛠️ Step-by-Step Installation

LAI consists of three main components:
1. **Database**: PostgreSQL 16 managed via Docker Compose.
2. **Backend**: FastAPI web server handling AI pipelines and storage operations.
3. **Frontend**: Vite + React 19 SPA.

Follow the instructions below matching your operating system.

### 1. Database Provisioning (All OS)

Ensure your Docker daemon is running, open your terminal at the project root directory, and launch the database container:

```bash
docker compose up -d
```

To verify the database is healthy and running, execute:
```bash
docker ps --filter name=lai-postgres
```

---

### 2. Operating System Setup Guides

Select the instructions below that match your operating system.

<details>
<summary><b>🪟 Windows 11 / 10 Installation</b></summary>

#### A. Backend Setup
1. Open PowerShell as Administrator or standard terminal in `Lai/backend`.
2. Create a virtual environment:
   ```powershell
   python -m venv .venv
   ```
3. Activate the virtual environment:
   ```powershell
   .venv\Scripts\activate
   ```
4. Install backend dependencies in editable mode:
   ```powershell
   pip install -e .
   ```
   > [!NOTE]
   > On Windows, some components of IBM Docling may compile C++ extensions. If you encounter errors, ensure you have **Visual Studio Build Tools** installed with the "Desktop development with C++" workload.
5. Create your environment file:
   ```powershell
   Copy-Item .env.example .env
   ```
6. Open `.env` and fill in your variables (see [Environment Variables](#-environment-variable-reference) below).

#### B. Frontend Setup
1. Navigate to the frontend directory:
   ```powershell
   cd ..\frontend
   ```
2. Install dependencies:
   ```powershell
   npm install
   ```

</details>

<details>
<summary><b>🍎 macOS (Intel / Apple Silicon) Installation</b></summary>

#### A. Backend Setup
1. Open terminal in `Lai/backend`.
2. Create a virtual environment:
   ```bash
   python3 -m venv .venv
   ```
3. Activate the virtual environment:
   ```bash
   source .venv/bin/activate
   ```
4. Install backend dependencies in editable mode:
   ```bash
   pip install -e .
   ```
   > [!TIP]
   > On Apple Silicon (M1/M2/M3), ensure you have Xcode Command Line Tools installed (`xcode-select --install`) before running pip install to compile any native parser dependencies.
5. Create your environment file:
   ```bash
   cp .env.example .env
   ```
6. Open `.env` and fill in your variables.

#### B. Frontend Setup
1. Navigate to the frontend directory:
   ```bash
   cd ../frontend
   ```
2. Install dependencies:
   ```bash
   npm install
   ```

</details>

<details>
<summary><b>🐧 Linux (Ubuntu / Debian / RHEL) Installation</b></summary>

#### A. Backend Setup
1. Open terminal in `Lai/backend`.
2. Install system build dependencies (required for document processing and psycopg2 compiling):
   ```bash
   sudo apt-get update && sudo apt-get install -y build-essential python3-dev libpq-dev
   ```
3. Create a virtual environment:
   ```bash
   python3 -m venv .venv
   ```
4. Activate the virtual environment:
   ```bash
   source .venv/bin/activate
   ```
5. Install backend dependencies in editable mode:
   ```bash
   pip install -e .
   ```
6. Create your environment file:
   ```bash
   cp .env.example .env
   ```
7. Open `.env` and fill in your variables.

#### B. Frontend Setup
1. Navigate to the frontend directory:
   ```bash
   cd ../frontend
   ```
2. Install dependencies:
   ```bash
   npm install
   ```

</details>

---

## 🔑 Environment Variable Reference

The backend uses a `.env` file in the `backend/` directory to manage local settings.

| Key | Example / Default | Required? | Purpose |
| :--- | :--- | :---: | :--- |
| `DATABASE_URL` | `postgresql://lai:lai_dev_password@localhost:5432/lai` | **Yes** | SQLAlchemy connection URI pointing to the PostgreSQL database. |
| `GEMINI_API_KEY` | `AIzaSyD...` | **Yes** | API key to access Google Gemini models via Google AI Studio. |

> [!IMPORTANT]
> Never commit your active `.env` file to version control. The `.gitignore` file is pre-configured to ignore `.env` files.

---

## ⚡ How to Run LAI

### 1. Running in Development Mode

The development environment supports hot-reloading for both backend endpoints and frontend components.

#### A. Start Backend
In the `backend` directory (with active `.venv`):
```bash
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```
- **FastAPI Server**: Running on `http://localhost:8000`
- **Interactive Swagger Docs**: Available at `http://localhost:8000/docs`
- **Alternative ReDoc Docs**: Available at `http://localhost:8000/redoc`

#### B. Start Frontend
In the `frontend` directory:
```bash
npm run dev
```
- **Vite Application**: Running on `http://localhost:5173`
- **API Proxy**: Requests made to `/api/*` are automatically proxied to `http://localhost:8000/api/*` in real-time.

---

### 2. Running in Production Mode

For staging or production deployments, follow these guidelines to optimize performance, scalability, and security.

#### A. Production Database
- Replace the local Docker Compose Postgres service with a managed database instance (e.g., AWS RDS, GCP Cloud SQL, or a hardened, backed-up PostgreSQL instance).
- Update the `DATABASE_URL` connection string to use SSL connections (`sslmode=require`).

#### B. Production Backend
Avoid using the uvicorn development reloader. Instead, run with a process manager or uvicorn with multi-worker configurations:
```bash
# Run backend with 4 workers in production
uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
```

#### C. Production Frontend
Compile and serve the frontend as optimized static assets:
1. Build the production build in the `frontend` directory:
   ```bash
   npm run build
   ```
2. This generates a production-ready bundle in `frontend/dist`.
3. Serve these static assets using a high-performance web server like **Nginx**. 

An example Nginx configuration routing static files and proxying API endpoints is shown below:

```nginx
server {
    listen 80;
    server_name lai-intelligence.domain;

    # Serve compiled static files
    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;
    }

    # Proxy backend API requests
    location /api {
        proxy_pass http://backend-api:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## 🔍 Common Issues & Troubleshooting

### 1. Database Port Conflict (Error: `port is already allocated`)
- **Symptoms**: Docker compose fails with errors claiming port `5432` is already in use.
- **Cause**: A local instance of PostgreSQL is already running on your host machine.
- **Remedy**:
  - Stop your local Postgres service (e.g. `services.msc` on Windows, or `brew services stop postgresql` on macOS).
  - Alternatively, map a different host port in `docker-compose.yml` (e.g. `"5433:5432"`) and update your `DATABASE_URL` in `.env` to point to port `5433`.

### 2. Missing or Expired Gemini API Key (`API_KEY_INVALID` / `403`)
- **Symptoms**: Document extraction or chat commands fail, and backend logs output HTTP Forbidden or authentication errors.
- **Cause**: Missing, copy-paste truncated, or incorrect `GEMINI_API_KEY` in `backend/.env`.
- **Remedy**:
  - Verify that your API key is correctly defined in `backend/.env` without enclosing quotes: `GEMINI_API_KEY=AIzaSy...`
  - Test your key with a simple `curl` request to the Gemini API directly to confirm status.

### 3. IBM Docling Model Download Hangs on Startup
- **Symptoms**: The first document parsing request takes several minutes or fails with a timeout.
- **Cause**: On its very first run, Docling downloads essential layout parsing and OCR model weights (approx. 100-300MB) from Hugging Face.
- **Remedy**:
  - Ensure your machine has a stable, unrestricted internet connection.
  - If you are behind a corporate proxy, set the `HTTP_PROXY` and `HTTPS_PROXY` environment variables in your active terminal environment before booting the backend.

### 4. Docling C++ Extension / Build Tool Errors on Windows
- **Symptoms**: `pip install -e .` fails with errors mentioning MSVC, `cl.exe`, or compiler directories.
- **Cause**: Certain parsing dependencies require compiler capabilities to build local libraries.
- **Remedy**:
  - Download and install [Visual Studio Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/).
  - Ensure you select **"Desktop development with C++"** during the installation wizard.
  - Restart your terminal and re-run `pip install -e .`.
