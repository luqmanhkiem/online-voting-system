# 🗳️ Online Voting System
| Name               | Matrics No.              |
| ------------------ | ------------------------ |
| Muhammad Daniel Bin Hasbulah   | B032310612 |
| Fuad Bin Mohd Razali| B032310705  |
| Muhammad Nizar Rifa'at Bin       | B032310611    |
| Muhammad Khawarizmi Bin Mohd Saffian   | B032310589 |
| Ahmad Luqmanul Hakiem Bin Rusli | B032310853 |
## 📌 Project Overview
The Online Voting System is a secure, web-based ballot platform designed to digitise elections in schools, universities, clubs and small–to–medium organisations.
It eliminates paper ballots, reduces human error, and provides real-time results while guaranteeing that every voter can vote only once.

## Commercial Value/Third Party Integration
| Value Driver         | Details                                                                                                                                                                                                                                                                     |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Market Potential** | • Ready-to-deploy Software as a Service for institutions that run annual elections (student councils, club committees, company or board-level-decision).                                                                                           |
| **Third-Party APIs** | • **Firebase Realtime Database** – Used to store and sync the voter list and voting status securely in real time. This enables: (a) Cloud-based backup of data. (b) Cross-device access to results. (c) Live updates for reports and audits|

## High Level Diagram
![hld](https://github.com/user-attachments/assets/70126354-7b7c-48e4-b0d9-51c8fb0978c7)

- **Frontend Apps**: Separate views for voters and admin
- **Backend**: PHP server handles API logic
- **Database**: MySQL stores local data; Firebase stores synced voter/export data
- **External Service**: Firebase for real-time monitoring and data export

---

## 🔧 Backend Application

### Technology Stack
| Component       | Technology        |
|----------------|-------------------|
| Language        | Java 17           |
| Framework       | Spring Boot 3.5.3 |
| Build Tool      | Maven             |
| Database        | MySQL             |
| ORM             | Spring Data JPA (Hibernate) |
| Security        | Spring Security   |
| API Format      | RESTful (JSON)    |
| External Service| Firebase Firestore|
| Authentication  | Token-based (matricNo / username) |


| # | Method | Endpoint & Query     | Headers / Body                              | cURL snippet                                                                                                      |
| - | ------ | -------------------- | ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| 1 | POST   | `/api/login`         | Body (JSON): `{ "matricNo": "B032310678" }` | `curl -X POST http://localhost:8080/api/login -H "Content-Type: application/json" -d '{"matricNo":"B032310678"}'` |
| 2 | GET    | `/api/teams`         | none                                        | `curl http://localhost:8080/api/teams`                                                                            |
| 3 | POST   | `/api/vote?teamId=2` | Header: `Authorization: Bearer B032310678`  | `curl -X POST "http://localhost:8080/api/vote?teamId=2" -H "Authorization: Bearer B032310678"`                    |


### Example Success and Error Responses 

**1. POST /login**

Success Response 200

```
{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "role": "voter",
  "expires_in": 3600
}
```

Error Response 401
```
{ "error": "Invalid credentials" }
```
**2. GET /teams**

Success Response 200
```
[
  { "id": 1, "name": "Team Kasturi", "votes": 124 },
  { "id": 2, "name": "Team Lestari", "votes": 98 },
  { "id": 3, "name": "Team Jebat",   "votes": 117 }
]
```
**3. POST /vote**

Success 201`
```
{ "message": "Vote recorded" }
```
Eror 409
```
{ "error": "Already voted" }
```

### 🔒 Security

- Basic token-based security:
  - Voters use matric number as login and as Bearer token
  - Staff use username + password (currently plaintext, with BCrypt support available)
- All protected routes require a valid Authorization header:
  - Voter routes require a valid matricNo token
  - Staff routes require a valid staff token
- Spring Security handles route protection and custom authentication logic via Authentica

### 📱 Frontend Applications

| # | Method | Endpoint            | Headers / Body                                                 | cURL snippet                                                                                                                             |
| - | ------ | ------------------- | -------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| 4 | POST   | /api/staff/login  | Body (JSON): { "username": "admin", "password": "admin123" } | curl -X POST http://localhost:8080/api/staff/login -H "Content-Type: application/json" -d '{"username":"admin","password":"admin123"}' |
| 5 | GET    | /api/staff/teams  | Header: Authorization: Bearer admin                          | curl http://localhost:8080/api/staff/teams -H "Authorization: Bearer admin"                                                            |
| 6 | POST   | /api/staff/export | Header: Authorization: Bearer admin                          | curl -X POST http://localhost:8080/api/staff/export -H "Authorization: Bearer admin"

#### 1️⃣ Voter Web App
- **Purpose**  
  Allows voters to log in with their **matric number**, view teams, and cast **one single vote**.

- **Technology Stack**  
  • Pure **HTML5**, **CSS3**, **vanilla JavaScript (ES6)**  
  • Zero build step; files served statically by Apache/Nginx

- **API Integration**  
  • All network calls via `fetch()`  
  • JWT stored in an **HttpOnly cookie** (automatically attached)  
  • After a successful vote the UI **disables the form** and displays “Already voted”

#### 2️⃣ Admin Web App
- **Purpose**
Central **real-time dashboard** for staff/admins to:
1. Log in with username & password.  
2. View **live vote counts** and **voter status** updated every 5 s.  
3. Trigger **one-click backup** of the voter roll into **Firebase Firestore** instead of Google Sheets.  
4. Observe export status in **real-time** via Firestore listeners.

#### Technology Stack
- **HTML5**, **CSS3**, **vanilla JavaScript (ES6)**  
- **Firebase JS SDK v10** (`firebase-app`, `firebase-firestore`)  
- **Chart.js v4** – live bar chart of team votes  
- No build step; served statically by Apache/Nginx

#### API Integration
| Method | Endpoint            | Headers / Body                                                 | cURL snippet                                                                                                                             |
| ------ | ------------------- | -------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| POST   | `/api/staff/login`  | Body (JSON): `{ "username": "admin", "password": "admin123" }` | `curl -X POST http://localhost:8080/api/staff/login -H "Content-Type: application/json" -d '{"username":"admin","password":"admin123"}'` |
| GET    | `/api/staff/teams`  | Header: `Authorization: Bearer <JWT>`                          | `curl http://localhost:8080/api/staff/teams -H "Authorization: Bearer <JWT>"`                                                            |
| POST   | `/api/staff/export` | Header: `Authorization: Bearer <JWT>`                          | `curl -X POST http://localhost:8080/api/staff/export -H "Authorization: Bearer <JWT>"`                                                   |

- **Authentication**  
  `POST /api/staff/login` returns a short-lived **admin JWT** stored in an HttpOnly cookie.  
- **Realtime Data**  
  Front-end uses the **Firebase JS SDK** (`onSnapshot`) on collection `exports/{electionId}/voters` for live updates.  
- **Export Flow**  
  1. Admin clicks **Export to Firebase**.  
  2. `POST /api/staff/export` writes the voter list into Firestore at  
     `exports/{electionId}/voters/{matric_no}` (fields: `matric_no`, `voted`, `voted_at`, `exported_at`).  
  3. Firestore listener instantly shows **Export completed** toast and displays the new document count.
     
## Database Design

### Entity-Relationship Diagram
![erddad (1)](https://github.com/user-attachments/assets/97643778-ec2b-4265-abc1-86b9c1fc2909)

 **Schema Justification**
- **TEAM** table keeps an atomic counter votes for fast read queries in the results dashboard
- **VOTER** use matric_no as natural PK to guarantee uniqueness per institution
- **ADMIN** is isolated from VOTER to avoid role confusion and simplifu auth logic
- Foreign key team_id in VOTER enables quick JOIN to know who voted which team for audit

## Business Logic And Data Validation

### Flowchart

**Voters Flowchart**

flowchart TD
    A[Student opens voter-app.html] --> B[Enter Matric No]
    B --> C[POST /api/login {matricNo}]
    C --> D{Valid?}
    D -->|No| B1[Show error & retry]
    D -->|Yes| E[Store token in localStorage]
    E --> F[GET /api/teams]
    F --> G[Display teams + progress bars]
    G --> H{Countdown active?}
    H -->|No| H1[Show "Voting ended"]
    H -->|Yes| I[Student clicks Vote]
    I --> J[POST /api/vote?teamId]
    J --> K{Already voted?}
    K -->|Yes| L[Show "Already voted"]
    K -->|No| M[DB transaction:\nvoter.voted = true\nteam.votes++\nvote row inserted]
    M --> N[Show "Vote successful"]


**Admins Flowchart**

![admin](https://github.com/user-attachments/assets/02ae50a4-3b90-4e6f-9a55-a4423a23e0f3)


## ✅ Data Validation

### 🧪 Frontend Validation

The frontend performs basic input validation to ensure data integrity and improve user experience:

- 🧑‍🎓 Voter Login:
  - Checks if the matric number field is not empty before sending a login request.
- 🧑‍💼 Staff Login:
  - Ensures both username and password fields are filled.
- 🗳️ Voting:
  - Requires a team to be selected before casting a vote.
  - Displays clear feedback if a vote has already been cast.

Validation feedback is displayed using Bulma styling (e.g., has-text-danger for error messages).

---

### 🛡️ Backend Validation

The backend includes robust validation and error handling to prevent unauthorized or invalid operations:

- 🔐 Voter Authentication:
  - Checks if the provided matric number exists in the database.
  - Throws an error if the matric number is not registered.

- 🔐 Staff Authentication:
  - Verifies if the username exists and if the password matches.
  - Password is stored securely using BCrypt hashing (if encoder is enabled).

- 🗳️ Voting Process:
  - Validates Authorization header and parses the token.
  - Ensures the voter has not already voted.
  - Confirms the team ID exists before registering a vote.
  - Records a vote only if all conditions are met (atomic transaction).

- 📤 Export Validation:
  - Validates Firebase initialization before exporting.
  - Clears existing Firestore documents to avoid duplication.
  - Catches and logs all exceptions during export.

---

### 🧱 Fail-Safe Responses

- All invalid or unauthorized requests result in appropriate error responses:
  - Missing or malformed tokens
  - Nonexistent matric numbers
  - Already voted condition
  - Firebase export failures

Errors are returned with meaningful messages to aid debugging and improve UX.
