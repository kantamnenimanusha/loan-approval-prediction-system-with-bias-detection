# 🏦 Loan Approval Prediction — Java Full Stack Project

Converted from the original Python/Jupyter notebook (`Loan_Data__Project_.ipynb`).

---

## Project Structure

```
loan-app/
├── backend/          Spring Boot REST API (Java 17)
│   ├── pom.xml
│   └── src/main/java/com/loanapp/
│       ├── LoanApprovalApplication.java   Entry point
│       ├── controller/LoanController.java REST endpoints
│       ├── model/
│       │   ├── LoanRequest.java           Input DTO
│       │   ├── LoanResponse.java          Output DTO
│       │   ├── LoanApplication.java       JPA Entity (H2)
│       │   └── LoanApplicationRepository.java
│       └── service/LoanPredictionService.java  Prediction logic
└── frontend/
    └── index.html    Standalone HTML/CSS/JS UI
```

---

## Tech Stack

| Layer    | Technology                                |
|----------|-------------------------------------------|
| Backend  | Java 17, Spring Boot 3.2, Spring Data JPA |
| Database | H2 (in-memory, no setup required)         |
| Frontend | Plain HTML5 + CSS3 + Vanilla JS           |
| Build    | Maven                                     |

---

## Prerequisites

- Java 17+ (`java -version`)
- Maven 3.8+ (`mvn -version`)
- A modern browser

---

## Running the Project

### 1. Start the Backend

```bash
cd backend
mvn spring-boot:run
```

The API will start on **http://localhost:8080**

You can verify it's running:
```
GET http://localhost:8080/api/loan/health
```

### 2. Open the Frontend

Simply open `frontend/index.html` in your browser (double-click the file, or use a local server):

```bash
# Option A: Just open the file
open frontend/index.html

# Option B: Use Python's HTTP server (optional)
cd frontend && python3 -m http.server 3000
# Then visit http://localhost:3000
```

---

## REST API Reference

| Method | Endpoint              | Description                          |
|--------|-----------------------|--------------------------------------|
| POST   | `/api/loan/predict`   | Predict loan approval                |
| GET    | `/api/loan/history`   | Last 10 predictions                  |
| GET    | `/api/loan/stats`     | Approval statistics for this session |
| GET    | `/api/loan/health`    | Health check                         |

### POST `/api/loan/predict` — Request Body

```json
{
  "gender": 1,
  "dependents": 0,
  "applicantIncome": 5000,
  "coapplicantIncome": 2000,
  "loanAmount": 150,
  "loanAmountTerm": 360,
  "creditHistory": 1.0,
  "propertyArea": 2
}
```

**Field Encodings** (same as notebook):

| Field           | Values                              |
|-----------------|-------------------------------------|
| gender          | 1 = Male, 0 = Female                |
| dependents      | 0, 1, 2, 3 (3 means "3+")          |
| applicantIncome | Monthly income in ₹                 |
| coapplicantIncome| Monthly income of co-applicant     |
| loanAmount      | In ₹ thousands (e.g. 150 = ₹1.5L)  |
| loanAmountTerm  | In months (360 = 30 years)          |
| creditHistory   | 1.0 = good, 0.0 = bad               |
| propertyArea    | 0 = Rural, 1 = Urban, 2 = Semiurban |

### Response

```json
{
  "approved": true,
  "status": "Approved",
  "confidence": 78.5,
  "riskLevel": "LOW",
  "message": "Congratulations! Your loan application looks strong..."
}
```

---

## Prediction Model

The `LoanPredictionService` implements a weighted scoring model derived from the
Random Forest Classifier in the original notebook. Features and weights reflect:

- **Credit History** — highest importance (matches notebook's RF analysis)
- **EMI-to-Income Ratio** — calculated from LoanAmount, Term, and combined income
- **Total Income** — absolute income thresholds
- **Property Area** — Semiurban had highest approval rate in dataset
- **Dependents**, **Loan Term**, **Gender** — minor supporting signals

To integrate an actual trained model (e.g., via ONNX or PMML), replace the
scoring logic inside `LoanPredictionService.computeScore()`.

---

## H2 Console (Development)

While the backend is running, visit:
```
http://localhost:8080/h2-console
```
- JDBC URL: `jdbc:h2:mem:loandb`
- Username: `sa`
- Password: *(leave blank)*

---

## Notebook → Java Mapping

| Notebook Step        | Java Component                         |
|----------------------|----------------------------------------|
| Data Preprocessing   | Handled on frontend (radio buttons)    |
| Feature Selection    | `LoanRequest` DTO fields               |
| Model Training (RF)  | `LoanPredictionService` scoring model  |
| Prediction           | `POST /api/loan/predict`               |
| Gradio UI            | `frontend/index.html`                  |
| Model evaluation %   | `GET /api/loan/stats`                  |
