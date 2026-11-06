# 🌐 TradeOS Platform

> A full-stack, AI-powered matchmaking and risk assessment platform for global trade and supply chains. Built with **FastAPI**, **React**, **MongoDB**, and **Google Gemini AI**.
> 
> 🤖 **Chatbot Integration**: *"Hi! I am the TradeOS LangChain Orchestrator. Tell me what you want to export or ask me for live commodity prices in plain text."*

![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)
![Python](https://img.shields.io/badge/Python-3.12-blue?style=flat-square)
![React](https://img.shields.io/badge/React-18-61DAFB?style=flat-square)
![FastAPI](https://img.shields.io/badge/FastAPI-0.104+-009688?style=flat-square)

---

## 📸 Screenshots

| Dashboard | Matchmaker Agent |
|:---:|:---:|
| ![Dashboard Overview](images/dashboard.png) | ![Matchmaker AI](images/matchmaker.png) |

> *Place your screenshots inside the `images/` folder and name them `dashboard.png` and `matchmaker.png` to have them appear here.*

---

## 📑 Table of Contents

- [🚀 Features](#-features)
- [🛠️ Tech Stack](#️-tech-stack)
- [🏗️ Architecture & Operation Flow](#️-architecture--operation-flow)
- [🤖 The AI Agent Swarm](#-the-ai-agent-swarm)
- [📁 Project Structure](#-project-structure)
- [🔑 Environment Variables & API Keys](#-environment-variables--api-keys)
- [⚙️ Setup & Installation](#️-setup--installation)
- [▶️ Usage](#️-usage)

---

## 🚀 Features

- **AI Matchmaking Agent** — Uses Google Gemini AI and TF-IDF to analyze and match suppliers based on multi-dimensional trade data.
- **Risk Assessment Agent** — Evaluates geopolitical, financial, and operational risks in real-time.
- **Smart Escrow** — Blockchain-ready escrow agent architecture using Web3.py.
- **Document OCR & Processing** — Automated document parsing using PyTesseract, OpenCV, and PyMuPDF.
- **Modern Web Dashboard** — A dynamic, responsive React frontend integrated with Tailwind CSS, Recharts, and Framer Motion.

---

## 🛠️ Tech Stack

### Backend
| Component | Technology |
|-----------|-----------|
| **Framework** | Python 3.12, FastAPI, Uvicorn |
| **AI Orchestration** | LangChain, Google Gemini (`gemini-2.5-flash`) |
| **Database** | MongoDB (Motor AsyncIO) |
| **Document/OCR** | OpenCV, PyTesseract, PyMuPDF, Pillow |
| **Blockchain** | Web3.py |
| **Security/Auth** | PyJWT, Passlib, Bcrypt |
| **Data/Market** | Scikit-learn, Numpy, YFinance, Frankfurter API |

### Frontend
| Component | Technology |
|-----------|-----------|
| **Framework** | React 18 |
| **Styling** | Tailwind CSS, Bootstrap, React Bootstrap |
| **Animations** | Framer Motion |
| **Data Visualization**| Recharts |
| **Routing** | React Router v6 |
| **Package Manager** | `npm` |

---

## 🏗️ Architecture & Operation Flow

TradeOS operates as a highly concurrent REST API utilizing FastAPI and LangChain for internal agent orchestration.

- **FastAPI REST Endpoints**: The platform is built purely on REST architecture. The frontend React application interacts with the backend via standard HTTP requests to isolated endpoints (e.g., `/api/orchestrator`, `/api/matchmaker`, `/api/escrow`).
- **LangChain AI Orchestrator**: The core controller. When a user interacts with the chatbot, the natural language query is sent to the `/api/orchestrator` endpoint. 
- **Tool Binding (`bind_tools`)**: The Orchestrator uses Gemini 2.5 and LangChain's `bind_tools` to dynamically parse the intent and route the request to deterministic Python functions (e.g., `trigger_matchmaker_search`, `assess_supplier_risk`).
- **Internal State Management (`TradeState`)**: TradeOS manages state directly in-memory (and ultimately backed by MongoDB). Complex trades progress through sequential states (like `BUYERS_MATCHED` to `NEGOTIATION_COMPLETE`) via a state object managed by the Orchestrator.
- **Asynchronous I/O**: High-latency tasks like database interactions (MongoDB via `motor`) and external API calls (e.g., OpenWeather, Frankfurter) are executed non-blockingly using Python's `asyncio` within the FastAPI event loop, ensuring high concurrency without the architectural overhead of external task queues (like Celery or RabbitMQ).

---

## 🤖 The AI Agent Swarm (Micro-Agents)

The TradeOS "Swarm" consists of distinct Python modules located in `app/agents/`. They communicate programmatically via the Orchestrator logic.

| Agent | Core Function |
|-------|---------------|
| **Orchestrator Agent** | The LangChain "Brain". Receives natural language from the frontend, maps it to internal Python tools, and returns structured execution plans or chat responses. |
| **Matchmaker Agent** | A deterministic recommendation engine using TF-IDF vectorization and geospatial filtering on MongoDB to connect buyers and sellers globally. |
| **Risk Agent** | The gatekeeper. Calculates real-time composite risk scores (Financial, Logistics, Compliance) by directly polling APIs like OpenWeather, AISStream, OpenRouteService, and Frankfurter. |
| **Escrow Agent** | Interacts with Ethereum-compatible smart contracts using `web3.py` to securely lock and release funds automatically based on trade state. |
| **Market Agent** | Uses libraries like `yfinance` to fetch live, real-time commodity data back to the Orchestrator. |
| **Document Agent** | Leverages OCR (`pytesseract`, `opencv`) and PDF processors to autonomously parse invoices and Bills of Lading. |
| **Logistics Agent** | Handles multi-modal routing and calculates optimal delivery paths directly through REST response cycles. |

---

## 📁 Project Structure

```text
TradeOS/
│
├── 📂 backend/                  # FastAPI Backend Server
│   ├── 📂 app/                  # Main application code
│   │   ├── 📂 agents/           # AI Agents (Matchmaker, Risk, Escrow, etc.)
│   │   ├── 📂 core/             # Configuration & Subscribers
│   │   └── main.py              # Application Entry Point
│   ├── .env                     # Environment variables (Create this!)
│   ├── requirements.txt         # Python dependencies
│   └── verify_matchmaker_csv_integration.py # DB verification script
│
├── 📂 frontend/                 # React Web Application
│   ├── 📂 public/               # Static assets
│   ├── 📂 src/                  # React components & UI logic
│   └── package.json             # NPM dependencies
│
├── 📂 docs/                     # Detailed architectural documentation
│   ├── MATCHMAKER_FINAL_STATUS.md
│   └── tradeos_analysis.md
│
├── 📂 images/                   # Screenshots for documentation
│   ├── dashboard.png
│   └── matchmaker.png
│
├── push_to_mongodb.py           # Script to populate MongoDB with initial data
└── README.md                    # This documentation file
```

---

## 🔑 Environment Variables & API Keys

Before running the application, you must configure the environment variables for the backend. 

Create a new file named `.env` exactly at this location: `backend/.env`

Paste the following configuration into your `.env` file and replace the placeholder values with your actual API keys:

```env
# ==========================================
# TradeOS API Keys & Environment Variables
# ==========================================

# 1. Google Gemini API Key (CRUCIAL for Matchmaker & AI Agents)
# Get it from: https://aistudio.google.com/
GEMINI_API_KEY=your_gemini_api_key_here

# 2. MongoDB Database Connection (CRUCIAL)
MONGODB_URL=mongodb://localhost:27017
DATABASE_NAME=tradeos_platform

# 3. Security/Auth Keys
SECRET_KEY=your_super_secret_key_change_in_production
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# ==========================================
# Optional Specialized Agent APIs
# ==========================================

# 4. OpenWeather API (For Risk Agent weather tracking)
# Get it from: https://openweathermap.org/api
OPENWEATHER_API_KEY=your_openweather_api_key_here

# 5. AISStream API (For Risk Agent real-time vessel tracking)
# Get it from: https://aisstream.io/
AISSTREAM_API_KEY=your_aisstream_api_key_here

# 6. FastForex API (For currency exchange rate data)
# Get it from: https://www.fastforex.io/
# NOTE: The Risk Agent will try FastForex first. If it fails or is empty, it automatically falls back to the free Frankfurter API.
FASTFOREX_API_KEY=your_fastforex_api_key_here

# 7. OpenRouteService API (For routing and logistics distances)
# Get it from: https://openrouteservice.org/
OPENROUTESERVICE_API_KEY=your_ors_api_key_here

# 8. Web3 Private Key (For Escrow Agent smart contracts)
WEB3_PRIVATE_KEY=your_ethereum_private_key_here
```

### How to get the API Keys:
- **Google Gemini**: Go to [Google AI Studio](https://aistudio.google.com/) and generate a free key.
- **OpenWeather**: Sign up at [OpenWeatherMap](https://openweathermap.org/api) for a free tier key.
- **AISStream**: Register at [Aisstream.io](https://aisstream.io/) for free real-time maritime tracking.
- **FastForex**: Register at [FastForex](https://www.fastforex.io/). *(Optional: App defaults to free Frankfurter API if omitted).*
- **OpenRouteService**: Generate a Standard token in the [ORS Dashboard](https://openrouteservice.org/dev/#/home).

---

## ⚙️ Setup & Installation

### 1. Backend Setup

Open a terminal and navigate to the backend directory:

```bash
cd e:\TradeOS\Trade-OS\backend

# Create and activate a Python virtual environment
python -m venv venv
.\venv\Scripts\activate

# Install all backend dependencies
pip install -r requirements.txt

# (Optional) Verify your database integration
python verify_matchmaker_csv_integration.py
```

### 2. Frontend Setup

Open a **separate** terminal and navigate to the frontend directory:

```bash
cd e:\TradeOS\Trade-OS\frontend

# Clean any existing corrupted modules
Remove-Item -Recurse -Force node_modules -ErrorAction SilentlyContinue
Remove-Item -Force package-lock.json -ErrorAction SilentlyContinue
npm cache clean --force

# Install frontend dependencies
npm install
```

---

## ▶️ Usage

To run the full platform, you need to run both the backend and frontend servers simultaneously in separate terminal windows.

**Terminal 1 — Backend API**
```bash
cd e:\TradeOS\Trade-OS\backend
.\venv\Scripts\activate
python -m uvicorn app.main:app --reload
```
*The backend API will be available at [http://localhost:8000](http://localhost:8000).*

**Terminal 2 — Frontend UI**
```bash
cd e:\TradeOS\Trade-OS\frontend
npm start
```
*The frontend dashboard will automatically open at [http://localhost:3000](http://localhost:3000).*

---

<div align="center">
  <sub>Built with ❤️ for Global Trade Automation.</sub>
</div>
