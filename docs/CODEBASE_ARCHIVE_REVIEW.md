# 📚 TradeOS Complete Codebase Review

This document provides a comprehensive, file-by-file audit of the TradeOS platform, detailing exactly what dependencies each file uses and what business logic it is responsible for.

---

## 1. 🧠 Backend Agents (`backend/app/agents/`)
This folder contains the LangChain and Python logic for the autonomous AI swarm.

### 📄 `orchestrator_agent.py`
- **Uses**: `langchain_google_genai`, `dotenv`, `app.core.trade_state`.
- **Responsibility**: The master "Brain" of the platform. It initializes the Google Gemini 2.5 LLM, wraps all other agents as `@tool`s using `bind_tools`, and parses raw natural language from the frontend chat. It routes intents to deterministic Python execution paths and manages the `TradeState` memory.

### 📄 `risk_agent.py`
- **Uses**: `requests`, `random`, `os`, `app.agents.market_agent`.
- **Responsibility**: The ultimate gatekeeper for trade safety. It aggregates 5 risk pillars (Supplier, Financial, Logistics, Compliance, Market). It explicitly polls external APIs: **OpenWeather** (for port storms), **AISStream** (for port congestion), **FastForex / Frankfurter** (for FX volatility), and **OpenRouteService** (for logistics duration).

### 📄 `escrow_agent.py` & `escrow_oracle.py`
- **Uses**: `web3`, `json`.
- **Responsibility**: Manages blockchain-based Trust logic. `escrow_agent.py` connects to the Ethereum/Polygon smart contract (`TradeEscrow.sol`) to lock and release funds automatically when logistics endpoints are met. `escrow_oracle.py` feeds real-world Web2 data (like shipment tracking) into the Web3 contract.

### 📄 `buyer_matchmaker_agent.py` & `catalog_agent.py`
- **Uses**: `motor.motor_asyncio`, TF-IDF libraries (`sklearn`).
- **Responsibility**: `catalog_agent.py` autonomously parses unstructured inventory uploads and structures them into the database. `buyer_matchmaker_agent.py` runs the deterministic recommendation engine to pair buyers with the exact structured supplier profiles.

### 📄 `document_agent.py` & `qc_agent.py`
- **Uses**: `pytesseract`, `opencv-python`, `PyMuPDF`.
- **Responsibility**: Handles digital paperwork. `document_agent.py` parses uploaded Invoices and Bills of Lading via OCR to extract data automatically. `qc_agent.py` runs Quality Control checks against compliance parameters.

### 📄 `market_agent.py` & `product_intelligence_agent.py`
- **Uses**: `yfinance`, HTTP requests.
- **Responsibility**: Feeds live market intelligence to the Orchestrator. `market_agent.py` retrieves real-time commodity pricing (e.g., Coffee, Turmeric) from external data oracles, ensuring trades aren't executed at heavily skewed prices.

---

## 2. 🔌 Backend APIs (`backend/app/api/` & `backend/app/routes/`)
These files map the core logic to the frontend via FastAPI REST endpoints.

### 📄 `app/main.py`
- **Uses**: `fastapi`, `CORSMiddleware`, `uvicorn`.
- **Responsibility**: The primary entry point. Configures the FastAPI server, mounts all the sub-routers (from `app.api` and `app.routes`), establishes the `motor` async MongoDB connection on startup, and configures CORS for the React frontend.

### 📄 `app/api/orchestrator_api.py`
- **Uses**: `fastapi.APIRouter`, `app.agents.orchestrator_agent`.
- **Responsibility**: Exposes the `/api/orchestrator/chat` endpoint. This is the single endpoint the frontend Chat UI hits, streaming the LLM response back to the user.

### 📄 `app/api/risk_api.py` & `app/api/escrow_api.py`
- **Uses**: FastAPI routers, Pydantic models.
- **Responsibility**: Exposes endpoints for the dashboard UI to manually trigger risk assessments or view the locked state of the Web3 escrow contract without talking to the chatbot.

### 📄 `app/routes/auth.py` & `app/routes/users.py`
- **Uses**: `passlib`, `python-jose` (JWT), `bcrypt`.
- **Responsibility**: Handles secure user authentication. Hashes passwords, verifies login credentials, and mints JWT access tokens for secure session management on the frontend.

---

## 3. 💾 Backend Core & Models (`backend/app/core/` & `backend/app/models/`)
Database schemas and system-wide configurations.

### 📄 `app/database.py`
- **Uses**: `motor.motor_asyncio`, `os.getenv`.
- **Responsibility**: Establishes the singleton connection to MongoDB. Uses AsyncIO to ensure that database queries never block the FastAPI event loop, achieving massive concurrency.

### 📄 `app/models/supplier.py` & `app/models/matchmaker.py`
- **Uses**: `pydantic.BaseModel`.
- **Responsibility**: Defines the strict data structures (schemas) for Suppliers and Trades. This ensures data validated before it is inserted into MongoDB, preventing NoSQL injection and malformed entries.

---

## 4. 🎨 Frontend React App (`frontend/src/`)
The UI layer consumed by the end-user.

### 📄 `App.js` & `index.js`
- **Uses**: `react-router-dom`, `react`.
- **Responsibility**: The root of the React application. Configures all the frontend routes (e.g., `/dashboard`, `/kyc-upload`, `/matchmaker`).

### 📄 `pages/EnhancedTradePlatform.js` & `pages/TradePlatform.js`
- **Uses**: `lucide-react`, `recharts`, `framer-motion`, `axios`.
- **Responsibility**: The massive, dynamic main dashboard you see when you log in. It fetches live data from the backend APIs, renders the beautiful charts using Recharts, handles animations with Framer Motion, and houses the Chatbot UI component on the side.

### 📄 `pages/KYCUpload.js` & `pages/OnboardingWizard.js`
- **Uses**: React state, FormData, `axios`.
- **Responsibility**: The user onboarding flow. Allows suppliers/buyers to upload their business licenses, Aadhaar cards, and tax documents. It sends these as `multipart/form-data` to the backend OCR APIs for autonomous validation.

### 📄 `pages/Login.js` & `pages/Register.js`
- **Uses**: `react-bootstrap`, `AuthContext`.
- **Responsibility**: User entry points. Captures credentials, hits the `/api/auth` backend endpoints, stores the returned JWT token in `localStorage`, and updates the global AuthContext so the rest of the app knows the user is logged in.

### 📄 `services/api.js`
- **Uses**: `axios`.
- **Responsibility**: A centralized Axios instance that automatically attaches the JWT bearer token to the headers of every outgoing request, ensuring secure communication with the FastAPI backend.

---

## 5. 🛠️ Configuration & Environment Files

### 📄 `backend/requirements.txt`
- **Responsibility**: Locks in the exact Python dependencies required for the backend (FastAPI, Motor, LangChain, Web3, PyTesseract, OpenCV).

### 📄 `backend/.env`
- **Responsibility**: Stores highly sensitive API keys (Google Gemini, Web3 Private Keys, OpenWeather, Frankfurter) and database connection strings.

### 📄 `frontend/package.json`
- **Responsibility**: Locks in the exact Node.js dependencies for the React app (Tailwind, Recharts, Framer Motion, Bootstrap).

### 📄 `frontend/tailwind.config.js`
- **Responsibility**: Configures the Tailwind CSS design system, defining the custom color palettes (vibrant blues, dark modes) and spacing utilities used across all React components.
