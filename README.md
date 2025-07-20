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
| **Third-Party APIs** | • **Google Sheets API** – enables non-technical staff to audit and share live voter data without database access. Chosen because: (a) zero-learning-curve for spreadsheet users, (b) built-in version history & sharing, (c) free tier covers most institutional use-cases. |

## High Level Diagram
 -gambar diagram
- **Frontend Apps**: Separate views for Voters and Admin
- **Backend**: PHP server handles business logic and API endpoints
- **Database**: MySQL stores users, teams, and votes
- **External Service**: Google Sheets API for data export

---

## 🔧 Backend Application

### Technology Stack
- **Language**: PHP
- **Database**: MySQL
- **External Service**: Google Sheets API
- **Security**: Basic Session-based Auth


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

---

#### 2️⃣ Admin Web App
- **Purpose**  
  Dashboard for **staff/admins** to observe **live results**, browse the **voter list**, and trigger a **Google-Sheet export**.

- **Technology Stack**  
  • Same as voter app: HTML, CSS, JS  
  • **Chart.js v4** for a real-time bar chart that refreshes every **10 s**

- **API Integration**  
  • Uses `fetch()` with the JWT cookie  
  • Polling endpoint `/teams` every 10 s for live updates  
  • On **Export** click, calls `/export/sheet` and opens the returned Google Sheet URL in a new tab

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

![fcvoters](https://github.com/user-attachments/assets/54a8d144-e175-4b81-bbff-23d9d989acdc)

**Admins Flowchart**

![fcadmin](https://github.com/user-attachments/assets/d026b894-f677-4175-90ec-1a6312dd0787)

### ✅ Data Validation Rules

| **Area** | **Field / Action** | **Rule** | **Frontend** | **Backend** |
| --- | --- | --- | --- | --- |
| **Login** | Matric Number | Regex `^A\d{2}[A-Z]{2}\d{4}$` (or institute pattern) | Inline red message | Unique in `voter.matric_no`; case-insensitive |
| **Vote** | Team selection | Not empty | Disable “Submit” if empty | FK must exist in `team.id`; 422 if missing |
| **Vote** | One vote only | — | Disable form if cookie shows “voted” | Transaction: check `voter.voted = 0`; 409 if already voted |
| **Admin** | Username | Alphanumeric 3-50 chars | HTML5 pattern | Unique in `staff.username` |
| **Admin** | Password | ≥ 8 chars, 1 upper, 1 lower, 1 digit | Real-time strength meter | Bcrypt hash stored |
| **Export** | Google token | — | — | Verify JWT role = admin; 403 otherwise |
| **API** | JWT | — | Stored in HttpOnly cookie | Verify signature & exp; 401 if invalid |

> All SQL parameters are bound via PDO prepared statements to prevent injection.
