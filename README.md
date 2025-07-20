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
