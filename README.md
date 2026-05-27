# 🚗 Vehicle Control & Information Management System

<p align="center">
  <img src="https://img.shields.io/badge/Oracle-F80000?style=for-the-badge&logo=oracle&logoColor=white"/>
  <img src="https://img.shields.io/badge/SQL-4479A1?style=for-the-badge&logo=postgresql&logoColor=white"/>
  <img src="https://img.shields.io/badge/Database_Design-8A2BE2?style=for-the-badge&logoColor=white"/>
</p>

> An Oracle SQL database system designed to help law enforcement in Bangladesh verify vehicles, identify their owners and drivers, and track cases filed against them — all through a normalized relational database.

---

## 📌 Table of Contents

- [About the Project](#-about-the-project)
- [Database Schema](#-database-schema)
- [ER Relationships](#-er-relationships)
- [Normalization](#-normalization)
- [Table Creation (DDL)](#-table-creation-ddl)
- [Query Design](#-query-design)
- [Repository Contents](#-repository-contents)
- [How to Run](#-how-to-run)
- [Team](#-team)

---

## 📖 About the Project

**Vehicle Control & Information Management System** is a database project developed at **American International University-Bangladesh (AIUB)**, Fall 2022–2023, for the course *Introduction to Database*, Section C, Group 6, under faculty **Mst. Tasnim Parveen**.

In Bangladesh, law enforcement units face difficulty verifying vehicle ownership and driver details on the spot, and cannot easily check whether an existing case has been filed against a vehicle. This system addresses that gap by providing a structured relational database that links vehicles to their drivers, owners, overseeing police officers, and filed cases.

---

## 🗄️ Database Schema

The system consists of **6 final normalized tables**:

### 1. `VEHICLE`
| Column | Type | Constraint |
|--------|------|------------|
| `VEHICLE_NUMBER` | NUMBER(10) | PRIMARY KEY |
| `VEHICLE_REG_DATE` | VARCHAR(20) | — |
| `VEHICLE_TYPE` | VARCHAR2(20) | NOT NULL |
| `VEHICLE_MODEL` | VARCHAR2(30) | UNIQUE |
| `DRIVER_LICENSE_NUMBER` | NUMBER(20) | FK → DRIVER |

### 2. `DRIVER`
| Column | Type | Constraint |
|--------|------|------------|
| `DRIVER_LICENSE_NUMBER` | NUMBER(15) | PRIMARY KEY |
| `DRIVER_NAME` | VARCHAR2(30) | NOT NULL |
| `DRIVER_PHONE_NO` | NUMBER(15) | UNIQUE |
| `DRIVER_HOME_DIST` | VARCHAR2(15) | — |

### 3. `OWNER`
| Column | Type | Constraint |
|--------|------|------------|
| `OWNER_ID` | NUMBER(15) | PRIMARY KEY |
| `OWNER_NAME` | VARCHAR2(30) | — |
| `OWNER_PHONE_NO` | NUMBER(15) | UNIQUE |
| `OWNER_ADDRESS` | VARCHAR2(40) | — |
| `OWNERE_MAIL` | VARCHAR2(40) | UNIQUE |
| `VEHICLE_NUMBER` | NUMBER(10) | FK → VEHICLE |

### 4. `POLICE`
| Column | Type | Constraint |
|--------|------|------------|
| `POLICE_ID` | NUMBER(10) | PRIMARY KEY |
| `POLICE_NAME` | VARCHAR2(30) | — |
| `POLICE_PHONE_NO_1` | NUMBER(15) | UNIQUE |
| `POLICE_WORKING_CITY` | VARCHAR2(10) | — |
| `POLICEWORKINGZONE` | VARCHAR2(10) | — |

### 5. `CASE`
| Column | Type | Constraint |
|--------|------|------------|
| `CASE_NO` | NUMBER(10) | PRIMARY KEY (via SEQUENCE) |
| `CASE_DATE` | DATE | — |
| `CASE_INFO` | VARCHAR2(40) | — |
| `CASE_FINE` | NUMBER(8) | — |
| `POLICE_ID` | NUMBER(20) | FK → POLICE |

> `CASE_NO` is auto-generated using a sequence (`CASE_CASENO`), starting at 101, incrementing by 1, with a max of 20,000.

### 6. `VP` *(Vehicle–Police junction table)*
| Column | Type | Constraint |
|--------|------|------------|
| `VP_ID` | NUMBER(20) | PRIMARY KEY |
| `POLICE_ID` | NUMBER(20) | FK → POLICE |
| `VEHICLE_NUMBER` | NUMBER(20) | FK → VEHICLE |

---

## 🔗 ER Relationships

| Relationship | Cardinality | Description |
|---|---|---|
| DRIVER **drives** VEHICLE | One-to-One | Each vehicle is assigned one driver |
| OWNER **owns** VEHICLE | One-to-Many | An owner can own multiple vehicles |
| POLICE **observes** VEHICLE | Many-to-Many | Resolved via the `VP` junction table |
| POLICE **gives** CASE | One-to-Many | A police officer can file multiple cases |

---

## 📐 Normalization

All relations were normalized from **UNF → 1NF → 2NF → 3NF**. The key normalization steps across each relationship were:

**DRIVER DRIVES VEHICLE**
- **1NF:** `DRIVER_PHONE_NO` identified as a multivalued attribute and separated
- **2NF:** Split into `VEHICLE` and `DRIVER` tables on their respective primary keys
- **3NF:** No transitive dependencies found

**OWNER OWNS VEHICLE**
- **1NF:** `OWNER_PHONE_NO` identified as a multivalued attribute
- **2NF:** Split into `VEHICLE` and `OWNER` tables
- **3NF:** No transitive dependencies found

**POLICE GIVES CASE**
- **1NF:** `POLICE_PHONE_NO` identified as a multivalued attribute
- **2NF:** Split into `POLICE` and `CASE` tables
- **3NF:** No transitive dependencies found

**POLICE OBSERVES VEHICLE** *(Many-to-Many)*
- **1NF:** `POLICE_PHONE_NO` identified as a multivalued attribute
- **2NF:** Three tables formed — `VEHICLE`, `POLICE`, and junction `VP`
- **3NF:** No transitive dependencies found

---

## 🏗️ Table Creation (DDL)

```sql
-- DRIVER table (create first — referenced by VEHICLE)
CREATE TABLE DRIVER (
    DRIVER_LICENSE_NUMBER NUMBER(15) CONSTRAINT DRIVER_LICENSE_NUMBER_PK PRIMARY KEY,
    DRIVER_NAME           VARCHAR2(30) NOT NULL,
    DRIVER_PHONE_NO       NUMBER(15) UNIQUE,
    DRIVER_HOME_DIST      VARCHAR2(15)
);

-- VEHICLE table
CREATE TABLE VEHICLE (
    VEHICLE_NUMBER        NUMBER(10) CONSTRAINT VEHICLE_NUMBER_PK PRIMARY KEY,
    VEHICLE_REG_DATE      VARCHAR(20),
    VEHICLE_TYPE          VARCHAR2(20) NOT NULL,
    VEHICLE_MODEL         VARCHAR2(30) UNIQUE,
    DRIVER_LICENSE_NUMBER NUMBER(20) CONSTRAINT D_LICENCE_NUMBER_FK REFERENCES DRIVER
);

-- OWNER table
CREATE TABLE OWNER (
    OWNER_ID         NUMBER(15) CONSTRAINT OWNER_ID_PK PRIMARY KEY,
    OWNER_NAME       VARCHAR2(30),
    OWNER_PHONE_NO   NUMBER(15) UNIQUE,
    OWNER_ADDRESS    VARCHAR2(40),
    OWNERE_MAIL      VARCHAR2(40) UNIQUE,
    VEHICLE_NUMBER   NUMBER(10) CONSTRAINT VEHICLE_OWNER_FK REFERENCES VEHICLE(VEHICLE_NUMBER)
);

-- POLICE table
CREATE TABLE POLICE (
    POLICE_ID            NUMBER(10) CONSTRAINT POLICE_ID_PK PRIMARY KEY,
    POLICE_NAME          VARCHAR2(30),
    POLICE_PHONE_NO_1    NUMBER(15) UNIQUE,
    POLICE_WORKING_CITY  VARCHAR2(10),
    POLICEWORKINGZONE    VARCHAR2(10)
);

-- CASE sequence and table
CREATE SEQUENCE CASE_CASENO
    INCREMENT BY 1
    START WITH 101
    MAXVALUE 20000
    NOCACHE
    NOCYCLE;

CREATE TABLE CASE (
    CASE_NO    NUMBER(10) CONSTRAINT CASE_NO_PK PRIMARY KEY,
    CASE_DATE  DATE,
    CASE_INFO  VARCHAR2(40),
    CASE_FINE  NUMBER(8),
    POLICE_ID  NUMBER(20) CONSTRAINT P_ID_FK REFERENCES POLICE(POLICE_ID)
);

-- VP junction table (Vehicle–Police many-to-many)
CREATE TABLE VP (
    VP_ID          NUMBER(20) CONSTRAINT VP_ID_PK PRIMARY KEY,
    POLICE_ID      NUMBER(20) CONSTRAINT PO_ID_FK REFERENCES POLICE(POLICE_ID),
    VEHICLE_NUMBER NUMBER(20) CONSTRAINT VEHI_NUMBER_FK REFERENCES VEHICLE(VEHICLE_NUMBER)
);
```

> ⚠️ **Create order matters:** `DRIVER` → `VEHICLE` → `OWNER`, `POLICE` → `CASE`, then `VP` last (depends on both `POLICE` and `VEHICLE`).

---

## 🔍 Query Design

The project demonstrates a range of SQL query types:

### Functions
```sql
-- Display owner names in uppercase
SELECT UPPER(OWNER_NAME) FROM OWNER;

-- Average case fine amount
SELECT AVG(CASE_FINE) FROM CASE;
```

### Single-Row Subquery
```sql
-- Case with the minimum fine
SELECT CASE_NO, CASE_INFO, CASE_DATE
FROM CASE
WHERE CASE_FINE = (SELECT MIN(CASE_FINE) FROM CASE);
```

### Multiple-Row Subquery
```sql
-- Cases with fines greater than the 'MODIFIED CAR' case
SELECT CASE_NO, CASE_INFO
FROM CASE
WHERE CASE_FINE > (SELECT CASE_FINE FROM CASE WHERE CASE_INFO = 'MODIFIED CAR');
```

### Join
```sql
-- Vehicle number, driver license, and case info together
SELECT V.VEHICLENUMBER AS "VEHICLE NUMBER",
       D.DRIVERLICENSENUMBER AS "DRIVER LICENSE NO",
       C.CASEINFO AS "CASE INFORMATION"
FROM VEHICLE V, DRIVER D, CASES C
WHERE V.VEHICLENUMBER = D.VEHICLENUMBER_F
  AND V.VEHICLENUMBER = C.VEHICLENUMBER_F;
```

### Views
```sql
-- View: full vehicle information (vehicle + owner + driver)
CREATE VIEW VEHICLE_INFO AS
SELECT VEHICLE.VEHICLE_NUMBER, OWNER.OWNER_NAME, OWNER.OWNER_ID,
       DRIVER.DRIVE_NAME, DRIVER.DRIVER_LICENSE_NUMBER
FROM VEHICLE, OWNER, DRIVER
WHERE VEHICLE.VEHICLENUMBER = OWNER.VEHICLENUMBER_F
  AND VEHICLE.VEHICLENUMBER = DRIVER.VEHICLENUMBER_F
ORDER BY VEHICLENUMBER ASC;

-- View: drivers based in Dhaka
CREATE VIEW DRIVER_DHAKA ("NAME", "LICENSE NUMBER") AS
SELECT DRIVERNAME, DRIVERLICENSENUMBER
FROM DRIVER
WHERE DRIVERHOMEDIST = 'DHAKA';
```

---

## 📁 Repository Contents

| File | Description |
|------|-------------|
| `Database project.docx` | Full project report — introduction, ER diagram, normalization, DDL, DML, queries |
| `GROUP-6_SEC(C).pptx` | Presentation slides covering all project phases |
| `README.md` | This file |

> ⚠️ **Note:** The SQL script files are embedded in the `.docx` report rather than provided as standalone `.sql` files. Copy the DDL and DML statements from the document to run them in Oracle SQL Developer or SQL*Plus.

---

## ▶️ How to Run

### Prerequisites
- Oracle Database (Express Edition or higher)
- Oracle SQL Developer or SQL*Plus

### Steps

1. Open **Oracle SQL Developer** and connect to your database instance.

2. Create tables in the correct order to respect foreign key dependencies:
   ```
   DRIVER → VEHICLE → OWNER
   POLICE → CASE
   VEHICLE + POLICE → VP
   ```

3. Copy and run each `CREATE TABLE` statement from the DDL section above.

4. Create the sequence before inserting into `CASE`:
   ```sql
   CREATE SEQUENCE CASE_CASENO START WITH 101 INCREMENT BY 1 MAXVALUE 20000 NOCACHE NOCYCLE;
   ```

5. Insert sample data using the `INSERT INTO` statements from `Database project.docx`.

6. Verify each table:
   ```sql
   SELECT * FROM VEHICLE;
   SELECT * FROM DRIVER;
   SELECT * FROM OWNER;
   SELECT * FROM POLICE;
   SELECT * FROM CASE;
   SELECT * FROM VP;
   ```

---

## 👩‍💻 Team

**Group No: 06 | Course: Introduction to Database | Section: C | AIUB | Fall 2022–2023**

| Name | Student ID |
|------|-----------|
| **Rifah Sanzida** | 22-47154-1 |
| **Md Samin Yeaser** | 22-47139-1 |
| **Tahmida Alamgir** | 22-46020-1 |

**Supervised by:** Mst. Tasnim Parveen

**Rifah Sanzida**
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/rifah-sanzida-b58141290/)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat&logo=github&logoColor=white)](https://github.com/Rifah22)

---

## 📄 License

This project is open source and available for educational purposes.
