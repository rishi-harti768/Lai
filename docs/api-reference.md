# LAI API Reference Guide

Welcome to the Legal Document Intelligence (LAI) API Reference. The LAI backend is a RESTful API built on FastAPI, designed to handle multi-format document parsing, LLM-powered legal clause extraction, risk assessment, side-by-side contract comparison, and conversational contract Q&A.

---

## 🌐 Global Settings & Conventions

- **Default Base URL**: `http://localhost:8000` (Local development)
- **Base Routing Prefix**: `/api`
- **Request Payloads**: All POST requests accept JSON payloads with `Content-Type: application/json` unless stated otherwise (e.g. multipart file uploads).
- **Response Payloads**: All success responses return a JSON object. Dates are formatted as ISO 8601 strings (`YYYY-MM-DDTHH:MM:SSZ`).

---

## 🏥 Health Endpoint

### 1. System Health Check
Check the service status, version, and confirm database connectivity.

- **HTTP Method**: `GET`
- **Path**: `/api/health`
- **Headers**:
  - `Accept: application/json`

#### Example Curl Command
```bash
curl -X GET "http://localhost:8000/api/health" \
     -H "Accept: application/json"
```

#### Response Example (`200 OK`)
```json
{
  "status": "healthy",
  "service": "Legal Document Intelligence System",
  "version": "0.1.0"
}
```

---

## 📁 Contracts Management Endpoints

### 1. Upload Contract File
Upload a contract document (PDF or DOCX format) for ingestion. This initializes a contract record with status set to `uploaded`.

- **HTTP Method**: `POST`
- **Path**: `/api/contracts/upload`
- **Content-Type**: `multipart/form-data`
- **Payload**:
  - `file`: Binary file data (Accepts `.pdf` or `.docx` extensions. Max size: 10MB).

#### Example Curl Command
```bash
curl -X POST "http://localhost:8000/api/contracts/upload" \
     -F "file=@/path/to/my_contract.pdf"
```

#### Response Example (`200 OK`)
```json
{
  "id": "7f805a41-3b7c-4734-91c6-cf881335cb8a",
  "filename": "my_contract.pdf",
  "file_type": "pdf",
  "status": "uploaded",
  "overall_risk_score": null,
  "risk_level": null,
  "risk_breakdown": null,
  "created_at": "2026-06-18T05:38:22.000Z",
  "updated_at": "2026-06-18T05:38:22.000Z"
}
```

---

### 2. List All Ingested Contracts
Fetch all uploaded contracts ordered by creation date (newest first).

- **HTTP Method**: `GET`
- **Path**: `/api/contracts`
- **Headers**:
  - `Accept: application/json`

#### Example Curl Command
```bash
curl -X GET "http://localhost:8000/api/contracts" \
     -H "Accept: application/json"
```

#### Response Example (`200 OK`)
```json
{
  "contracts": [
    {
      "id": "7f805a41-3b7c-4734-91c6-cf881335cb8a",
      "filename": "my_contract.pdf",
      "file_type": "pdf",
      "status": "complete",
      "overall_risk_score": 38.5,
      "risk_level": "medium",
      "risk_breakdown": {
        "financial": 20.0,
        "operational": 45.0,
        "legal": 50.0,
        "reputational": 10.0
      },
      "created_at": "2026-06-18T05:38:22.000Z",
      "updated_at": "2026-06-18T05:41:12.000Z"
    }
  ]
}
```

---

### 3. Get Contract Details
Retrieve complete details for a single contract, including its AI executive summary and nested extracted clauses.

- **HTTP Method**: `GET`
- **Path**: `/api/contracts/{contract_id}`
- **Path Parameters**:
  - `contract_id`: (String, UUID) Unique identifier of the contract.

#### Example Curl Command
```bash
curl -X GET "http://localhost:8000/api/contracts/7f805a41-3b7c-4734-91c6-cf881335cb8a" \
     -H "Accept: application/json"
```

#### Response Example (`200 OK`)
```json
{
  "id": "7f805a41-3b7c-4734-91c6-cf881335cb8a",
  "filename": "my_contract.pdf",
  "file_type": "pdf",
  "status": "complete",
  "overall_risk_score": 38.5,
  "risk_level": "medium",
  "risk_breakdown": {
    "financial": 20.0,
    "operational": 45.0,
    "legal": 50.0,
    "reputational": 10.0
  },
  "created_at": "2026-06-18T05:38:22.000Z",
  "updated_at": "2026-06-18T05:41:12.000Z",
  "executive_summary": "{\"formatted\": \"### Executive Summary\\nThis agreement outlines standard services...\", \"structured\": {}}",
  "clauses": [
    {
      "id": "a82df6a0-53cf-42bb-ae4b-70337c76251b",
      "clause_type": "indemnity",
      "section_number": "Sec 8.1",
      "title": "Indemnification",
      "original_text": "Supplier shall defend, indemnify and hold harmless customer from...",
      "plain_english_summary": "Supplier protects customer from third party legal claims.",
      "risk_score": 25.0,
      "risk_level": "low",
      "risk_category": "legal",
      "market_deviation": "standard",
      "deviation_explanation": "Matches standard commercial vendor protections."
    }
  ]
}
```

---

### 4. Delete Ingested Contract
Delete a contract from the system. This permanently removes the uploaded file from the server's disk and cascades deletion to all database clauses associated with it.

- **HTTP Method**: `DELETE`
- **Path**: `/api/contracts/{contract_id}`
- **Path Parameters**:
  - `contract_id`: (String, UUID) Unique identifier of the contract.

#### Example Curl Command
```bash
curl -X DELETE "http://localhost:8000/api/contracts/7f805a41-3b7c-4734-91c6-cf881335cb8a"
```

#### Response Example (`200 OK`)
```json
{
  "message": "Contract deleted successfully"
}
```

---

## 🧠 AI Analysis Pipeline Endpoints

### 1. Trigger AI Analysis Pipeline
Launches the asynchronous AI analysis background pipeline. The contract status changes to `parsing` immediately.

- **HTTP Method**: `POST`
- **Path**: `/api/contracts/{contract_id}/analyze`
- **Path Parameters**:
  - `contract_id`: (String, UUID) Unique identifier of the contract.

#### Example Curl Command
```bash
curl -X POST "http://localhost:8000/api/contracts/7f805a41-3b7c-4734-91c6-cf881335cb8a/analyze" \
     -H "Content-Length: 0"
```

#### Response Example (`200 OK`)
```json
{
  "contract_id": "7f805a41-3b7c-4734-91c6-cf881335cb8a",
  "status": "parsing",
  "progress": 0.0,
  "message": "Analysis pipeline started"
}
```

---

### 2. Get Analysis Status
Polls the active processing state and maps current status to completion progress percentages.

- **HTTP Method**: `GET`
- **Path**: `/api/contracts/{contract_id}/status`
- **Path Parameters**:
  - `contract_id`: (String, UUID) Unique identifier of the contract.

#### Status Mapping Heuristics
- `uploaded` (0.0)
- `parsing` (0.2)
- `parsed` (0.4)
- `analyzing` (0.6)
- `complete` (1.0)
- `error` (0.0)

#### Example Curl Command
```bash
curl -X GET "http://localhost:8000/api/contracts/7f805a41-3b7c-4734-91c6-cf881335cb8a/status"
```

#### Response Example (`200 OK`)
```json
{
  "contract_id": "7f805a41-3b7c-4734-91c6-cf881335cb8a",
  "status": "analyzing",
  "progress": 0.6,
  "message": "Contract is currently: analyzing"
}
```

---

### 3. Get Extracted Clauses
Get the full array of AI-extracted and categorized legal clauses.

- **HTTP Method**: `GET`
- **Path**: `/api/contracts/{contract_id}/clauses`
- **Path Parameters**:
  - `contract_id`: (String, UUID) Unique identifier of the contract.

#### Example Curl Command
```bash
curl -X GET "http://localhost:8000/api/contracts/7f805a41-3b7c-4734-91c6-cf881335cb8a/clauses"
```

#### Response Example (`200 OK`)
```json
[
  {
    "id": "a82df6a0-53cf-42bb-ae4b-70337c76251b",
    "contract_id": "7f805a41-3b7c-4734-91c6-cf881335cb8a",
    "clause_type": "indemnity",
    "section_number": "Sec 8.1",
    "title": "Indemnification",
    "original_text": "Supplier shall defend, indemnify and hold harmless...",
    "plain_english_summary": "Supplier protects customer from third party legal claims.",
    "risk_score": 25.0,
    "risk_level": "low",
    "risk_category": "legal",
    "market_deviation": "standard",
    "deviation_explanation": "Matches standard commercial vendor protections."
  }
]
```

---

### 4. Get Executive Summary
Retrieves the AI-generated high-level executive summary of a contract once analysis is complete.

- **HTTP Method**: `GET`
- **Path**: `/api/contracts/{contract_id}/summary`
- **Path Parameters**:
  - `contract_id`: (String, UUID) Unique identifier.

#### Response Schema Structure
- `summary`: Serialized JSON block containing `"formatted"` markdown and `"structured"` context items.
- `key_terms`: Core key value properties.
- `risk_carrier`: Entity carrying major risks.
- `top_issues`: Bulleted list of negotiation opportunities.

#### Example Curl Command
```bash
curl -X GET "http://localhost:8000/api/contracts/7f805a41-3b7c-4734-91c6-cf881335cb8a/summary"
```

#### Response Example (`200 OK`)
```json
{
  "summary": "{\"formatted\": \"### Executive Summary\\nThis services contract assigns legal liabilities...\"}",
  "key_terms": [],
  "risk_carrier": "",
  "top_issues": []
}
```

---

### 5. Get Risk Breakdown Scores
Get risk metrics of a contract divided by legal, financial, operational, and reputational risk categories.

- **HTTP Method**: `GET`
- **Path**: `/api/contracts/{contract_id}/risks`
- **Path Parameters**:
  - `contract_id`: (String, UUID) Unique identifier.

#### Example Curl Command
```bash
curl -X GET "http://localhost:8000/api/contracts/7f805a41-3b7c-4734-91c6-cf881335cb8a/risks"
```

#### Response Example (`200 OK`)
```json
{
  "financial": 20.0,
  "operational": 45.0,
  "legal": 50.0,
  "reputational": 10.0
}
```

---

## ⚖️ Comparative & Chat Interactive Endpoints

### 1. Cross-Contract Clause Comparison
Compare identical clause types (e.g. `liability` or `indemnity`) across multiple contracts side-by-side using Gemini AI to flag deviations and pick the safest version.

- **HTTP Method**: `POST`
- **Path**: `/api/compare`
- **Body JSON Format**:
```json
{
  "contract_ids": ["7f805a41-3b7c-4734-91c6-cf881335cb8a", "de9e144a-a48e-4a6c-9418-4f27df7a3641"],
  "clause_type": "liability"
}
```

#### Example Curl Command
```bash
curl -X POST "http://localhost:8000/api/compare" \
     -H "Content-Type: application/json" \
     -d '{"contract_ids": ["7f805a41-3b7c-4734-91c6-cf881335cb8a", "de9e144a-a48e-4a6c-9418-4f27df7a3641"], "clause_type": "liability"}'
```

#### Response Example (`200 OK`)
```json
{
  "clause_type": "liability",
  "comparisons": [
    {
      "contract_id": "7f805a41-3b7c-4734-91c6-cf881335cb8a",
      "contract_name": "SaaS_Contract_A.pdf",
      "clause_text": "Supplier's aggregate liability under this agreement is capped at...",
      "risk_score": 60.0,
      "deviation": "unfavourable"
    },
    {
      "contract_id": "de9e144a-a48e-4a6c-9418-4f27df7a3641",
      "contract_name": "Vendor_Agreement_B.pdf",
      "clause_text": "Either party's liability under this agreement is limited to 12 months fee paid...",
      "risk_score": 30.0,
      "deviation": "standard"
    }
  ],
  "ai_summary": "Agreement B offers mutual protections, whereas Agreement A caps only Supplier liability.",
  "key_differences": [
    {
      "attribute": "Mutuality",
      "difference": "Agreement A is unilateral, Agreement B is bilateral."
    }
  ],
  "most_favorable": "Vendor_Agreement_B.pdf",
  "most_risky": "SaaS_Contract_A.pdf",
  "recommendation": "Use the liability cap framework in Vendor Agreement B as a template."
}
```

---

### 2. Chat with Contract (Conversational Q&A)
Ask questions regarding a specific contract and receive answers grounded inside the raw text with source citations of clauses.

- **HTTP Method**: `POST`
- **Path**: `/api/contracts/{contract_id}/chat`
- **Path Parameters**:
  - `contract_id`: (String, UUID) Unique identifier of the contract.
- **Body JSON Format**:
```json
{
  "message": "What is the notice period required to terminate this agreement?"
}
```

#### Example Curl Command
```bash
curl -X POST "http://localhost:8000/api/contracts/7f805a41-3b7c-4734-91c6-cf881335cb8a/chat" \
     -H "Content-Type: application/json" \
     -d '{"message": "What is the notice period required to terminate this agreement?"}'
```

#### Response Example (`200 OK`)
```json
{
  "response": "The contract requires a written notice of at least thirty (30) days prior to the expiration of the current term to prevent auto-renewal.",
  "citations": [
    {
      "clause_id": "b182bf90-34fa-48b2-b13c-823cd7f6259c",
      "section_number": "Sec 12.2",
      "snippet": "Either party may terminate this Agreement by providing at least thirty (30) days written notice..."
    }
  ]
}
```

---

## 📈 Baselines Configuration Endpoints

### 1. Reseed Baseline Clauses
Reseeds standard contract templates and clauses into the PostgreSQL database. This is safe to run multiple times (idempotent) and updates the rows in-place.

- **HTTP Method**: `POST`
- **Path**: `/api/baselines/seed`

#### Example Curl Command
```bash
curl -X POST "http://localhost:8000/api/baselines/seed"
```

#### Response Example (`200 OK`)
```json
{
  "status": "ok",
  "inserted": 4,
  "updated": 10
}
```

---

### 2. List All Grouped Baselines
Fetch all standard baselines stored in the database, grouped by contract type (e.g., `nda`, `saas`, `employment`, `services`).

- **HTTP Method**: `GET`
- **Path**: `/api/baselines`

#### Example Curl Command
```bash
curl -X GET "http://localhost:8000/api/baselines"
```

#### Response Example (`200 OK`)
```json
{
  "baselines": {
    "nda": [
      {
        "id": "e89df9a0-f3cb-456b-ae4b-55447c76251b",
        "clause_type": "confidentiality",
        "standard_text": "Each party agrees to maintain strict confidentiality of all disclosed proprietary information...",
        "description": "Standard mutual confidentiality commitment for non-disclosure agreements.",
        "acceptable_variations": ["reasonable representative disclosure", "written exclusions"]
      }
    ]
  }
}
```

---

### 3. List Baselines for Specific Contract Type
List standard baseline clauses available for a specific category of contract.

- **HTTP Method**: `GET`
- **Path**: `/api/baselines/{contract_type}`
- **Path Parameters**:
  - `contract_type`: String (one of `nda`, `saas`, `employment`, `services`).

#### Example Curl Command
```bash
curl -X GET "http://localhost:8000/api/baselines/nda"
```

#### Response Example (`200 OK`)
```json
{
  "contract_type": "nda",
  "baselines": [
    {
      "id": "e89df9a0-f3cb-456b-ae4b-55447c76251b",
      "clause_type": "confidentiality",
      "standard_text": "Each party agrees to maintain strict confidentiality of all disclosed proprietary information...",
      "description": "Standard mutual confidentiality commitment for non-disclosure agreements.",
      "acceptable_variations": ["reasonable representative disclosure", "written exclusions"]
    }
  ]
}
```

---

## 🚫 Standard Error Codes Reference

When an API call returns an HTTP error, the response body contains a detail message formatted as follows:

```json
{
  "detail": "Detailed explanation of what went wrong."
}
```

| HTTP Code | Error Classification | Typical Cause(s) |
| :---: | :--- | :--- |
| `400` | **Bad Request** | Uploaded file size exceeds 10MB; uploaded file is not a `.pdf` or `.docx`; trying to compare less than 2 contracts; chat requested on a contract whose analysis is not complete. |
| `404` | **Not Found** | Specified contract ID or baseline contract type does not exist in the database; summary or risk results not ready. |
| `409` | **Conflict** | Triggering analysis on a contract that is already currently in `parsing` or `analyzing` states. |
| `500` | **Internal Server Error** | Database connection outage; unexpected exception during Docling local file parsing; Gemini API rate limits or timeout failures. |

---

## 📮 Postman Collection Support

To make testing local and remote deployments as frictionless as possible, we have packaged a pre-built Postman collection for you.

### Quick Import Steps:
1. Locate the packaged JSON file: [docs/lai.postman_collection.json](file:///C:/workspace/rishi-harti768/Lai/docs/lai.postman_collection.json).
2. Open Postman, click **Import** in the upper left corner.
3. Drag and drop the `lai.postman_collection.json` file into the upload zone.
4. Set the Postman Environment variable `base_url` to `http://localhost:8000` to quickly reference all endpoints.
