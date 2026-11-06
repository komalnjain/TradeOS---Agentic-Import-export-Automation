# TradeOS Complete Analysis

## 1. What is TradeOS?
TradeOS is an intelligent B2B recommendation engine and platform designed to connect buyers, suppliers, and logistics partners using AI-driven matching techniques. 

Its core intelligence is driven by the **Matchmaker Agent**, which performs a multi-phase matching pipeline:
1. **Rule-Based Filtering:** Basic eligibility screening (e.g., verifying capacity, HS codes).
2. **Trust Score Calculation:** Analyzes historical performance (success rate, on-time delivery, ratings).
3. **Similarity Matching:** Uses AI to find content similarity between trade requirements and supplier profiles.
4. **Risk Assessment:** Evaluates trade volume, rating, geographic, and response time risks.
5. **Compliance Checking:** Checks verification status and sanctions.
6. **Recommendation:** Provides final ranked recommendations.

The platform currently operates on a dataset of around 1,000 verified suppliers (loaded via CSV into a MongoDB database) across various categories like Agriculture, Spices, Textiles, and Electronics.

## 2. Project Structure
The project is divided into two main components:
- **`backend/`**: A Python FastAPI backend that houses the core agents (Matchmaker, Risk, Escrow, Catalog, etc.), API routes, and database integration. It heavily relies on AI models (like Google Gemini) to parse natural language trade requests and match them with suppliers.
- **`frontend/`**: A React application bootstrapped with `react-scripts`, styled using TailwindCSS and Framer Motion for animations. It serves as the user interface for the platform.

## 3. Required APIs & Environment Variables
To fully utilize the platform's agentic capabilities, you will need to set up several API keys. These should be placed in a `.env` file inside the `backend/` directory.

### Core AI & Database APIs:
- **`GEMINI_API_KEY`**: (Crucial) Google Gemini API key used by various agents (Matchmaker, Catalog, Product Intelligence, Quality Control, Logistics) to parse requests and process natural language. You can get this from [Google AI Studio](https://aistudio.google.com/).
- **`MONGODB_URL`** (or `MONGO_URI`): Connection string for MongoDB. Defaults to `mongodb://localhost:27017` if running locally, but can be a MongoDB Atlas URI.
- **`DATABASE_NAME`**: The name of the database (e.g., `tradeos_platform` or `tradeos_db`).

### Specialized Agent APIs:
- **`OPENWEATHER_API_KEY`**: Used by the Risk Agent for weather-related risk assessment. Get from [OpenWeatherMap](https://openweathermap.org/api).
- **`AISSTREAM_API_KEY`**: Used by the Risk Agent for real-time vessel tracking. Get from [AISStream.io](https://aisstream.io/).
- **`FASTFOREX_API_KEY`**: Used by the Risk Agent for currency exchange rate data. Get from [FastForex](https://www.fastforex.io/).
- **`OPENROUTESERVICE_API_KEY`**: Used by the Risk Agent to calculate routing and distances. Get from [OpenRouteService](https://openrouteservice.org/).
- **`WEB3_PRIVATE_KEY`**: Used by the Escrow Agent for blockchain/smart contract interactions. 

### Security/Authentication (Optional for local dev but required for production):
- **`SECRET_KEY`**: For JWT token generation.
- **`ALGORITHM`**: Typically `HS256`.
- **`ACCESS_TOKEN_EXPIRE_MINUTES`**: Token expiry time (e.g., 30).

## 4. How to Run the Project

### Step 1: Database Setup
Ensure you have MongoDB running locally on port 27017, or set up a MongoDB Atlas cluster and get your connection string.

### Step 2: Backend Setup
1. Open a terminal and navigate to the backend directory:
   ```bash
   cd backend
   ```
2. Create and activate a Python virtual environment:
   ```bash
   python -m venv venv
   # On Windows:
   .\venv\Scripts\activate
   # On macOS/Linux:
   source venv/bin/activate
   ```
3. Install the dependencies:
   ```bash
   pip install -r requirements.txt
   ```
4. Create a `.env` file in the `backend` directory and add your API keys (especially `GEMINI_API_KEY` and `MONGODB_URL`).
5. (Optional but recommended) Run the verification script to ensure your database and CSV data are properly loaded:
   ```bash
   python verify_matchmaker_csv_integration.py
   ```
6. Start the FastAPI server:
   ```bash
   python -m uvicorn app.main:app --reload
   ```
   The backend will be available at `http://localhost:8000`.

### Step 3: Frontend Setup
1. Open a new terminal window and navigate to the frontend directory:
   ```bash
   cd frontend
   ```
2. Install the Node.js dependencies:
   ```bash
   npm install
   ```
3. Start the React development server:
   ```bash
   npm start
   ```
   The frontend will be available at `http://localhost:3000`.

## 5. Testing the Matchmaker
Once the backend is running, you can test the Matchmaker API directly via a `cURL` request or API testing tool (like Postman).

Example cURL request:
```bash
curl -X POST "http://localhost:8000/api/matchmaker/trade-request" \
  -H "Content-Type: application/json" \
  -d '{
    "product_name": "Organic Coffee Beans",
    "hs_code": "090111",
    "quantity": 1000,
    "unit": "kg",
    "destination_country": "US",
    "delivery_deadline": "2026-04-06",
    "budget_max": 35000,
    "certifications_required": ["USDA Organic"]
  }'
```
This will parse the trade request and return a list of top suppliers with their calculated trust scores and matching reasons.
