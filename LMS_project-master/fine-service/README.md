#  Fine Service

## 📚 Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Folder Structure](#folder-structure)
- [REST API Endpoints](#rest-api-endpoints)
- [Data Model](#data-model)
- [Module Architecture Diagram](#module-architecture-diagram)
- [Component Diagram](#component-diagram)
- [Sequence Diagram](#sequence-diagram)
- [Integration](#integration)
- [Run Locally](#run-locally)

## Overview
- The **Fine Service** handles overdue tracking and fine calculations in the Library Management System. It monitors late returns from the Borrowing Service, calculates penalties based on overdue days, and enables members to view and pay fines. It operates independently and is integrated through Eureka for discoverability.

---

##  Features
- Track and calculate fines for overdue books
- Associate fines with borrowing transactions or members
- Mark fines as paid or pending
- Provide fine summaries by member
- Support fine payment status updates

---
## Folder Structure
<pre>
src/
└── main/
    ├── java/
    │   └── com.library.book/
    │       ├── controller/       # REST controllers
    │       ├── dto/              # Data Transfer Objects
    │       ├── entity/           # JPA Entities
    │       ├── repository/       # Spring Data Repositories
    │       └── service/          # Business logic layer
    └── resources/
        └── application.properties  # App configuration
</pre>
---
##  REST API Endpoints

| Method | Endpoint                    | Description                      |
|--------|-----------------------------|----------------------------------|
| GET    | `/api/fines/{memberId}`     | Get fines for a specific member |
| POST   | `/api/fines`                | Create fine record (overdue)    |
| POST   | `/api/fines/pay/{fineId}`   | Mark a fine as paid             |

> Swagger UI available at: `/swagger-ui/index.html`

---

##  Data Model

### `Fine` Entity

| Field            | Type      | Description                           |
|------------------|-----------|---------------------------------------|
| fineId           | BIGINT    | Primary key                           |
| memberId         | BIGINT    | Foreign key to `Member`               |
| amount           | DECIMAL   | Fine amount calculated                |
| status           | VARCHAR   | PENDING / PAID                        |
| transactionDate  | DATE      | When fine was assessed or paid        |

---
## Component diagram
 
```mermaid
flowchart LR
  subgraph React_Frontend [React Frontend]
    direction TB
    A[Fine UI Components]
    B[Fine API Client]
  end
  subgraph Backend [Spring Boot ]
    direction TB
    C[FineController]
    D[FineService]
    E[FineRepository]
  end
  subgraph Database [Relational Database]
    direction TB
    F[(Fine Table)]
  end
  G[Fine DTO]
  H[Fine Entity]
  %% Connections
  B -->|HTTP/REST| C
  C -->|Calls| D
  D -->|Calls| E
  E -->|ORM / JDBC| F
  C ---|uses| G
  E ---|maps to| H
  %% Styling
  classDef ui fill:#dae8fc,stroke:#6c8ebf,color:#1a237e
  classDef backend fill:#d5e8d4,stroke:#82b366,color:#1b4332
  classDef database fill:#f3e5f5,stroke:#ab47bc,color:#4a148c
  classDef model fill:#fff2cc,stroke:#d6b656,color:#7f4f24
  class A,B ui
  class C,D,E backend
  class F database
  class G,H model
```
---
## Sequence Diagram
```mermaid
 
sequenceDiagram
  actor UI as React Frontend
  participant C as FineController
  participant S as FineService
  participant R as FineRepository
  participant DB as Database
  UI->>C: HTTP POST /api/books (FineDto)
  C->>S: registerMember(fineDto)
  S->>R: save(FineEntity)
  R->>DB: INSERT INTO Fine
  R-->>S: Return Saved Fine
  S-->>C: Return FineDto
  C-->>UI: 201 Created (FineDto)
  %% Styling (as comments for Mermaid, no visual impact)
  %% UI:       #dae8fc (soft blue)
  %% Controller/Service/Repo: #d5e8d4 (light green)
  %% DB:       #f3e5f5 (lavender)
```
---
##  Integration

- **Borrowing Service**: triggers fine creation when books are overdue
- **Notification Service** *(optional)*: sends reminders to users with pending fines

---
##  Run Locally
```bash
# Clone this repo
# Navigate to the folder
cd member-service
# Build and run
mvn clean install
mvn spring-boot:run
