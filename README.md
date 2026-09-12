# Farmer Decision & Value-Sharing Platform — Backend

A **Java 17 + Spring Boot 3 + MySQL** REST API backend that helps a farmer decide
**where to sell a crop** (mandi vs company) by calculating the **final farmer
realization** for every option and recommending the best one.

> This is **backend only** — no frontend code is included.

---

## 1. What it does

| Module | What you get |
|---|---|
| Farmer Management | CRUD APIs for farmers |
| Crop Management | CRUD APIs for crops (linked to a farmer) |
| Mandi Data | Mandi prices, search by crop/location, best-price mandi |
| Company Management | CRUD for buyer companies |
| Company Offers | Offers with base price + value-sharing %, realization auto-calculated |
| Transport | Route storage + cost/time calculation used in comparisons |
| Calculation Engine | `farmerRealization = sellingPrice − transportCost − otherCosts + valueShare` |
| Recommendation Engine | Compares mandis + companies, ranks them, explains the winner |
| Trust & Verification | Verification records, history, 0–100 trust score |
| Transactions | Completed sales, auto invoice numbers, payment status |
| AI Decision Engine | `/api/ai/decision` — clean interface, LLM/LangGraph-ready |

---

## 2. Folder structure

```
farmer-platform/
│
├── pom.xml                                  ← Maven build + all dependencies
├── README.md                                ← this file
│
└── src/main/
    ├── java/com/farmerplatform/
    │   ├── FarmerPlatformApplication.java   ← main class (starts the app)
    │   │
    │   ├── controller/                      ← REST endpoints (HTTP layer)
    │   │   ├── FarmerController.java
    │   │   ├── CropController.java
    │   │   ├── MandiController.java
    │   │   ├── CompanyController.java
    │   │   ├── CompanyOfferController.java
    │   │   ├── TransportController.java
    │   │   ├── CalculationController.java
    │   │   ├── RecommendationController.java
    │   │   ├── TrustController.java
    │   │   ├── TransactionController.java
    │   │   └── AiDecisionController.java
    │   │
    │   ├── service/                         ← business logic layer
    │   │   ├── FarmerService.java
    │   │   ├── CropService.java
    │   │   ├── MandiService.java
    │   │   ├── CompanyService.java
    │   │   ├── CompanyOfferService.java
    │   │   ├── TransportService.java
    │   │   ├── CalculationService.java      ← THE realization formula
    │   │   ├── RecommendationService.java   ← decision/ranking engine
    │   │   ├── TrustService.java            ← trust score 0–100
    │   │   └── TransactionService.java
    │   │
    │   ├── repository/                      ← database access (Spring Data JPA)
    │   │   ├── FarmerRepository.java
    │   │   ├── CropRepository.java
    │   │   ├── MandiRepository.java
    │   │   ├── CompanyRepository.java
    │   │   ├── CompanyOfferRepository.java
    │   │   ├── TransportRepository.java
    │   │   ├── TransactionRepository.java
    │   │   ├── VerificationRepository.java
    │   │   └── RecommendationRepository.java
    │   │
    │   ├── model/                           ← JPA entities = MySQL tables
    │   │   ├── Farmer.java
    │   │   ├── Crop.java
    │   │   ├── Mandi.java
    │   │   ├── Company.java
    │   │   ├── CompanyOffer.java
    │   │   ├── Transport.java
    │   │   ├── Transaction.java
    │   │   ├── Verification.java
    │   │   ├── Recommendation.java
    │   │   ├── BuyerType.java
    │   │   ├── PaymentStatus.java
    │   │   ├── VerifiedType.java
    │   │   └── VerificationStatus.java
    │   │
    │   ├── dto/                             ← request/response shapes + validation
    │   ├── ai/                              ← AiDecisionEngine interface + rule-based impl
    │   ├── exception/                       ← @RestControllerAdvice global error handling
    │   ├── config/                          ← CORS, Swagger, DataLoader (sample data)
    │   └── util/                            ← MoneyUtils (safe BigDecimal maths)
    │
    └── resources/
        └── application.properties           ← DB config, port, JPA, Swagger
```

**Layer rule (beginner-friendly):**
`Controller` receives HTTP → `Service` does the thinking → `Repository` talks to MySQL.
Every layer only talks to the one below it.

---

## 3. Requirements

1. **JDK 17+** (works with JDK 17–25)
2. **Maven 3.6+**
3. **MySQL 8**
4. **Postman** (for testing) or just use Swagger in the browser

Check your setup:

```bash
java -version
mvn -v
mysql --version
```

---

## 4. MySQL setup (step by step)

1. Open the **MySQL command line** (or MySQL Workbench):

```bash
mysql -u root -p
```

2. Create the database (table creation is automatic):

```sql
CREATE DATABASE farmer_platform_db;
```

3. Open `src/main/resources/application.properties` and set **your** username/password:

```properties
spring.datasource.username=root
spring.datasource.password=YOUR_PASSWORD_HERE
```

4. That's it. On startup, Hibernate creates all tables automatically
   (`ddl-auto=update`) and the sample data loader fills them
   (`app.data-loader.enabled=true`).

> Tip: the JDBC URL also contains `createDatabaseIfNotExist=true`, so even step 2
> is optional — the app creates the database itself if the user has permission.

---

## 5. How to run

### Option A — in VS Code (recommended for beginners)

1. Install the **Extension Pack for Java** and **Spring Boot Extension Pack**.
2. File → Open Folder → select this project folder.
3. Wait for Java projects to finish loading (bottom-right).
4. Open `FarmerPlatformApplication.java` → click **Run** above `main`.
5. Wait until the console shows:
   `Farmer Platform Backend is running!`

### Option B — terminal

```bash
mvn spring-boot:run
```

### Option C — packaged jar

```bash
mvn package -DskipTests
java -jar target/farmer-platform-0.0.1-SNAPSHOT.jar
```

The backend starts on **http://localhost:8080**.

**Swagger UI:** http://localhost:8080/swagger-ui.html
(OpenAPI JSON: http://localhost:8080/v3/api-docs)

---

## 6. Sample data (loaded automatically)

| Data | Values |
|---|---|
| Farmers | Ramesh Patil (id 1, Nashik), Sunita Devi (id 2, Varanasi) |
| Crops | Tomato 50q, Onion 120q, Wheat 200q (farmer 1), Potato 80q (farmer 2) |
| Mandis | Pimpalgaon, Nashik, Mumbai Vashi, Lasalgaon (Tomato + Onion + Wheat prices) |
| Companies | Fresh Foods Pvt Ltd (verified, pickup), Agro Exports India (verified), GreenGrocer (pending verification) |
| Offers | 3 Tomato offers from the 3 companies |
| Transport routes | Pimpalgaon↔Nashik/Mumbai/Lasalgaon, Baragaon↔Varanasi |
| Verifications | 2 VERIFIED companies, 1 PENDING |

Sample data loads **only if the farmers table is empty**, and only when
`app.data-loader.enabled=true`. Set it to `false` to stop it.

---

## 7. API testing examples (Postman)

Base URL: `http://localhost:8080`
All responses are wrapped like: `{ "success": true, "message": "...", "data": {...} }`

### 7.1 Create a farmer — `POST /api/farmers`

Body (raw → JSON):

```json
{
  "name": "Ramesh Patil",
  "phone": "9876543210",
  "location": "Pimpalgaon",
  "state": "Maharashtra",
  "district": "Nashik"
}
```

Response (201):

```json
{
  "success": true,
  "message": "Farmer created successfully",
  "data": {
    "id": 1,
    "name": "Ramesh Patil",
    "phone": "9876543210",
    "location": "Pimpalgaon",
    "state": "Maharashtra",
    "district": "Nashik"
  }
}
```

Other farmer endpoints:

- `GET /api/farmers/1`
- `GET /api/farmers`
- `GET /api/farmers/state/Maharashtra`
- `PUT /api/farmers/1` (same body as POST)
- `DELETE /api/farmers/1`

### 7.2 Add a crop — `POST /api/crops`

```json
{
  "farmerId": 1,
  "cropName": "Tomato",
  "quantity": 50,
  "quality": "Grade A",
  "harvestDate": "2026-09-09",
  "location": "Pimpalgaon"
}
```

Response (201):

```json
{
  "success": true,
  "message": "Crop added successfully",
  "data": {
    "id": 1,
    "farmerId": 1,
    "cropName": "Tomato",
    "quantity": 50,
    "quality": "Grade A",
    "harvestDate": "2026-09-09",
    "location": "Pimpalgaon"
  }
}
```

Other crop endpoints: `GET /api/crops/1`, `GET /api/crops?farmerId=1`,
`PUT /api/crops/1`, `DELETE /api/crops/1`.

### 7.3 Add mandi data — `POST /api/mandis`

```json
{
  "mandiName": "Nashik Mandi",
  "location": "Nashik",
  "crop": "Tomato",
  "price": 2200,
  "marketTiming": "05:00 AM - 12:00 PM",
  "distance": 40
}
```

Response (201):

```json
{
  "success": true,
  "message": "Mandi data added successfully",
  "data": {
    "id": 2,
    "mandiName": "Nashik Mandi",
    "location": "Nashik",
    "crop": "Tomato",
    "price": 2200,
    "marketTiming": "05:00 AM - 12:00 PM",
    "distance": 40
  }
}
```

Search and best options:

- `GET /api/mandis/search?crop=Tomato&location=Nashik`
- `GET /api/mandis/search?crop=Tomato`
- `GET /api/mandis/best?crop=Tomato` → highest price mandi
- `GET /api/mandis/crop/Tomato` → all mandis for the crop, best price first

### 7.4 Register a company — `POST /api/companies`

```json
{
  "companyName": "Fresh Foods Pvt Ltd",
  "location": "Nashik",
  "requiredCrop": "Tomato",
  "requiredQuantity": 100,
  "qualityRequirements": "Grade A only",
  "pickupAvailable": true,
  "paymentTerms": "Payment within 7 days",
  "verified": true
}
```

Response (201):

```json
{
  "success": true,
  "message": "Company registered successfully",
  "data": {
    "id": 1,
    "companyName": "Fresh Foods Pvt Ltd",
    "location": "Nashik",
    "requiredCrop": "Tomato",
    "requiredQuantity": 100,
    "qualityRequirements": "Grade A only",
    "pickupAvailable": true,
    "paymentTerms": "Payment within 7 days",
    "verified": true
  }
}
```

Other endpoints: `GET /api/companies`, `GET /api/companies?crop=Tomato`,
`GET /api/companies/1`, `PUT /api/companies/1`, `DELETE /api/companies/1`.

### 7.5 Create a company offer — `POST /api/offers`

```json
{
  "companyId": 1,
  "crop": "Tomato",
  "basePrice": 2300,
  "valueSharingPercent": 10,
  "transportCost": 0,
  "otherCosts": 50
}
```

Response (201) — note `finalExpectedFarmerRealization` is calculated by the backend:

```json
{
  "success": true,
  "message": "Offer created successfully",
  "data": {
    "id": 1,
    "companyId": 1,
    "crop": "Tomato",
    "basePrice": 2300,
    "valueSharingPercent": 10,
    "transportCost": 0,
    "otherCosts": 50,
    "finalExpectedFarmerRealization": 2480.00
  }
}
```

(2300 + 10% share (230) − 0 − 50 = 2480 per quintal)

Search offers: `GET /api/offers/search?crop=Tomato`, `GET /api/offers/company/1`.

### 7.6 Calculate transport — `POST /api/transport/calculate`

```json
{
  "source": "Pimpalgaon",
  "destination": "Nashik",
  "estimatedDeliveryTime": ""
}
```

Response (200) — uses the saved route (2500 ₹ for 40 km):

```json
{
  "success": true,
  "message": "Transport calculation done",
  "data": {
    "id": 1,
    "source": "Pimpalgaon",
    "destination": "Nashik",
    "distance": 40.00,
    "transportCost": 2500.00,
    "estimatedDeliveryTime": "2 hours"
  }
}
```

If no route is saved, the backend estimates:
`cost = distance × 40 ₹/km × 1.2` (part load) and `time ≈ 1 h per 40 km`.

### 7.7 Calculate realization — `POST /api/calculation/realization`

**Mode 1 — use a saved option (mandi id 2, 50 quintals):**

```json
{
  "crop": "Tomato",
  "optionType": "MANDI",
  "optionId": 2,
  "quantity": 50
}
```

Response (200):

```json
{
  "success": true,
  "message": "Calculation completed",
  "data": {
    "crop": "Tomato",
    "quantity": 50,
    "sellingPrice": 2200,
    "transportCost": 2500.00,
    "otherCosts": 7500.00,
    "valueShare": 0,
    "farmerRealization": 100000.00,
    "realizationPerQuintal": 2000.00,
    "formula": "farmerRealization = sellingPrice - transportCost - otherCosts + valueShare  [source: Mandi: Nashik Mandi]"
  }
}
```

(`50 × 2200 − 2500 − 7500 = 100,000` → `2000/q`)

**Mode 2 — direct prices (company offer id 1):**

```json
{
  "crop": "Tomato",
  "optionType": "COMPANY",
  "optionId": 1,
  "quantity": 50
}
```

Here valueShare comes from the offer's `valueSharingPercent`:
`50 × 2300 − 0 − 2500 + 50×230 = 124,000` → `2480/q`
(same as the offer's stored `finalExpectedFarmerRealization`).

### 7.8 Get a recommendation — `POST /api/recommendations` ⭐

The main endpoint. Ask: *"I have 50 q of Tomato in Pimpalgaon — where to sell?"*

```json
{
  "farmerId": 1,
  "crop": "Tomato",
  "quantity": 50
}
```

Response (200), shape matches the requirement example:

```json
{
  "success": true,
  "message": "Recommendation generated",
  "data": {
    "recommendedOption": "Agro Exports India",
    "recommendedOptionType": "COMPANY",
    "finalRealization": 138750.00,
    "reasons": [
      "Base price 2500 per quintal",
      "Value sharing 15% of company profit",
      "Verified company",
      "Trust score 50/100",
      "Higher net realization than Fresh Foods Pvt Ltd (+295.00 per quintal)",
      "Better payment terms: 50% advance, rest on delivery"
    ],
    "alternatives": [ ...ranked options from 2nd place... ],
    "rankedOptions": [ ...all options, best first... ],
    "recommendationId": 1
  }
}
```

Each ranked option contains grossPrice, transportCost, otherCosts, valueShare,
finalRealization, realizationPerQuintal, distance, trustScore and reasons.

### 7.9 Trust APIs

- `GET /api/trust/company/1` →

```json
{
  "success": true,
  "message": "Trust report",
  "data": {
    "buyerType": "COMPANY",
    "buyerId": 1,
    "buyerName": "Fresh Foods Pvt Ltd",
    "trustScore": 80,
    "verificationStatus": "VERIFIED",
    "lastVerificationDate": "2026-08-12",
    "completedTransactions": 6,
    "disputedTransactions": 0,
    "explanation": "verification=50 + transactions=30 - disputes=0 => trustScore=80"
  }
}
```

- `GET /api/trust/mandi/1`
- `GET /api/trust/company/1/history` — verification history
- `POST /api/trust/verification` — record a new verification:

```json
{
  "companyId": 3,
  "verifiedType": "COMPANY",
  "documentsChecked": "GST certificate, trade licence, bank statement",
  "status": "VERIFIED",
  "remarks": "All documents valid"
}
```

### 7.10 Transactions

Record a sale — `POST /api/transactions`:

```json
{
  "farmerId": 1,
  "buyerName": "Fresh Foods Pvt Ltd",
  "buyerType": "COMPANY",
  "crop": "Tomato",
  "quantity": 50,
  "sellingPrice": 2300,
  "transportCost": 0,
  "transactionDate": "2026-09-11",
  "paymentStatus": "PENDING"
}
```

Response (201) — invoice generated automatically:

```json
{
  "success": true,
  "message": "Transaction recorded",
  "data": {
    "id": 1,
    "farmerId": 1,
    "buyerName": "Fresh Foods Pvt Ltd",
    "buyerType": "COMPANY",
    "crop": "Tomato",
    "quantity": 50,
    "sellingPrice": 2300,
    "transportCost": 0,
    "finalRealization": 115000.00,
    "transactionDate": "2026-09-11",
    "paymentStatus": "PENDING",
    "invoiceReference": "TXN-20260911-4821"
  }
}
```

More endpoints:

- `GET /api/transactions/1`
- `GET /api/transactions?farmerId=1` — farmer's sale history
- `GET /api/transactions/invoice/TXN-20260911-4821`
- `PATCH /api/transactions/1/payment-status?status=PAID`

### 7.11 AI decision endpoint — `POST /api/ai/decision`

```json
{
  "farmerId": 1,
  "crop": "Tomato",
  "quantity": 50,
  "farmerNotes": "I need money within 3 days"
}
```

Runs the full pipeline (farmer input → mandi data → company offers → costs →
realization → compare → trust → rank → recommendation) and returns the
recommendation plus pipeline notes. **No external AI API is needed** — the
default implementation is rule-based. To plug in an LLM later, implement the
`AiDecisionEngine` interface in a new class and mark it `@Primary`.

### 7.12 Error responses (all errors look like this)

Validation error — `POST /api/farmers` with a bad phone:

```json
{
  "status": 400,
  "error": "Bad Request",
  "message": "Validation failed. Please check the fields.",
  "timestamp": "2026-09-11T20:30:00.123",
  "path": "/api/farmers",
  "fieldErrors": {
    "phone": "Phone must be a valid 10-digit Indian mobile number"
  }
}
```

Not found: `GET /api/farmers/999` → 404 with
`"message": "Farmer not found with id: 999"`.
Duplicate: creating a farmer with an existing phone → 409.

---

## 8. How the calculation works

**The formula (one place — `CalculationService.calculateRealization`):**

```
farmerRealization = sellingPrice − transportCost − otherCosts + valueShare
```

- **Mandi option:** selling price = mandi price × quantity;
  transport from saved routes (or estimate);
  other costs = 150 ₹/quintal (commission + loading + grading); no value share.
- **Company offer:** selling price = base price × quantity;
  value share = base × valueSharingPercent / 100;
  transport = 0 if company pickup, else offer's per-quintal transport cost;
  other costs from the offer.

**Trust score (0–100):**

```
trustScore = verificationPoints + transactionPoints − disputePenalty
  verification: VERIFIED=50, PENDING=20, else 0
  transactions: +5 each, max +30
  disputes:     −10 each
```

**Recommendation ranking:** highest `realizationPerQuintal` wins;
tie-break by trust score. Reasons are generated by comparing the winner
with the runner-up (net realization, transport cost, trust, payment terms,
verified status, value sharing).

---

## 9. Common problems

| Problem | Fix |
|---|---|
| `Access denied for user 'root'@'localhost'` | Wrong MySQL password in `application.properties` |
| `Unknown database 'farmer_platform_db'` | Run `CREATE DATABASE farmer_platform_db;` (or rely on `createDatabaseIfNotExist=true`) |
| `Port 8080 was already in use` | Change `server.port` in `application.properties` |
| Lombok errors in VS Code | Install "Extension Pack for Java", enable annotation processing |
| Tables not created | Check MySQL is running; check username/password; see console logs |
| `Public Key Retrieval is not allowed` | Already handled by `allowPublicKeyRetrieval=true` in the JDBC URL |

---

## 10. What to build next (suggested)

- Spring Security + JWT login for farmers/companies (kept out for simplicity)
- Replace `TransportService.estimateDistance` with Google Maps Distance Matrix API
- Real LLM/LangGraph implementation of `AiDecisionEngine` (interface is ready)
- Admin role for verification workflows
- Daily price-update job for mandi data
