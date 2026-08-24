# 03 — Database & SQL (Senior SDET Interview Prep)

> **Kaise use karein:** Ye document **zero se** shuru hota hai. Maan ke chal raha hoon ki tumne SQL kabhi seriously likhi hi nahi. Isliye har cheez ka structure fixed hai:
> **Kya hai → Kyun QA ko chahiye → Worked example (asli data + asli result rows) → Interview answer (English) → Cross-question + uska jawab**.
>
> Explanation **Hinglish** mein hai taaki concept dimaag mein baithe. Lekin jo `> **Interview answer:**` blockquote mein likha hai — **wahi bolna hai, English mein**. Ratta mat maaro; structure yaad rakho, apne shabdon mein bolo.
>
> `> **[REAL]**` boxes tumhare Merlin AI (construction ERP) project ke actual examples hain. Merlin ka database **MongoDB** hai, SQL nahi — lekin **interview mein SQL 100% poocha jayega**. Isliye do hisse hain:
> - **Part A–B: SQL** — interview clear karne ke liye. Zero se.
> - **Part C: MongoDB** — tumhara actual kaam. Yahan tum **sabse strong** dikh sakte ho, kyunki tumhare paas real production stories hain.
>
> **Sabse important line jo tum interview mein bol sakte ho:**
> *"Our primary datastore is MongoDB, so day to day I write aggregation pipelines and verify index behaviour. But I've learned relational modelling as well because most of the concepts — indexes, transactions, isolation, normalisation — carry over."*
> Ye line honest bhi hai aur senior bhi lagti hai.

---

## Table of Contents

| # | Section |
|---|---|
| — | **PART A — SQL FROM ZERO** |
| 0 | [The Example Schema (sab examples isi par honge)](#0--the-example-schema) |
| 1 | [SELECT aur WHERE](#1--select-aur-where) |
| 2 | [NULL — SQL ka sabse bada trap](#2--null--sql-ka-sabse-bada-trap) |
| 3 | [ORDER BY](#3--order-by) |
| 4 | [LIMIT / OFFSET aur pagination at scale](#4--limit--offset-aur-pagination-at-scale) |
| 5 | [DISTINCT](#5--distinct) |
| 6 | [Aggregate functions — COUNT ka pura sach](#6--aggregate-functions) |
| 7 | [GROUP BY](#7--group-by) |
| 8 | [HAVING vs WHERE](#8--having-vs-where) |
| 9 | [SQL logical execution order](#9--sql-logical-execution-order) |
| 10 | [JOINs — sab types, worked](#10--joins) |
| 11 | [Subqueries — scalar, IN, EXISTS, correlated](#11--subqueries) |
| 12 | [CTEs (WITH) + Recursive CTE](#12--ctes-with--recursive-cte) |
| 13 | [CASE WHEN](#13--case-when) |
| 14 | [Window functions](#14--window-functions) |
| 15 | [Set operations — UNION, INTERSECT, EXCEPT](#15--set-operations) |
| 16 | [INSERT / UPDATE / DELETE / UPSERT](#16--insert--update--delete--upsert) |
| 17 | [Keys — primary, foreign, composite, surrogate](#17--keys) |
| 18 | [Constraints](#18--constraints) |
| 19 | [Indexes — B-tree se EXPLAIN tak](#19--indexes) |
| 20 | [Transactions aur ACID](#20--transactions-aur-acid) |
| 21 | [Isolation levels aur 3 anomalies](#21--isolation-levels-aur-3-anomalies) |
| 22 | [Deadlocks](#22--deadlocks) |
| 23 | [Normalization — 1NF, 2NF, 3NF, BCNF](#23--normalization) |
| 24 | [Views aur Materialized Views](#24--views-aur-materialized-views) |
| 25 | [Stored Procedures aur Triggers](#25--stored-procedures-aur-triggers) |
| — | **PART B — PRACTICE** |
| 26 | [30 SQL practice questions with full solutions](#26--30-sql-practice-questions) |
| — | **PART C — MONGODB (tumhara actual kaam)** |
| 27 | [Document model vs relational](#27--document-model-vs-relational) |
| 28 | [MongoDB CRUD](#28--mongodb-crud) |
| 29 | [Query operators](#29--mongodb-query-operators) |
| 30 | [Aggregation pipeline](#30--aggregation-pipeline) |
| 31 | [Mongo indexes, compound key order, partial indexes](#31--mongo-indexes) |
| 32 | [Transactions in MongoDB](#32--transactions-in-mongodb) |
| 33 | [SQL → MongoDB translation table](#33--sql--mongodb-translation-table) |
| — | **PART D — PRACTICAL QA** |
| 34 | ["UI shows payment successful" — DB verification](#34--ui-shows-payment-successful--what-do-you-verify-in-the-db) |
| 35 | ["API returned 200 but UI is wrong" — RCA](#35--api-returned-200-but-ui-shows-wrong-data--rca) |
| 36 | [Test data setup & cleanup](#36--test-data-setup--cleanup) |
| 37 | [Data integrity checks QA should automate](#37--data-integrity-checks-qa-should-automate) |
| 38 | [Verifying migrations](#38--verifying-migrations) |
| 39 | [DB access from Python tests](#39--db-access-from-python-tests) |
| — | **PART E — SENIOR** |
| 40 | [Senior scenario questions](#40--senior-scenario-questions) |
| 41 | [Red flags — ye jawab mat dena](#41--red-flags--ye-jawab-mat-dena) |
| 42 | [Quick revision cheatsheet](#42--quick-revision-cheatsheet) |

---
---

# PART A — SQL FROM ZERO

---

# 0 — The Example Schema

## 0.1 Kyun ek hi schema

Log SQL isliye nahi seekh paate kyunki har tutorial naya table deta hai. Yahan **ek hi schema** hai — construction ERP jaisa, tumhare Merlin domain se milta-julta — aur **poore document mein wahi** use hoga. 20 examples ke baad ye data tumhe zubaani yaad ho jayega, aur tab SQL asaan lagegi.

Domain: **projects** (sites), **suppliers** (vendors), **users** (staff), **orders** (purchase orders), **order_items** (line items), **invoices**, **payments**.

```
  projects ─────┐
                │  1..N
  suppliers ────┼──► orders ──1..N──► order_items
                │       │
  users ────────┘       │ 1..N
                        ▼
                    invoices ──1..N──► payments

  users.manager_id ──► users.user_id   (self-reference)
```

## 0.2 DDL — CREATE TABLE

```sql
CREATE TABLE suppliers (
    supplier_id  INT PRIMARY KEY,
    name         VARCHAR(100) NOT NULL,
    city         VARCHAR(50),
    rating       INT,                         -- nullable on purpose
    created_at   TIMESTAMP NOT NULL DEFAULT now()
);

CREATE TABLE projects (
    project_id   INT PRIMARY KEY,
    name         VARCHAR(100) NOT NULL,
    city         VARCHAR(50),
    budget       NUMERIC(14,2),
    status       VARCHAR(20) NOT NULL          -- ACTIVE / CLOSED / ON_HOLD
);

CREATE TABLE users (
    user_id     INT PRIMARY KEY,
    name        VARCHAR(100) NOT NULL,
    email       VARCHAR(100) NOT NULL UNIQUE,
    role        VARCHAR(20)  NOT NULL,         -- ADMIN / PM / SITE_ENGINEER
    manager_id  INT REFERENCES users(user_id), -- self FK
    salary      NUMERIC(10,2)
);

CREATE TABLE orders (
    order_id     INT PRIMARY KEY,
    project_id   INT NOT NULL REFERENCES projects(project_id),
    supplier_id  INT REFERENCES suppliers(supplier_id),   -- nullable: draft PO
    created_by   INT REFERENCES users(user_id),
    order_date   DATE NOT NULL,
    status       VARCHAR(20) NOT NULL,          -- DRAFT/APPROVED/DELIVERED/CANCELLED
    total_amount NUMERIC(14,2) NOT NULL DEFAULT 0,
    is_deleted   BOOLEAN NOT NULL DEFAULT FALSE -- soft delete
);

CREATE TABLE order_items (
    item_id    INT PRIMARY KEY,
    order_id   INT NOT NULL REFERENCES orders(order_id) ON DELETE CASCADE,
    material   VARCHAR(50)  NOT NULL,
    quantity   NUMERIC(12,2) NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(12,2) NOT NULL CHECK (unit_price >= 0)
);

CREATE TABLE invoices (
    invoice_id   INT PRIMARY KEY,
    order_id     INT NOT NULL REFERENCES orders(order_id),
    invoice_no   VARCHAR(30) NOT NULL,          -- NOTE: no UNIQUE — jaan-boojh kar
    invoice_date DATE NOT NULL,
    amount       NUMERIC(14,2) NOT NULL,
    status       VARCHAR(20) NOT NULL           -- PENDING/PARTIALLY_PAID/PAID
);

CREATE TABLE payments (
    payment_id      INT PRIMARY KEY,
    invoice_id      INT NOT NULL REFERENCES invoices(invoice_id),
    paid_on         DATE NOT NULL,
    amount          NUMERIC(14,2) NOT NULL,
    method          VARCHAR(20),                -- NEFT / UPI / CHEQUE
    idempotency_key VARCHAR(64) UNIQUE
);
```

> `invoices.invoice_no` par UNIQUE **deliberately nahi** hai — aage "find duplicates" question mein isi ka fayda uthayenge, aur ye exactly wahi bug pattern hai jo tumhare Merlin codebase mein tha (`(org, emailAddress)` par unique constraint na hone se duplicate contacts).

## 0.3 Sample data

**suppliers**

| supplier_id | name | city | rating |
|---|---|---|---|
| 1 | Sharma Cement Co | Jaipur | 4 |
| 2 | Verma Steels | Delhi | 5 |
| 3 | Nakoda Traders | Jaipur | NULL |
| 4 | Bombay Hardware | Mumbai | 3 |
| 5 | Gupta Sanitary | Delhi | NULL |
| 6 | Ajmer Aggregates | Ajmer | 2 |

**projects**

| project_id | name | city | budget | status |
|---|---|---|---|---|
| 1 | Skyline Towers | Jaipur | 50000000.00 | ACTIVE |
| 2 | Metro Depot | Delhi | 120000000.00 | ACTIVE |
| 3 | Green Villa | Jaipur | 8000000.00 | CLOSED |
| 4 | Riverfront Mall | Mumbai | 30000000.00 | ON_HOLD |

**users**

| user_id | name | email | role | manager_id | salary |
|---|---|---|---|---|---|
| 1 | Anil Mehta | anil@merlin.co | ADMIN | NULL | 250000.00 |
| 2 | Ritik Chaturvedi | ritik@merlin.co | PM | 1 | 90000.00 |
| 3 | Sunita Rao | sunita@merlin.co | PM | 1 | 110000.00 |
| 4 | Kabir Singh | kabir@merlin.co | SITE_ENGINEER | 2 | 60000.00 |
| 5 | Meera Nair | meera@merlin.co | SITE_ENGINEER | 2 | 60000.00 |
| 6 | Farhan Ali | farhan@merlin.co | SITE_ENGINEER | 3 | 75000.00 |

**orders**

| order_id | project_id | supplier_id | created_by | order_date | status | total_amount | is_deleted |
|---|---|---|---|---|---|---|---|
| 1 | 1 | 1 | 2 | 2025-01-05 | DELIVERED | 120000.00 | false |
| 2 | 1 | 2 | 2 | 2025-01-18 | DELIVERED | 450000.00 | false |
| 3 | 2 | 2 | 3 | 2025-02-02 | APPROVED | 300000.00 | false |
| 4 | 2 | 4 | 3 | 2025-02-14 | DELIVERED | 75000.00 | false |
| 5 | 1 | 1 | 4 | 2025-02-27 | CANCELLED | 60000.00 | false |
| 6 | 3 | 3 | 3 | 2025-03-03 | DELIVERED | 200000.00 | false |
| 7 | 2 | NULL | 2 | 2025-03-11 | DRAFT | 0.00 | false |
| 8 | 4 | 5 | 6 | 2025-03-19 | APPROVED | 180000.00 | false |
| 9 | 1 | 2 | 2 | 2025-04-02 | DELIVERED | 500000.00 | false |
| 10 | 2 | 1 | 3 | 2025-04-15 | DELIVERED | 250000.00 | false |
| 11 | 3 | 4 | 4 | 2025-04-21 | APPROVED | 90000.00 | **true** |
| 12 | 1 | 3 | 5 | 2025-05-06 | DRAFT | 45000.00 | false |

**order_items**

| item_id | order_id | material | quantity | unit_price | (line total) |
|---|---|---|---|---|---|
| 1 | 1 | Cement OPC 53 | 200 | 400.00 | 80000 |
| 2 | 1 | Sand | 100 | 400.00 | 40000 |
| 3 | 2 | TMT Bar 12mm | 5000 | 65.00 | 325000 |
| 4 | 2 | TMT Bar 16mm | 2000 | 62.50 | 125000 |
| 5 | 3 | TMT Bar 12mm | 4000 | 65.00 | 260000 |
| 6 | 3 | Binding Wire | 500 | 80.00 | 40000 |
| 7 | 4 | Plywood | 150 | 500.00 | 75000 |
| 8 | 5 | Cement OPC 53 | 150 | 400.00 | 60000 |
| 9 | 6 | Bricks | 40000 | 5.00 | 200000 |
| 10 | 8 | CP Fittings | 300 | 600.00 | 180000 |
| 11 | 9 | TMT Bar 12mm | 6000 | 65.00 | 390000 |
| 12 | 9 | Cement OPC 53 | 275 | 400.00 | 110000 |
| 13 | 10 | Cement OPC 53 | 500 | 400.00 | 200000 |
| 14 | 10 | Sand | 125 | 400.00 | 50000 |
| 15 | 11 | Plywood | 180 | 500.00 | 90000 |

> **Note kar lo:** order **7** aur order **12** ke koi items hi nahi hain. Order 12 ka `total_amount` 45000 hai lekin items zero — ye ek **deliberate data-integrity bug** hai jise hum QA section mein pakdenge. Supplier **6 (Ajmer Aggregates)** ka koi order nahi hai — LEFT JOIN example ke liye.

**invoices**

| invoice_id | order_id | invoice_no | invoice_date | amount | status |
|---|---|---|---|---|---|
| 1 | 1 | INV-2025-001 | 2025-01-20 | 120000.00 | PAID |
| 2 | 2 | INV-2025-002 | 2025-02-01 | 450000.00 | PARTIALLY_PAID |
| 3 | 4 | INV-2025-003 | 2025-02-28 | 75000.00 | PAID |
| 4 | 6 | INV-2025-004 | 2025-03-15 | 200000.00 | PENDING |
| 5 | 9 | INV-2025-005 | 2025-04-10 | 500000.00 | PARTIALLY_PAID |
| 6 | 10 | INV-2025-006 | 2025-04-25 | 250000.00 | PAID |
| 7 | 2 | **INV-2025-002** | 2025-02-01 | 450000.00 | PENDING |
| 8 | 8 | INV-2025-009 | 2025-03-25 | 180000.00 | PENDING |

> Invoice 7 ka `invoice_no` **duplicate** hai (INV-2025-002). Aur sequence mein **007, 008 missing** hain. Dono cheezein practice questions mein use hongi.

**payments**

| payment_id | invoice_id | paid_on | amount | method | idempotency_key |
|---|---|---|---|---|---|
| 1 | 1 | 2025-01-25 | 120000.00 | NEFT | idem-a1 |
| 2 | 2 | 2025-02-10 | 200000.00 | NEFT | idem-b1 |
| 3 | 2 | 2025-03-05 | 100000.00 | UPI | idem-b2 |
| 4 | 3 | 2025-03-01 | 75000.00 | CHEQUE | idem-c1 |
| 5 | 5 | 2025-04-20 | 250000.00 | NEFT | idem-d1 |
| 6 | 6 | 2025-05-02 | 250000.00 | NEFT | idem-e1 |
| 7 | 5 | 2025-05-10 | 100000.00 | UPI | idem-d2 |

**Insert script (agar tum local Postgres mein practice karna chaho — aur tumhe karna chahiye):**

```sql
INSERT INTO suppliers (supplier_id, name, city, rating) VALUES
 (1,'Sharma Cement Co','Jaipur',4), (2,'Verma Steels','Delhi',5),
 (3,'Nakoda Traders','Jaipur',NULL), (4,'Bombay Hardware','Mumbai',3),
 (5,'Gupta Sanitary','Delhi',NULL), (6,'Ajmer Aggregates','Ajmer',2);

INSERT INTO projects VALUES
 (1,'Skyline Towers','Jaipur',50000000,'ACTIVE'),
 (2,'Metro Depot','Delhi',120000000,'ACTIVE'),
 (3,'Green Villa','Jaipur',8000000,'CLOSED'),
 (4,'Riverfront Mall','Mumbai',30000000,'ON_HOLD');

INSERT INTO users VALUES
 (1,'Anil Mehta','anil@merlin.co','ADMIN',NULL,250000),
 (2,'Ritik Chaturvedi','ritik@merlin.co','PM',1,90000),
 (3,'Sunita Rao','sunita@merlin.co','PM',1,110000),
 (4,'Kabir Singh','kabir@merlin.co','SITE_ENGINEER',2,60000),
 (5,'Meera Nair','meera@merlin.co','SITE_ENGINEER',2,60000),
 (6,'Farhan Ali','farhan@merlin.co','SITE_ENGINEER',3,75000);

INSERT INTO orders VALUES
 (1,1,1,2,'2025-01-05','DELIVERED',120000,false),
 (2,1,2,2,'2025-01-18','DELIVERED',450000,false),
 (3,2,2,3,'2025-02-02','APPROVED',300000,false),
 (4,2,4,3,'2025-02-14','DELIVERED',75000,false),
 (5,1,1,4,'2025-02-27','CANCELLED',60000,false),
 (6,3,3,3,'2025-03-03','DELIVERED',200000,false),
 (7,2,NULL,2,'2025-03-11','DRAFT',0,false),
 (8,4,5,6,'2025-03-19','APPROVED',180000,false),
 (9,1,2,2,'2025-04-02','DELIVERED',500000,false),
 (10,2,1,3,'2025-04-15','DELIVERED',250000,false),
 (11,3,4,4,'2025-04-21','APPROVED',90000,true),
 (12,1,3,5,'2025-05-06','DRAFT',45000,false);

INSERT INTO order_items VALUES
 (1,1,'Cement OPC 53',200,400),(2,1,'Sand',100,400),
 (3,2,'TMT Bar 12mm',5000,65),(4,2,'TMT Bar 16mm',2000,62.50),
 (5,3,'TMT Bar 12mm',4000,65),(6,3,'Binding Wire',500,80),
 (7,4,'Plywood',150,500),(8,5,'Cement OPC 53',150,400),
 (9,6,'Bricks',40000,5),(10,8,'CP Fittings',300,600),
 (11,9,'TMT Bar 12mm',6000,65),(12,9,'Cement OPC 53',275,400),
 (13,10,'Cement OPC 53',500,400),(14,10,'Sand',125,400),
 (15,11,'Plywood',180,500);

INSERT INTO invoices VALUES
 (1,1,'INV-2025-001','2025-01-20',120000,'PAID'),
 (2,2,'INV-2025-002','2025-02-01',450000,'PARTIALLY_PAID'),
 (3,4,'INV-2025-003','2025-02-28',75000,'PAID'),
 (4,6,'INV-2025-004','2025-03-15',200000,'PENDING'),
 (5,9,'INV-2025-005','2025-04-10',500000,'PARTIALLY_PAID'),
 (6,10,'INV-2025-006','2025-04-25',250000,'PAID'),
 (7,2,'INV-2025-002','2025-02-01',450000,'PENDING'),
 (8,8,'INV-2025-009','2025-03-25',180000,'PENDING');

INSERT INTO payments VALUES
 (1,1,'2025-01-25',120000,'NEFT','idem-a1'),
 (2,2,'2025-02-10',200000,'NEFT','idem-b1'),
 (3,2,'2025-03-05',100000,'UPI','idem-b2'),
 (4,3,'2025-03-01',75000,'CHEQUE','idem-c1'),
 (5,5,'2025-04-20',250000,'NEFT','idem-d1'),
 (6,6,'2025-05-02',250000,'NEFT','idem-e1'),
 (7,5,'2025-05-10',100000,'UPI','idem-d2');
```

> **Seriously — ye 10 minute lagao.** `docker run --name pg -e POSTGRES_PASSWORD=pass -p 5432:5432 -d postgres:16` chala do, `psql` se connect karo, upar ka script paste karo. Jo SQL tumne apne haath se chala kar dekhi hai, wahi interview mein confidently bolne aayegi. Sirf padhne se SQL nahi aati.

---

# 1 — SELECT aur WHERE

## 1.1 Kya hai

`SELECT` = "mujhe ye columns do". `FROM` = "is table se". `WHERE` = "sirf wo rows jo ye condition satisfy karti hain".

```sql
SELECT column1, column2
FROM   table_name
WHERE  condition;
```

`SELECT *` ka matlab "saare columns". Production code mein `*` avoid karte hain (column add hone par output silently badal jaata hai), par debugging mein theek hai.

## 1.2 Kyun QA ko chahiye

QA ka 80% DB kaam yahi hai: **"UI pe ye dikh raha hai — DB mein actually kya pada hai?"** Bug report mein API response ke saath DB row paste kar dena tumhe turant senior dikhata hai, kyunki tumne developer ka aadha debugging kaam khud kar diya.

## 1.3 Worked example

```sql
SELECT order_id, project_id, status, total_amount
FROM   orders
WHERE  status = 'DELIVERED';
```

**Result:**

| order_id | project_id | status | total_amount |
|---|---|---|---|
| 1 | 1 | DELIVERED | 120000.00 |
| 2 | 1 | DELIVERED | 450000.00 |
| 4 | 2 | DELIVERED | 75000.00 |
| 6 | 3 | DELIVERED | 200000.00 |
| 9 | 1 | DELIVERED | 500000.00 |
| 10 | 2 | DELIVERED | 250000.00 |

## 1.4 Saare WHERE operators

| Operator | Matlab | Example |
|---|---|---|
| `=` | equal | `WHERE status = 'DRAFT'` |
| `<>` ya `!=` | not equal | `WHERE status <> 'DRAFT'` |
| `>` `<` `>=` `<=` | comparison | `WHERE total_amount >= 200000` |
| `BETWEEN a AND b` | range, **dono ends inclusive** | `WHERE total_amount BETWEEN 100000 AND 300000` |
| `IN (…)` | list mein se koi ek | `WHERE status IN ('APPROVED','DELIVERED')` |
| `NOT IN (…)` | list mein nahi | `WHERE city NOT IN ('Delhi')` |
| `LIKE` | pattern match | `WHERE name LIKE 'Sharma%'` |
| `ILIKE` (Postgres) | case-insensitive LIKE | `WHERE name ILIKE '%cement%'` |
| `IS NULL` / `IS NOT NULL` | null check | `WHERE supplier_id IS NULL` |
| `AND` `OR` `NOT` | combine | `WHERE a = 1 AND (b = 2 OR c = 3)` |

**LIKE wildcards:**

| Wildcard | Matlab | Example | Match karega |
|---|---|---|---|
| `%` | zero ya zyada characters | `'S%'` | Sharma, Sand, S |
| `_` | **exactly ek** character | `'_and'` | Sand, band |
| `'%cement%'` | kahin bhi contains | | Sharma Cement Co |

```sql
-- Sab suppliers jinke naam mein 'a' ke baad kuch bhi ho aur jo Jaipur/Delhi ke hain
SELECT supplier_id, name, city, rating
FROM   suppliers
WHERE  city IN ('Jaipur','Delhi')
  AND  name LIKE '%a%';
```

**Result:**

| supplier_id | name | city | rating |
|---|---|---|---|
| 1 | Sharma Cement Co | Jaipur | 4 |
| 2 | Verma Steels | Delhi | 5 |
| 3 | Nakoda Traders | Jaipur | NULL |

(Gupta Sanitary bhi 'a' contains karta hai — haan, `Gupta` mein 'a' hai. Toh actually 5 bhi aayega.)

Chalo isko theek se karte hain — `name LIKE 'S%'` (naam S se shuru):

```sql
SELECT supplier_id, name, city
FROM   suppliers
WHERE  name LIKE 'S%';
```

| supplier_id | name | city |
|---|---|---|
| 1 | Sharma Cement Co | Jaipur |

**BETWEEN example:**

```sql
SELECT order_id, total_amount
FROM   orders
WHERE  total_amount BETWEEN 100000 AND 300000
ORDER  BY total_amount;
```

| order_id | total_amount |
|---|---|
| 1 | 120000.00 |
| 8 | 180000.00 |
| 6 | 200000.00 |
| 10 | 250000.00 |
| 3 | 300000.00 |

Dhyan do: 100000 aur 300000 **dono include** hain. `BETWEEN` inclusive hota hai — ye ek classic boundary-value bug source hai. Dates mein toh aur khatarnak: `order_date BETWEEN '2025-01-01' AND '2025-01-31'` timestamp column par 31 Jan 00:00:00 ke baad ka data **miss** kar dega. Isliye dates ke liye hamesha half-open range:

```sql
WHERE order_date >= '2025-01-01' AND order_date < '2025-02-01'
```

> **Interview answer:**
> "WHERE filters rows before grouping. The operators I use most are equality, comparison, BETWEEN, IN, LIKE and IS NULL. Two things I'm careful about as a tester: BETWEEN is inclusive on both ends, so for timestamp ranges I always use a half-open range — greater-than-or-equal the start and strictly less-than the next day — otherwise I silently drop everything after midnight on the last day. And LIKE with a leading wildcard, like percent-cement-percent, cannot use a normal B-tree index, so it forces a full scan. That's a performance smell I flag in reviews."

> **Cross-question: "`IN` aur `OR` mein kya difference hai?"**
> Functionally kuch nahi — `status IN ('A','B')` aur `status='A' OR status='B'` same hain, optimizer dono ko same plan deta hai. `IN` readable hai. Real difference tab aata hai jab `IN` ke andar **subquery** ho — tab ye semi-join ban jaata hai, jo alag cheez hai.

> **Cross-question: "`NOT IN` mein kya khatra hai?"**
> Agar list mein ek bhi `NULL` aa gaya toh **poora result empty** ho jayega. `x NOT IN (1, NULL)` ka matlab hai `x<>1 AND x<>NULL` → second part `UNKNOWN` → poora `UNKNOWN` → row reject. Isliye subquery ke saath `NOT IN` ki jagah `NOT EXISTS` prefer karo. (Section 11 mein detail.)

---

# 2 — NULL — SQL ka sabse bada trap

## 2.1 Kya hai

`NULL` ka matlab **"zero" nahi**, **"empty string" nahi** — matlab hai **"pata nahi / value hai hi nahi"**. Isliye NULL par koi bhi comparison `TRUE`/`FALSE` nahi, **`UNKNOWN`** deta hai. Aur `WHERE` sirf `TRUE` rows ko paas karta hai — `UNKNOWN` reject ho jaati hai.

```
NULL = NULL        →  UNKNOWN   (NOT TRUE!)
NULL <> 'DRAFT'    →  UNKNOWN
NULL > 5           →  UNKNOWN
NULL IS NULL       →  TRUE      ← sirf yahi kaam karta hai
```

## 2.2 Kyun QA ko chahiye

Ye **sabse zyada real bugs** paida karne wala concept hai. "Report mein 12 orders hain par list mein 11 dikh rahe hain" — 90% baar culprit NULL hota hai.

## 2.3 The classic trap — worked

Question: **"Wo saare orders jinka supplier Sharma Cement Co (id=1) nahi hai."** Naive query:

```sql
SELECT order_id, supplier_id
FROM   orders
WHERE  supplier_id <> 1;
```

**Result — 10 rows:**

| order_id | supplier_id |
|---|---|
| 2 | 2 |
| 3 | 2 |
| 4 | 4 |
| 6 | 3 |
| 8 | 5 |
| 9 | 2 |
| 10 | 1 → ❌ nahi, 10 ka supplier 1 hai, exclude |

Chalo dobara sahi se likhta hoon. Orders jinka supplier_id ≠ 1: orders 2(2), 3(2), 4(4), 6(3), 8(5), 9(2), 11(4), 12(3). **Order 7 ka supplier_id NULL hai — wo result mein NAHI aayega.**

| order_id | supplier_id |
|---|---|
| 2 | 2 |
| 3 | 2 |
| 4 | 4 |
| 6 | 3 |
| 8 | 5 |
| 9 | 2 |
| 11 | 4 |
| 12 | 3 |

**Order 7 gayab hai** — jabki uska supplier Sharma Cement Co nahi hai (uska supplier hai hi nahi). Business ke hisaab se ye galat hai.

**Fix:**

```sql
SELECT order_id, supplier_id
FROM   orders
WHERE  supplier_id <> 1 OR supplier_id IS NULL;

-- ya cleaner (Postgres/standard SQL):
WHERE  supplier_id IS DISTINCT FROM 1;
```

Ab order 7 bhi aa jayega — 9 rows.

## 2.4 NULL aur aggregates

```sql
SELECT COUNT(*)        AS total_rows,
       COUNT(rating)   AS rows_with_rating,
       AVG(rating)     AS avg_rating,
       SUM(rating)     AS sum_rating
FROM   suppliers;
```

**Result:**

| total_rows | rows_with_rating | avg_rating | sum_rating |
|---|---|---|---|
| 6 | 4 | 3.5000 | 14 |

`AVG` = 14/4 = 3.5, **not** 14/6 = 2.33. Aggregates NULL ko **skip** karte hain, denominator mein count nahi karte. Ye ek super common reporting bug hai: "average rating 3.5 kyun dikha raha hai jab 6 suppliers hain?"

## 2.5 NULL handling functions

| Function | Kaam |
|---|---|
| `COALESCE(a, b, c)` | pehla non-NULL return karta hai |
| `NULLIF(a, b)` | agar a = b toh NULL, warna a (divide-by-zero se bachne ke liye) |
| `x IS DISTINCT FROM y` | NULL-safe `<>` |
| `x IS NOT DISTINCT FROM y` | NULL-safe `=` |

```sql
SELECT supplier_id, name, COALESCE(rating, 0) AS rating_safe
FROM   suppliers;
```

| supplier_id | name | rating_safe |
|---|---|---|
| 1 | Sharma Cement Co | 4 |
| 2 | Verma Steels | 5 |
| 3 | Nakoda Traders | 0 |
| 4 | Bombay Hardware | 3 |
| 5 | Gupta Sanitary | 0 |
| 6 | Ajmer Aggregates | 2 |

> **[REAL]** Merlin mein documents ke optional fields — jaise ek purchase order ka `supplier` ya `deliveryDate` — MongoDB mein aksar **field hi missing** hoti hai, `null` bhi nahi. Isliye Mongo mein `{supplier: {$ne: "X"}}` aur `{supplier: {$exists: false}}` do alag cheezein hain. SQL ka NULL-trap Mongo mein "missing field vs null field" trap ban jaata hai — same bug, alag shakal. Interview mein ye connection banana bahut strong lagta hai.

> **Interview answer:**
> "NULL means unknown, not zero and not empty string. Any comparison with NULL evaluates to UNKNOWN, and WHERE only passes rows that are TRUE — so a filter like `status <> 'DRAFT'` silently drops every row where status is NULL. That's a real class of bug: the count on the dashboard doesn't match the count in the list. The fix is either an explicit `OR col IS NULL`, or `IS DISTINCT FROM`, which is NULL-safe. Aggregates also skip NULLs, so AVG divides by the count of non-null values, not the row count — I've seen that produce an average that looks impossibly high."

> **Cross-question: "`COUNT(*)` aur `COUNT(column)` mein farak?"**
> `COUNT(*)` saari rows ginta hai. `COUNT(column)` sirf wo rows ginta hai jahan wo column NULL nahi hai. Upar ke example mein 6 vs 4.

> **Cross-question: "Do NULLs `GROUP BY` mein same group mein aayenge?"**
> Haan. `=` ke liye NULL ≠ NULL, lekin `GROUP BY`, `DISTINCT` aur `UNION` NULLs ko **ek dusre ke barabar** treat karte hain. Ye SQL ki inconsistency hai jo interviewers ko poochna pasand hai.

---

# 3 — ORDER BY

## 3.1 Kya hai

Result rows ko sort karta hai. **Bina `ORDER BY` ke SQL kisi order ki guarantee nahi deta** — chahe aaj same order aa raha ho.

```sql
SELECT ... FROM ... WHERE ...
ORDER BY col1 [ASC|DESC], col2 [ASC|DESC];
```

Default `ASC`. Postgres mein `NULLS FIRST` / `NULLS LAST` bhi de sakte ho.

## 3.2 Kyun QA ko chahiye

Ye direct test-flakiness ka source hai. Agar API `ORDER BY created_at` karta hai aur do rows ka `created_at` same hai, toh unka aapsi order **run-to-run badal sakta hai** — aur tumhara assertion `response[0].id == 5` random fail karega. **Fix: hamesha ek deterministic tie-breaker maango** — `ORDER BY created_at DESC, id DESC`.

## 3.3 Worked examples

**Single column:**

```sql
SELECT order_id, total_amount
FROM   orders
ORDER  BY total_amount DESC
LIMIT  4;
```

| order_id | total_amount |
|---|---|
| 9 | 500000.00 |
| 2 | 450000.00 |
| 3 | 300000.00 |
| 10 | 250000.00 |

**Multiple columns** — pehle project, phir amount descending:

```sql
SELECT project_id, order_id, total_amount
FROM   orders
WHERE  is_deleted = false
ORDER  BY project_id ASC, total_amount DESC;
```

| project_id | order_id | total_amount |
|---|---|---|
| 1 | 9 | 500000.00 |
| 1 | 2 | 450000.00 |
| 1 | 1 | 120000.00 |
| 1 | 5 | 60000.00 |
| 1 | 12 | 45000.00 |
| 2 | 3 | 300000.00 |
| 2 | 10 | 250000.00 |
| 2 | 4 | 75000.00 |
| 2 | 7 | 0.00 |
| 3 | 6 | 200000.00 |
| 4 | 8 | 180000.00 |

**NULLS FIRST / LAST:**

```sql
SELECT supplier_id, name, rating
FROM   suppliers
ORDER  BY rating DESC NULLS LAST;
```

| supplier_id | name | rating |
|---|---|---|
| 2 | Verma Steels | 5 |
| 1 | Sharma Cement Co | 4 |
| 4 | Bombay Hardware | 3 |
| 6 | Ajmer Aggregates | 2 |
| 3 | Nakoda Traders | NULL |
| 5 | Gupta Sanitary | NULL |

Postgres mein default: `ASC` → NULLS LAST, `DESC` → NULLS **FIRST**. MySQL mein ulta (NULLs sabse chhote maane jaate hain). Isliye cross-database code mein hamesha explicitly likho.

> **Interview answer:**
> "ORDER BY sorts the final result set, and it's the last thing to run before LIMIT. The important thing I've learned from writing automated tests is that without ORDER BY, SQL gives no ordering guarantee at all — it can change when the plan changes or when the table grows. And even with ORDER BY, if the sort column has ties, the order within the tie is undefined. So for any paginated or asserted-on endpoint I ask the dev to add a unique tie-breaker column, usually the primary key. That single change removed a whole category of flaky tests for us."

> **Cross-question: "ORDER BY costly kyun hai?"**
> Agar sort column par index nahi hai toh DB ko poora result set memory ya disk pe sort karna padta hai (`work_mem` se bada hua toh external merge sort — disk I/O). Index sorted order mein hota hai, toh matching index ho toh sort skip ho jaata hai. `EXPLAIN` mein `Sort` node dikhe toh wahi jagah hai.

---

# 4 — LIMIT / OFFSET aur pagination at scale

## 4.1 Kya hai

```sql
SELECT ... ORDER BY ... LIMIT 10 OFFSET 20;   -- rows 21–30
```

`LIMIT n` = kitni rows chahiye. `OFFSET m` = pehli m rows chhod do.
(SQL Server / Oracle mein: `OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY`.)

## 4.2 Worked example

```sql
SELECT order_id, order_date, total_amount
FROM   orders
WHERE  is_deleted = false
ORDER  BY order_date, order_id
LIMIT  3 OFFSET 3;      -- "page 2", page size 3
```

| order_id | order_date | total_amount |
|---|---|---|
| 4 | 2025-02-14 | 75000.00 |
| 5 | 2025-02-27 | 60000.00 |
| 6 | 2025-03-03 | 200000.00 |

## 4.3 OFFSET scale par kyun tootta hai

Do problems:

**Problem 1 — Performance.** `OFFSET 100000` ka matlab DB ko pehle **100,000 rows padhni aur discard karni** padti hain, phir agli 10 deni hain. Ye O(offset) hai. Page 1 fast, page 10,000 timeout.

```
OFFSET 0     : [XXXXXXXXXX] ......................  → 10 rows read
OFFSET 100000: read & throw away 100000 rows, then [XXXXXXXXXX]
               ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ pure waste
```

**Problem 2 — Correctness (page drift).** Tum page 1 padh rahe ho, tabhi koi naya order insert ho gaya jo sorting mein upar aata hai. Ab page 2 par jo row pehle page 2 par thi wo **shift ho kar page 1 pe chali gayi** — matlab tumne wo row **kabhi nahi dekhi**. Delete ke case mein ek row **do baar** dikh sakti hai.

```
t0  page1 = [A B C]   page2 = [D E F]
t1  new row "A0" inserted at top
t2  page2 = [C D E]     ← "C" repeat, "F" abhi bhi aage khisak gaya
```

## 4.4 Fix — keyset / cursor pagination

`OFFSET` ki jagah **"last seen value"** se aage padho:

```sql
-- Page 1
SELECT order_id, order_date FROM orders
ORDER BY order_date, order_id
LIMIT 3;
-- maan lo last row: order_date='2025-02-02', order_id=3

-- Page 2 — keyset
SELECT order_id, order_date FROM orders
WHERE (order_date, order_id) > (DATE '2025-02-02', 3)
ORDER BY order_date, order_id
LIMIT 3;
```

Ye index par direct seek karta hai — O(log n), page number chahe jo ho. Trade-off: random page ("page 500 pe jao") support nahi karta, sirf next/prev.

> **[REAL]** Merlin mein list APIs (purchase orders, suppliers) `orgId + isDeleted` par filter karte hain aur `createdAt` se sort. Jab tum pagination test karo toh **do checks** karo: (1) saare pages ke IDs collect karke `set` banao — length total count ke barabar honi chahiye, koi duplicate/missing nahi; (2) concurrent insert karte hue paginate karo — page drift dikh jayega. Ye do checks likh dena tumhe alag level pe dikhata hai.

> **Interview answer:**
> "LIMIT and OFFSET are fine for the first few pages, but OFFSET is O(offset) — the database physically reads and discards every skipped row, so deep pages get slow. Worse, OFFSET pagination isn't stable: if rows are inserted or deleted while a user pages through, rows can be skipped entirely or shown twice, because the offset is positional, not anchored to data. For large or live datasets I expect keyset pagination — carry the last seen sort key as a cursor and use a comparison against it, which is an index seek regardless of depth. When I test pagination I collect IDs across all pages into a set and assert no duplicates and no gaps against the total count, and I run one pass while concurrently inserting rows to expose drift."

> **Cross-question: "`LIMIT` bina `ORDER BY` ke chalaoge toh?"**
> Non-deterministic. DB koi bhi 10 rows de sakta hai, aur do baar chalane par alag rows aa sakti hain. Ye ek genuine bug hai jo QA ko pakadna chahiye.

---

# 5 — DISTINCT

## 5.1 Kya hai

Duplicate rows hata deta hai. **Poori selected row** par kaam karta hai, sirf pehle column par nahi.

```sql
SELECT DISTINCT city FROM suppliers;
```

| city |
|---|
| Jaipur |
| Delhi |
| Mumbai |
| Ajmer |

```sql
SELECT DISTINCT city, rating FROM suppliers ORDER BY city;
```

| city | rating |
|---|---|
| Ajmer | 2 |
| Delhi | 5 |
| Delhi | NULL |
| Jaipur | 4 |
| Jaipur | NULL |
| Mumbai | 3 |

Dekho — `Delhi` do baar aaya kyunki **(city, rating) ka combination** alag hai. Ye ek classic misunderstanding hai.

## 5.2 DISTINCT ON (Postgres-only, par bahut useful)

```sql
-- Har project ka sabse mehenga order
SELECT DISTINCT ON (project_id) project_id, order_id, total_amount
FROM   orders
WHERE  is_deleted = false
ORDER  BY project_id, total_amount DESC;
```

| project_id | order_id | total_amount |
|---|---|---|
| 1 | 9 | 500000.00 |
| 2 | 3 | 300000.00 |
| 3 | 6 | 200000.00 |
| 4 | 8 | 180000.00 |

## 5.3 DISTINCT ka smell

Agar tumhe JOIN ke baad `DISTINCT` lagana pad raha hai, toh **99% baar JOIN galat hai** — kahin fan-out ho raha hai (one-to-many join ne rows multiply kar di). `DISTINCT` us bug ko **chhupa** deta hai. Sahi fix: aggregate karo ya `EXISTS` use karo.

```sql
-- ❌ SMELL: DISTINCT se duplicate chhupa rahe hain
SELECT DISTINCT o.order_id, o.total_amount
FROM   orders o JOIN order_items i ON i.order_id = o.order_id;

-- ✅ Intent clear: "orders jinke items hain"
SELECT o.order_id, o.total_amount
FROM   orders o
WHERE  EXISTS (SELECT 1 FROM order_items i WHERE i.order_id = o.order_id);
```

> **Interview answer:**
> "DISTINCT de-duplicates the entire selected row, not just the first column — that surprises people. The bigger point I'd make is that DISTINCT after a join is usually a code smell. A one-to-many join fans rows out, and slapping DISTINCT on top hides that rather than fixing it. If the real intent is existence, EXISTS says so directly and doesn't fan out; if the intent is a total, aggregate properly. As a tester, when I see DISTINCT in a reporting query I go look for a fan-out bug — that's exactly how double-counted totals happen."

> **Cross-question: "DISTINCT vs GROUP BY?"**
> Bina aggregate ke `SELECT DISTINCT a, b` aur `SELECT a, b … GROUP BY a, b` same result dete hain aur aksar same plan bhi. `GROUP BY` tab chahiye jab aggregate function (`SUM`, `COUNT`) bhi chahiye. Intent ke hisaab se choose karo.

---

# 6 — Aggregate functions

## 6.1 Kya hai

Kai rows ko **ek value** mein sikodne wale functions.

| Function | Kaam | NULL handling |
|---|---|---|
| `COUNT(*)` | rows ki ginti | NULLs bhi count |
| `COUNT(col)` | non-NULL values ki ginti | NULLs skip |
| `COUNT(DISTINCT col)` | unique non-NULL values | NULLs skip |
| `SUM(col)` | jod | NULLs skip; **sab NULL → result NULL, 0 nahi** |
| `AVG(col)` | average = SUM/COUNT(col) | NULLs skip |
| `MIN` / `MAX` | sabse chhota / bada | NULLs skip |
| `STRING_AGG(col, ',')` | strings jodna (Postgres) | |

## 6.2 COUNT ka pura sach — worked

```sql
SELECT COUNT(*)                    AS a_all_rows,
       COUNT(supplier_id)          AS b_non_null_supplier,
       COUNT(DISTINCT supplier_id) AS c_unique_suppliers,
       COUNT(1)                    AS d_count_one
FROM   orders;
```

**Result:**

| a_all_rows | b_non_null_supplier | c_unique_suppliers | d_count_one |
|---|---|---|---|
| 12 | 11 | 5 | 12 |

- **12** — total rows (deleted order 11 bhi shaamil).
- **11** — order 7 ka `supplier_id` NULL hai, isliye skip.
- **5** — distinct supplier_ids = {1,2,3,4,5}. Supplier 6 ka koi order nahi.
- **12** — `COUNT(1)` bilkul `COUNT(*)` jaisa hai. **Ye faster nahi hai** — ek purani myth hai; modern optimizers dono ko identical treat karte hain.

## 6.3 SUM ka NULL trap

```sql
SELECT SUM(total_amount) FROM orders WHERE status = 'REJECTED';
```

Koi row match nahi karti → result **NULL**, `0` nahi. Agar API ne isko directly serialize kar diya toh UI par "₹null" ya blank dikhega.

```sql
SELECT COALESCE(SUM(total_amount), 0) AS total FROM orders WHERE status='REJECTED';
-- → 0
```

Lekin `COUNT` empty set par hamesha `0` deta hai, NULL nahi. Ye asymmetry yaad rakho.

## 6.4 Aur examples

```sql
SELECT MIN(total_amount) AS min_amt,
       MAX(total_amount) AS max_amt,
       ROUND(AVG(total_amount), 2) AS avg_amt,
       SUM(total_amount) AS sum_amt,
       COUNT(*) AS n
FROM   orders
WHERE  is_deleted = false;
```

Non-deleted orders ke amounts: 120000, 450000, 300000, 75000, 60000, 200000, 0, 180000, 500000, 250000, 45000 → 11 rows, sum = 2,180,000.

| min_amt | max_amt | avg_amt | sum_amt | n |
|---|---|---|---|---|
| 0.00 | 500000.00 | 198181.82 | 2180000.00 | 11 |

> **Interview answer:**
> "COUNT star counts rows, COUNT of a column counts only rows where that column is not null, and COUNT DISTINCT counts unique non-null values — so on a nullable column all three give different numbers. COUNT(1) is identical to COUNT star; the idea that it's faster is a myth. The one that actually bites in production is SUM: over an empty set SUM returns NULL, not zero, while COUNT returns zero. If that NULL flows straight into a JSON response you get a blank or 'null' on the dashboard instead of a zero total. I wrap money aggregates in COALESCE and I explicitly test the empty-data case for every report."

> **Cross-question: "`COUNT(*)` bade table pe slow kyun hai Postgres mein?"**
> Postgres MVCC ki wajah se har row ki visibility check karni padti hai, isliye exact `COUNT(*)` full scan hai (ya index-only scan agar visibility map fresh ho). MySQL InnoDB mein bhi similar. Approximate chahiye toh `pg_class.reltuples` use karte hain.

---

# 7 — GROUP BY

## 7.1 Kya hai

`GROUP BY` rows ko **buckets** mein baant deta hai, aur har bucket **ek output row** banti hai. Aggregate functions har bucket ke andar chalte hain.

```
orders (project_id ke hisaab se):

  project 1: [o1 o2 o5 o9 o12]  ──► ek row: project_id=1, COUNT=5, SUM=1175000
  project 2: [o3 o4 o7 o10]     ──► ek row: project_id=2, COUNT=4, SUM=625000
  project 3: [o6]               ──► ek row
  project 4: [o8]               ──► ek row
```

## 7.2 The golden rule

**`SELECT` mein sirf wahi columns aa sakte hain jo (a) `GROUP BY` mein hain, ya (b) kisi aggregate function ke andar hain.** Kyunki baaki columns ka bucket ke andar ek hi value nahi hai — DB kaunsi chune?

```sql
-- ❌ ERROR: column "o.order_id" must appear in the GROUP BY clause
SELECT project_id, order_id, SUM(total_amount)
FROM   orders GROUP BY project_id;
```

MySQL (old, `ONLY_FULL_GROUP_BY` off) ise chalne deta tha aur **random value** return karta tha — silent data corruption ka source. Postgres error deta hai, jo behtar hai.

## 7.3 Worked example

```sql
SELECT project_id,
       COUNT(*)                       AS order_count,
       SUM(total_amount)              AS total_value,
       ROUND(AVG(total_amount), 2)    AS avg_value,
       MAX(order_date)                AS last_order
FROM   orders
WHERE  is_deleted = false
GROUP  BY project_id
ORDER  BY project_id;
```

**Result:**

| project_id | order_count | total_value | avg_value | last_order |
|---|---|---|---|---|
| 1 | 5 | 1175000.00 | 235000.00 | 2025-05-06 |
| 2 | 4 | 625000.00 | 156250.00 | 2025-04-15 |
| 3 | 1 | 200000.00 | 200000.00 | 2025-03-03 |
| 4 | 1 | 180000.00 | 180000.00 | 2025-03-19 |

Check: project 1 = 120000+450000+60000+500000+45000 = 1,175,000 ✓ (order 11 deleted, project 3 se bhi hata)

## 7.4 Multi-column GROUP BY

```sql
SELECT project_id, status, COUNT(*) AS cnt, SUM(total_amount) AS amt
FROM   orders
WHERE  is_deleted = false
GROUP  BY project_id, status
ORDER  BY project_id, status;
```

| project_id | status | cnt | amt |
|---|---|---|---|
| 1 | CANCELLED | 1 | 60000.00 |
| 1 | DELIVERED | 3 | 1070000.00 |
| 1 | DRAFT | 1 | 45000.00 |
| 2 | APPROVED | 1 | 300000.00 |
| 2 | DELIVERED | 2 | 325000.00 |
| 2 | DRAFT | 1 | 0.00 |
| 3 | DELIVERED | 1 | 200000.00 |
| 4 | APPROVED | 1 | 180000.00 |

## 7.5 GROUP BY ke saath JOIN — fan-out ka khatra

```sql
-- ❌ GALAT: order ka total_amount har item ke saath repeat ho raha hai
SELECT o.project_id, SUM(o.total_amount) AS wrong_total
FROM   orders o
JOIN   order_items i ON i.order_id = o.order_id
GROUP  BY o.project_id;
```

Order 1 ke 2 items hain, toh `o.total_amount` (120000) **do baar** count hoga. Ye **double counting** ka classic bug hai — aur real reporting bugs isi se aate hain.

**Result (galat):** project 1 → 120000×2 + 450000×2 + 60000×1 + 500000×2 = 240000+900000+60000+1000000 = 2,200,000 (jabki sahi 1,175,000 hai, aur order 12 toh items na hone ki wajah se gayab bhi ho gaya).

**Sahi tareeka — pehle aggregate, phir join:**

```sql
SELECT o.project_id, SUM(o.total_amount) AS right_total
FROM   orders o
WHERE  o.is_deleted = false
GROUP  BY o.project_id;
```

Ya agar items ka sum chahiye toh **item level par aggregate karo**:

```sql
SELECT o.project_id, SUM(i.quantity * i.unit_price) AS item_total
FROM   orders o
JOIN   order_items i ON i.order_id = o.order_id
WHERE  o.is_deleted = false
GROUP  BY o.project_id;
```

| project_id | item_total |
|---|---|
| 1 | 1130000.00 |
| 2 | 375000.00 |
| 3 | 200000.00 |
| 4 | 180000.00 |

> **Ye 1,130,000 vs 1,175,000 ka farak hi toh asli bug hai** — order 12 ka `total_amount` 45000 hai par uske items nahi hain. Ek QA ke roop mein tumne abhi ek data-integrity defect khoj liya. Section 37 mein isko automate karenge.

> **Interview answer:**
> "GROUP BY collapses rows into buckets and produces one output row per bucket. The rule is that anything in the SELECT list must either be in the GROUP BY or wrapped in an aggregate, because otherwise the value is ambiguous within the bucket. The mistake I look for in review is aggregating a parent-level column after joining to a child table — joining orders to order items multiplies the order row once per line item, so SUM of the order total double-counts. That's the single most common cause of a report showing a number that's too high. The fix is to aggregate the child table first in a subquery or CTE and then join the already-aggregated result."

> **Cross-question: "GROUP BY implicit sorting deta hai?"**
> Nahi. Purane MySQL versions dete the (kyunki wo sort-based grouping karta tha), lekin hash aggregation mein order random hota hai. Order chahiye toh `ORDER BY` explicitly likho.

---

# 8 — HAVING vs WHERE

## 8.1 Kya hai

- **`WHERE`** — grouping se **pehle** individual rows filter karta hai.
- **`HAVING`** — grouping ke **baad** groups filter karta hai. Yahan aggregate functions use kar sakte ho.

```
FROM orders
   ↓
WHERE is_deleted = false          ← individual rows chhaanti hain
   ↓
GROUP BY project_id               ← buckets bante hain
   ↓
HAVING SUM(total_amount) > 500000 ← poore buckets chhaante jaate hain
   ↓
SELECT project_id, SUM(...)
```

## 8.2 Worked example

```sql
SELECT project_id, COUNT(*) AS cnt, SUM(total_amount) AS total
FROM   orders
WHERE  is_deleted = false           -- row-level filter
GROUP  BY project_id
HAVING SUM(total_amount) > 500000   -- group-level filter
ORDER  BY total DESC;
```

**Grouping ke baad (HAVING se pehle):**

| project_id | cnt | total |
|---|---|---|
| 1 | 5 | 1175000.00 |
| 2 | 4 | 625000.00 |
| 3 | 1 | 200000.00 |
| 4 | 1 | 180000.00 |

**HAVING ke baad — final result:**

| project_id | cnt | total |
|---|---|---|
| 1 | 5 | 1175000.00 |
| 2 | 4 | 625000.00 |

## 8.3 Kya galti hoti hai

```sql
-- ❌ ERROR: aggregate functions are not allowed in WHERE
SELECT project_id, SUM(total_amount)
FROM orders WHERE SUM(total_amount) > 500000 GROUP BY project_id;
```

Kyunki jab `WHERE` chalta hai, groups abhi bane hi nahi — `SUM` ka koi matlab hi nahi.

```sql
-- ⚠️ Chalega, par galat/slow: HAVING mein non-aggregate condition
SELECT project_id, SUM(total_amount)
FROM orders GROUP BY project_id
HAVING project_id = 1;      -- ← ye WHERE mein hona chahiye
```

Ye technically valid hai par **saare projects ke liye grouping karega, phir 3 groups phenk dega**. Row-level filter hamesha `WHERE` mein daalo — kaam kam hoga aur index use ho sakega.

## 8.4 Dono ek saath

```sql
-- Un suppliers ke naam jinhone 2025 mein 2 se zyada delivered orders diye
SELECT s.name, COUNT(*) AS delivered_orders, SUM(o.total_amount) AS value
FROM   suppliers s
JOIN   orders o ON o.supplier_id = s.supplier_id
WHERE  o.status = 'DELIVERED'
  AND  o.is_deleted = false
GROUP  BY s.supplier_id, s.name
HAVING COUNT(*) >= 2
ORDER  BY value DESC;
```

DELIVERED orders: 1(s1), 2(s2), 4(s4), 6(s3), 9(s2), 10(s1).
Groups: s1 → 2 orders (120000+250000=370000); s2 → 2 orders (450000+500000=950000); s3 → 1; s4 → 1.
`HAVING COUNT(*) >= 2` ke baad:

| name | delivered_orders | value |
|---|---|---|
| Verma Steels | 2 | 950000.00 |
| Sharma Cement Co | 2 | 370000.00 |

> **Interview answer:**
> "WHERE filters individual rows before grouping; HAVING filters the groups after aggregation, which is why aggregate functions are legal in HAVING but not in WHERE. Practically, the rule I follow is: any condition that only depends on a single row belongs in WHERE, and only conditions on aggregates belong in HAVING. Putting a row-level condition in HAVING still works, but the database groups everything first and then throws groups away — it does more work and it can't use an index for that predicate. So it's a correctness-neutral but performance-negative mistake, and it's a good thing to catch in query review."

> **Cross-question: "HAVING bina GROUP BY ke use ho sakta hai?"**
> Haan. Us case mein poori table **ek hi group** maani jaati hai. `SELECT SUM(total_amount) FROM orders HAVING SUM(total_amount) > 1000000;` — ek row ya zero rows dega.

> **Cross-question: "HAVING mein SELECT alias use kar sakte ho?"**
> Standard SQL mein nahi (kyunki `SELECT` `HAVING` ke baad evaluate hota hai), lekin **Postgres aur MySQL dono allow karte hain** ek extension ke roop mein. `ORDER BY` mein alias sab jagah chalta hai. `WHERE` mein kahin nahi.

---

# 9 — SQL logical execution order

## 9.1 Kya hai

SQL jis order mein **likhi** jaati hai, us order mein **chalti nahi**. Logical order ye hai:

```
 1. FROM        ← tables uthao
 2. JOIN / ON   ← unhe jodo
 3. WHERE       ← rows chhaano
 4. GROUP BY    ← buckets banao
 5. HAVING      ← buckets chhaano
 6. SELECT      ← columns choose karo, aliases yahan bante hain
 7. DISTINCT    ← duplicates hatao
 8. ORDER BY    ← sort karo (aliases yahan available hain)
 9. LIMIT/OFFSET← kaato
```

Window functions `SELECT` ke saath (step 6 ke aas-paas) evaluate hote hain — **`WHERE` ke baad, `ORDER BY` se pehle**. Isliye window function ka result `WHERE` mein use nahi kar sakte; usko CTE/subquery mein wrap karna padta hai.

## 9.2 Ye kya explain karta hai

**(a) SELECT alias `WHERE` mein kaam kyun nahi karta:**

```sql
-- ❌ ERROR: column "amt" does not exist
SELECT total_amount AS amt FROM orders WHERE amt > 100000;
```

`WHERE` step 3 par chalta hai, `amt` step 6 par banta hai. Abhi wo exist hi nahi karta.

```sql
-- ✅ ORDER BY mein chalega — wo step 8 hai
SELECT total_amount AS amt FROM orders WHERE total_amount > 100000 ORDER BY amt DESC;
```

**(b) `WHERE` mein aggregate kyun nahi:** aggregates step 4 par bante hain, `WHERE` step 3 par chalta hai.

**(c) Window function `WHERE` mein kyun nahi:**

```sql
-- ❌ ERROR: window functions are not allowed in WHERE
SELECT order_id, ROW_NUMBER() OVER (ORDER BY total_amount DESC) AS rn
FROM orders WHERE rn <= 3;

-- ✅ CTE mein wrap karo
WITH ranked AS (
    SELECT order_id, total_amount,
           ROW_NUMBER() OVER (ORDER BY total_amount DESC) AS rn
    FROM   orders WHERE is_deleted = false
)
SELECT order_id, total_amount FROM ranked WHERE rn <= 3;
```

| order_id | total_amount | 
|---|---|
| 9 | 500000.00 |
| 2 | 450000.00 |
| 3 | 300000.00 |

## 9.3 Ek query jisme sab dikhe

```sql
SELECT   p.name AS project,                     -- 6
         COUNT(*) AS order_count                -- 6
FROM     orders o                               -- 1
JOIN     projects p ON p.project_id=o.project_id-- 2
WHERE    o.is_deleted = false                   -- 3
GROUP BY p.project_id, p.name                   -- 4
HAVING   COUNT(*) >= 2                          -- 5
ORDER BY order_count DESC                       -- 8  (alias allowed)
LIMIT    5;                                     -- 9
```

| project | order_count |
|---|---|
| Skyline Towers | 5 |
| Metro Depot | 4 |

> **Interview answer:**
> "SQL is declarative, so the written order isn't the evaluation order. Logically it's FROM and JOIN first, then WHERE, then GROUP BY, then HAVING, then SELECT, then DISTINCT, then ORDER BY, then LIMIT. That single fact explains three things people trip over: you can't use a SELECT alias in WHERE because the alias doesn't exist yet, you can't use an aggregate in WHERE because groups haven't been formed, and you can't filter on a window function in WHERE because windows are computed with SELECT — you have to wrap it in a CTE and filter outside. Note this is the logical order; the optimizer is free to physically execute differently as long as the result is identical."

> **Cross-question: "Toh optimizer isi order mein chalata hai?"**
> Nahi. Ye **logical/semantic** order hai — result ka matlab define karta hai. Physical plan optimizer decide karta hai: wo predicates ko join ke neeche push kar sakta hai (predicate pushdown), join order badal sakta hai, aggregate pehle kar sakta hai. Guarantee sirf ye hai ki **result wahi aayega** jo logical order se aata.

---

# 10 — JOINs

## 10.1 Kya hai

Do (ya zyada) tables ki rows ko ek condition par jodna.

```sql
SELECT ...
FROM   A
<JOIN TYPE> B ON A.key = B.key;
```

## 10.2 Visual

```
        A               B
     ┌──────┐       ┌──────┐
     │  ▓▓▓ │███████│ ▓▓▓  │
     │      │       │      │
     └──────┘       └──────┘
       only A   both   only B

INNER JOIN        →  sirf ███  (dono mein match)
LEFT  JOIN        →  ▓▓▓ + ███ (saara A, match hua B)
RIGHT JOIN        →  ███ + ▓▓▓ (match hua A, saara B)
FULL OUTER JOIN   →  ▓▓▓ + ███ + ▓▓▓ (sab kuch)
CROSS JOIN        →  A ka har row × B ka har row (cartesian)
SELF JOIN         →  table apne aap se join (hierarchy, comparisons)
```

**Anti-join pattern (bahut poocha jaata hai):**

```
LEFT JOIN + WHERE b.key IS NULL  →  sirf ▓▓▓ (A jinka B mein match nahi)
```

## 10.3 INNER JOIN

Sirf wo rows jahan **dono taraf match** ho.

```sql
SELECT o.order_id, o.total_amount, s.name AS supplier
FROM   orders o
INNER  JOIN suppliers s ON s.supplier_id = o.supplier_id
WHERE  o.is_deleted = false
ORDER  BY o.order_id;
```

**Result — 10 rows** (order 7 gayab kyunki `supplier_id` NULL hai; order 11 deleted):

| order_id | total_amount | supplier |
|---|---|---|
| 1 | 120000.00 | Sharma Cement Co |
| 2 | 450000.00 | Verma Steels |
| 3 | 300000.00 | Verma Steels |
| 4 | 75000.00 | Bombay Hardware |
| 5 | 60000.00 | Sharma Cement Co |
| 6 | 200000.00 | Nakoda Traders |
| 8 | 180000.00 | Gupta Sanitary |
| 9 | 500000.00 | Verma Steels |
| 10 | 250000.00 | Sharma Cement Co |
| 12 | 45000.00 | Nakoda Traders |

> **QA lens:** ye query "saare orders" ki report ke liye use ho toh **order 7 chup-chaap gayab** ho jayega. "Dashboard 11 bolta hai, list 10 dikhati hai" — classic INNER-JOIN-should-have-been-LEFT-JOIN bug.

## 10.4 LEFT JOIN (LEFT OUTER JOIN)

Left table ki **saari** rows aayengi; right side match na mile toh `NULL`.

```sql
SELECT o.order_id, o.total_amount, s.name AS supplier
FROM   orders o
LEFT   JOIN suppliers s ON s.supplier_id = o.supplier_id
WHERE  o.is_deleted = false
ORDER  BY o.order_id;
```

**Result — 11 rows:**

| order_id | total_amount | supplier |
|---|---|---|
| 1 | 120000.00 | Sharma Cement Co |
| 2 | 450000.00 | Verma Steels |
| 3 | 300000.00 | Verma Steels |
| 4 | 75000.00 | Bombay Hardware |
| 5 | 60000.00 | Sharma Cement Co |
| 6 | 200000.00 | Nakoda Traders |
| **7** | **0.00** | **NULL** |
| 8 | 180000.00 | Gupta Sanitary |
| 9 | 500000.00 | Verma Steels |
| 10 | 250000.00 | Sharma Cement Co |
| 12 | 45000.00 | Nakoda Traders |

## 10.5 RIGHT JOIN

Ulta LEFT JOIN. Right table ki saari rows.

```sql
SELECT s.name AS supplier, o.order_id
FROM   orders o
RIGHT  JOIN suppliers s ON s.supplier_id = o.supplier_id
ORDER  BY s.supplier_id, o.order_id;
```

**Result — supplier 6 bhi aayega (NULL order ke saath):**

| supplier | order_id |
|---|---|
| Sharma Cement Co | 1 |
| Sharma Cement Co | 5 |
| Sharma Cement Co | 10 |
| Verma Steels | 2 |
| Verma Steels | 3 |
| Verma Steels | 9 |
| Nakoda Traders | 6 |
| Nakoda Traders | 12 |
| Bombay Hardware | 4 |
| Bombay Hardware | 11 |
| Gupta Sanitary | 8 |
| **Ajmer Aggregates** | **NULL** |

Practically log RIGHT JOIN kam likhte hain — `FROM suppliers LEFT JOIN orders` likhna zyada padhne-yogya hai. Interview mein bol dena: *"They're symmetric; I prefer LEFT JOIN and reorder the tables, because reading a query where the driving table is first is much easier."*

## 10.6 FULL OUTER JOIN

Dono taraf ka sab kuch, match ho ya na ho.

```sql
SELECT o.order_id, s.name AS supplier
FROM   orders o
FULL   OUTER JOIN suppliers s ON s.supplier_id = o.supplier_id
ORDER  BY o.order_id NULLS LAST;
```

Result mein 12 order rows + `Ajmer Aggregates` ke liye ek `(NULL, Ajmer Aggregates)` row = **13 rows**. Order 7 ke saamne supplier NULL, Ajmer ke saamne order NULL.

**FULL OUTER JOIN ka sabse acha QA use — reconciliation.** Do systems ka data compare karna:

```sql
-- Legacy vs new: kaunsi rows kahan hain, aur kahan mismatch hai
SELECT COALESCE(a.order_id, b.order_id) AS order_id,
       a.total_amount AS old_amt,
       b.total_amount AS new_amt,
       CASE WHEN a.order_id IS NULL THEN 'MISSING_IN_OLD'
            WHEN b.order_id IS NULL THEN 'MISSING_IN_NEW'
            WHEN a.total_amount <> b.total_amount THEN 'AMOUNT_MISMATCH'
            ELSE 'OK' END AS verdict
FROM   orders_legacy a
FULL   OUTER JOIN orders b ON b.order_id = a.order_id
WHERE  a.order_id IS NULL
   OR  b.order_id IS NULL
   OR  a.total_amount IS DISTINCT FROM b.total_amount;
```

Ye **migration verification** ka ek-line jawab hai — Section 38 mein aur detail.

## 10.7 CROSS JOIN

Har row × har row. Koi `ON` nahi.

```sql
SELECT p.name AS project, s.name AS supplier
FROM   projects p CROSS JOIN suppliers s;
```

4 projects × 6 suppliers = **24 rows**. Har possible combination.

Kab kaam aata hai: **complete matrix banana** (har project × har month ka report grid, jahan missing combinations ke liye 0 dikhana hai).

```sql
-- Har project ka har status — jahan data nahi wahan 0
SELECT p.name, st.status, COALESCE(COUNT(o.order_id), 0) AS cnt
FROM   projects p
CROSS  JOIN (VALUES ('DRAFT'),('APPROVED'),('DELIVERED'),('CANCELLED')) AS st(status)
LEFT   JOIN orders o
       ON o.project_id = p.project_id AND o.status = st.status AND o.is_deleted=false
GROUP  BY p.name, st.status
ORDER  BY p.name, st.status;
```

Ye 16 rows dega (4 projects × 4 statuses), jisme zyadatar 0 honge. Bina CROSS JOIN ke report mein rows **missing** hoti, 0 nahi dikhta.

**Danger:** accidental cross join. Agar `ON` clause bhool gaye ya galat likha, toh 1 lakh × 1 lakh = 10 billion rows. Query hang, DB down. `EXPLAIN` mein `Nested Loop` bina join condition ke dikhe toh alarm bajao.

## 10.8 SELF JOIN

Table apne aap se join. `users.manager_id → users.user_id` ke liye.

```sql
SELECT e.name AS employee, e.role, m.name AS manager
FROM   users e
LEFT   JOIN users m ON m.user_id = e.manager_id
ORDER  BY e.user_id;
```

| employee | role | manager |
|---|---|---|
| Anil Mehta | ADMIN | NULL |
| Ritik Chaturvedi | PM | Anil Mehta |
| Sunita Rao | PM | Anil Mehta |
| Kabir Singh | SITE_ENGINEER | Ritik Chaturvedi |
| Meera Nair | SITE_ENGINEER | Ritik Chaturvedi |
| Farhan Ali | SITE_ENGINEER | Sunita Rao |

Agar `INNER JOIN` karte toh Anil Mehta gayab ho jaata (uska koi manager nahi). Ye LEFT vs INNER ka perfect demo hai.

**Doosra self-join use — same table mein comparison:**

```sql
-- Wo pairs of orders jo same project mein hain aur same din ke hain
SELECT a.order_id AS o1, b.order_id AS o2, a.project_id, a.order_date
FROM   orders a
JOIN   orders b ON b.project_id = a.project_id
               AND b.order_date = a.order_date
               AND b.order_id > a.order_id;   -- ← duplicate pairs se bachne ke liye
```

`b.order_id > a.order_id` ka trick yaad rakho — warna har pair do baar aayega aur har row apne se bhi join ho jayegi.

## 10.9 Anti-join — "A jinka B mein match nahi"

Ye **sabse zyada poocha jaane wala JOIN pattern** hai. Teen tareeke:

**(a) LEFT JOIN + IS NULL:**

```sql
SELECT o.order_id, o.total_amount
FROM   orders o
LEFT   JOIN order_items i ON i.order_id = o.order_id
WHERE  i.item_id IS NULL;
```

| order_id | total_amount |
|---|---|
| 7 | 0.00 |
| 12 | 45000.00 |

**(b) NOT EXISTS:**

```sql
SELECT o.order_id, o.total_amount
FROM   orders o
WHERE  NOT EXISTS (SELECT 1 FROM order_items i WHERE i.order_id = o.order_id);
```

**(c) NOT IN — ⚠️ NULL-unsafe:**

```sql
SELECT order_id FROM orders
WHERE order_id NOT IN (SELECT order_id FROM order_items);
```

Agar `order_items.order_id` nullable hota aur usme ek NULL hota, toh ye **zero rows** deta. Isliye `NOT EXISTS` prefer karo.

**Doosra example — suppliers jinka koi order nahi:**

```sql
SELECT s.supplier_id, s.name
FROM   suppliers s
LEFT   JOIN orders o ON o.supplier_id = s.supplier_id
WHERE  o.order_id IS NULL;
```

| supplier_id | name |
|---|---|
| 6 | Ajmer Aggregates |

## 10.10 The `ON` vs `WHERE` trap in OUTER JOINs

Ye interview mein **bahut** poocha jaata hai aur real bug bhi hai.

```sql
-- (A) condition ON mein
SELECT o.order_id, i.item_id
FROM   orders o
LEFT   JOIN order_items i ON i.order_id = o.order_id AND i.material = 'Sand';
```

Saare 12 orders aayenge; jinke paas 'Sand' item nahi, unke saamne `NULL`.

| order_id | item_id |
|---|---|
| 1 | 2 |
| 2 | NULL |
| 3 | NULL |
| … | … |
| 10 | 14 |
| … | NULL |

```sql
-- (B) same condition WHERE mein
SELECT o.order_id, i.item_id
FROM   orders o
LEFT   JOIN order_items i ON i.order_id = o.order_id
WHERE  i.material = 'Sand';
```

Sirf 2 rows! Kyunki `WHERE` join ke **baad** chalta hai, aur `NULL = 'Sand'` false hai — saari unmatched rows filter ho gayi. **LEFT JOIN effectively INNER JOIN ban gaya.**

| order_id | item_id |
|---|---|
| 1 | 2 |
| 10 | 14 |

**Rule:** OUTER JOIN mein, **right table ki condition `ON` mein** daalo. **Left (driving) table ki condition `WHERE` mein** daalo.

> **[REAL]** Merlin mein har collection `orgId` se scoped hai. Agar `$lookup` (Mongo ka join) ke baad `$match` par `orgId` filter lagaya jaye — instead of lookup ke `pipeline` ke andar — toh performance bhi kharaab hoti hai aur outer-join semantics bhi badal sakti hain. Same principle: **filter ko join ke andar/pehle le jao**.

> **Interview answer (JOINs — one consolidated answer):**
> "INNER JOIN returns only matching rows from both sides. LEFT JOIN keeps every row from the left table and fills NULLs where the right has no match; RIGHT is the mirror, and FULL OUTER keeps unmatched rows from both sides. CROSS JOIN is the Cartesian product, which I use deliberately to build a complete matrix so a report shows zeros instead of missing rows. SELF JOIN is joining a table to itself, typically for hierarchies like employee-to-manager.
> Two things I watch for as a tester. First, the anti-join pattern: LEFT JOIN plus `WHERE right.id IS NULL` finds rows in A with no match in B — that's how I find orphaned or missing records, and I'd write it as NOT EXISTS in production because NOT IN breaks silently if the subquery returns a NULL. Second, in an outer join, putting a condition on the right table in the WHERE clause instead of the ON clause turns the outer join back into an inner join, because the NULL rows fail the WHERE predicate. That one has caused real missing-data bugs — the dashboard count and the list count disagree."

> **Cross-question: "`JOIN` ka default type kya hai?"**
> `INNER`. `JOIN` = `INNER JOIN`. Aise hi `LEFT JOIN` = `LEFT OUTER JOIN`.

> **Cross-question: "Ek query mein 5 tables join karne se pehle kya sochoge?"**
> (1) Kya har join condition indexed column par hai? (2) Kya koi one-to-many join fan-out kar raha hai — agar haan toh aggregate pehle. (3) Kya sab tables sach mein chahiye ya kuch sirf `EXISTS` check ke liye hain? (4) `EXPLAIN` dekh kar join order aur estimated vs actual rows compare karo — badi galti wahin dikhegi.

> **Cross-question: "Nested loop, hash join, merge join — kya hai?"**
> Physical join algorithms. **Nested loop** — chhoti outer table, indexed inner; small results ke liye best. **Hash join** — ek side ka hash table banata hai; bade unsorted sets ke liye best, equality joins only. **Merge join** — dono side sorted ho toh, ya index se sorted mile toh. QA ko itna hi jaanna kaafi hai ki `EXPLAIN` mein bade table pe **Nested Loop + Seq Scan** dikhna aksar missing index ka signal hai.

---

# 11 — Subqueries

## 11.1 Kya hai

Ek query ke andar dusri query. Char flavours:

| Type | Kya return karta hai | Kahan use hota hai |
|---|---|---|
| **Scalar** | ek single value (1 row, 1 col) | `SELECT`, `WHERE`, `HAVING` |
| **IN / ANY / ALL** | ek column ki list | `WHERE x IN (…)` |
| **EXISTS** | bas TRUE/FALSE | `WHERE EXISTS (…)` |
| **Derived table** | poori table | `FROM (…) AS t` |
| **Correlated** | outer query ke har row ke liye alag chalta hai | kahin bhi |

## 11.2 Scalar subquery

```sql
SELECT order_id, total_amount,
       (SELECT AVG(total_amount) FROM orders WHERE is_deleted=false) AS avg_all,
       total_amount - (SELECT AVG(total_amount) FROM orders WHERE is_deleted=false) AS diff
FROM   orders
WHERE  is_deleted = false AND total_amount > (SELECT AVG(total_amount) FROM orders WHERE is_deleted=false)
ORDER  BY total_amount DESC;
```

Average = 198181.82. Us se upar wale orders:

| order_id | total_amount | avg_all | diff |
|---|---|---|---|
| 9 | 500000.00 | 198181.82 | 301818.18 |
| 2 | 450000.00 | 198181.82 | 251818.18 |
| 3 | 300000.00 | 198181.82 | 101818.18 |
| 10 | 250000.00 | 198181.82 | 51818.18 |
| 6 | 200000.00 | 198181.82 | 1818.18 |

**Khatra:** agar scalar subquery **ek se zyada row** return kar de toh runtime error aata hai — "more than one row returned by a subquery used as an expression".

> **[REAL]** Ye exactly wahi bug pattern hai jo tumhare Merlin codebase mein tha: `findByOrgAndEmailAddress` ka return type `Optional<Contact>` tha, lekin `(org, emailAddress)` par koi unique constraint nahi tha. Jis din ek org mein do contacts same email se ban gaye, Spring Data ne `IncorrectResultSizeDataAccessException` phenka, jo API layer mein **HTTP 400** ban gaya — jabki asli problem 400 (bad input) thi hi nahi, wo **data integrity** ki thi. SQL mein ye "subquery returned more than one row" ban ke aata. Sabak same: **agar code ek hi row maan raha hai, toh database mein us par unique constraint hona chahiye.** Ye story interview mein zaroor sunana — ye tumhe senior dikhati hai.

## 11.3 IN subquery

```sql
-- Un projects ke naam jinke paas kam se kam ek DELIVERED order hai
SELECT project_id, name
FROM   projects
WHERE  project_id IN (SELECT project_id FROM orders
                      WHERE status='DELIVERED' AND is_deleted=false);
```

| project_id | name |
|---|---|
| 1 | Skyline Towers |
| 2 | Metro Depot |
| 3 | Green Villa |

## 11.4 EXISTS (aur NOT EXISTS)

`EXISTS` sirf poochta hai **"koi row hai?"** — value se matlab nahi. Isliye `SELECT 1` likhte hain (`SELECT *` bhi chalega, koi farak nahi padta).

```sql
SELECT p.project_id, p.name
FROM   projects p
WHERE  EXISTS (SELECT 1 FROM orders o
               WHERE o.project_id = p.project_id      -- ← correlation
                 AND o.status = 'DELIVERED'
                 AND o.is_deleted = false);
```

Same result as above.

```sql
-- Ulta: projects jinka koi DELIVERED order nahi
SELECT p.project_id, p.name
FROM   projects p
WHERE  NOT EXISTS (SELECT 1 FROM orders o
                   WHERE o.project_id = p.project_id AND o.status='DELIVERED');
```

| project_id | name |
|---|---|
| 4 | Riverfront Mall |

## 11.5 Correlated subquery

Inner query outer query ke column par depend karti hai — isliye **har outer row ke liye** logically dobara chalti hai.

```sql
-- Har order ke saath uske items ki count
SELECT o.order_id, o.total_amount,
       (SELECT COUNT(*) FROM order_items i WHERE i.order_id = o.order_id) AS item_count
FROM   orders o
WHERE  o.is_deleted = false
ORDER  BY o.order_id;
```

| order_id | total_amount | item_count |
|---|---|---|
| 1 | 120000.00 | 2 |
| 2 | 450000.00 | 2 |
| 3 | 300000.00 | 2 |
| 4 | 75000.00 | 1 |
| 5 | 60000.00 | 1 |
| 6 | 200000.00 | 1 |
| 7 | 0.00 | 0 |
| 8 | 180000.00 | 1 |
| 9 | 500000.00 | 2 |
| 10 | 250000.00 | 2 |
| 12 | 45000.00 | **0** |

Order 12 par 0 items par 45000 amount — **bug confirmed**.

**Correlated subquery ka classic use — "har group ka max":**

```sql
-- Har project ka sabse mehenga order
SELECT o.project_id, o.order_id, o.total_amount
FROM   orders o
WHERE  o.is_deleted = false
  AND  o.total_amount = (SELECT MAX(o2.total_amount) FROM orders o2
                         WHERE o2.project_id = o.project_id AND o2.is_deleted=false);
```

| project_id | order_id | total_amount |
|---|---|---|
| 1 | 9 | 500000.00 |
| 2 | 3 | 300000.00 |
| 3 | 6 | 200000.00 |
| 4 | 8 | 180000.00 |

**Note:** agar tie ho toh ye **dono** rows dega. Window function wale approach mein `ROW_NUMBER` ek hi degi, `RANK` dono degi — ye difference interviewer poochta hai.

## 11.6 Derived table (FROM ke andar subquery)

```sql
SELECT t.project_id, t.item_total
FROM   (SELECT o.project_id, SUM(i.quantity*i.unit_price) AS item_total
        FROM   orders o JOIN order_items i ON i.order_id=o.order_id
        WHERE  o.is_deleted=false
        GROUP  BY o.project_id) t
WHERE  t.item_total > 200000;
```

| project_id | item_total |
|---|---|
| 1 | 1130000.00 |
| 2 | 375000.00 |

(Postgres mein derived table ko alias dena **mandatory** hai — `AS t`.)

## 11.7 EXISTS vs IN — performance

| | `IN (subquery)` | `EXISTS (subquery)` |
|---|---|---|
| Semantics | subquery ki poori list se compare | pehla match milte hi ruk jaata hai (short-circuit) |
| NULL behaviour | `NOT IN` NULL par **poora result kha jaata hai** | `NOT EXISTS` NULL-safe |
| Subquery bada | list materialize karni pad sakti hai | short-circuit, aksar behtar |
| Outer bada | aksar behtar | har row ke liye probe (agar optimizer semi-join na banaye) |
| Modern optimizers | dono ko **semi-join** mein rewrite kar dete hain | same |

**Practical rule jo interview mein bolna hai:**
- Correctness ke liye: `NOT EXISTS` > `NOT IN`. Hamesha.
- Performance ke liye: modern Postgres/Oracle/SQL Server dono ko same semi-join plan dete hain, toh farak aksar zero hai. **Measure karo, guess mat karo.**

> **Interview answer:**
> "A subquery is a query nested inside another. The four kinds I use are scalar — returns one value, and it'll throw a runtime error if the data ever returns more than one row, which is why it needs a matching unique constraint; IN — compares against a list; EXISTS — a boolean existence check that short-circuits on the first match; and correlated subqueries, which reference the outer row.
> On EXISTS versus IN: functionally the big difference is NULL handling. NOT IN with a subquery that can return NULL returns zero rows overall, silently — that's a genuine production trap. NOT EXISTS is NULL-safe, so I default to it. Performance-wise, modern optimisers usually rewrite both into a semi-join and the plans come out the same, so I don't make blanket claims — I check EXPLAIN."

> **Cross-question: "`IN` vs `JOIN` — kaun better?"**
> `JOIN` **fan-out** kar sakta hai (agar right side mein duplicate keys hain toh left rows multiply ho jayengi); `IN`/`EXISTS` semi-join hai, kabhi fan-out nahi karta. Agar tumhe sirf filter chahiye, right table se koi column nahi chahiye → `EXISTS` use karo. Agar right table ke columns chahiye → `JOIN`.

---

# 12 — CTEs (WITH) + Recursive CTE

## 12.1 Kya hai

CTE = **Common Table Expression** = "named temporary result set" jo sirf us ek query ke liye zinda rehta hai. `WITH` se banta hai.

```sql
WITH name AS ( SELECT … ),
     name2 AS ( SELECT … FROM name … )   -- pichhle CTE ko use kar sakte ho
SELECT … FROM name2;
```

## 12.2 Kyun use karein

- **Readability** — nested subqueries ki jagah top-to-bottom steps.
- **Reuse** — ek hi CTE ko query mein kai baar reference kar sakte ho.
- **Debuggability (QA ke liye sabse bada fayda)** — tum CTE ek-ek karke run karke dekh sakte ho ki intermediate result kya hai. Report ka total galat hai? CTE-by-CTE chal ke pakdo ki kis step par number bigda.

## 12.3 Worked example — step by step

```sql
WITH active_orders AS (
    SELECT * FROM orders WHERE is_deleted = false AND status <> 'CANCELLED'
),
order_totals AS (
    SELECT ao.order_id, ao.project_id,
           COALESCE(SUM(i.quantity * i.unit_price), 0) AS computed_total,
           ao.total_amount AS stored_total
    FROM   active_orders ao
    LEFT   JOIN order_items i ON i.order_id = ao.order_id
    GROUP  BY ao.order_id, ao.project_id, ao.total_amount
)
SELECT order_id, project_id, stored_total, computed_total,
       stored_total - computed_total AS drift
FROM   order_totals
WHERE  stored_total <> computed_total
ORDER  BY order_id;
```

**Result — data integrity violation pakda gaya:**

| order_id | project_id | stored_total | computed_total | drift |
|---|---|---|---|---|
| 12 | 1 | 45000.00 | 0.00 | 45000.00 |

**Ye ek real QA query hai.** Ise nightly job mein daal do aur alert lagao — Section 37 mein detail.

## 12.4 Recursive CTE

Apne aap ko reference karne wala CTE. Hierarchies (org chart, BOM, category tree), graph traversal, aur series generation ke liye.

**Structure:**

```sql
WITH RECURSIVE cte AS (
    -- 1. ANCHOR: starting rows
    SELECT … FROM base WHERE <start condition>
    UNION ALL
    -- 2. RECURSIVE: cte ko reference karta hai
    SELECT … FROM base JOIN cte ON …
)
SELECT * FROM cte;
```

Engine anchor chalata hai, phir recursive part ko baar-baar chalata hai jab tak nayi rows aana band na ho jaye.

**Example — Anil Mehta ke neeche poori reporting chain:**

```sql
WITH RECURSIVE org_chart AS (
    -- anchor: top boss
    SELECT user_id, name, manager_id, 1 AS level, name::text AS path
    FROM   users
    WHERE  manager_id IS NULL

    UNION ALL

    -- recursive: har us banda jiska manager already chart mein hai
    SELECT u.user_id, u.name, u.manager_id, oc.level + 1,
           oc.path || ' > ' || u.name
    FROM   users u
    JOIN   org_chart oc ON oc.user_id = u.manager_id
)
SELECT level, user_id, name, path
FROM   org_chart
ORDER  BY path;
```

**Result:**

| level | user_id | name | path |
|---|---|---|---|
| 1 | 1 | Anil Mehta | Anil Mehta |
| 2 | 2 | Ritik Chaturvedi | Anil Mehta > Ritik Chaturvedi |
| 3 | 4 | Kabir Singh | Anil Mehta > Ritik Chaturvedi > Kabir Singh |
| 3 | 5 | Meera Nair | Anil Mehta > Ritik Chaturvedi > Meera Nair |
| 2 | 3 | Sunita Rao | Anil Mehta > Sunita Rao |
| 3 | 6 | Farhan Ali | Anil Mehta > Sunita Rao > Farhan Ali |

**Iterations kaise chale:**

```
Iteration 0 (anchor) : [Anil(L1)]
Iteration 1          : [Ritik(L2), Sunita(L2)]     ← manager = Anil
Iteration 2          : [Kabir(L3), Meera(L3), Farhan(L3)]
Iteration 3          : []  ← khali, ruk gaya
```

**Doosra example — date series generate karna** (reports mein missing months bharne ke liye):

```sql
WITH RECURSIVE months AS (
    SELECT DATE '2025-01-01' AS m
    UNION ALL
    SELECT (m + INTERVAL '1 month')::date FROM months WHERE m < DATE '2025-05-01'
)
SELECT m FROM months;
```

| m |
|---|
| 2025-01-01 |
| 2025-02-01 |
| 2025-03-01 |
| 2025-04-01 |
| 2025-05-01 |

(Postgres mein `generate_series('2025-01-01','2025-05-01', '1 month')` simpler hai, par recursive CTE portable hai aur interview mein wahi poochte hain.)

**Infinite loop se bachna:** agar data mein cycle ho (A ka manager B, B ka manager A) toh recursive CTE kabhi nahi rukega. Bachav: `path` array rakho aur `WHERE NOT (u.user_id = ANY(oc.path_ids))` lagao, ya `level < 20` ka depth cap.

> **Interview answer:**
> "A CTE is a named temporary result defined with WITH, scoped to a single statement. I reach for them mainly for readability and debuggability: a five-step report reads top to bottom instead of as four levels of nested subqueries, and when a total is wrong I can run each CTE in isolation and find exactly which step introduces the bad number. A recursive CTE has an anchor member and a recursive member joined by UNION ALL; the engine keeps re-running the recursive part until it produces no new rows. I use it for hierarchies — an employee-to-manager chain, or a bill-of-materials — and for generating a date series so a report shows zero-value months instead of missing rows. The thing to guard against is a cycle in the data, which makes it loop forever, so I add either a visited-path check or a depth limit."

> **Cross-question: "CTE ka performance impact?"**
> Postgres 11 se pehle CTE ek **optimization fence** tha — hamesha alag se materialize hota tha, predicates push nahi hote the. PG12+ mein wo **inline** ho jaate hain agar sirf ek baar reference ho aur side-effect na ho; force karne ke liye `MATERIALIZED` / `NOT MATERIALIZED` keywords hain. SQL Server aur Oracle CTE ko hamesha inline karte hain. Toh "CTE slow hota hai" ab blanket sach nahi hai — version-dependent hai.

> **Cross-question: "CTE vs temp table vs view?"**
> CTE — ek statement ke liye, memory mein. Temp table — session ke liye, indexes bhi bana sakte ho, badi intermediate results ke liye behtar. View — permanently stored query definition, har baar chalta hai.

---

# 13 — CASE WHEN

## 13.1 Kya hai

SQL ka if-else.

```sql
CASE WHEN condition1 THEN result1
     WHEN condition2 THEN result2
     ELSE default_result
END
```

`ELSE` na do toh unmatched rows par `NULL` aata hai — ye ek chupa hua bug source hai.

## 13.2 Worked example — bucketing

```sql
SELECT order_id, total_amount,
       CASE WHEN total_amount = 0            THEN 'EMPTY'
            WHEN total_amount < 100000       THEN 'SMALL'
            WHEN total_amount < 300000       THEN 'MEDIUM'
            ELSE                                  'LARGE'
       END AS size_bucket
FROM   orders
WHERE  is_deleted = false
ORDER  BY total_amount;
```

| order_id | total_amount | size_bucket |
|---|---|---|
| 7 | 0.00 | EMPTY |
| 12 | 45000.00 | SMALL |
| 5 | 60000.00 | SMALL |
| 4 | 75000.00 | SMALL |
| 1 | 120000.00 | MEDIUM |
| 8 | 180000.00 | MEDIUM |
| 6 | 200000.00 | MEDIUM |
| 10 | 250000.00 | MEDIUM |
| 3 | 300000.00 | LARGE |
| 2 | 450000.00 | LARGE |
| 9 | 500000.00 | LARGE |

Dhyan do: `WHEN` **upar se neeche** evaluate hote hain, pehla TRUE jeet jaata hai. Isliye order matter karta hai. 300000 `< 300000` nahi hai isliye LARGE — **boundary!** Ye exactly wo jagah hai jahan QA ko boundary value test likhna chahiye.

## 13.3 CASE inside aggregate — pivot / conditional count

Ye **bahut powerful** pattern hai aur interview mein impress karta hai.

```sql
SELECT p.name AS project,
       COUNT(*)                                                     AS total,
       COUNT(*) FILTER (WHERE o.status = 'DRAFT')                    AS draft,
       SUM(CASE WHEN o.status = 'APPROVED'  THEN 1 ELSE 0 END)       AS approved,
       SUM(CASE WHEN o.status = 'DELIVERED' THEN 1 ELSE 0 END)       AS delivered,
       SUM(CASE WHEN o.status = 'CANCELLED' THEN 1 ELSE 0 END)       AS cancelled,
       SUM(CASE WHEN o.status = 'DELIVERED' THEN o.total_amount ELSE 0 END) AS delivered_value
FROM   projects p
LEFT   JOIN orders o ON o.project_id = p.project_id AND o.is_deleted = false
GROUP  BY p.project_id, p.name
ORDER  BY p.project_id;
```

**Result — rows ko columns mein pivot kar diya:**

| project | total | draft | approved | delivered | cancelled | delivered_value |
|---|---|---|---|---|---|---|
| Skyline Towers | 5 | 1 | 0 | 3 | 1 | 1070000.00 |
| Metro Depot | 4 | 1 | 1 | 2 | 0 | 325000.00 |
| Green Villa | 1 | 0 | 0 | 1 | 0 | 200000.00 |
| Riverfront Mall | 1 | 0 | 1 | 0 | 0 | 0.00 |

`COUNT(*) FILTER (WHERE …)` Postgres ka cleaner syntax hai; `SUM(CASE …)` har database mein chalta hai. Dono ka matlab same.

## 13.4 CASE in ORDER BY — custom sort

```sql
SELECT order_id, status
FROM   orders WHERE is_deleted = false
ORDER  BY CASE status WHEN 'DRAFT' THEN 1 WHEN 'APPROVED' THEN 2
                      WHEN 'DELIVERED' THEN 3 ELSE 4 END,
          order_id;
```

Workflow order mein sort — alphabetical nahi.

## 13.5 CASE in UPDATE

```sql
UPDATE invoices i
SET    status = CASE
         WHEN paid.total IS NULL       THEN 'PENDING'
         WHEN paid.total >= i.amount   THEN 'PAID'
         ELSE                               'PARTIALLY_PAID'
       END
FROM   (SELECT invoice_id, SUM(amount) AS total FROM payments GROUP BY invoice_id) paid
WHERE  paid.invoice_id = i.invoice_id;
```

> **Interview answer:**
> "CASE WHEN is SQL's conditional expression. Branches are evaluated top to bottom and the first true one wins, so the order of the WHEN clauses is part of the logic — and if there's no ELSE, unmatched rows silently become NULL, which I always check for. The pattern I use most is CASE inside an aggregate — SUM of CASE WHEN status equals X THEN 1 ELSE 0 — which pivots status rows into status columns in one pass over the table. That's how you build a status-breakdown dashboard without four separate queries. Postgres has a cleaner form, COUNT star FILTER WHERE, that does the same thing."

> **Cross-question: "`CASE x WHEN 1 THEN…` aur `CASE WHEN x=1 THEN…` mein farak?"**
> Pehla **simple CASE** — sirf equality compare karta hai, aur `NULL` ko match nahi karega (kyunki `x = NULL` UNKNOWN hai). Doosra **searched CASE** — koi bhi boolean condition le sakta hai, jaise `x IS NULL`, ranges, `AND`/`OR`. Searched CASE zyada flexible hai.

---

# 14 — Window functions

## 14.1 Kya hai

Window function har row ke liye ek value nikaalta hai **doosri rows ko dekh kar** — lekin rows ko **collapse nahi karta**. Yahi `GROUP BY` se sabse bada farak hai.

```
GROUP BY:   5 rows  →  1 row  (details gayab)
WINDOW:     5 rows  →  5 rows (+ ek extra computed column)
```

**Syntax:**

```sql
function() OVER (
    PARTITION BY col     -- groups mein baanto (optional)
    ORDER BY col         -- group ke andar order (kuch functions ke liye zaroori)
    ROWS/RANGE …         -- frame (optional)
)
```

## 14.2 ROW_NUMBER vs RANK vs DENSE_RANK — the classic

Sabse zyada poocha jaane wala window question. Difference sirf **ties** par dikhta hai.

```sql
SELECT user_id, name, salary,
       ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num,
       RANK()       OVER (ORDER BY salary DESC) AS rnk,
       DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rnk
FROM   users;
```

**Result** (Kabir aur Meera dono 60000 — tie):

| user_id | name | salary | row_num | rnk | dense_rnk |
|---|---|---|---|---|---|
| 1 | Anil Mehta | 250000 | 1 | 1 | 1 |
| 3 | Sunita Rao | 110000 | 2 | 2 | 2 |
| 2 | Ritik Chaturvedi | 90000 | 3 | 3 | 3 |
| 6 | Farhan Ali | 75000 | 4 | 4 | 4 |
| 4 | Kabir Singh | 60000 | 5 | **5** | **5** |
| 5 | Meera Nair | 60000 | 6 | **5** | **5** |

Agar ek aur banda 50000 par hota:

| name | salary | row_num | rnk | dense_rnk |
|---|---|---|---|---|
| Kabir | 60000 | 5 | 5 | 5 |
| Meera | 60000 | 6 | 5 | 5 |
| (naya) | 50000 | 7 | **7** | **6** |

**Yaad rakhne ka tareeka:**
- `ROW_NUMBER` — hamesha 1,2,3,4… **kabhi tie nahi**, arbitrary tie-break (isliye deterministic chahiye toh `ORDER BY salary DESC, user_id` likho).
- `RANK` — ties ko same rank, phir **gap chhodta hai** (5,5,7).
- `DENSE_RANK` — ties ko same rank, **gap nahi** (5,5,6).

## 14.3 PARTITION BY

`PARTITION BY` = "har group ke andar alag se counting shuru karo".

```sql
SELECT project_id, order_id, total_amount,
       ROW_NUMBER() OVER (PARTITION BY project_id ORDER BY total_amount DESC) AS rn_in_project,
       SUM(total_amount) OVER (PARTITION BY project_id) AS project_total,
       ROUND(100.0 * total_amount / SUM(total_amount) OVER (PARTITION BY project_id), 1) AS pct_of_project
FROM   orders
WHERE  is_deleted = false
ORDER  BY project_id, rn_in_project;
```

**Result:**

| project_id | order_id | total_amount | rn_in_project | project_total | pct_of_project |
|---|---|---|---|---|---|
| 1 | 9 | 500000.00 | 1 | 1175000.00 | 42.6 |
| 1 | 2 | 450000.00 | 2 | 1175000.00 | 38.3 |
| 1 | 1 | 120000.00 | 3 | 1175000.00 | 10.2 |
| 1 | 5 | 60000.00 | 4 | 1175000.00 | 5.1 |
| 1 | 12 | 45000.00 | 5 | 1175000.00 | 3.8 |
| 2 | 3 | 300000.00 | 1 | 625000.00 | 48.0 |
| 2 | 10 | 250000.00 | 2 | 625000.00 | 40.0 |
| 2 | 4 | 75000.00 | 3 | 625000.00 | 12.0 |
| 2 | 7 | 0.00 | 4 | 625000.00 | 0.0 |
| 3 | 6 | 200000.00 | 1 | 200000.00 | 100.0 |
| 4 | 8 | 180000.00 | 1 | 180000.00 | 100.0 |

**Dekho jo `GROUP BY` se possible nahi tha:** individual row **aur** uska group total, **ek hi row mein**. Ye "% of total" wale reports ka jawab hai.

## 14.4 Top-N per group

```sql
-- Har project ke top 2 orders
WITH ranked AS (
    SELECT project_id, order_id, total_amount,
           ROW_NUMBER() OVER (PARTITION BY project_id ORDER BY total_amount DESC, order_id) AS rn
    FROM   orders WHERE is_deleted = false
)
SELECT project_id, order_id, total_amount
FROM   ranked WHERE rn <= 2
ORDER  BY project_id, rn;
```

| project_id | order_id | total_amount |
|---|---|---|
| 1 | 9 | 500000.00 |
| 1 | 2 | 450000.00 |
| 2 | 3 | 300000.00 |
| 2 | 10 | 250000.00 |
| 3 | 6 | 200000.00 |
| 4 | 8 | 180000.00 |

## 14.5 LAG aur LEAD

`LAG(col, n)` = pichhli row ki value. `LEAD(col, n)` = agli row ki value.

```sql
-- Har order ke saath us project ka pichhla order aur gap
SELECT project_id, order_id, order_date, total_amount,
       LAG(total_amount) OVER (PARTITION BY project_id ORDER BY order_date) AS prev_amount,
       total_amount - LAG(total_amount) OVER (PARTITION BY project_id ORDER BY order_date) AS change,
       order_date - LAG(order_date) OVER (PARTITION BY project_id ORDER BY order_date) AS days_gap
FROM   orders
WHERE  is_deleted = false AND project_id = 1
ORDER  BY order_date;
```

**Result (project 1):**

| project_id | order_id | order_date | total_amount | prev_amount | change | days_gap |
|---|---|---|---|---|---|---|
| 1 | 1 | 2025-01-05 | 120000.00 | NULL | NULL | NULL |
| 1 | 2 | 2025-01-18 | 450000.00 | 120000.00 | 330000.00 | 13 |
| 1 | 5 | 2025-02-27 | 60000.00 | 450000.00 | -390000.00 | 40 |
| 1 | 9 | 2025-04-02 | 500000.00 | 60000.00 | 440000.00 | 34 |
| 1 | 12 | 2025-05-06 | 45000.00 | 500000.00 | -455000.00 | 34 |

Pehli row par `LAG` `NULL` deta hai. Default de sakte ho: `LAG(total_amount, 1, 0)`.

## 14.6 Running total — SUM OVER with frame

```sql
SELECT order_id, order_date, total_amount,
       SUM(total_amount) OVER (ORDER BY order_date, order_id
                               ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM   orders
WHERE  is_deleted = false
ORDER  BY order_date, order_id;
```

| order_id | order_date | total_amount | running_total |
|---|---|---|---|
| 1 | 2025-01-05 | 120000.00 | 120000.00 |
| 2 | 2025-01-18 | 450000.00 | 570000.00 |
| 3 | 2025-02-02 | 300000.00 | 870000.00 |
| 4 | 2025-02-14 | 75000.00 | 945000.00 |
| 5 | 2025-02-27 | 60000.00 | 1005000.00 |
| 6 | 2025-03-03 | 200000.00 | 1205000.00 |
| 7 | 2025-03-11 | 0.00 | 1205000.00 |
| 8 | 2025-03-19 | 180000.00 | 1385000.00 |
| 9 | 2025-04-02 | 500000.00 | 1885000.00 |
| 10 | 2025-04-15 | 250000.00 | 2135000.00 |
| 12 | 2025-05-06 | 45000.00 | 2180000.00 |

**Frame clause — ROWS vs RANGE (interviewers ise pasand karte hain):**

| Frame | Matlab |
|---|---|
| `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` | shuru se ab tak, **physical rows** |
| `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` | **default** jab `ORDER BY` ho — ties ko ek saath treat karta hai |
| `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW` | moving window of 3 rows (moving average) |
| bina `ORDER BY` | poora partition |

**Trap:** `ORDER BY` ke saath default frame `RANGE … CURRENT ROW` hai, jo **tied rows ko ek hi group** maanta hai. Agar do orders ki same date ho, `RANGE` dono ka sum ek saath dega — running total "chhalang" maar jayega. Isliye running total ke liye **explicitly `ROWS`** likho.

**Moving average:**

```sql
SELECT order_id, order_date, total_amount,
       ROUND(AVG(total_amount) OVER (ORDER BY order_date
             ROWS BETWEEN 2 PRECEDING AND CURRENT ROW), 2) AS mov_avg_3
FROM   orders WHERE is_deleted = false ORDER BY order_date;
```

## 14.7 "2nd highest" — teen tareeke (classic interview question)

**Target: 2nd highest salary = 110000 (Sunita Rao).**

**Way 1 — DENSE_RANK (best, ties handle karta hai):**

```sql
WITH r AS (
    SELECT name, salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS dr FROM users
)
SELECT name, salary FROM r WHERE dr = 2;
```

| name | salary |
|---|---|
| Sunita Rao | 110000.00 |

**Way 2 — Subquery with MAX:**

```sql
SELECT MAX(salary) AS second_highest
FROM   users
WHERE  salary < (SELECT MAX(salary) FROM users);
```

| second_highest |
|---|
| 110000.00 |

**Way 3 — OFFSET on DISTINCT:**

```sql
SELECT DISTINCT salary FROM users ORDER BY salary DESC LIMIT 1 OFFSET 1;
```

| salary |
|---|
| 110000.00 |

**Kaunsa kab:**

| Approach | Ties | Nth generalize | Notes |
|---|---|---|---|
| DENSE_RANK | saare tied employees deta hai | `dr = N` — trivial | best answer |
| MAX subquery | distinct value deta hai | N=3 ke liye ugly nesting | simple, cheap |
| LIMIT/OFFSET | `DISTINCT` chahiye warna galat | `OFFSET N-1` | agar 2nd highest exist na kare toh **zero rows**, NULL nahi |

**Ye zaroor bolo:** *"I'd also ask what 'second highest' means when there are ties — is it the second distinct value, or the second person? Those are different queries. DENSE_RANK gives you the second distinct value and every person at it; ROW_NUMBER gives you exactly one person."* **Yeh line senior candidate ki pehchaan hai.**

## 14.8 FIRST_VALUE / LAST_VALUE / NTH_VALUE

```sql
SELECT project_id, order_id, order_date, total_amount,
       FIRST_VALUE(order_id) OVER (PARTITION BY project_id ORDER BY order_date) AS first_order,
       LAST_VALUE(order_id)  OVER (PARTITION BY project_id ORDER BY order_date
                                   ROWS BETWEEN UNBOUNDED PRECEDING
                                            AND UNBOUNDED FOLLOWING) AS last_order
FROM   orders WHERE is_deleted=false AND project_id=1 ORDER BY order_date;
```

| project_id | order_id | order_date | first_order | last_order |
|---|---|---|---|---|
| 1 | 1 | 2025-01-05 | 1 | 12 |
| 1 | 2 | 2025-01-18 | 1 | 12 |
| 1 | 5 | 2025-02-27 | 1 | 12 |
| 1 | 9 | 2025-04-02 | 1 | 12 |
| 1 | 12 | 2025-05-06 | 1 | 12 |

**Bahut bada trap:** `LAST_VALUE` bina explicit frame ke **current row hi** deta hai (default frame `UNBOUNDED PRECEDING AND CURRENT ROW` hai). `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` likhna **zaroori** hai. Ye interview mein poochha jaata hai.

## 14.9 NTILE — bucketing

```sql
SELECT name, salary, NTILE(3) OVER (ORDER BY salary DESC) AS tier
FROM   users;
```

| name | salary | tier |
|---|---|---|
| Anil Mehta | 250000 | 1 |
| Sunita Rao | 110000 | 1 |
| Ritik Chaturvedi | 90000 | 2 |
| Farhan Ali | 75000 | 2 |
| Kabir Singh | 60000 | 3 |
| Meera Nair | 60000 | 3 |

> **Interview answer:**
> "Window functions compute a value for each row based on a set of related rows, but unlike GROUP BY they don't collapse the rows — so you get the detail row and its aggregate side by side. That's how you do 'this order's share of the project total' in a single pass.
> The three ranking functions differ only on ties: ROW_NUMBER always gives 1, 2, 3 with no ties, so I always add a unique tie-breaker to the ORDER BY to make it deterministic. RANK gives tied rows the same rank and then skips — 1, 2, 2, 4. DENSE_RANK gives the same rank without skipping — 1, 2, 2, 3.
> Two gotchas I've hit. Window functions are evaluated with SELECT, so you can't filter on them in WHERE — you wrap the query in a CTE and filter outside. And with an ORDER BY, the default frame is RANGE, not ROWS, which lumps ties together — so for a running total I write ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW explicitly. Same reason LAST_VALUE returns the current row unless you widen the frame to UNBOUNDED FOLLOWING."

> **Cross-question: "Window function ke saath GROUP BY use kar sakte ho?"**
> Haan, aur window `GROUP BY` ke **baad** chalta hai — matlab wo grouped rows par operate karega, original rows par nahi.
> ```sql
> SELECT project_id, SUM(total_amount) AS t,
>        RANK() OVER (ORDER BY SUM(total_amount) DESC) AS rnk
> FROM orders WHERE is_deleted=false GROUP BY project_id;
> ```
> Result: project 1 → rank 1, project 2 → rank 2, project 3 → 3, project 4 → 4.

---

# 15 — Set operations

## 15.1 Kya hai

Do query results ko **vertically** jodna (JOIN horizontally jodta hai).

**Rule:** dono queries mein **same number of columns**, **compatible data types**, **same order** hone chahiye.

```
   Query A rows          Query B rows
   ┌───┬───┬───┐        ┌───┬───┬───┐
   │ 1 │ 2 │ 3 │        │ 3 │ 4 │ 5 │
   └───┴───┴───┘        └───┴───┴───┘

UNION      → 1 2 3 4 5    (duplicates hataye, SORTED/hashed)
UNION ALL  → 1 2 3 3 4 5  (sab kuch, tez)
INTERSECT  → 3            (dono mein)
EXCEPT     → 1 2          (A mein hai, B mein nahi)
```

## 15.2 UNION vs UNION ALL

```sql
SELECT city FROM suppliers
UNION
SELECT city FROM projects
ORDER BY city;
```

| city |
|---|
| Ajmer |
| Delhi |
| Jaipur |
| Mumbai |

```sql
SELECT city FROM suppliers
UNION ALL
SELECT city FROM projects;
```

| city |
|---|
| Jaipur, Delhi, Jaipur, Mumbai, Delhi, Ajmer (suppliers — 6 rows) |
| Jaipur, Delhi, Jaipur, Mumbai (projects — 4 rows) |

= **10 rows**, duplicates ke saath.

**Difference jo interview mein bolna hai:** `UNION` ko duplicates hatane ke liye **saare rows sort ya hash karne padte hain** — memory + CPU. `UNION ALL` bas concatenate karta hai. **Agar tumhe pata hai ki duplicates ho hi nahi sakte, `UNION ALL` use karo.** Bade datasets mein ye 2–10x farak la sakta hai.

## 15.3 INTERSECT

```sql
-- Wo cities jahan supplier bhi hai aur project bhi
SELECT city FROM suppliers
INTERSECT
SELECT city FROM projects;
```

| city |
|---|
| Jaipur |
| Delhi |
| Mumbai |

## 15.4 EXCEPT (MySQL mein nahi hai; Oracle mein `MINUS`)

```sql
-- Wo cities jahan supplier hai par koi project nahi
SELECT city FROM suppliers
EXCEPT
SELECT city FROM projects;
```

| city |
|---|
| Ajmer |

```sql
-- Ulta: project hai par koi supplier nahi
SELECT city FROM projects EXCEPT SELECT city FROM suppliers;
```

Zero rows.

## 15.5 QA ka killer use — table diffing

Migration ya refactor ke baad **do queries ka result bilkul same hai ya nahi** — ye **ek** query mein prove karo:

```sql
-- Agar ye zero rows de, toh dono results identical hain
(SELECT order_id, project_id, total_amount FROM orders_old
 EXCEPT
 SELECT order_id, project_id, total_amount FROM orders_new)
UNION ALL
(SELECT order_id, project_id, total_amount FROM orders_new
 EXCEPT
 SELECT order_id, project_id, total_amount FROM orders_old);
```

Isko **"symmetric difference"** kehte hain. Dono direction check karna zaroori hai — sirf ek direction se "missing rows" pata chalega par "extra rows" nahi.

Ise Python test mein daal do:

```python
def test_migration_produces_identical_rows(db):
    rows = db.query(SYMMETRIC_DIFF_SQL)
    assert rows == [], f"Migration drift on {len(rows)} rows: {rows[:10]}"
```

**Ye ek asli senior-level automation hai** jo tum interview mein bata sakte ho.

**Note:** `EXCEPT`/`INTERSECT` set semantics use karte hain (duplicates collapse ho jaate hain) — agar duplicate rows bhi pakadni hain toh `EXCEPT ALL` use karo (Postgres support karta hai).

> **Interview answer:**
> "UNION combines two result sets and removes duplicates; UNION ALL just concatenates them. The removal isn't free — the database has to sort or hash everything to de-duplicate — so if I know the inputs are already disjoint I use UNION ALL, and on large sets that's a meaningful speed-up. INTERSECT returns rows present in both, EXCEPT returns rows in the first but not the second.
> The way I actually use these as a tester is for diffing. After a migration or a query refactor I run a symmetric difference — old EXCEPT new, UNION ALL, new EXCEPT old. If that returns zero rows, the two result sets are provably identical, in both directions. I turn that into an automated assertion in the migration test rather than eyeballing counts, because equal row counts don't prove equal rows."

> **Cross-question: "UNION mein `ORDER BY` kahan lagega?"**
> Sirf **poore statement ke end mein**, ek baar. Individual branches par `ORDER BY` allowed nahi (kuch DBs allow karte hain agar parentheses mein `LIMIT` bhi ho). Aur `ORDER BY` mein pehli query ke column names/positions use hote hain.

> **Cross-question: "`UNION` NULLs ko duplicate maanta hai?"**
> Haan — `UNION`, `DISTINCT`, `GROUP BY` sab NULL ko NULL ke barabar maante hain (`=` operator ke ulta). Do NULL rows `UNION` ke baad ek hi bachegi.

---

# 16 — INSERT / UPDATE / DELETE / UPSERT

## 16.1 INSERT

```sql
-- Single row, columns explicitly named (ALWAYS do this)
INSERT INTO suppliers (supplier_id, name, city, rating)
VALUES (7, 'Kota Stone Works', 'Kota', 4);

-- Multiple rows in one statement (faster: one round trip, one transaction)
INSERT INTO suppliers (supplier_id, name, city, rating) VALUES
  (8, 'Udaipur Marbles', 'Udaipur', 3),
  (9, 'Jodhpur Sand Co', 'Jodhpur', NULL);

-- INSERT from a SELECT
INSERT INTO orders_archive (order_id, project_id, total_amount)
SELECT order_id, project_id, total_amount FROM orders WHERE order_date < '2025-02-01';

-- RETURNING: naya generated id turant wapas lo (Postgres) — test code ke liye gold
INSERT INTO suppliers (supplier_id, name, city) VALUES (10, 'Test Supplier', 'Jaipur')
RETURNING supplier_id, created_at;
```

**Hamesha column list likho.** `INSERT INTO suppliers VALUES (…)` tab tootegi jab koi column add karega — aur silently galat column mein data ja sakta hai agar types match kar gaye.

## 16.2 UPDATE

```sql
UPDATE orders
SET    status = 'APPROVED', total_amount = 46000
WHERE  order_id = 12;
```

**UPDATE with a JOIN (Postgres syntax):**

```sql
UPDATE orders o
SET    total_amount = t.computed
FROM   (SELECT order_id, SUM(quantity*unit_price) AS computed
        FROM order_items GROUP BY order_id) t
WHERE  t.order_id = o.order_id
  AND  o.total_amount <> t.computed;
```

## 16.3 DELETE

```sql
DELETE FROM order_items WHERE order_id = 12;
DELETE FROM orders WHERE order_id = 12;   -- ON DELETE CASCADE ho toh items apne aap
```

**DELETE vs TRUNCATE vs DROP — ye poocha jaata hai:**

| | DELETE | TRUNCATE | DROP |
|---|---|---|---|
| Kya hatta hai | rows (WHERE ke saath selective) | **saari** rows | poori table + structure |
| Type | DML | DDL | DDL |
| Rollback | haan (transaction mein) | Postgres: haan; MySQL/Oracle: **nahi** (implicit commit) | Postgres: haan |
| Triggers | fire karte hain | **nahi** karte | N/A |
| Speed | slow (row by row, WAL) | bahut fast (file truncate) | fastest |
| Identity/sequence | reset nahi | `RESTART IDENTITY` se reset | N/A |
| FK ke saath | RESTRICT/CASCADE respect | referenced ho toh fail (ya CASCADE) | fail |

## 16.4 The danger — WHERE bhoolna

```sql
UPDATE orders SET status = 'CANCELLED';   -- 💀 SAARE 12 orders cancelled
DELETE FROM orders;                        -- 💀 sab uda diya
```

**Isse bachne ke 5 tareeke — ye batao interview mein, ye maturity dikhata hai:**

1. **Pehle SELECT likho, phir UPDATE mein badlo.**
   ```sql
   SELECT * FROM orders WHERE order_id = 12;      -- verify: 1 row aayi?
   -- ab SELECT * ko UPDATE … SET … se replace karo, WHERE waisa hi rakho
   ```
2. **Transaction mein wrap karo, row count check karke commit karo.**
   ```sql
   BEGIN;
   UPDATE orders SET status='CANCELLED' WHERE order_id = 12;
   -- output: UPDATE 1   ← agar UPDATE 12 dikha toh...
   ROLLBACK;  -- ya COMMIT;
   ```
3. **psql mein safe-update mode** — MySQL mein `SET SQL_SAFE_UPDATES=1` non-key WHERE ke bina UPDATE/DELETE block kar deta hai.
4. **Production mein read-only credentials** use karo. QA ko write access shared prod DB par nahi chahiye (Section 39).
5. **`RETURNING` se verify karo** ki kya-kya change hua.
   ```sql
   UPDATE orders SET status='CANCELLED' WHERE order_id=12 RETURNING order_id, status;
   ```

## 16.5 UPSERT

"Insert karo agar nahi hai, warna update kar do."

**Postgres — `ON CONFLICT`:**

```sql
INSERT INTO suppliers (supplier_id, name, city, rating)
VALUES (1, 'Sharma Cement Co', 'Jaipur', 5)
ON CONFLICT (supplier_id)
DO UPDATE SET name = EXCLUDED.name,
              city = EXCLUDED.city,
              rating = EXCLUDED.rating;
```

`EXCLUDED` = wo row jo insert hone waali thi.

```sql
-- "insert karo, agar hai toh chhod do" — idempotent insert
INSERT INTO payments (payment_id, invoice_id, paid_on, amount, idempotency_key)
VALUES (99, 2, '2025-06-01', 50000, 'idem-b3')
ON CONFLICT (idempotency_key) DO NOTHING;
```

**MySQL:** `INSERT … ON DUPLICATE KEY UPDATE col = VALUES(col)`
**Standard SQL:** `MERGE INTO … WHEN MATCHED THEN UPDATE … WHEN NOT MATCHED THEN INSERT`

**Zaroori baat:** `ON CONFLICT` ke liye us column par **unique constraint ya unique index hona hi chahiye**. Warna Postgres error dega. Ye conceptually important hai: **upsert database ke unique constraint par bharosa karta hai, application code par nahi** — isliye ye concurrency-safe hai.

> **[REAL]** Merlin mein "ek scope sirf ek baar sold ho sakta hai" wala rule ek **unique partial index** se enforce hota hai. Do concurrent requests aayen toh ek jeet-ti hai aur doosri ko **duplicate key error** milta hai, jo API layer mein **HTTP 409 Conflict** ban jaata hai. Ye bilkul wahi principle hai — **uniqueness ka faisla database karta hai, application ka `if (exists) …` check nahi**, kyunki wo check aur insert ke beech mein doosra thread ghus sakta hai (TOCTOU race). Interview mein ye bolna:
> *"We enforce single-sale with a unique partial index rather than an application-level existence check, because a check-then-insert has a race window. Under concurrency one writer wins and the other gets a duplicate key error, which we map to a 409. I test that specifically by firing two concurrent requests and asserting exactly one 200 and one 409, and that the collection has exactly one document."*

## 16.6 Soft delete

```sql
-- Hard delete
DELETE FROM orders WHERE order_id = 11;

-- Soft delete — data rehta hai, sirf flag lag jaata hai
UPDATE orders SET is_deleted = true WHERE order_id = 11;
```

Soft delete ke saath **har single query mein** `is_deleted = false` chahiye. Ek bhi query bhool jaye toh deleted data leak ho jaata hai. Ye **QA ka favourite bug-hunting ground** hai (Section 40.4).

> **Interview answer:**
> "INSERT, UPDATE and DELETE are the write path, and the rule I hold myself to is that I never type an UPDATE or DELETE without first running it as a SELECT with the exact same WHERE clause and checking the row count. On anything shared I wrap it in an explicit transaction, look at the reported row count, and only then commit — if it says twelve rows when I expected one, I roll back. For upsert, Postgres uses INSERT ON CONFLICT and MySQL uses ON DUPLICATE KEY UPDATE, and both require a unique constraint on the conflict target. That's actually the important part: the uniqueness decision is made by the database under a lock, not by an application-level 'does it exist' check, which has a race window between the check and the insert."

> **Cross-question: "Kya `UPDATE` ki jagah `DELETE` + `INSERT` kar sakte ho?"**
> Technically haan, par: FK cascade fire ho sakta hai, auto-increment id badal jayegi (references toot jayenge), audit trail mein galat entry banegi, triggers alag fire honge, aur transaction ke bahar kiya toh beech mein row **exist hi nahi karegi** — koi concurrent reader use miss kar dega. `UPDATE` hi karo.

---

# 17 — Keys

## 17.1 Primary Key

Row ko **uniquely identify** karta hai.

- Ek table mein **sirf ek** PK.
- Automatically `UNIQUE` + `NOT NULL`.
- Zyadatar DBs isme automatically ek index banate hain.

```sql
CREATE TABLE suppliers (supplier_id INT PRIMARY KEY, …);

-- Composite PK — do columns milkar unique
CREATE TABLE order_tags (
    order_id INT,
    tag      VARCHAR(30),
    PRIMARY KEY (order_id, tag)
);
```

## 17.2 Foreign Key

Doosri table ki PK ko point karta hai. **Referential integrity** enforce karta hai — matlab DB tumhe aisi row insert hi nahi karne dega jo kisi non-existent parent ko point kare.

```sql
ALTER TABLE orders
  ADD CONSTRAINT fk_orders_project
  FOREIGN KEY (project_id) REFERENCES projects(project_id);
```

```sql
INSERT INTO orders (order_id, project_id, …) VALUES (99, 999, …);
-- ERROR: insert or update on table "orders" violates foreign key constraint
```

**Yaad rakho: FK apne aap index nahi banata** (PK banata hai, FK nahi — Postgres, Oracle, SQL Server mein). Child table ke FK column par manually index banana padta hai, warna:
- `WHERE project_id = 1` slow hoga
- Parent row delete karne par DB ko child table full-scan karna padega → **DELETE bahut slow**

Ye ek **real performance bug** hai jo QA load testing mein pakad sakta hai.

## 17.3 Composite key

Do ya zyada columns milkar key.

```sql
CREATE TABLE project_supplier_rates (
    project_id  INT,
    supplier_id INT,
    material    VARCHAR(50),
    rate        NUMERIC(10,2),
    PRIMARY KEY (project_id, supplier_id, material)
);
```

**Column order matter karta hai** — composite PK ka index left-to-right prefix par kaam karta hai. `WHERE project_id=1` index use karega; `WHERE material='Sand'` **nahi** karega (Section 19).

## 17.4 Unique key

PK jaisa unique, lekin:
- Ek table mein **kai** unique constraints ho sakte hain.
- **NULL allow karta hai** — aur (Postgres/Oracle/MySQL mein) **kai NULLs allow karta hai**, kyunki NULL ≠ NULL.

```sql
ALTER TABLE users ADD CONSTRAINT uq_users_email UNIQUE (email);

-- Composite unique — business rule as a constraint
ALTER TABLE invoices ADD CONSTRAINT uq_invoice_no UNIQUE (invoice_no);
-- ↑ ye add karte hi hamara duplicate INV-2025-002 wala data fail kar dega — accha hai!
```

**QA insight:** agar business bolti hai "ek org mein ek email sirf ek contact ka ho sakta hai", toh wo **DB constraint honi chahiye**, sirf service-layer validation nahi. Warna: concurrency race, direct DB writes, aur buggy import scripts duplicate bana denge.

> **[REAL]** Merlin ka `findByOrgAndEmailAddress` bug bilkul yahi tha — code `Optional<Contact>` maan raha tha (matlab "at most one"), par `(org, emailAddress)` par koi unique index nahi tha. Duplicate ban gaye → `IncorrectResultSizeDataAccessException` → HTTP 400. **Do-line fix:** (1) `(orgId, emailAddress)` par unique partial index (`isDeleted: false` wale documents par), (2) tab tak repository return type `List<Contact>` karo taaki crash na ho. Interview mein bolna: *"The type in the code claimed a uniqueness the schema didn't guarantee. Any time a repository returns Optional, I check that there's a matching unique index — otherwise it's a latent 500-or-400 waiting for duplicate data."*

## 17.5 Surrogate vs Natural key

| | **Surrogate** | **Natural** |
|---|---|---|
| Kya hai | meaningless auto-generated id (`SERIAL`, `UUID`, Mongo `ObjectId`) | asli business value (email, PAN, invoice_no) |
| Badalti hai? | kabhi nahi | badal sakti hai (email change, invoice renumber) |
| Size | chhota int, fast joins | aksar bada string |
| Privacy | PII leak nahi | PK URLs/logs mein PII daal deta hai |
| Business meaning | zero — extra unique constraint chahiye | khud hi rule enforce karta hai |

**Industry practice (aur sahi jawab):** **surrogate PK use karo, aur natural key par ek alag UNIQUE constraint lagao.** Dono ke fayde mil jaate hain — stable joins + enforced business rule.

```sql
CREATE TABLE invoices (
    invoice_id  BIGSERIAL PRIMARY KEY,          -- surrogate
    invoice_no  VARCHAR(30) NOT NULL UNIQUE,    -- natural, still enforced
    …
);
```

**UUID vs auto-increment integer:**

| | Auto-increment | UUID / ObjectId |
|---|---|---|
| Generate kahan | DB (round-trip chahiye) | client-side bhi ho sakta hai |
| Index locality | sequential — B-tree ke liye acha | random — page splits, index bloat (UUIDv4) |
| Enumeration risk | `/orders/5` → `/orders/6` guess kar sakte ho (IDOR testing!) | guess nahi kar sakte |
| Distributed / sharding | collision problem | safe |
| Size | 4–8 bytes | 16 bytes |

> **QA note:** auto-increment IDs ka matlab hai ki **IDOR (Insecure Direct Object Reference)** test karna zaroori hai — Org A ke user se Org B ke order ka id hit karo, 403/404 aana chahiye, 200 nahi. Mongo `ObjectId` bhi fully random nahi hai (timestamp + machine + counter), toh usko bhi security control mat maano — **authorization check hi security hai**.

> **Interview answer:**
> "A primary key uniquely identifies a row — one per table, implicitly unique and not null. A foreign key points at another table's key and gives you referential integrity, so the database refuses to create orphans. A composite key spans multiple columns, and the column order matters because it defines the index prefix. A unique key is like a primary key but you can have several per table and it permits NULLs — and since NULL isn't equal to NULL, most engines allow multiple NULL rows, which surprises people.
> On surrogate versus natural: I default to a surrogate primary key — a generated ID with no business meaning — because natural keys change, and a changing key breaks every reference to it. But I put a UNIQUE constraint on the natural key as well, so the business rule is still enforced by the database. The failure mode I've actually seen is code assuming uniqueness that the schema never guaranteed — a lookup returning Optional where there was no unique index behind it. When duplicate data appeared it threw an incorrect-result-size exception and surfaced as a 400 to the user. So now, whenever I see a single-result lookup, I go check that there's a matching unique index."

> **Cross-question: "FK hamesha honi chahiye? Log kyun hata dete hain?"**
> Log hataate hain: (a) write performance — har insert par parent check; (b) sharding — cross-shard FK possible hi nahi; (c) bulk load/migration ki asaani; (d) microservices — data alag DBs mein hai. **Lekin QA ke liye ye red flag hai:** FK hataoge toh orphan rows banenge, aur phir wo consistency application code mein enforce karni padegi — jo eventually koi na koi path miss kar deta hai. Agar FK nahi hai toh **orphan-detection query nightly chalao** (Section 37).

---

# 18 — Constraints

## 18.1 Saare constraints

| Constraint | Kya karta hai | Example |
|---|---|---|
| `NOT NULL` | column khali nahi ho sakta | `name VARCHAR(100) NOT NULL` |
| `UNIQUE` | duplicate values nahi | `email VARCHAR UNIQUE` |
| `PRIMARY KEY` | UNIQUE + NOT NULL, ek per table | `id INT PRIMARY KEY` |
| `FOREIGN KEY` | doosri table mein exist karna chahiye | `REFERENCES projects(project_id)` |
| `CHECK` | koi bhi boolean rule | `CHECK (quantity > 0)` |
| `DEFAULT` | value na do toh ye lag jaayegi | `DEFAULT false` |
| `EXCLUDE` (PG) | generalized unique (ranges, geometry) | overlapping bookings rokna |

## 18.2 CHECK — business rules DB mein

```sql
ALTER TABLE order_items ADD CONSTRAINT chk_qty_positive CHECK (quantity > 0);
ALTER TABLE order_items ADD CONSTRAINT chk_price_nonneg CHECK (unit_price >= 0);

ALTER TABLE orders ADD CONSTRAINT chk_status
  CHECK (status IN ('DRAFT','APPROVED','DELIVERED','CANCELLED'));

ALTER TABLE payments ADD CONSTRAINT chk_amount_pos CHECK (amount > 0);

-- Multi-column CHECK
ALTER TABLE invoices ADD CONSTRAINT chk_invoice_dates
  CHECK (invoice_date <= CURRENT_DATE);
```

Test:

```sql
INSERT INTO order_items VALUES (99, 1, 'Cement', -5, 400);
-- ERROR: new row for relation "order_items" violates check constraint "chk_qty_positive"
```

**CHECK ka gotcha:** `CHECK (col > 0)` par agar `col` NULL ho toh condition `UNKNOWN` deti hai, aur CHECK **UNKNOWN ko pass** kar deta hai (`WHERE` ke ulta!). Toh `NOT NULL` alag se chahiye.

## 18.3 DEFAULT

```sql
CREATE TABLE orders (
    …,
    status     VARCHAR(20) NOT NULL DEFAULT 'DRAFT',
    is_deleted BOOLEAN NOT NULL DEFAULT false,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

**QA ka test:** `INSERT` karo bina us column ke — kya default laga? Aur **`NULL` explicitly bhejo** — DEFAULT **nahi** lagega, NULL insert hoga (aur `NOT NULL` ho toh error). Ye API testing mein important hai: *"field omitted"* vs *"field sent as null"* do **alag** cases hain, aur bahut saare bugs is farak mein chhupe hote hain.

## 18.4 FK actions — ON DELETE / ON UPDATE

```sql
FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE
```

| Action | Parent delete hone par child ka kya hoga |
|---|---|
| `RESTRICT` | delete **rok deta hai** (turant check) |
| `NO ACTION` | default; delete rokta hai, par check statement ke end tak defer ho sakta hai |
| `CASCADE` | child rows bhi delete ho jaati hain |
| `SET NULL` | child ka FK column NULL ho jaata hai (column nullable hona chahiye) |
| `SET DEFAULT` | child ka FK column default value le leta hai |

**Kya kab:**
- `CASCADE` — jab child ka parent ke bina koi wajood na ho. `order_items` bina `order` ke bekaar hai → CASCADE sahi.
- `RESTRICT` — jab accidental data loss se bachna ho. `supplier` delete karne par uske `orders` udd jaayein? **Bilkul nahi** → RESTRICT.
- `SET NULL` — optional relationship. Manager chala gaya, employee reh gaya → `manager_id` NULL.

**⚠️ CASCADE ka khatra:** ek `DELETE FROM projects WHERE project_id=1` chain reaction se orders → order_items → invoices → payments sab uda sakta hai. Agar tumhare schema mein deep cascade chain hai, **QA ko wo chain map karni chahiye** aur test karna chahiye ki accidental delete kitna damage karega. Ye ek badiya interview point hai.

```
DELETE projects(1)
   └─ CASCADE → orders (5 rows)
        └─ CASCADE → order_items (9 rows)
        └─ CASCADE → invoices (3 rows)
             └─ CASCADE → payments (4 rows)
   Total silent deletion: 21 rows from one statement
```

## 18.5 Kyun DB constraints > application validation

| | App validation | DB constraint |
|---|---|---|
| Bypass ho sakta hai? | haan — dusra service, script, manual SQL, bug | **nahi** |
| Concurrency-safe? | nahi — check-then-act race | haan — lock ke andar |
| Legacy/bad data? | naye writes hi rukte hain | migration ke waqt hi pata chal jaata hai |
| Error message | achha, user-friendly | technical, translate karna padta hai |
| Performance | free | thoda cost |

**Sahi jawab: dono.** App validation achhe UX ke liye, DB constraint **truth** ke liye. Interview mein ye "defence in depth" phrase use karo.

> **Interview answer:**
> "The main constraints are NOT NULL, UNIQUE, PRIMARY KEY, FOREIGN KEY, CHECK and DEFAULT. My position is that constraints belong in the database, not only in application code — application validation gives a nice error message, but it can be bypassed by another service, a migration script, or a manual query, and a check-then-insert in application code has a race window that a unique index doesn't. So I treat app validation as UX and database constraints as truth.
> One thing I always check on foreign keys is the ON DELETE action. CASCADE is right when the child can't exist without the parent — order items under an order. But a deep cascade chain means a single delete can silently remove rows from four tables, so I map the chain and test what a delete actually destroys. For anything a user could delete by accident, RESTRICT is safer. And a subtle one: CHECK constraints pass when the expression is NULL, unlike WHERE, so a CHECK on a nullable column needs a separate NOT NULL."

> **Cross-question: "Ek constraint add karni hai bade production table par jisme already violating data hai — kaise?"**
> Postgres mein `ADD CONSTRAINT … NOT VALID` — naye writes par enforce hoga, purane data par nahi, aur table lock nahi lagega. Phir purana data clean karo, phir `VALIDATE CONSTRAINT` (jo lighter lock leta hai). QA ka kaam: (1) violating rows ki list nikaalo aur count do, (2) clean-up ka plan verify karo, (3) `NOT VALID` phase mein test karo ki naye writes reject ho rahe hain, (4) `VALIDATE` ke baad count zero confirm karo.

---

# 19 — Indexes

## 19.1 Kya hai

Index ek **alag data structure** hai jo bolti hai "ye value in rows par hai" — bilkul kitaab ke index ki tarah. Bina index ke DB ko **poori table padhni** padti hai (sequential/full scan).

```
Bina index — "supplier_id = 4 wale orders":
  row1? no  row2? no  row3? no  row4? YES  row5? no … row12? no
  → 12 rows padhi, 1 mili.  O(n)

Index ke saath (B-tree on supplier_id):
       [3]
      /   \
   [1,2]  [4,5]
             \
            (4 → rows 4, 11)
  → 2-3 nodes padhe, seedha row par pahunche.  O(log n)
```

## 19.2 B-tree intuition

Zyadatar indexes **B-tree** hote hain (balanced tree):

```
                    ┌──────────────┐
        root        │ 100  |  500  │
                    └───┬──┬───┬───┘
              ┌─────────┘  │   └─────────┐
        ┌─────▼────┐  ┌────▼─────┐  ┌────▼─────┐
internal│ 20 | 60  │  │ 200|350  │  │ 700|900  │
        └──┬──┬──┬─┘  └──────────┘  └──────────┘
     ┌─────┘  │  └────┐
  ┌──▼──┐  ┌──▼──┐ ┌──▼──┐
  │10,15│─▶│25,40│▶│65,80│   leaf nodes: sorted + linked list
  └─────┘  └─────┘ └─────┘        ↑ isliye RANGE scan bhi fast hai
```

**Isse ye samajh aata hai ki B-tree kya support karta hai:**

| Query | Index use hoga? | Kyun |
|---|---|---|
| `WHERE x = 5` | ✅ | direct descent |
| `WHERE x > 5` / `BETWEEN` | ✅ | leaves sorted + linked |
| `ORDER BY x` | ✅ | leaves already sorted — sort skip |
| `WHERE x LIKE 'abc%'` | ✅ | prefix = range scan |
| `WHERE x LIKE '%abc'` | ❌ | leading wildcard — koi starting point nahi |
| `WHERE UPPER(x) = 'ABC'` | ❌ | function ne value badal di → expression index chahiye |
| `WHERE x + 1 = 5` | ❌ | same — `x = 4` likho |
| `WHERE x <> 5` | ❌ (usually) | zyadatar table match karti hai, seq scan sasta |
| `WHERE x IS NULL` | ✅ (Postgres) | PG NULLs ko index karta hai |

**Ye table interview mein sona hai.** "Index kab use nahi hota" ka jawab yahi hai.

## 19.3 Index banana

```sql
CREATE INDEX idx_orders_project   ON orders(project_id);
CREATE INDEX idx_orders_status    ON orders(status);
CREATE UNIQUE INDEX uq_invoice_no ON invoices(invoice_no);

-- Composite
CREATE INDEX idx_orders_proj_status_date ON orders(project_id, status, order_date);

-- Partial (Postgres) — sirf kuch rows index karo
CREATE INDEX idx_orders_active ON orders(project_id) WHERE is_deleted = false;

-- Expression index
CREATE INDEX idx_users_email_lower ON users(LOWER(email));

-- Production-safe (table lock nahi lagta, par slow)
CREATE INDEX CONCURRENTLY idx_orders_supplier ON orders(supplier_id);
```

## 19.4 Composite index — column ORDER matters (ye bahut poocha jaata hai)

```sql
CREATE INDEX idx_orders_pso ON orders(project_id, status, order_date);
```

Ise **phone directory** samjho jo `(last_name, first_name, city)` par sorted hai:

| Query | Index use? | Explanation |
|---|---|---|
| `WHERE project_id=1` | ✅ full | leftmost prefix |
| `WHERE project_id=1 AND status='DRAFT'` | ✅ full | 2-column prefix |
| `WHERE project_id=1 AND status='DRAFT' AND order_date>'2025-01-01'` | ✅ full | poora index |
| `WHERE project_id=1 AND order_date>'2025-01-01'` | ⚠️ partial | `project_id` se seek, `order_date` filter as extra — `status` skip hone se range narrow nahi hua |
| `WHERE status='DRAFT'` | ❌ | leading column skip — last name jaane bina directory bekaar |
| `WHERE order_date > '2025-01-01'` | ❌ | same |
| `WHERE project_id=1 ORDER BY status` | ✅ | sort free mil gaya |

**Rule — "leftmost prefix rule".** Index `(A,B,C)` in queries pe kaam karta hai: `A`, `(A,B)`, `(A,B,C)`. `B` ya `C` akele par nahi.

**Column order kaise choose karein:**
1. **Equality columns pehle**, range columns baad mein. `WHERE a=1 AND b>5` → index `(a, b)`, `(b, a)` nahi.
2. **Zyada selective (zyada unique values) pehle** — jab dono equality hon.
3. **`ORDER BY` ke columns** range column ke turant baad.

> **[REAL]** Merlin ke Mongo indexes bilkul yahi pattern follow karte hain: `{orgId: 1, isDeleted: 1, status: 1}`. `orgId` pehle hai kyunki **har** query multi-tenancy ki wajah se usko filter karti hai; `isDeleted` doosra kyunki wo bhi har query mein hai; `status` teesra kyunki wo optional hai. Agar order `{status: 1, orgId: 1}` hota toh `status` ke bina wali queries index use hi nahi kar paatin. **Leftmost prefix rule Mongo mein bilkul same hai.** Ye tumhara sabse strong interview point hai — SQL ka concept, Mongo ka real example.

## 19.5 Clustered vs Non-clustered

| | **Clustered** | **Non-clustered (secondary)** |
|---|---|---|
| Kya hai | **table ka physical order hi** index hai | alag structure, row ka pointer rakhta hai |
| Kitne | ek table mein **ek hi** | kai |
| Leaf mein kya | **poori row** | key + row pointer/PK |
| Range scan | bahut fast (rows saath-saath) | random I/O (har hit ke liye table jump) |
| SQL Server / MySQL InnoDB | PK = clustered index (default) | baaki sab |
| Postgres | **clustered index nahi hai** — sab non-clustered (heap tables). `CLUSTER` command ek baar physically reorder karta hai, maintain nahi karta | sab |

**Non-clustered index se rows nikalna:**
```
index leaf: (status='DRAFT') → row pointer (ctid / PK)
                                       │
                                       ▼
                             heap/table pe jaao aur poori row padho   ← extra I/O
```
Isi extra hop ko **"bookmark lookup"** kehte hain, aur isi se covering index ka fayda samajh aata hai.

## 19.6 Covering index

Agar index mein **saare columns hain jo query chahti hai**, toh DB ko table chhoona hi nahi padta — **index-only scan**.

```sql
-- Query
SELECT project_id, status FROM orders WHERE project_id = 1;

-- Ye index covering hai: dono columns index mein hain
CREATE INDEX idx_cover ON orders(project_id, status);
```

`EXPLAIN` mein `Index Only Scan` dikhega, `Index Scan` nahi. Ye bada difference hai.

Postgres mein `INCLUDE` se non-key columns bhi daal sakte ho (search ke liye nahi, sirf covering ke liye):

```sql
CREATE INDEX idx_cover2 ON orders(project_id) INCLUDE (status, total_amount);
```

## 19.7 Indexes kab NUKSAAN karte hain

Ye sabse important part hai — junior candidate "index laga do" bolta hai, senior **trade-off** batata hai.

1. **Har write slow hota hai.** Har `INSERT`/`UPDATE`/`DELETE` ko **har** affected index bhi update karna padta hai. 8 indexes wali table par insert 8x extra kaam hai.
2. **Storage.** Index table jitna bada bhi ho sakta hai. 500GB table + 10 indexes = terabytes.
3. **Low-cardinality columns par bekaar.** `is_deleted` (2 values) par index? 50% rows match karengi — optimizer seq scan hi chunega, index bekaar pada rahega. **Lekin composite index ke andar** ya **partial index ke condition** mein wahi column bahut useful hai.
4. **Unused indexes** — pure overhead. Postgres mein `pg_stat_user_indexes.idx_scan = 0` se dhoondo.
5. **Duplicate/redundant indexes.** `(a)` aur `(a,b)` dono hain? `(a)` redundant hai, `(a,b)` uska kaam kar deta hai. Drop it.
6. **Bloat.** Bahut updates/deletes ke baad B-tree bloat hota hai; `REINDEX` chahiye.
7. **Optimizer confusion.** Bahut zyada overlapping indexes se planner galat chun sakta hai.

## 19.8 EXPLAIN padhna

```sql
EXPLAIN ANALYZE
SELECT o.order_id, o.total_amount, s.name
FROM   orders o JOIN suppliers s ON s.supplier_id = o.supplier_id
WHERE  o.project_id = 1 AND o.is_deleted = false;
```

**Output (typical, bade table par):**

```
Hash Join  (cost=1.14..30.55 rows=5 width=48) (actual time=0.045..0.061 rows=5 loops=1)
  Hash Cond: (o.supplier_id = s.supplier_id)
  ->  Seq Scan on orders o  (cost=0.00..29.35 rows=5 width=20)
                            (actual time=0.012..0.025 rows=5 loops=1)
        Filter: ((NOT is_deleted) AND (project_id = 1))
        Rows Removed by Filter: 7
  ->  Hash  (cost=1.06..1.06 rows=6 width=36) (actual ...)
        ->  Seq Scan on suppliers s  (cost=0.00..1.06 rows=6 width=36)
Planning Time: 0.310 ms
Execution Time: 0.098 ms
```

**Kaise padhein:**
- **Neeche se upar, andar se bahar** padho. Sabse andar wala node pehle chalta hai.
- `cost=startup..total` — arbitrary units, absolute value se matlab nahi; **comparison** ke liye hai.
- `rows=` estimate hai; `actual … rows=` sach hai. **Agar dono mein 10x+ ka farak hai → stale statistics.** `ANALYZE table;` chalao. Ye sabse common "query suddenly slow" ki wajah hai.
- `loops=` — nested loop mein andar wala node kitni baar chala. `actual time` **per loop** hai, toh total = time × loops.
- `Rows Removed by Filter` bada hai → DB bahut rows padh ke phenk raha hai → **index chahiye**.
- `EXPLAIN ANALYZE` **query actually chalata hai** — production par `UPDATE`/`DELETE` ke saath sirf transaction mein chalao aur rollback karo!

**Node types — kya dekhna hai:**

| Node | Matlab | Kab chinta karein |
|---|---|---|
| `Seq Scan` | poori table padhi | **badi table par + selective WHERE** = missing index |
| `Index Scan` | index se seek, phir table se row | theek hai |
| `Index Only Scan` | sirf index se, table chhua hi nahi | best |
| `Bitmap Heap Scan` | bahut sari rows match hui — index se bitmap banaya, phir table | usually fine |
| `Nested Loop` | har outer row ke liye inner probe | outer rows bahut zyada + inner Seq Scan = disaster |
| `Hash Join` | hash table banaya | bade sets ke liye fine |
| `Merge Join` | dono sorted | fine |
| `Sort` | explicit sorting | `Sort Method: external merge Disk: 50000kB` = work_mem kam |
| `Materialize` | intermediate result store kiya | thoda memory |

**Chhoti table par `Seq Scan` normal hai** — 12 rows ke liye index kholna mehnga hai. Ye clarification interview mein zaroor karo, warna lagega tum rat ke aaye ho.

## 19.9 QA ka index workflow

1. Feature ki **slowest query** identify karo (APM ya `pg_stat_statements`).
2. `EXPLAIN ANALYZE` chalao **production-jaise data volume** par — 100 rows par sab fast hai. Ye sabse badi galti hai.
3. `Seq Scan` + bada `Rows Removed by Filter` dhoondo.
4. Index suggest karo, **column order justify karo** (equality pehle, range baad mein).
5. Index ke baad **dobara measure** karo — aur **write path bhi measure karo** (insert/bulk import slow to nahi hua?).
6. `pg_stat_user_indexes` se confirm karo ki naya index **actually use ho raha hai**.

> **Interview answer:**
> "An index is a separate sorted structure — usually a B-tree — that lets the database jump straight to matching rows instead of scanning the whole table, turning a linear scan into a logarithmic descent. Because the leaves are sorted and linked, a B-tree serves equality, range queries, prefix LIKE, and ORDER BY. It can't serve a leading-wildcard LIKE or a predicate wrapped in a function, because the indexed value isn't what's being compared — those need an expression index.
> For composite indexes, column order is the thing people get wrong. An index on A, B, C can be used for A, for A and B, and for all three, but not for B alone — that's the leftmost prefix rule. So I put equality columns first, the range column last, and I order the equality columns by how often they appear in queries. In our system the compound index leads with the tenant ID because literally every query filters on it.
> The trade-off I always state is that indexes make reads faster and writes slower — every insert has to maintain every index — plus storage and bloat. So I don't recommend an index without checking the write path too. And to verify, I read EXPLAIN ANALYZE: I compare the estimated rows to the actual rows, because a big gap means stale statistics rather than a missing index, and I look at 'rows removed by filter' — if it's reading thousands to return ten, that's where the index belongs."

> **Cross-question: "Query kal fast thi, aaj slow hai. Schema nahi badla. Kya hua?"**
> Sabse pehle **statistics**: table badhi, planner ke stats purane, wo galat plan chun raha hai → `ANALYZE`. Doosra, **data distribution badal gayi** (ek org ne 10 lakh rows daal diye — pehle index selective tha, ab nahi). Teesra, **parameter sniffing / cached plan** (SQL Server mein bada issue). Chautha, **index bloat** ya autovacuum peeche reh gaya. Paanchva, **lock contention** — query slow nahi hai, wo **wait** kar rahi hai; `pg_stat_activity` mein `wait_event` dekho. Chhatva, **connection pool exhaustion** — query fast hai, connection milne mein time lag raha hai.
> Ye 6 possibilities bolna tumhe turant senior dikhata hai, kyunki junior sirf "index add kar do" bolta hai.

---

# 20 — Transactions aur ACID

## 20.1 Kya hai

Transaction = **kai statements ka ek group jo "all or nothing" chalta hai**. Ya toh sab commit honge, ya sab rollback.

```sql
BEGIN;                                    -- ya START TRANSACTION;

UPDATE accounts SET balance = balance - 5000 WHERE account_id = 'A';
UPDATE accounts SET balance = balance + 5000 WHERE account_id = 'B';

COMMIT;                                   -- dono pakke
-- ya
ROLLBACK;                                 -- dono udd gaye, kuch nahi hua
```

**SAVEPOINT** — transaction ke andar partial rollback:

```sql
BEGIN;
INSERT INTO orders (order_id, project_id, order_date, status, total_amount)
VALUES (100, 1, '2025-06-01', 'DRAFT', 0);

SAVEPOINT after_order;

INSERT INTO order_items VALUES (200, 100, 'Cement', -5, 400);   -- CHECK fail!
-- ERROR

ROLLBACK TO SAVEPOINT after_order;    -- sirf item wapas gaya, order bacha hua hai
INSERT INTO order_items VALUES (200, 100, 'Cement', 5, 400);    -- sahi value
COMMIT;
```

**Autocommit:** default mein har single statement apni transaction mein chalta hai. `BEGIN` likhne se hi explicit transaction shuru hota hai. Ye jaanna zaroori hai — QA ne `psql` mein `UPDATE` chala diya bina `BEGIN` ke, toh **turant commit ho gaya, rollback ka option hi nahi**.

## 20.2 ACID — bank transfer se har letter

**Scenario:** A ke account se B ko ₹5000 bhejne hain. Shuru mein A = ₹10,000, B = ₹2,000.

### A — Atomicity ("sab ya kuch nahi")

Transaction ke andar ke saare steps **ek unit** hain. Beech mein crash ho jaye toh **kuch bhi** apply nahi hota.

```
BEGIN
  Step 1: A -= 5000   →  A = 5000   ✓
  Step 2: B += 5000   →  💥 SERVER CRASH
COMMIT (kabhi nahi hua)

Recovery ke baad:  A = 10000, B = 2000     ← Step 1 bhi undo ho gaya
```

Bina atomicity ke: A ke 5000 gayab, B ko mile hi nahi. **Paisa hawa mein.**

Kaise hota hai: DB **write-ahead log (WAL / undo log)** rakhta hai. Restart par uncommitted transactions ko log se undo kar deta hai.

**QA test:** transaction ke beech mein service kill karo (`kill -9`), phir DB state check karo. Partial write bilkul nahi honi chahiye. Ye **chaos-style test** hai jo bahut kam QA log karte hain — interview mein bolne layak.

### C — Consistency ("rules kabhi nahi tootenge")

Transaction DB ko **ek valid state se doosre valid state** mein le jaata hai. Saare constraints (PK, FK, CHECK, UNIQUE) aur business invariants transaction ke **end mein** satisfy hone chahiye.

```
Invariant: SUM(all balances) = constant (paisa banta ya gayab nahi hota)

Pehle:  A=10000, B=2000   → total 12000
Baad:   A=5000,  B=7000   → total 12000   ✓ Consistency bani rahi

Agar sirf Step 1 commit hota:  A=5000, B=2000 → total 7000  ✗ INVALID
```

```sql
-- Consistency ko DB se enforce karo, code se nahi
ALTER TABLE accounts ADD CONSTRAINT chk_no_overdraft CHECK (balance >= 0);
```

Ab agar A ke paas sirf ₹3000 hote, toh Step 1 hi fail ho jaata aur poora transaction rollback.

**Note:** ACID ka "C" aur CAP theorem ka "C" **alag cheezein hain**. ACID-C = "constraints hold". CAP-C = "har node same data dekhta hai". Ye clarification interview mein bahut acha lagta hai.

### I — Isolation ("concurrent transactions ek dusre ko gandagi nahi dikhate")

Do transactions ek saath chal rahe hain — har ek ko lagna chahiye ki wo **akela** chal raha hai.

```
T1: A → B transfer 5000        T2: total balance report

Bina isolation ke:
  T1: A -= 5000   (A = 5000)
                                T2: reads A=5000, B=2000 → total 7000  ✗ GALAT
  T1: B += 5000   (B = 7000)
  T1: COMMIT

Isolation ke saath:
  T2 ko ya toh (10000, 2000)=12000 dikhega ya (5000, 7000)=12000. Beech ka nahi.
```

Isolation **degrees** mein aata hai — poora Section 21.

### D — Durability ("commit ka matlab pakka")

`COMMIT` ka reply mil gaya = data **permanent** hai, chahe agle second power chali jaye.

```
COMMIT  →  WAL disk par fsync  →  client ko "OK"
                                        │
                                     💥 POWER CUT
                                        │
                                  Restart: WAL replay → A=5000, B=7000  ✓
```

Kaise: **WAL pehle disk par fsync hota hai**, tab commit acknowledge hota hai. Actual data pages baad mein background mein likhi jaati hain.

**QA angle:** kuch systems performance ke liye durability weaken kar dete hain — Postgres mein `synchronous_commit = off`, MySQL mein `innodb_flush_log_at_trx_commit = 2`, MongoDB mein `writeConcern: {w: 1}` bina `journal`. **Ye conscious trade-off hai** — crash par last few ms ka data ja sakta hai. QA ko poochna chahiye: *"What's our write concern / synchronous_commit setting, and is the business okay losing the last 200 ms of writes on a crash?"* Payments ke liye jawab **nahi** hona chahiye.

## 20.3 ACID summary table

| Letter | Guarantee | Bank example | Toot jaye toh | Kaise implement hota hai |
|---|---|---|---|---|
| **A**tomicity | all-or-nothing | debit + credit dono ya koi nahi | paisa gayab | undo log / WAL, rollback |
| **C**onsistency | rules hold | total balance constant | invalid state (negative balance) | constraints, triggers, FK |
| **I**solation | concurrent txns don't interfere | report kabhi half-transfer nahi dekhta | dirty/phantom reads | locks / MVCC |
| **D**urability | committed = permanent | power cut ke baad bhi transfer hua | acknowledged data gayab | WAL + fsync, replication |

> **Interview answer:**
> "A transaction groups statements so they succeed or fail as one unit, using BEGIN, COMMIT and ROLLBACK, with SAVEPOINT for partial rollback inside one transaction.
> ACID with a bank transfer: Atomicity means the debit and the credit both happen or neither does — if the server crashes between them, recovery undoes the debit, so money never vanishes. Consistency means the transaction moves the database from one valid state to another — the sum of all balances is the invariant, and constraints like a non-negative balance check enforce it. Isolation means a concurrent report query never sees the half-done transfer — it sees the totals either before or after, never in between. Durability means once COMMIT returns, the transfer survives a power cut, because the write-ahead log is flushed to disk before the commit is acknowledged.
> The one I actually test is Atomicity: I kill the service mid-transaction and then assert the database has no partial write. And I ask what the durability setting is — if someone has turned synchronous commit off for speed, that's fine for analytics but not for payments, and that's a conversation the team should have deliberately rather than by accident."

> **Cross-question: "ACID ka 'C' aur CAP ka 'C' same hai?"**
> Nahi. ACID-C = database constraints/invariants hold karte hain (single node ki baat). CAP-C = distributed system mein har node ko **same latest data** dikhta hai (linearizability). Bilkul alag concepts, bas naam same hai. Ye poocha jaata hai aur zyadatar log confuse karte hain.

> **Cross-question: "Transaction lamba rakhna kyun bura hai?"**
> (1) Locks lambe time tak held rehte hain → doosre blocked. (2) Postgres mein purani row versions vacuum nahi ho sakti → **table bloat**. (3) Rollback mehnga ho jaata hai. (4) Deadlock ka chance badhta hai. (5) Connection pool block hota hai. **Rule: transaction ke andar kabhi network call (HTTP, email, payment gateway) mat karo.** Ye ek real design review point hai — Merlin mein bhi check karne layak.

---

# 21 — Isolation levels aur 3 anomalies

**Ye section sabse zyada poocha jaata hai. Ise theek se yaad karo.**

## 21.1 Teen classic anomalies

| Anomaly | Ek line mein | Kis level par rukti hai |
|---|---|---|
| **Dirty read** | doosre ka **uncommitted** data padh liya | READ COMMITTED |
| **Non-repeatable read** | **same row** dobara padhi, value badal gayi | REPEATABLE READ |
| **Phantom read** | **same query** dobara chalayi, **nayi rows** aa gayi | SERIALIZABLE |

Yaad rakhne ka tareeka: **Dirty = uncommitted. Non-repeatable = row badli. Phantom = row count badla.**

## 21.2 Anomaly 1 — Dirty Read (timeline)

T2 ne wo data padh liya jo T1 ne **abhi commit hi nahi kiya**. Phir T1 rollback ho gaya — matlab T2 ne aisa data padha jo **kabhi exist hi nahi kiya**.

```
 TIME   T1 (order update)                    T2 (report)
 ────   ─────────────────────────────────    ─────────────────────────────────
  t1    BEGIN
  t2    UPDATE orders SET total_amount=999999
        WHERE order_id=1;
        (uncommitted, sirf T1 ke memory mein)
  t3                                         BEGIN
  t4                                         SELECT total_amount FROM orders
                                             WHERE order_id=1;
                                             → 999999   💀 DIRTY
  t5    ROLLBACK;                            (asli value wapas 120000)
  t6                                         Report mein 999999 chhap gaya
                                             — ye number kabhi exist hi nahi kiya
```

**Kahan possible:** sirf `READ UNCOMMITTED` par. **Postgres mein bilkul possible nahi** — Postgres `READ UNCOMMITTED` maangne par bhi `READ COMMITTED` deta hai. MySQL InnoDB mein `READ UNCOMMITTED` actually kaam karta hai.

## 21.3 Anomaly 2 — Non-Repeatable Read (timeline)

T1 ne **ek hi row** do baar padhi aur **do alag values** mili — kyunki beech mein T2 ne commit kar diya.

```
 TIME   T1 (invoice reconciliation)          T2 (payment applied)
 ────   ─────────────────────────────────    ─────────────────────────────────
  t1    BEGIN
  t2    SELECT amount FROM invoices
        WHERE invoice_id=2;
        → 450000
  t3                                         BEGIN
  t4                                         UPDATE invoices SET amount=460000
                                             WHERE invoice_id=2;
  t5                                         COMMIT;
  t6    SELECT amount FROM invoices
        WHERE invoice_id=2;
        → 460000   💀 SAME ROW, ALAG VALUE
  t7    -- T1 ke andar hi do alag sach!
        -- Agar T1 ne pehle wale 450000 se
        -- calculation kar ke ab 460000 se
        -- validate kiya, toh mismatch bug.
```

**Real bug shape:** ek batch job pehle rows read karta hai, calculation karta hai, phir dobara read kar ke verify karta hai. `READ COMMITTED` par ye reliably fail karega jab traffic ho.

**Kahan rukti hai:** `REPEATABLE READ` aur upar. Postgres mein `REPEATABLE READ` snapshot isolation deta hai — poore transaction ko wahi snapshot dikhta hai jo `BEGIN` (pehli statement) ke waqt tha.

## 21.4 Anomaly 3 — Phantom Read (timeline)

T1 ne **same query** do baar chalayi aur doosri baar **nayi rows** aa gayi. Individual rows nahi badli — **set** badal gaya.

```
 TIME   T1 (project audit)                   T2 (naya order)
 ────   ─────────────────────────────────    ─────────────────────────────────
  t1    BEGIN
  t2    SELECT COUNT(*) FROM orders
        WHERE project_id=1 AND is_deleted=false;
        → 5
  t3                                         BEGIN
  t4                                         INSERT INTO orders VALUES
                                             (13,1,2,2,'2025-06-01',
                                              'DRAFT',10000,false);
  t5                                         COMMIT;
  t6    SELECT COUNT(*) FROM orders
        WHERE project_id=1 AND is_deleted=false;
        → 6   👻 PHANTOM — ek nayi row prakat ho gayi
  t7    -- T1 ne pehle 5 orders ke basis par
        -- budget check pass kiya tha; ab 6 hain.
        -- Budget overshoot ho sakta hai.
```

**Kahan rukti hai:** `SERIALIZABLE`. (Note: Postgres ka `REPEATABLE READ` snapshot-based hai, isliye wo bhi phantoms rok deta hai read ke liye — standard se **strict** hai. MySQL InnoDB `REPEATABLE READ` par gap locks se phantoms rokta hai. Ye nuance batana interviewer ko impress karta hai.)

## 21.5 Chautha (bonus) — Lost Update / Write Skew

Ye standard ke "3 anomalies" mein nahi hai par **real applications mein sabse zyada hota hai**.

**Lost update:**

```
 TIME   T1 (User A edits PO)                 T2 (User B edits same PO)
 ────   ─────────────────────────────────    ─────────────────────────────────
  t1    BEGIN
  t2    SELECT total_amount FROM orders
        WHERE order_id=1;  → 120000
  t3                                         BEGIN
  t4                                         SELECT total_amount FROM orders
                                             WHERE order_id=1;  → 120000
  t5    UPDATE orders
        SET total_amount = 120000 + 5000
        WHERE order_id=1;
  t6    COMMIT;   (= 125000)
  t7                                         UPDATE orders
                                             SET total_amount = 120000 + 3000
                                             WHERE order_id=1;
  t8                                         COMMIT;  (= 123000)

  Final: 123000.  T1 ka +5000 GAYAB. "Last write wins" — silent data loss.
```

**Teen fixes (ye interview mein bolna):**

1. **Atomic update** — read-modify-write hi mat karo:
   ```sql
   UPDATE orders SET total_amount = total_amount + 5000 WHERE order_id = 1;
   ```
2. **Pessimistic lock** — read karte waqt hi lock lo:
   ```sql
   BEGIN;
   SELECT total_amount FROM orders WHERE order_id=1 FOR UPDATE;  -- ← row lock
   UPDATE orders SET total_amount = … WHERE order_id=1;
   COMMIT;
   ```
3. **Optimistic locking (version column)** — best for web/REST:
   ```sql
   UPDATE orders SET total_amount = 125000, version = version + 1
   WHERE order_id = 1 AND version = 7;
   -- affected rows = 0  →  kisi aur ne beech mein badal diya  →  HTTP 409 return karo
   ```

**Write skew** — dono transactions alag rows likhte hain par ek shared invariant tod dete hain:

```
Rule: kisi project par kam se kam ek APPROVED order hona chahiye.
T1: order 3 ko CANCELLED kar raha hai (dekhta hai order 10 APPROVED hai → ok)
T2: order 10 ko CANCELLED kar raha hai (dekhta hai order 3 APPROVED hai → ok)
Dono commit → ab zero APPROVED orders. Invariant toot gaya.
```
Ye **sirf `SERIALIZABLE`** rokta hai (ya explicit locking).

## 21.6 Isolation levels — the matrix

| Level | Dirty read | Non-repeatable read | Phantom read | Lost update | Cost |
|---|---|---|---|---|---|
| **READ UNCOMMITTED** | ✅ possible | ✅ possible | ✅ possible | ✅ | sabse fast |
| **READ COMMITTED** | ❌ prevented | ✅ possible | ✅ possible | ✅ | fast (default: PG, Oracle, SQL Server) |
| **REPEATABLE READ** | ❌ | ❌ prevented | ✅ possible* | ❌ (PG mein error) | medium (default: MySQL) |
| **SERIALIZABLE** | ❌ | ❌ | ❌ prevented | ❌ | slowest, retries chahiye |

\* Postgres ka REPEATABLE READ (snapshot isolation) phantoms bhi rok deta hai. MySQL InnoDB gap locks se rokta hai. Standard **allow** karta hai — isliye "possible" likha hai.

```sql
-- Set karna
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN;
…
COMMIT;

-- Ya session ke liye
SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Current level dekhna
SHOW transaction_isolation;
```

## 21.7 SERIALIZABLE ka hidden cost — retries

Postgres `SERIALIZABLE` **SSI (Serializable Snapshot Isolation)** use karta hai. Wo blocking nahi karta — wo **detect** karta hai ki serial order possible nahi tha aur transaction ko **abort** kar deta hai:

```
ERROR: could not serialize access due to read/write dependencies among transactions
SQLSTATE 40001
```

**Matlab: application ko retry logic likhna hoga.** Agar tumhari team `SERIALIZABLE` use karti hai aur retry nahi hai, toh load par random 500s aayenge.

```python
# QA ka test: SERIALIZABLE par retry hai ya nahi
def test_serializable_conflict_is_retried(client):
    """Do concurrent conflicting writes: dono eventually succeed hone chahiye,
    ya ek ko clean 409 milna chahiye — 500 kabhi nahi."""
    results = run_concurrently(lambda: client.post("/orders/1/approve"), n=2)
    codes = sorted(r.status_code for r in results)
    assert 500 not in codes, f"Serialization failure leaked as 500: {codes}"
    assert codes in ([200, 200], [200, 409]), codes
```

## 21.8 QA ka concurrency test template

```python
import threading

def test_lost_update_is_prevented(db, api):
    """Do users same PO par simultaneously +5000 aur +3000 karte hain.
       Final = 128000 hona chahiye, 125000 ya 123000 nahi."""
    db.execute("UPDATE orders SET total_amount=120000 WHERE order_id=1")

    barrier = threading.Barrier(2)
    def bump(delta):
        barrier.wait()                    # dono ko exactly same waqt chhodo
        return api.patch("/orders/1", json={"amountDelta": delta})

    with ThreadPoolExecutor(2) as ex:
        r1, r2 = ex.submit(bump, 5000), ex.submit(bump, 3000)
        codes = [r1.result().status_code, r2.result().status_code]

    final = db.scalar("SELECT total_amount FROM orders WHERE order_id=1")
    # Do valid outcomes:
    #  (a) dono succeed → 128000  (atomic increment ya proper locking)
    #  (b) ek 409 → 125000 ya 123000  (optimistic locking, honest conflict)
    assert (codes == [200,200] and final == 128000) or \
           (sorted(codes) == [200,409] and final in (125000, 123000)), (codes, final)
```

**`threading.Barrier` ka use yaad rakho** — bina uske threads staggered chalte hain aur race kabhi reproduce hi nahi hota. Ye chhoti si detail interview mein bolne se pata chalta hai ki tumne concurrency test **sach mein** likha hai.

> **[REAL]** Merlin mein "ek scope sirf ek baar sold" wala rule isolation level se nahi, **unique partial index** se enforce hai — aur ye zyada solid design hai. Do concurrent writes aayen toh isolation level chahe jo ho, **index level par ek hi jeet-ta hai**; doosre ko duplicate-key error milta hai jo API layer mein **409** ban jaata hai. Interview mein bolna: *"For a single-record invariant I prefer a unique index over relying on isolation level, because a unique index is enforced at the storage layer regardless of isolation, it doesn't need retry logic, and it can't be bypassed by a different code path. Isolation levels I'd reach for when the invariant spans multiple rows — that's where SERIALIZABLE or explicit row locks earn their cost."*

> **Interview answer (isolation — the full one):**
> "Isolation levels trade correctness against concurrency, and they're defined by which anomalies they permit.
> A dirty read is reading another transaction's uncommitted data — if that transaction rolls back, you've read a value that never existed. Only READ UNCOMMITTED allows it; Postgres doesn't implement it at all.
> A non-repeatable read is reading the same row twice inside one transaction and getting two different values, because another transaction committed an update in between. READ COMMITTED allows this — and READ COMMITTED is the default in Postgres, Oracle and SQL Server, so most systems have it.
> A phantom read is running the same query twice and getting new rows the second time — the individual rows didn't change, the result set did. Standard SQL says only SERIALIZABLE prevents it, though Postgres's REPEATABLE READ is snapshot-based so it prevents phantoms too, and MySQL InnoDB uses gap locks.
> The one that actually causes most production bugs isn't in that list — it's lost update: two users read a value, both compute from it, and the second write silently overwrites the first. I fix that three ways depending on the case: an atomic increment in SQL rather than read-modify-write, a SELECT FOR UPDATE if I need a pessimistic lock, or optimistic locking with a version column where a zero-row update means someone else won and I return a 409.
> One practical caveat on SERIALIZABLE in Postgres: it doesn't block, it detects and aborts with a serialization failure, so the application must have retry logic. If a team turns on SERIALIZABLE without retries, they get intermittent 500s under load — that's a specific thing I'd test for."

> **Cross-question: "Default isolation level kya hai?"**
> Postgres, Oracle, SQL Server: `READ COMMITTED`. MySQL InnoDB: `REPEATABLE READ`. **Ye difference asli migration bug hai** — MySQL se Postgres shift karne par jo code `REPEATABLE READ` maan raha tha wo `READ COMMITTED` par non-repeatable reads dekhne lagega.

> **Cross-question: "MVCC kya hai?"**
> **Multi-Version Concurrency Control.** Row ko update karne par purani version delete nahi hoti — nayi version banti hai, aur har transaction ko apne snapshot ke hisaab se sahi version dikhta hai. Fayda: **readers writers ko block nahi karte, writers readers ko block nahi karte**. Cost: purani versions ka garbage collection chahiye — Postgres mein `VACUUM`. Agar autovacuum peeche reh gaya toh **table bloat** hota hai aur queries slow. Ye ek real Postgres production issue hai jo QA ko pata hona chahiye.

> **Cross-question: "`SELECT FOR UPDATE` aur `FOR SHARE` mein farak?"**
> `FOR UPDATE` exclusive row lock leta hai — koi aur na padh sakta hai (lock ke saath) na likh sakta hai. `FOR SHARE` shared lock — doosre bhi `FOR SHARE` le sakte hain, par koi update nahi kar sakta. `FOR UPDATE SKIP LOCKED` ek badiya variant hai — job queue implement karne ke liye (locked rows chhod ke agli le lo). `FOR UPDATE NOWAIT` — lock na mile toh turant error, wait mat karo.

---

# 22 — Deadlocks

## 22.1 Kya hai

Do (ya zyada) transactions ek dusre ka lock chhodne ka **wait** kar rahe hain — aur na koi aage badh sakta hai, na peeche.

```
 TIME   T1                                   T2
 ────   ─────────────────────────────────    ─────────────────────────────────
  t1    BEGIN                                BEGIN
  t2    UPDATE orders WHERE order_id=1;
        🔒 order 1 par lock
  t3                                         UPDATE orders WHERE order_id=2;
                                             🔒 order 2 par lock
  t4    UPDATE orders WHERE order_id=2;
        ⏳ T2 ka lock chahiye — WAIT
  t5                                         UPDATE orders WHERE order_id=1;
                                             ⏳ T1 ka lock chahiye — WAIT

        ┌──────────────────────────────────────────┐
        │  T1 waits for T2  →  T2 waits for T1     │
        │  ══════════ CYCLE ══════════             │
        └──────────────────────────────────────────┘

  t6    DB deadlock detector cycle pakadta hai:
        → ek transaction ko VICTIM chunta hai aur abort karta hai
        ERROR: deadlock detected  (Postgres SQLSTATE 40P01)
        DETAIL: Process 123 waits for ShareLock on transaction 456;
                Process 456 waits for ShareLock on transaction 123.
```

Database **hang nahi hota** — wo cycle detect karke ek ko maar deta hai. Loser ko error milta hai. Postgres `deadlock_timeout` (default 1s) ke baad check karta hai.

## 22.2 Common causes

1. **Alag-alag order mein rows lock karna** — upar wala classic case. Sabse aam.
2. **Lock upgrade** — pehle `SELECT` (shared), phir `UPDATE` (exclusive) same row par, do transactions mein.
3. **Foreign key locks** — child insert karne par parent row par lock lagta hai. Do transactions alag order mein parents chhoo rahe hon toh deadlock.
4. **Index vs table lock ordering** — unique index maintenance apne locks leta hai.
5. **Gap locks (MySQL)** — `REPEATABLE READ` par range locks unexpected overlap kar sakte hain.
6. **Lamba transaction + user interaction** — transaction khol ke user ke input ka wait karna. **Kabhi mat karo.**

## 22.3 Bachne ke tareeke

| Technique | Kaise |
|---|---|
| **Consistent lock order** | hamesha ascending PK order mein rows chhoo. `ORDER BY order_id` laga kar batch update karo. **Ye sabse bada fix hai.** |
| **Transactions chhote rakho** | lock ka window jitna chhota, collision utna kam |
| **Transaction mein network call mat karo** | HTTP/email/payment gateway ke liye locks 2 second held rakhna = deadlock factory |
| **Sahi isolation level** | zaroorat se zyada strict level = zyada locks |
| **`SELECT FOR UPDATE` upfront** | jo rows chahiye, sabko **ek hi statement** mein, sorted order mein lock kar lo |
| **`NOWAIT` / `SKIP LOCKED`** | wait hi mat karo — turant fail ya skip |
| **Retry logic** | deadlock ek **transient** error hai. Exponential backoff ke saath 3 retries. |
| **Rows par kam contention** | hot counter row (`SELECT balance FROM totals WHERE id=1 FOR UPDATE`) ko sharded counters mein todo |

**Consistent ordering ka code example:**

```sql
-- ❌ Deadlock-prone: order random hai
UPDATE orders SET status='APPROVED' WHERE order_id IN (2, 1);

-- ✅ Deterministic order
BEGIN;
SELECT order_id FROM orders WHERE order_id IN (1,2) ORDER BY order_id FOR UPDATE;
UPDATE orders SET status='APPROVED' WHERE order_id IN (1,2);
COMMIT;
```

Application mein bhi: agar do accounts ke beech transfer hai, hamesha **chhote account_id ko pehle lock** karo — chahe paisa kis direction mein ja raha ho.

```python
a, b = sorted([from_account, to_account])   # ← ek line jo deadlock khatam kar deti hai
lock(a); lock(b)
```

## 22.4 QA kya kare

1. **Logs mein `deadlock detected` grep karo.** Staging/perf run ke baad ye check karna chahiye. Zyadatar teams nahi karti.
   ```bash
   grep -c "deadlock detected" postgres.log
   ```
2. **Reproduce karo** — 2 threads, ulta order, `threading.Barrier` se sync.
3. **Verify karo ki retry hai.** Deadlock hone par user ko **500 nahi** dikhna chahiye — ya retry ho jaye, ya clean 409.
4. **Postgres se live locks dekho:**
   ```sql
   SELECT blocked.pid AS blocked_pid, blocked.query AS blocked_query,
          blocking.pid AS blocking_pid, blocking.query AS blocking_query
   FROM   pg_stat_activity blocked
   JOIN   pg_stat_activity blocking
          ON blocking.pid = ANY(pg_blocking_pids(blocked.pid))
   WHERE  cardinality(pg_blocking_pids(blocked.pid)) > 0;
   ```

**Reproduce test:**

```python
def test_no_deadlock_on_reverse_order_updates(db_pool):
    barrier = threading.Barrier(2)
    errors = []

    def txn(first, second):
        conn = db_pool.getconn()
        try:
            with conn:
                cur = conn.cursor()
                cur.execute("UPDATE orders SET total_amount=total_amount+1 WHERE order_id=%s", (first,))
                barrier.wait()                      # dono ne pehla lock le liya
                cur.execute("UPDATE orders SET total_amount=total_amount+1 WHERE order_id=%s", (second,))
        except psycopg2.errors.DeadlockDetected as e:
            errors.append(e)
        finally:
            db_pool.putconn(conn)

    t1 = threading.Thread(target=txn, args=(1, 2))
    t2 = threading.Thread(target=txn, args=(2, 1))     # ← ULTA order
    t1.start(); t2.start(); t1.join(); t2.join()

    assert not errors, "Application locks rows in inconsistent order — deadlock reproduced"
```

> **Interview answer:**
> "A deadlock is a cycle of waits: transaction one holds a lock that transaction two needs, and two holds a lock that one needs, so neither can proceed. The database doesn't hang — it has a deadlock detector, it finds the cycle, picks a victim and aborts it with a deadlock error. The loser has to retry.
> The most common cause by far is acquiring locks in inconsistent order — two code paths touching the same two rows in opposite order. The single most effective fix is to always lock in a deterministic order, usually ascending primary key; in application code that's literally sorting the IDs before you lock them. Beyond that: keep transactions short, never do a network call or wait for user input inside a transaction, and add retry with backoff, because a deadlock is a transient error, not a bug in the data.
> As a tester, two things. I grep the database log for deadlock messages after every performance run, because they're usually invisible in the application if there's a retry — and if there isn't a retry, users see a 500. And I reproduce them deliberately with two threads doing the updates in opposite order, synchronised on a barrier so the window actually overlaps."

> **Cross-question: "Deadlock aur lock wait / blocking mein farak?"**
> **Blocking** — ek transaction wait kar raha hai, doosra aage badh raha hai. Eventually resolve ho jayega (ya `lock_timeout` par fail). **Deadlock** — cycle hai, kabhi resolve nahi hoga, isliye DB ko intervene karna padta hai. Slow query ke complaints aksar blocking hoti hain, deadlock nahi.

> **Cross-question: "Victim kaun chuna jaata hai?"**
> Postgres aam taur par wo transaction chunta hai jo cycle detect karte waqt wait kar raha tha. SQL Server "deadlock priority" aur rollback cost (kitna kaam undo karna padega) dekhta hai — sasta wala mara jaata hai. MySQL bhi kam rows modify karne wale ko victim banata hai. **Matlab: chhota transaction zyada baar victim banta hai** — isliye retry logic usi par sabse zaroori hai.

---

# 23 — Normalization

## 23.1 Kya hai

Normalization = table ko is tarah tod-na ki **har fact sirf ek jagah store ho**. Maqsad: **redundancy hatana** aur **anomalies rokna**.

**Teen anomalies jo redundancy paida karti hai — ye naam se yaad rakho:**

| Anomaly | Kya hota hai | Example |
|---|---|---|
| **Insert anomaly** | naya fact daal hi nahi sakte jab tak koi aur fact na ho | naya supplier tab tak nahi daal sakte jab tak uska ek order na ho |
| **Update anomaly** | ek fact badalne ke liye N rows badalni padti hain; ek chooti toh **data contradict** karega | supplier ka city badla → 50 rows update; 1 miss hui → do alag cities |
| **Delete anomaly** | ek cheez delete karne par doosri cheez ka data bhi udd jaata hai | aakhri order delete kiya → supplier ka address hi gaya |

## 23.2 Starting point — ek badi flat table

Ye hai `po_flat` — sab kuch ek table mein (jaise Excel sheet):

| po_id | supplier_name | supplier_city | project_name | project_city | materials |
|---|---|---|---|---|---|
| 1 | Sharma Cement Co | Jaipur | Skyline Towers | Jaipur | Cement OPC 53 (200 @ 400), Sand (100 @ 400) |
| 2 | Verma Steels | Delhi | Skyline Towers | Jaipur | TMT Bar 12mm (5000 @ 65), TMT Bar 16mm (2000 @ 62.50) |
| 4 | Bombay Hardware | Mumbai | Metro Depot | Delhi | Plywood (150 @ 500) |

**Problems saaf dikh rahe hain:**
- `materials` mein ek cell ke andar **kai values** — SQL se query kaise karoge? `WHERE materials LIKE '%Sand%'` — bhayanak.
- `Skyline Towers` ka city do baar likha hai. Ek jagah typo hua toh?
- Sharma Cement Co ka koi order na ho toh uska address kahin nahi rahega.

## 23.3 1NF — First Normal Form

**Rule: har cell mein ek hi atomic value. Koi repeating group, koi comma-separated list, koi array nahi.**

Har material ko apni row do:

| po_id | material | quantity | unit_price | supplier_id | supplier_name | supplier_city | project_id | project_name | project_city |
|---|---|---|---|---|---|---|---|---|---|
| 1 | Cement OPC 53 | 200 | 400.00 | 1 | Sharma Cement Co | Jaipur | 1 | Skyline Towers | Jaipur |
| 1 | Sand | 100 | 400.00 | 1 | Sharma Cement Co | Jaipur | 1 | Skyline Towers | Jaipur |
| 2 | TMT Bar 12mm | 5000 | 65.00 | 2 | Verma Steels | Delhi | 1 | Skyline Towers | Jaipur |
| 2 | TMT Bar 16mm | 2000 | 62.50 | 2 | Verma Steels | Delhi | 1 | Skyline Towers | Jaipur |
| 4 | Plywood | 150 | 500.00 | 4 | Bombay Hardware | Mumbai | 2 | Metro Depot | Delhi |

**Primary key ab: `(po_id, material)`** — composite.

✅ Ab `WHERE material = 'Sand'` kaam karega, `SUM(quantity)` kaam karega, index lag sakta hai.
❌ Lekin redundancy **badh gayi** — "Sharma Cement Co / Jaipur" ab 2 rows mein, "Skyline Towers / Jaipur" 4 rows mein.

## 23.4 2NF — Second Normal Form

**Rule: 1NF ho, AUR koi non-key column composite key ke *sirf ek hisse* par depend na kare (no partial dependency).**

Composite key hai `(po_id, material)`. Ab dekho:

```
(po_id, material)  →  quantity, unit_price     ✅ poore key par depend
 po_id             →  supplier_id, supplier_name, supplier_city,
                      project_id, project_name, project_city    ❌ PARTIAL!
```

Supplier ka naam sirf `po_id` par depend karta hai — material se koi lena-dena nahi. Isliye **do tables mein todo**:

**`po`** (PK = `po_id`)

| po_id | supplier_id | supplier_name | supplier_city | project_id | project_name | project_city |
|---|---|---|---|---|---|---|
| 1 | 1 | Sharma Cement Co | Jaipur | 1 | Skyline Towers | Jaipur |
| 2 | 2 | Verma Steels | Delhi | 1 | Skyline Towers | Jaipur |
| 4 | 4 | Bombay Hardware | Mumbai | 2 | Metro Depot | Delhi |

**`po_item`** (PK = `(po_id, material)`)

| po_id | material | quantity | unit_price |
|---|---|---|---|
| 1 | Cement OPC 53 | 200 | 400.00 |
| 1 | Sand | 100 | 400.00 |
| 2 | TMT Bar 12mm | 5000 | 65.00 |
| 2 | TMT Bar 16mm | 2000 | 62.50 |
| 4 | Plywood | 150 | 500.00 |

✅ Supplier ka naam ab 4 rows ki jagah 3 rows mein.
❌ Lekin `Skyline Towers / Jaipur` abhi bhi 2 baar hai.

**Note:** agar PK single column hota (composite nahi), toh 2NF automatically satisfy ho jaata — partial dependency possible hi nahi. Ye ek acha cross-question ka jawab hai.

## 23.5 3NF — Third Normal Form

**Rule: 2NF ho, AUR koi non-key column doosre non-key column par depend na kare (no transitive dependency).**

`po` table mein:

```
po_id  →  supplier_id  →  supplier_name, supplier_city     ❌ TRANSITIVE
po_id  →  project_id   →  project_name, project_city       ❌ TRANSITIVE
```

`supplier_name` ka asli determinant `supplier_id` hai, `po_id` nahi. Isliye **alag tables**:

**`po`** (PK = `po_id`)

| po_id | supplier_id | project_id | order_date | status |
|---|---|---|---|---|
| 1 | 1 | 1 | 2025-01-05 | DELIVERED |
| 2 | 2 | 1 | 2025-01-18 | DELIVERED |
| 4 | 4 | 2 | 2025-02-14 | DELIVERED |

**`suppliers`** (PK = `supplier_id`)

| supplier_id | name | city |
|---|---|---|
| 1 | Sharma Cement Co | Jaipur |
| 2 | Verma Steels | Delhi |
| 4 | Bombay Hardware | Mumbai |

**`projects`** (PK = `project_id`)

| project_id | name | city |
|---|---|---|
| 1 | Skyline Towers | Jaipur |
| 2 | Metro Depot | Delhi |

**`po_item`** (PK = `(po_id, material)`) — waise ka waisa.

✅ **Ab har fact exactly ek jagah hai.** Supplier ka city badalna = **ek row ka update**.
✅ **Aur dhyan do — ye bilkul wahi schema hai jo humne Section 0 mein banaya tha.** Ye koi theory nahi thi; normalized design apne aap yahi shakal leta hai.

**Yaad rakhne ki line (interviewers ise sunna chahte hain):**
> *"Every non-key attribute depends on the key, the whole key, and nothing but the key — so help me Codd."*
> - **"the key"** → 1NF
> - **"the whole key"** → 2NF
> - **"nothing but the key"** → 3NF

## 23.6 BCNF — Boyce-Codd Normal Form

**Rule: har determinant (jo cheez kisi aur ko determine karti hai) ek candidate key honi chahiye.**

BCNF 3NF se **thoda strict** hai. Farak sirf tab dikhta hai jab **overlapping candidate keys** hon.

**Example — site engineer assignment:**

Business rules:
1. Har project par, har material category ke liye **exactly ek** site engineer hota hai.
2. Har site engineer **sirf ek** material category handle karta hai (uski specialization).

**`assignment`**

| project_id | material_category | engineer_id |
|---|---|---|
| 1 | CIVIL | 4 |
| 1 | ELECTRICAL | 5 |
| 2 | CIVIL | 6 |
| 3 | ELECTRICAL | 5 |

**Functional dependencies:**
```
(project_id, material_category)  →  engineer_id      [rule 1]
 engineer_id                     →  material_category [rule 2]
```

**Candidate keys:** `(project_id, material_category)` **aur** `(project_id, engineer_id)` — dono unique hain. Ye **overlap** karte hain (`project_id` dono mein).

**Kya ye 3NF mein hai?** Haan. 3NF ka violation tab hota hai jab non-key → non-key. Yahan `engineer_id → material_category` mein `material_category` ek **prime attribute** hai (kisi candidate key ka hissa), isliye 3NF ki definition ise allow kar deti hai.

**Kya ye BCNF mein hai?** **Nahi.** `engineer_id` ek determinant hai par candidate key nahi hai.

**Practical problem jo isse hoti hai:**

```
Kabir (4) ka specialization CIVIL se PLUMBING karna hai.
→ Kabir ki SAARI rows update karni padengi.
→ Ek row miss hui? Ab Kabir ek project par CIVIL hai aur doosre par PLUMBING.
→ Rule 2 toot gaya, aur DB ne rok bhi nahi paya.
```

**BCNF decomposition:**

**`engineer_specialization`** (PK = `engineer_id`)

| engineer_id | material_category |
|---|---|
| 4 | CIVIL |
| 5 | ELECTRICAL |
| 6 | CIVIL |

**`project_engineer`** (PK = `(project_id, engineer_id)`)

| project_id | engineer_id |
|---|---|
| 1 | 4 |
| 1 | 5 |
| 2 | 6 |
| 3 | 5 |

Ab Kabir ka specialization **ek row ka update** hai. ✅

**Trade-off (ye zaroor bolna):** ye decomposition rule 1 ko ("ek project par ek category ke liye ek hi engineer") **schema level par enforce nahi kar sakti** — ab wo constraint do tables mein faili hui hai. Isliye BCNF kabhi-kabhi **dependency-preserving nahi** hota. Ye BCNF vs 3NF ka classic trade-off hai aur senior interviews mein poocha jaata hai.

## 23.7 Normal forms — summary

| NF | Rule | Kya khatam hota hai |
|---|---|---|
| **1NF** | atomic values, no repeating groups | comma-separated lists, arrays in cells |
| **2NF** | 1NF + no partial dependency on composite key | ek key-part se judi duplicate info |
| **3NF** | 2NF + no transitive dependency (non-key → non-key) | lookup data ka duplication |
| **BCNF** | har determinant candidate key ho | overlapping candidate keys wali anomalies |
| **4NF** | no multi-valued dependency | independent many-to-many ek table mein |
| **5NF** | no join dependency | (theory, practically dur-lab) |

**Practical reality:** industry mein **3NF tak jaate hain**, phir zaroorat par selectively denormalize karte hain. BCNF/4NF/5NF ka naam jaanna kaafi hai; unka theory-level example (jo upar diya hai) ek bolna aa jaye toh full marks.

## 23.8 Denormalization — kab sahi hai

Denormalization = **jaan-boojh kar** redundancy add karna, performance ke liye. Ye galti nahi hai — ye ek **conscious trade-off** hai.

**Kab sahi hai:**

| Case | Kya karte hain | Kyun |
|---|---|---|
| **Read-heavy reports** | pre-computed totals column | har page load par 5-table join mehnga |
| **Analytics / data warehouse** | star schema (fact + wide dimensions) | joins hi bottleneck hain |
| **Historical snapshot** | invoice par supplier ka naam/address **copy** karo | supplier ka address kal badla toh purani invoice **purana** address dikhaye — legal requirement! |
| **Hot lookup** | order row par `project_name` bhi rakho | list API pe join bachta hai |
| **Distributed/sharded** | data locality | cross-shard join possible hi nahi |
| **NoSQL document model** | embed karo, reference mat karo | ek document = ek read |

**Historical snapshot wala point bahut important hai** — ye "redundancy" nahi hai, ye **alag fact** hai. "Aaj supplier ka address" aur "invoice ke waqt supplier ka address" do **alag** cheezein hain. Ise normalize karna hi galti hoti.

**Denormalization ki keemat:**
- **Data drift** — copies desync ho jaati hain. Isko **automated check** chahiye (Section 37).
- Writes complex ho jaate hain (kai jagah update).
- Storage badhta hai.

**Rule:** denormalize karo toh **synchronization ka mechanism** aur **drift-detection ka test** dono chahiye. Bina test ke denormalization = time bomb.

> **[REAL]** Merlin ka schema exactly ye trade-off dikhata hai. Har document mein `org` ek **DBRef** hai (normalized reference) **aur uske saath ek denormalized scalar `orgId` (ObjectId)** bhi hai. Kyun?
> - DBRef se query par filter/index nahi lag sakta efficiently — har query ko `orgId` par index chahiye kyunki **har** multi-tenant query us par filter karti hai.
> - Isliye scalar `orgId` denormalized rakha gaya, aur compound indexes usse lead karte hain: `{orgId: 1, isDeleted: 1, status: 1}`.
>
> **QA ka kaam yahan:** ek **drift check** likho — koi bhi document jahan `orgId` aur `org.$id` match na karein. Agar wo kabhi diverge kare toh ek org ka data doosre org ko dikh sakta hai — ye ek **security bug** hai, sirf data bug nahi.
> ```javascript
> db.purchaseOrders.aggregate([
>   { $addFields: { refOrgId: "$org.$id" } },
>   { $match: { $expr: { $ne: ["$orgId", "$refOrgId"] } } },
>   { $project: { _id: 1, orgId: 1, refOrgId: 1 } },
>   { $limit: 20 }
> ])
> // Expected: 0 documents
> ```
> Interview mein ye bolna: *"We denormalise the tenant ID as a scalar next to the reference, because every query filters on tenant and the reference form can't lead a compound index. The cost of that denormalisation is drift, so I wrote a check that asserts the scalar always equals the reference — and I treat a mismatch as a security issue, not just a data issue, because tenant scoping is what isolates one customer's data from another's."*

> **Interview answer:**
> "Normalization is organising a schema so every fact is stored in exactly one place, which eliminates insert, update and delete anomalies. First normal form means atomic values — no comma-separated lists in a cell. Second normal form removes partial dependencies, where a column depends on only part of a composite key. Third normal form removes transitive dependencies, where a non-key column depends on another non-key column — that's where you split supplier name and city out into a suppliers table. The mnemonic is: every non-key attribute depends on the key, the whole key, and nothing but the key. BCNF is slightly stricter — every determinant must be a candidate key — and it only differs from 3NF when you have overlapping candidate keys.
> In practice I aim for 3NF and then denormalise deliberately where it pays. Two cases where denormalisation is correct, not a shortcut. First, historical snapshots: an invoice should store the supplier's address as it was at invoice time, because that's a different fact from the supplier's current address — normalising that away is actually a bug. Second, tenant scoping in our system: we keep a denormalised scalar tenant ID alongside the reference, because every query filters on tenant and the compound index has to lead with it.
> The rule I hold to is that any denormalisation needs an automated consistency check. If two copies of a fact exist, they will drift, so I write a query that asserts they're equal and run it nightly."

> **Cross-question: "Normalization se performance kharaab hoti hai?"**
> Zaroori nahi. Normalized tables **chhoti** hoti hain → zyada rows ek page mein → kam I/O → better cache hit ratio. Writes **bahut** faster hote hain (ek jagah update). Cost sirf joins ki hai, aur indexed joins sasta hai. Blanket "normalization slow hai" bolna junior signal hai. Sahi jawab: *"Normalisation usually helps writes and hurts nothing until you have a specific read path that's measurably too slow — then I denormalise that one path, with a consistency check."*

> **Cross-question: "1NF ke saath JSON/array column kaise fit hota hai?"**
> Strictly, JSON column 1NF violate karta hai. Practically, modern Postgres `JSONB` aur Mongo documents deliberately ye rule todte hain. Sahi jawab: *"It's a violation of the strict definition, and I'd only accept it when the nested data is genuinely opaque to queries — like an audit payload or a third-party API response — or when it's always read and written as a whole with its parent. The moment you need to filter, aggregate or join on something inside the JSON, it should have been a column or a table."*

---

# 24 — Views aur Materialized Views

## 24.1 View — kya hai

View ek **saved query** hai jise table ki tarah use kar sakte ho. Isme **data store nahi hota** — har baar query chalti hai.

```sql
CREATE VIEW v_active_orders AS
SELECT o.order_id, o.project_id, p.name AS project_name,
       o.supplier_id, s.name AS supplier_name,
       o.order_date, o.status, o.total_amount
FROM   orders o
JOIN   projects p  ON p.project_id  = o.project_id
LEFT   JOIN suppliers s ON s.supplier_id = o.supplier_id
WHERE  o.is_deleted = false;
```

```sql
SELECT project_name, COUNT(*) FROM v_active_orders GROUP BY project_name;
```

| project_name | count |
|---|---|
| Skyline Towers | 5 |
| Metro Depot | 4 |
| Green Villa | 1 |
| Riverfront Mall | 1 |

## 24.2 Views kis kaam ke

| Use | Explanation |
|---|---|
| **Complexity chhupana** | 5-table join ek naam ke peeche |
| **Security** | user ko base table par nahi, view par access do — sensitive columns hata do |
| **Multi-tenancy / soft delete** | `WHERE is_deleted=false` ko ek jagah enforce karo, har query mein repeat mat karo |
| **Backward compatibility** | column rename kiya? purane naam se view bana do, purana code chalta rahe |
| **Consistent business logic** | "active order" ki definition ek jagah |

**Security view example:**

```sql
CREATE VIEW v_users_public AS
SELECT user_id, name, role FROM users;    -- salary aur email nahi

REVOKE ALL ON users FROM reporting_role;
GRANT SELECT ON v_users_public TO reporting_role;
```

## 24.3 Updatable views

Simple views (ek table, no aggregate/DISTINCT/GROUP BY) par `INSERT`/`UPDATE` chal jaata hai. Complex views par nahi — unke liye `INSTEAD OF` trigger chahiye.

```sql
CREATE VIEW v_draft_orders AS
SELECT * FROM orders WHERE status = 'DRAFT' WITH CHECK OPTION;
```

`WITH CHECK OPTION` — view ke through aisi row insert/update nahi kar sakte jo view ki condition satisfy na kare. Bina iske tum ek DRAFT order ko view se `APPROVED` kar sakte the aur wo **view se gayab** ho jaata (`row disappears` bug).

## 24.4 Materialized View

Materialized view **result ko physically store** karta hai — ek cached table ki tarah. Fast read, par **stale ho sakta hai**.

```sql
CREATE MATERIALIZED VIEW mv_project_spend AS
SELECT o.project_id, p.name AS project_name,
       COUNT(*) AS order_count,
       SUM(o.total_amount) AS total_spend,
       MAX(o.order_date) AS last_order_date
FROM   orders o JOIN projects p ON p.project_id = o.project_id
WHERE  o.is_deleted = false
GROUP  BY o.project_id, p.name;

-- Index bhi bana sakte ho (normal view par nahi bana sakte)
CREATE UNIQUE INDEX ON mv_project_spend (project_id);

-- Refresh
REFRESH MATERIALIZED VIEW mv_project_spend;                -- table lock leta hai
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_project_spend;   -- lock nahi, UNIQUE index chahiye
```

**Result:**

| project_id | project_name | order_count | total_spend | last_order_date |
|---|---|---|---|---|
| 1 | Skyline Towers | 5 | 1175000.00 | 2025-05-06 |
| 2 | Metro Depot | 4 | 625000.00 | 2025-04-15 |
| 3 | Green Villa | 1 | 200000.00 | 2025-03-03 |
| 4 | Riverfront Mall | 1 | 180000.00 | 2025-03-19 |

## 24.5 View vs Materialized View vs Table

| | View | Materialized View | Table |
|---|---|---|---|
| Data store hota hai? | ❌ | ✅ | ✅ |
| Hamesha fresh? | ✅ | ❌ (refresh tak stale) | ✅ |
| Read speed | base query jitni | bahut fast | fast |
| Index laga sakte ho? | ❌ (base tables par lagta hai) | ✅ | ✅ |
| Storage | zero | poora | poora |
| Write | limited (simple views) | ❌ (refresh se hi) | ✅ |

## 24.6 QA ke liye — views ke bugs

**Ye section interview mein bolne layak hai, kyunki zyadatar log views ko test hi nahi karte.**

1. **Staleness.** Materialized view ka refresh kab hota hai? Agar nightly hai aur UI "live spend" bolta hai — **wo jhooth hai**. Test: data badlo, page refresh karo, purana number dikha? Bug (ya at least "Last updated" timestamp UI par dikhna chahiye).
2. **Refresh failure silently.** Agar refresh job fail ho gaya, koi alert hai? Test: refresh ko fail karao, dekho koi cheekha ya nahi. Zyadatar cases mein nahi cheekhta — **dashboard chup-chaap purana data dikhata rehta hai.** Ye ek bahut acha bug report hai.
3. **View mein soft-delete filter chhoot gaya.** `v_active_orders` mein `is_deleted=false` hai — par kya **har** view mein hai? Ek view mein miss = deleted data leak.
4. **Nested views ka performance.** View upar view upar view — planner ko poora expand karna padta hai. `EXPLAIN` par 12-table join dikh sakta hai jabki query 1-table lagti hai.
5. **View aur base table ka drift after migration.** Column rename hua, view purane naam se banaya gaya — logic silently badal sakta hai.
6. **`WITH CHECK OPTION` missing** — view se insert kiya, row view se hi gayab.

```sql
-- QA check: materialized view kitna stale hai?
SELECT (SELECT SUM(total_amount) FROM orders WHERE is_deleted=false) AS live_total,
       (SELECT SUM(total_spend)  FROM mv_project_spend)              AS mv_total,
       (SELECT SUM(total_amount) FROM orders WHERE is_deleted=false)
       - (SELECT SUM(total_spend) FROM mv_project_spend)             AS drift;
```

| live_total | mv_total | drift |
|---|---|---|
| 2180000.00 | 2180000.00 | 0.00 |

Isko test suite mein daal do. Non-zero drift = refresh job ki problem.

> **Interview answer:**
> "A view is a stored query — no data is kept, it runs against the base tables every time, so it's always fresh. I use views to hide join complexity, to enforce a filter like the soft-delete flag in one place instead of repeating it in every query, and for security, granting a role access to a view that omits sensitive columns rather than to the table.
> A materialized view actually stores the result, so reads are fast and you can index it, but it's stale until you refresh it. Postgres has REFRESH CONCURRENTLY, which avoids locking readers but needs a unique index on the view.
> What I test around views is mostly the staleness contract. If a dashboard reads a materialized view refreshed nightly but the UI implies live data, that's a defect even though every number is 'correct'. And the failure mode I always check is a silent refresh failure — if the refresh job dies, does anything alert, or does the dashboard just keep serving yesterday's numbers? Usually nothing alerts. So I add an automated check that compares the view's total against the live query and fails if they drift."

> **Cross-question: "Materialized view ki jagah summary table + trigger?"**
> Trade-off: trigger-maintained summary table **real-time** rehti hai, par har write ko mehnga banati hai aur contention/deadlock badhati hai. Materialized view writes ko sasta rakhta hai par stale hai. Teesra option: **incremental refresh** (sirf badle hue rows) — Postgres native support nahi karta, khud likhna padta hai. Chunav business ki staleness tolerance par depend karta hai.

---

# 25 — Stored Procedures aur Triggers

## 25.1 Stored Procedure / Function

DB ke andar store kiya hua code jo SQL se zyada kar sakta hai (loops, conditionals, variables).

```sql
CREATE OR REPLACE FUNCTION recalc_order_total(p_order_id INT)
RETURNS NUMERIC AS $$
DECLARE
    v_total NUMERIC;
BEGIN
    SELECT COALESCE(SUM(quantity * unit_price), 0)
      INTO v_total
      FROM order_items
     WHERE order_id = p_order_id;

    UPDATE orders SET total_amount = v_total WHERE order_id = p_order_id;

    RETURN v_total;
END;
$$ LANGUAGE plpgsql;
```

```sql
SELECT recalc_order_total(12);
```

| recalc_order_total |
|---|
| 0.00 |

Aur ab `orders.total_amount` for order 12 = `0.00` — humara data bug fix ho gaya.

**Function vs Procedure (Postgres 11+):** function ek value return karta hai aur transaction ke andar chalta hai; `PROCEDURE` `CALL` se chalta hai aur **apne andar `COMMIT`/`ROLLBACK` kar sakta hai**. Interview ke liye itna kaafi.

**Pros:** network round-trips kam (loop DB ke andar chala), permissions tightly control ho sakti hain, logic ek jagah for multiple apps.
**Cons (ye zyada important hain):** version control aur code review se aksar bahar, unit test karna mushkil, debugging painful, DB CPU mehnga hai aur horizontally scale nahi hota, deployment app ke saath sync nahi hota, vendor lock-in.

**Modern consensus (ye bolna):** *"Business logic belongs in the application; stored procedures are justified for data-intensive operations where moving the data to the app is the bottleneck — bulk transforms, set-based maintenance jobs — or where multiple independent applications must share the exact same rule."*

## 25.2 Triggers

Trigger = code jo kisi event par **apne aap** chal jaata hai — koi explicitly call nahi karta.

```sql
-- 1. Trigger function
CREATE OR REPLACE FUNCTION trg_audit_order_status()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.status IS DISTINCT FROM OLD.status THEN
        INSERT INTO order_audit (order_id, old_status, new_status, changed_at, changed_by)
        VALUES (NEW.order_id, OLD.status, NEW.status, now(), current_user);
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- 2. Trigger
CREATE TRIGGER audit_order_status
AFTER UPDATE ON orders
FOR EACH ROW
EXECUTE FUNCTION trg_audit_order_status();
```

Ab:

```sql
UPDATE orders SET status='DELIVERED' WHERE order_id=3;
SELECT * FROM order_audit;
```

| audit_id | order_id | old_status | new_status | changed_at | changed_by |
|---|---|---|---|---|---|
| 1 | 3 | APPROVED | DELIVERED | 2025-06-01 10:22:31 | app_user |

**Trigger ke prakar:**

| Dimension | Options |
|---|---|
| Timing | `BEFORE` (row modify kar sakta hai via `NEW`), `AFTER` (side effects), `INSTEAD OF` (views par) |
| Event | `INSERT`, `UPDATE`, `DELETE`, `TRUNCATE` |
| Level | `FOR EACH ROW` (har row), `FOR EACH STATEMENT` (ek statement mein ek baar) |
| Condition | `WHEN (OLD.status IS DISTINCT FROM NEW.status)` |

`NEW` = naya row, `OLD` = purana row. `INSERT` par `OLD` NULL hai; `DELETE` par `NEW` NULL hai.

**Common legitimate uses:** audit trail, `updated_at` maintain karna, denormalized counter update, cross-table validation jo CHECK se possible nahi.

## 25.3 QA ko trigger ka pata kyun hona CHAHIYE

**Ye is section ka asli point hai — aur interview mein sabse valuable line.**

Trigger ek **invisible side effect** hai. Tum ek `UPDATE` chalate ho, aur peeche paanch aur cheezein ho jaati hain jo kisi code file mein likhi nahi hain. Ye jo bugs banata hai:

| Symptom | Trigger ki wajah |
|---|---|
| **"Test cleanup kaam nahi kar raha"** | `DELETE FROM orders` par `ON DELETE` trigger ne kuch aur likh diya, ya cascade ne aur tables chhoo li |
| **"Row count expected se zyada"** | ek insert ne trigger se do aur rows bana di |
| **"Update slow hai"** | har row par trigger, har trigger ek insert — 10,000-row bulk update = 10,000 extra inserts |
| **"Bulk import ne data corrupt kar diya"** | import ne triggers bypass kiye (`COPY` kuch triggers skip kar sakta hai, ya DBA ne disable kar diya tha) |
| **"Deadlock random aa raha hai"** | trigger ne ek aur table par lock liya, lock order badal gaya |
| **"Same operation API se aur SQL se alag behave karta hai"** | app logic aur trigger dono kaam kar rahe hain — **double** ho raha hai |
| **"Rollback ke baad bhi audit row hai"** | trigger ne `dblink`/notification se transaction ke bahar likha |
| **"Recursion / infinite loop"** | trigger A ne table B likhi, B ke trigger ne A likhi |

**Isliye QA ko sabse pehle ye chalana chahiye jab kisi table ka behaviour samajh na aaye:**

```sql
-- Postgres: is table par kaunse triggers hain?
SELECT tgname AS trigger_name,
       tgrelid::regclass AS table_name,
       pg_get_triggerdef(oid) AS definition
FROM   pg_trigger
WHERE  NOT tgisinternal
  AND  tgrelid = 'orders'::regclass;

-- Saare triggers, poore DB mein
SELECT event_object_table, trigger_name, action_timing, event_manipulation
FROM   information_schema.triggers
ORDER  BY event_object_table;
```

```sql
-- Saare functions/procedures
SELECT routine_name, routine_type FROM information_schema.routines
WHERE routine_schema = 'public';
```

**MongoDB mein equivalent:** change streams / Atlas Triggers. Same problem — ek document write, aur peeche kuch aur ho gaya. Merlin mein agar koi change stream listener hai, toh QA ko wo bhi map karna chahiye.

**Trigger-aware test design:**

```python
def test_status_change_writes_exactly_one_audit_row(db):
    before = db.scalar("SELECT COUNT(*) FROM order_audit WHERE order_id=3")
    api.patch("/orders/3", json={"status": "DELIVERED"})
    after = db.scalar("SELECT COUNT(*) FROM order_audit WHERE order_id=3")
    # EXACTLY one — 2 ka matlab app aur trigger dono likh rahe hain (double audit bug)
    assert after - before == 1, f"Expected 1 audit row, got {after - before}"
```

**"Exactly one, not at least one"** — ye assertion style seniority dikhati hai.

> **Interview answer:**
> "A stored procedure is code stored and executed inside the database; a trigger is code the database runs automatically when an insert, update or delete happens. Triggers can fire before or after the event, for each row or once per statement.
> The reason I care about them as a tester is that a trigger is an invisible side effect — it isn't in the application code, so reading the service tells you nothing about it. Concretely, it explains several bug classes I've had to chase: test cleanup that doesn't clean up because a delete fires other writes; a row count that's higher than expected because one insert produced two rows; an update that's unexpectedly slow because a per-row trigger runs ten thousand times; and the worst one, double writes, where the application also implements the same logic the trigger does, so an audit entry or a counter gets applied twice — and the API path and a direct SQL path behave differently.
> So when I'm testing a table whose behaviour surprises me, the first thing I run is a query against information_schema.triggers to see what's actually attached. And I write the assertion as 'exactly one audit row', not 'at least one', because 'at least one' passes on a double-write bug.
> On whether to use them: I'd keep business logic in the application, where it's version-controlled, reviewable and unit-testable. Triggers are reasonable for cross-cutting mechanical concerns like an audit trail or an updated-at timestamp, where you genuinely want them to be impossible to bypass."

> **Cross-question: "BEFORE aur AFTER trigger mein farak?"**
> `BEFORE` row write hone se **pehle** chalta hai aur `NEW` ko **modify** kar sakta hai (ya `RETURN NULL` se operation cancel kar sakta hai). `AFTER` row write hone ke **baad** chalta hai — modify nahi kar sakta, par row committed-shape mein dikhti hai, isliye audit/notification ke liye sahi hai. Rule: **data badalna ho → BEFORE. Side effect karna ho → AFTER.**

> **Cross-question: "Statement-level trigger row-level se kab better hai?"**
> Jab tumhe har row se matlab nahi, sirf "kuch hua" se matlab hai — jaise cache invalidate karna ya ek summary recompute karna. 10,000-row update par row-level trigger 10,000 baar chalega; statement-level ek baar. Bulk operations mein ye 100x farak hai.

---
---

# PART B — PRACTICE

---

# 26 — 30 SQL Practice Questions

> **Kaise practice karo:** har question padh kar **pehle khud likho** (kaagaz par bhi chalega), phir solution dekho. Solution dekh kar "haan yahi to socha tha" bolna practice nahi hai — wo dhoka hai. Aur **approach ka explanation** solution se zyada important hai, kyunki interview mein interviewer tumhe bolte hue sunna chahta hai.
>
> Sab questions Section 0 ke schema aur data par hain. Results **actual** hain.

---

## Q1 — Orders above ₹1,00,000, newest first  🟢

```sql
SELECT order_id, order_date, status, total_amount
FROM   orders
WHERE  is_deleted = false
  AND  total_amount > 100000
ORDER  BY order_date DESC;
```

| order_id | order_date | status | total_amount |
|---|---|---|---|
| 10 | 2025-04-15 | DELIVERED | 250000.00 |
| 9 | 2025-04-02 | DELIVERED | 500000.00 |
| 8 | 2025-03-19 | APPROVED | 180000.00 |
| 6 | 2025-03-03 | DELIVERED | 200000.00 |
| 3 | 2025-02-02 | APPROVED | 300000.00 |
| 2 | 2025-01-18 | DELIVERED | 450000.00 |
| 1 | 2025-01-05 | DELIVERED | 120000.00 |

**Approach:** *"Straight filter and sort. The one thing I'd add unprompted is the soft-delete filter — in a system with an is_deleted flag, forgetting it is the most common source of wrong counts. And if this feeds a paginated API I'd add order_id as a tie-breaker, because two orders on the same date would otherwise come back in an unstable order."*

---

## Q2 — Suppliers jinki rating set nahi hai  🟢

```sql
SELECT supplier_id, name, city, rating
FROM   suppliers
WHERE  rating IS NULL;
```

| supplier_id | name | city | rating |
|---|---|---|---|
| 3 | Nakoda Traders | Jaipur | NULL |
| 5 | Gupta Sanitary | Delhi | NULL |

**Approach:** *"IS NULL, never `= NULL` — comparison with NULL is UNKNOWN, and WHERE only passes TRUE, so `rating = NULL` returns zero rows every time without erroring. That silent-zero-rows behaviour is exactly why it's a good interview question."*

---

## Q3 — Har status ka order count aur value  🟢

```sql
SELECT status, COUNT(*) AS order_count, SUM(total_amount) AS total_value
FROM   orders
WHERE  is_deleted = false
GROUP  BY status
ORDER  BY total_value DESC;
```

| status | order_count | total_value |
|---|---|---|
| DELIVERED | 6 | 1595000.00 |
| APPROVED | 2 | 480000.00 |
| CANCELLED | 1 | 60000.00 |
| DRAFT | 2 | 45000.00 |

**Approach:** *"Basic GROUP BY. As a tester I'd check that the counts sum to the total row count — six plus two plus one plus two is eleven, and we have eleven non-deleted orders. If a status column were nullable, a NULL status would form its own group, and that's a group people forget exists."*

---

## Q4 — Projects jinka total order value ₹5,00,000 se zyada hai  🟢

```sql
SELECT o.project_id, p.name AS project,
       COUNT(*) AS order_count,
       SUM(o.total_amount) AS total_value
FROM   orders o
JOIN   projects p ON p.project_id = o.project_id
WHERE  o.is_deleted = false
GROUP  BY o.project_id, p.name
HAVING SUM(o.total_amount) > 500000
ORDER  BY total_value DESC;
```

| project_id | project | order_count | total_value |
|---|---|---|---|
| 1 | Skyline Towers | 5 | 1175000.00 |
| 2 | Metro Depot | 4 | 625000.00 |

**Approach:** *"The row-level condition — not deleted — goes in WHERE, and the aggregate condition goes in HAVING. Putting the is_deleted filter in HAVING would still be correct but slower, because it would group everything first and then discard groups. Also note p.name must be in the GROUP BY even though it's functionally dependent on project_id; Postgres allows omitting it if you group by the primary key, but I write it explicitly for portability."*

---

## Q5 — Saare suppliers, unke order count ke saath (zero wale bhi)  🟡

```sql
SELECT s.supplier_id, s.name,
       COUNT(o.order_id) AS order_count,
       COALESCE(SUM(o.total_amount), 0) AS total_value
FROM   suppliers s
LEFT   JOIN orders o ON o.supplier_id = s.supplier_id AND o.is_deleted = false
GROUP  BY s.supplier_id, s.name
ORDER  BY order_count DESC, s.supplier_id;
```

| supplier_id | name | order_count | total_value |
|---|---|---|---|
| 1 | Sharma Cement Co | 3 | 430000.00 |
| 2 | Verma Steels | 3 | 1250000.00 |
| 3 | Nakoda Traders | 2 | 245000.00 |
| 4 | Bombay Hardware | 1 | 75000.00 |
| 5 | Gupta Sanitary | 1 | 180000.00 |
| **6** | **Ajmer Aggregates** | **0** | **0.00** |

**Approach — do critical details, dono bolna:**
1. *"`COUNT(o.order_id)`, not `COUNT(*)`. With a LEFT JOIN, an unmatched supplier still produces one row with NULLs, so `COUNT(*)` would report 1 for Ajmer Aggregates instead of 0. COUNT of a nullable joined column gives the right answer."*
2. *"The `is_deleted = false` goes in the ON clause, not WHERE. In WHERE it would filter out the NULL row for Ajmer and turn the LEFT JOIN back into an INNER JOIN — the supplier with zero orders would vanish, which is the exact row we're trying to show."*

**Ye do points ek saath bolna is question ka poora jawab hai.**

---

## Q6 — Har order apne project aur supplier ke naam ke saath  🟢

```sql
SELECT o.order_id, p.name AS project,
       COALESCE(s.name, '(no supplier yet)') AS supplier,
       o.status, o.total_amount
FROM   orders o
JOIN   projects p  ON p.project_id  = o.project_id
LEFT   JOIN suppliers s ON s.supplier_id = o.supplier_id
WHERE  o.is_deleted = false
ORDER  BY o.order_id;
```

| order_id | project | supplier | status | total_amount |
|---|---|---|---|---|
| 1 | Skyline Towers | Sharma Cement Co | DELIVERED | 120000.00 |
| 2 | Skyline Towers | Verma Steels | DELIVERED | 450000.00 |
| 3 | Metro Depot | Verma Steels | APPROVED | 300000.00 |
| 4 | Metro Depot | Bombay Hardware | DELIVERED | 75000.00 |
| 5 | Skyline Towers | Sharma Cement Co | CANCELLED | 60000.00 |
| 6 | Green Villa | Nakoda Traders | DELIVERED | 200000.00 |
| 7 | Metro Depot | **(no supplier yet)** | DRAFT | 0.00 |
| 8 | Riverfront Mall | Gupta Sanitary | APPROVED | 180000.00 |
| 9 | Skyline Towers | Verma Steels | DELIVERED | 500000.00 |
| 10 | Metro Depot | Sharma Cement Co | DELIVERED | 250000.00 |
| 12 | Skyline Towers | Nakoda Traders | DRAFT | 45000.00 |

**Approach:** *"Two different join types in one query, deliberately. project_id is NOT NULL and has a foreign key, so an INNER JOIN can't lose rows — and if it ever did, that would itself be a data bug worth surfacing. supplier_id is nullable for draft POs, so it must be a LEFT JOIN, otherwise order 7 silently disappears. Choosing the join type per relationship based on nullability is the actual skill here."*

---

## Q7 — Duplicate invoice numbers dhoondo  🟡

**Ye interview ka evergreen question hai.**

```sql
SELECT invoice_no,
       COUNT(*)                    AS occurrences,
       ARRAY_AGG(invoice_id ORDER BY invoice_id) AS invoice_ids,
       MIN(invoice_id)             AS keep_this,
       ARRAY_AGG(order_id)         AS order_ids
FROM   invoices
GROUP  BY invoice_no
HAVING COUNT(*) > 1;
```

| invoice_no | occurrences | invoice_ids | keep_this | order_ids |
|---|---|---|---|---|
| INV-2025-002 | 2 | {2,7} | 2 | {2,2} |

**Poore duplicate rows dekhne ke liye (window function version — ye zyada powerful hai):**

```sql
SELECT *
FROM (
  SELECT i.*,
         COUNT(*)     OVER (PARTITION BY invoice_no) AS dup_count,
         ROW_NUMBER() OVER (PARTITION BY invoice_no ORDER BY invoice_id) AS rn
  FROM   invoices i
) t
WHERE dup_count > 1
ORDER BY invoice_no, rn;
```

| invoice_id | order_id | invoice_no | amount | status | dup_count | rn |
|---|---|---|---|---|---|---|
| 2 | 2 | INV-2025-002 | 450000.00 | PARTIALLY_PAID | 2 | 1 |
| 7 | 2 | INV-2025-002 | 450000.00 | PENDING | 2 | 2 |

**Approach:** *"GROUP BY the business key with HAVING COUNT greater than one is the classic form, and it's enough if you only need the list of duplicated values. But usually I need the full rows, so I prefer the window-function version: COUNT OVER PARTITION BY gives every row its group's size without collapsing anything, and ROW_NUMBER in the same pass tells me which one to keep. That single query is both the detection and the input to the cleanup.
> And the real finding here isn't the duplicate — it's that `invoice_no` has no unique constraint. Detecting duplicates is a symptom; the fix is a unique index, so they can't be created again."*

> **[REAL]** Ye bilkul Merlin ka `findByOrgAndEmailAddress` wala bug hai — code ne uniqueness assume ki, schema ne guarantee nahi ki. Interview mein Q7 ke baad turant ye story sunana.

---

## Q8 — Second highest order amount — 3 tareeke  🟡

**Answer: ₹4,50,000 (order 2).**

**Way 1 — DENSE_RANK (best, tie-aware):**

```sql
WITH ranked AS (
    SELECT order_id, total_amount,
           DENSE_RANK() OVER (ORDER BY total_amount DESC) AS dr
    FROM   orders WHERE is_deleted = false
)
SELECT order_id, total_amount FROM ranked WHERE dr = 2;
```

| order_id | total_amount |
|---|---|
| 2 | 450000.00 |

**Way 2 — Correlated MAX:**

```sql
SELECT MAX(total_amount) AS second_highest
FROM   orders
WHERE  is_deleted = false
  AND  total_amount < (SELECT MAX(total_amount) FROM orders WHERE is_deleted = false);
```

| second_highest |
|---|
| 450000.00 |

**Way 3 — DISTINCT + OFFSET:**

```sql
SELECT DISTINCT total_amount
FROM   orders WHERE is_deleted = false
ORDER  BY total_amount DESC
LIMIT  1 OFFSET 1;
```

| total_amount |
|---|
| 450000.00 |

**Approach — ye clarification pehle poochna, uske baad solution dena:**
> *"Before I write it — with ties, do you want the second distinct amount, or the second row? If three orders tie at the top, 'second highest' is ambiguous. DENSE_RANK gives you the second distinct value and every row at it; ROW_NUMBER gives you exactly one row regardless. I'd default to DENSE_RANK. The MAX-of-less-than-MAX version is neat and cheap but doesn't generalise to Nth without ugly nesting, and the LIMIT/OFFSET version needs the DISTINCT or ties break it — and it returns zero rows rather than NULL if there is no second value, which is a different thing for the caller to handle."*

---

## Q9 — Har supplier ke top 2 orders  🟡

```sql
WITH ranked AS (
    SELECT o.supplier_id, s.name AS supplier, o.order_id, o.total_amount,
           ROW_NUMBER() OVER (PARTITION BY o.supplier_id
                              ORDER BY o.total_amount DESC, o.order_id) AS rn
    FROM   orders o
    JOIN   suppliers s ON s.supplier_id = o.supplier_id
    WHERE  o.is_deleted = false
)
SELECT supplier, order_id, total_amount, rn
FROM   ranked
WHERE  rn <= 2
ORDER  BY supplier_id, rn;
```

| supplier | order_id | total_amount | rn |
|---|---|---|---|
| Sharma Cement Co | 10 | 250000.00 | 1 |
| Sharma Cement Co | 1 | 120000.00 | 2 |
| Verma Steels | 9 | 500000.00 | 1 |
| Verma Steels | 2 | 450000.00 | 2 |
| Nakoda Traders | 6 | 200000.00 | 1 |
| Nakoda Traders | 12 | 45000.00 | 2 |
| Bombay Hardware | 4 | 75000.00 | 1 |
| Gupta Sanitary | 8 | 180000.00 | 1 |

**Approach:** *"Top-N-per-group is the canonical window-function problem. PARTITION BY restarts the numbering per supplier, ORDER BY inside the window defines 'top'. The filter has to go outside the CTE because window functions are computed with SELECT and aren't available in WHERE.
> Two choices I'd call out: I use ROW_NUMBER, not RANK, because I want exactly two rows per supplier even under ties — RANK would return three if two orders tied for second. And I add order_id as a secondary sort key so ties break deterministically; without it the same query can return different rows on different runs, which makes an automated test flaky."*

---

## Q10 — Payments ka running total  🟡

```sql
SELECT payment_id, paid_on, amount,
       SUM(amount) OVER (ORDER BY paid_on, payment_id
                         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM   payments
ORDER  BY paid_on, payment_id;
```

| payment_id | paid_on | amount | running_total |
|---|---|---|---|
| 1 | 2025-01-25 | 120000.00 | 120000.00 |
| 2 | 2025-02-10 | 200000.00 | 320000.00 |
| 4 | 2025-03-01 | 75000.00 | 395000.00 |
| 3 | 2025-03-05 | 100000.00 | 495000.00 |
| 5 | 2025-04-20 | 250000.00 | 745000.00 |
| 6 | 2025-05-02 | 250000.00 | 995000.00 |
| 7 | 2025-05-10 | 100000.00 | 1095000.00 |

**Approach:** *"SUM as a window function with a frame that runs from the start of the partition to the current row. The detail that matters: with an ORDER BY, the default frame is RANGE, not ROWS, and RANGE groups tied sort keys together — so if two payments landed on the same date, the running total would jump by both at once on both rows. Writing ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW explicitly makes it row-by-row. I've seen a 'running balance' report that was subtly wrong for exactly this reason, and it only misbehaved on days with more than one transaction."*

---

## Q11 — Month-over-month change in order value  🔴

```sql
WITH monthly AS (
    SELECT DATE_TRUNC('month', order_date)::date AS month,
           COUNT(*)          AS order_count,
           SUM(total_amount) AS total_value
    FROM   orders
    WHERE  is_deleted = false
    GROUP  BY 1
)
SELECT month, order_count, total_value,
       LAG(total_value) OVER (ORDER BY month) AS prev_month_value,
       total_value - LAG(total_value) OVER (ORDER BY month) AS mom_change,
       ROUND(100.0 * (total_value - LAG(total_value) OVER (ORDER BY month))
             / NULLIF(LAG(total_value) OVER (ORDER BY month), 0), 2) AS mom_pct
FROM   monthly
ORDER  BY month;
```

| month | order_count | total_value | prev_month_value | mom_change | mom_pct |
|---|---|---|---|---|---|
| 2025-01-01 | 2 | 570000.00 | NULL | NULL | NULL |
| 2025-02-01 | 3 | 435000.00 | 570000.00 | -135000.00 | -23.68 |
| 2025-03-01 | 3 | 380000.00 | 435000.00 | -55000.00 | -12.64 |
| 2025-04-01 | 2 | 750000.00 | 380000.00 | 370000.00 | 97.37 |
| 2025-05-01 | 1 | 45000.00 | 750000.00 | -705000.00 | -94.00 |

**Approach — teen cheezein bolna:**
1. *"Aggregate to month first in a CTE, then apply LAG over the aggregated rows. Doing it in one step doesn't work — LAG operates on rows, and I need it to operate on months."*
2. *"NULLIF on the denominator. If a month had zero value, the percentage would be a division by zero; NULLIF turns the zero into NULL, and NULL divided by anything is NULL, so I get an empty cell instead of an error."*
3. *"This has a real gap: if a month had no orders at all, it just isn't in the result, so LAG would compare across the gap and silently understate the drop. If the report needs to show zero months, I'd generate the full month series and LEFT JOIN the aggregate onto it. That's a defect I'd file against a real report rather than a query I'd write differently — 'missing month' versus 'zero month' looks the same on the chart and means something completely different."*

**Ye teesra point sunkar interviewer ko lagta hai ki tumne asli reporting bug dekha hai.**

---

## Q12 — Invoice number sequence mein gaps  🔴

```sql
WITH nums AS (
    SELECT DISTINCT CAST(SUBSTRING(invoice_no FROM 10) AS INT) AS n
    FROM   invoices
    WHERE  invoice_no ~ '^INV-\d{4}-\d{3}$'
),
full_range AS (
    SELECT generate_series((SELECT MIN(n) FROM nums), (SELECT MAX(n) FROM nums)) AS n
)
SELECT f.n AS missing_seq,
       'INV-2025-' || LPAD(f.n::text, 3, '0') AS missing_invoice_no
FROM   full_range f
LEFT   JOIN nums x ON x.n = f.n
WHERE  x.n IS NULL
ORDER  BY f.n;
```

**Present numbers:** 1, 2, 3, 4, 5, 6, 9.

| missing_seq | missing_invoice_no |
|---|---|
| 7 | INV-2025-007 |
| 8 | INV-2025-008 |

**Alternative — LEAD se gap ranges (bade sequences par better):**

```sql
WITH nums AS (
    SELECT DISTINCT CAST(SUBSTRING(invoice_no FROM 10) AS INT) AS n FROM invoices
),
gaps AS (
    SELECT n AS gap_start_after,
           LEAD(n) OVER (ORDER BY n) AS next_n
    FROM   nums
)
SELECT gap_start_after + 1 AS gap_from,
       next_n - 1          AS gap_to,
       next_n - gap_start_after - 1 AS missing_count
FROM   gaps
WHERE  next_n - gap_start_after > 1;
```

| gap_from | gap_to | missing_count |
|---|---|---|
| 7 | 8 | 2 |

**Approach:** *"Two approaches with different cost profiles. The generate-series version materialises every number in the range and anti-joins — simple and readable, but if IDs run to ten million with a handful of gaps, you're building ten million rows to find five. The LEAD version only looks at the rows that exist and reports gap ranges, so it scales with the number of rows, not the size of the range. On a big table I'd use LEAD.
> The reason this question matters for a tester is that gaps in an invoice sequence are legally significant in many countries — regulators expect a continuous series, and a gap means either a voided document that must be recorded, or a lost record. So this isn't a puzzle; it's a compliance check I'd run nightly."*

---

## Q13 — Orders jinke koi line items nahi hain  🟡

**Teeno tareeke, aur kaunsa kyun:**

```sql
-- (a) LEFT JOIN + IS NULL
SELECT o.order_id, o.status, o.total_amount
FROM   orders o
LEFT   JOIN order_items i ON i.order_id = o.order_id
WHERE  o.is_deleted = false
  AND  i.item_id IS NULL;

-- (b) NOT EXISTS  ← production mein yahi likhna
SELECT o.order_id, o.status, o.total_amount
FROM   orders o
WHERE  o.is_deleted = false
  AND  NOT EXISTS (SELECT 1 FROM order_items i WHERE i.order_id = o.order_id);

-- (c) NOT IN  ← NULL-unsafe, avoid
SELECT o.order_id, o.status, o.total_amount
FROM   orders o
WHERE  o.is_deleted = false
  AND  o.order_id NOT IN (SELECT order_id FROM order_items);
```

| order_id | status | total_amount |
|---|---|---|
| 7 | DRAFT | 0.00 |
| **12** | **DRAFT** | **45000.00** |

**Approach:** *"This is the anti-join pattern. All three give the same answer here, but I'd write NOT EXISTS in production. NOT IN is the dangerous one: if the subquery ever returns a NULL, the whole predicate becomes UNKNOWN for every row and the query returns zero rows — silently, with no error. Right now order_items.order_id is NOT NULL so it's safe, but that's a schema property I'd have to keep checking. NOT EXISTS doesn't have that failure mode at all.
> The finding itself is the interesting part. Order 7 is an empty draft, which is legitimate. Order 12 is a draft with a stored total of forty-five thousand and no line items — the total doesn't come from anywhere. That's a data-integrity defect, and it's exactly the kind of thing I'd turn into a nightly assertion rather than finding by accident."*

---

## Q14 — Suppliers jinhone required materials mein se HAR EK supply kiya ho  🔴

**"Customers who ordered every product" ka asli naam: relational division.** Interview mein ye sabse tough SQL question maana jaata hai.

**Requirement: supplier ne `Cement OPC 53` aur `Sand` — dono supply kiye hon.**

**Way 1 — GROUP BY + HAVING COUNT (sabse readable, ye likhna):**

```sql
WITH required(material) AS (VALUES ('Cement OPC 53'), ('Sand'))
SELECT s.supplier_id, s.name,
       COUNT(DISTINCT i.material) AS matched
FROM   suppliers s
JOIN   orders o      ON o.supplier_id = s.supplier_id AND o.is_deleted = false
JOIN   order_items i ON i.order_id    = o.order_id
JOIN   required r    ON r.material    = i.material
GROUP  BY s.supplier_id, s.name
HAVING COUNT(DISTINCT i.material) = (SELECT COUNT(*) FROM required);
```

| supplier_id | name | matched |
|---|---|---|
| 1 | Sharma Cement Co | 2 |

**Verify:** Sharma Cement Co ne Cement (orders 1, 5, 10) aur Sand (orders 1, 10) dono diye ✅. Verma Steels ne Cement (order 9) diya par Sand kabhi nahi ❌.

**Way 2 — double NOT EXISTS (asli relational division, "there is no required material that this supplier has not supplied"):**

```sql
WITH required(material) AS (VALUES ('Cement OPC 53'), ('Sand'))
SELECT s.supplier_id, s.name
FROM   suppliers s
WHERE  NOT EXISTS (
         SELECT 1 FROM required r
         WHERE NOT EXISTS (
             SELECT 1
             FROM   orders o
             JOIN   order_items i ON i.order_id = o.order_id
             WHERE  o.supplier_id = s.supplier_id
               AND  o.is_deleted = false
               AND  i.material = r.material
         )
       );
```

| supplier_id | name |
|---|---|
| 1 | Sharma Cement Co |

**Aur agar poora catalogue maango — "har material jo kabhi kisi ne order kiya":**

```sql
SELECT s.supplier_id, s.name
FROM   suppliers s
WHERE  NOT EXISTS (
         SELECT 1 FROM (SELECT DISTINCT material FROM order_items) allm
         WHERE NOT EXISTS (
             SELECT 1 FROM orders o JOIN order_items i ON i.order_id=o.order_id
             WHERE o.supplier_id = s.supplier_id AND i.material = allm.material));
```

**Result: 0 rows.** Koi supplier saare 8 materials supply nahi karta. **Ye sahi answer hai** — empty result bhi ek valid answer hai.

**Approach:** *"This is relational division — 'find X that relates to every Y'. Two standard forms. The double-NOT-EXISTS is the literal translation of the logic: there is no required material for which there does not exist a supply by this supplier. It's the textbook answer and it short-circuits, so it can be efficient. But honestly the COUNT DISTINCT form is what I'd write and what I'd want to maintain — join to the required set, count how many distinct required items each supplier matched, and keep the ones whose count equals the size of the required set.
> Two traps. COUNT DISTINCT, not COUNT, because one supplier supplying cement on three separate orders must count once. And the required set has to be its own relation — hard-coding `IN (a, b)` with `HAVING COUNT(*) = 2` works until someone adds a third item and forgets the 2."*

---

## Q15 — Har project ka pehla aur aakhri order  🟡

**Way 1 — window functions (ek pass, best):**

```sql
WITH marked AS (
    SELECT o.project_id, p.name AS project, o.order_id, o.order_date, o.total_amount,
           ROW_NUMBER() OVER (PARTITION BY o.project_id ORDER BY o.order_date,  o.order_id)      AS rn_first,
           ROW_NUMBER() OVER (PARTITION BY o.project_id ORDER BY o.order_date DESC, o.order_id DESC) AS rn_last
    FROM   orders o JOIN projects p ON p.project_id = o.project_id
    WHERE  o.is_deleted = false
)
SELECT project,
       MAX(CASE WHEN rn_first=1 THEN order_id   END) AS first_order_id,
       MAX(CASE WHEN rn_first=1 THEN order_date END) AS first_order_date,
       MAX(CASE WHEN rn_first=1 THEN total_amount END) AS first_amount,
       MAX(CASE WHEN rn_last =1 THEN order_id   END) AS last_order_id,
       MAX(CASE WHEN rn_last =1 THEN order_date END) AS last_order_date,
       MAX(CASE WHEN rn_last =1 THEN total_amount END) AS last_amount
FROM   marked
GROUP  BY project_id, project
ORDER  BY project_id;
```

| project | first_order_id | first_order_date | first_amount | last_order_id | last_order_date | last_amount |
|---|---|---|---|---|---|---|
| Skyline Towers | 1 | 2025-01-05 | 120000.00 | 12 | 2025-05-06 | 45000.00 |
| Metro Depot | 3 | 2025-02-02 | 300000.00 | 10 | 2025-04-15 | 250000.00 |
| Green Villa | 6 | 2025-03-03 | 200000.00 | 6 | 2025-03-03 | 200000.00 |
| Riverfront Mall | 8 | 2025-03-19 | 180000.00 | 8 | 2025-03-19 | 180000.00 |

**Way 2 — Postgres `DISTINCT ON` (bahut chhota, par Postgres-only):**

```sql
SELECT f.project_id, f.order_id AS first_order, l.order_id AS last_order
FROM (SELECT DISTINCT ON (project_id) project_id, order_id
      FROM orders WHERE is_deleted=false ORDER BY project_id, order_date, order_id) f
JOIN (SELECT DISTINCT ON (project_id) project_id, order_id
      FROM orders WHERE is_deleted=false ORDER BY project_id, order_date DESC, order_id DESC) l
  USING (project_id);
```

**Way 3 — FIRST_VALUE / LAST_VALUE:**

```sql
SELECT DISTINCT project_id,
       FIRST_VALUE(order_id) OVER w AS first_order,
       LAST_VALUE(order_id)  OVER w AS last_order
FROM   orders
WHERE  is_deleted = false
WINDOW w AS (PARTITION BY project_id ORDER BY order_date, order_id
             ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING);
```

**Approach:** *"The naive approach is MIN and MAX of the date, then join back to find the matching rows — that's two extra scans and it breaks if two orders share the same date. Instead I number the rows both ascending and descending in one pass and pivot with conditional aggregation, so I get first and last in a single scan.
> The gotcha worth mentioning: LAST_VALUE with an ORDER BY defaults to a frame ending at the current row, so it returns the current row, not the last one. You have to widen the frame to UNBOUNDED FOLLOWING explicitly. That surprises almost everyone the first time. I use the named WINDOW clause when the same frame is reused, so the definition lives in one place."*

---

## Q16 — Invoices jo poori tarah paid nahi hain (outstanding ke saath)  🟡

```sql
SELECT i.invoice_id, i.invoice_no, i.status, i.amount,
       COALESCE(SUM(p.amount), 0)            AS paid,
       i.amount - COALESCE(SUM(p.amount), 0) AS outstanding
FROM   invoices i
LEFT   JOIN payments p ON p.invoice_id = i.invoice_id
GROUP  BY i.invoice_id, i.invoice_no, i.status, i.amount
HAVING i.amount - COALESCE(SUM(p.amount), 0) > 0
ORDER  BY outstanding DESC;
```

| invoice_id | invoice_no | status | amount | paid | outstanding |
|---|---|---|---|---|---|
| 7 | INV-2025-002 | PENDING | 450000.00 | 0.00 | 450000.00 |
| 4 | INV-2025-004 | PENDING | 200000.00 | 0.00 | 200000.00 |
| 8 | INV-2025-009 | PENDING | 180000.00 | 0.00 | 180000.00 |
| 2 | INV-2025-002 | PARTIALLY_PAID | 450000.00 | 300000.00 | 150000.00 |
| 5 | INV-2025-005 | PARTIALLY_PAID | 500000.00 | 350000.00 | 150000.00 |

**Total outstanding: ₹11,30,000.**

**Approach:** *"LEFT JOIN so invoices with zero payments still appear — those are the ones you most want in an ageing report, and an INNER JOIN would drop exactly them. COALESCE turns the NULL sum into zero so the arithmetic works; without it, outstanding would be NULL for unpaid invoices and the HAVING would reject them, which is the same bug the LEFT JOIN was meant to avoid.
> One thing I'd flag on this data: invoice 7 is the duplicate we found in Q7, so ₹4,50,000 of this 'outstanding' is phantom. An ageing report built on a table without a unique constraint on invoice number will overstate receivables — that's a finance-impacting bug, not a cosmetic one."*

---

## Q17 — Invoice ka stored status vs actual payments — mismatch  🔴

**Ye ek asli QA data-integrity check hai.**

```sql
WITH paid AS (
    SELECT i.invoice_id, i.invoice_no, i.amount, i.status AS stored_status,
           COALESCE(SUM(p.amount), 0) AS paid_total
    FROM   invoices i
    LEFT   JOIN payments p ON p.invoice_id = i.invoice_id
    GROUP  BY i.invoice_id, i.invoice_no, i.amount, i.status
)
SELECT invoice_id, invoice_no, amount, paid_total, stored_status,
       CASE WHEN paid_total = 0            THEN 'PENDING'
            WHEN paid_total >= amount      THEN 'PAID'
            ELSE                                'PARTIALLY_PAID' END AS expected_status
FROM   paid
WHERE  stored_status <> CASE WHEN paid_total = 0       THEN 'PENDING'
                             WHEN paid_total >= amount THEN 'PAID'
                             ELSE 'PARTIALLY_PAID' END;
```

**Result: 0 rows.**

**Approach — aur ye samjhaana ki 0 rows ka matlab kya hai:** *"Zero rows is the pass condition, not a failed query. This is an invariant check, and the whole value of it is that it runs every night and returns nothing — until the day a code path updates payments without recomputing the invoice status, and then it returns exactly the broken rows.
> The point I'd make in a design review is that the status column is derived data. It's denormalised, so it can drift from the payments it's derived from. That's a legitimate denormalisation for read performance, but it comes with an obligation: a check that asserts the derived value still matches its source. Any denormalised field without a drift check is a bug waiting for the right race condition."*

---

## Q18 — Business rule violation: invoice exists for a non-DELIVERED order  🟡

```sql
SELECT i.invoice_id, i.invoice_no, i.amount,
       o.order_id, o.status AS order_status
FROM   invoices i
JOIN   orders o ON o.order_id = i.order_id
WHERE  o.status <> 'DELIVERED'
ORDER  BY i.invoice_id;
```

| invoice_id | invoice_no | amount | order_id | order_status |
|---|---|---|---|---|
| 8 | INV-2025-009 | 180000.00 | 8 | **APPROVED** |

**Approach:** *"Order 8 has been invoiced but never delivered. Whether that's a bug depends on the business rule — some ERPs allow advance invoicing, some don't — so the first thing I'd do is confirm the rule with the product owner rather than filing it as a defect.
> But this is the shape of check I care about most: not 'is the data well-formed' but 'is the data consistent with the state machine'. Foreign keys and CHECK constraints can't express 'an invoice may only exist for a delivered order' across two tables, so nothing in the schema prevents it. Cross-table state rules like this have to be either enforced in the application and verified by a query, or enforced by a trigger. Either way QA should be running the query."*

---

## Q19 — Orders jahan stored total ≠ line items ka sum  🔴

**Ye is document ka sabse important QA query hai.**

```sql
SELECT o.order_id, o.status, o.total_amount AS stored_total,
       COALESCE(SUM(i.quantity * i.unit_price), 0) AS computed_total,
       o.total_amount - COALESCE(SUM(i.quantity * i.unit_price), 0) AS drift
FROM   orders o
LEFT   JOIN order_items i ON i.order_id = o.order_id
GROUP  BY o.order_id, o.status, o.total_amount
HAVING o.total_amount <> COALESCE(SUM(i.quantity * i.unit_price), 0)
ORDER  BY ABS(o.total_amount - COALESCE(SUM(i.quantity * i.unit_price), 0)) DESC;
```

| order_id | status | stored_total | computed_total | drift |
|---|---|---|---|---|
| 12 | DRAFT | 45000.00 | 0.00 | 45000.00 |

**Approach:** *"Order 12 claims forty-five thousand rupees with no line items backing it. Where did that number come from? Either items were deleted without recalculating the total, or the total was written before the items and the item write failed — which would point at a missing transaction boundary between the two writes.
> That second possibility is the one I'd chase, because it's a design bug, not a data bug. If creating an order and creating its items are two separate transactions, then any failure between them leaves this exact footprint. The test is straightforward: force the item insert to fail and assert the order doesn't exist either.
> I'd add the LEFT JOIN detail: with an INNER JOIN, order 12 wouldn't appear at all, because it has no items to join to — the very rows this check exists to find would be invisible. That's the trap in writing this check."*

---

## Q20 — Suppliers ko delivered value ke hisaab se rank karo  🟡

```sql
SELECT s.name AS supplier,
       COUNT(*)               AS delivered_orders,
       SUM(o.total_amount)    AS delivered_value,
       RANK()       OVER (ORDER BY SUM(o.total_amount) DESC) AS rnk,
       DENSE_RANK() OVER (ORDER BY SUM(o.total_amount) DESC) AS dense_rnk
FROM   orders o
JOIN   suppliers s ON s.supplier_id = o.supplier_id
WHERE  o.status = 'DELIVERED' AND o.is_deleted = false
GROUP  BY s.supplier_id, s.name
ORDER  BY rnk;
```

| supplier | delivered_orders | delivered_value | rnk | dense_rnk |
|---|---|---|---|---|
| Verma Steels | 2 | 950000.00 | 1 | 1 |
| Sharma Cement Co | 2 | 370000.00 | 2 | 2 |
| Nakoda Traders | 1 | 200000.00 | 3 | 3 |
| Bombay Hardware | 1 | 75000.00 | 4 | 4 |

**Approach:** *"A window function applied on top of a GROUP BY. That works because window functions are evaluated after grouping, so `SUM(o.total_amount)` inside the OVER clause refers to the group's sum, not to individual rows. People assume you need a CTE for this; you don't.
> RANK and DENSE_RANK are identical here because there are no ties in the totals. I'd show both so the difference is visible the moment two suppliers do tie — RANK would give 1, 2, 2, 4 and DENSE_RANK 1, 2, 2, 3."*

---

## Q21 — Har supplier ka % contribution to delivered value  🟡

```sql
SELECT s.name AS supplier,
       SUM(o.total_amount) AS delivered_value,
       SUM(SUM(o.total_amount)) OVER () AS grand_total,
       ROUND(100.0 * SUM(o.total_amount) / SUM(SUM(o.total_amount)) OVER (), 2) AS pct
FROM   orders o
JOIN   suppliers s ON s.supplier_id = o.supplier_id
WHERE  o.status = 'DELIVERED' AND o.is_deleted = false
GROUP  BY s.supplier_id, s.name
ORDER  BY pct DESC;
```

| supplier | delivered_value | grand_total | pct |
|---|---|---|---|
| Verma Steels | 950000.00 | 1595000.00 | 59.56 |
| Sharma Cement Co | 370000.00 | 1595000.00 | 23.20 |
| Nakoda Traders | 200000.00 | 1595000.00 | 12.54 |
| Bombay Hardware | 75000.00 | 1595000.00 | 4.70 |

**Percentages sum: 100.00** ✅ — ye check hamesha karo, rounding se aksar 99.99 ya 100.01 aata hai.

**Approach:** *"`SUM(SUM(x)) OVER ()` looks bizarre the first time but it's exactly right: the inner SUM is the group aggregate, the outer SUM is a window over all groups with an empty OVER clause, meaning the whole result set. That gives me the grand total on every row without a second query or a self-join.
> The tester's habit here: always assert the percentages sum to 100. Rounding each row independently often lands on 99.99 or 100.01, and on a pie chart that shows up as a visible sliver. If the total has to be exactly 100, you round the largest bucket last and give it the remainder."*

---

## Q22 — Employees jinki salary apne role ke average se zyada hai  🟡

**Way 1 — window function (ek pass):**

```sql
SELECT name, role, salary, role_avg
FROM (
    SELECT name, role, salary,
           ROUND(AVG(salary) OVER (PARTITION BY role), 2) AS role_avg
    FROM   users
) t
WHERE salary > role_avg
ORDER BY role, salary DESC;
```

| name | role | salary | role_avg |
|---|---|---|---|
| Sunita Rao | PM | 110000.00 | 100000.00 |
| Farhan Ali | SITE_ENGINEER | 75000.00 | 65000.00 |

**Way 2 — correlated subquery (classic form):**

```sql
SELECT u.name, u.role, u.salary
FROM   users u
WHERE  u.salary > (SELECT AVG(u2.salary) FROM users u2 WHERE u2.role = u.role);
```

**Approach:** *"Both are correct. The window version computes each role's average once and reuses it, so it's one pass over the table. The correlated subquery is logically re-evaluated per row — modern optimisers often rewrite it into the same thing, but on a large table without an index on role I'd expect the window version to be noticeably faster, and I'd confirm with EXPLAIN rather than asserting it.
> Note Anil Mehta is the only ADMIN, so his own salary is the ADMIN average and he isn't strictly greater than it — he's correctly excluded. Single-member groups are the boundary case worth checking in a query like this."*

---

## Q23 — Employees jinki salary apne manager se zyada hai  🟡

```sql
SELECT e.name AS employee, e.salary AS emp_salary,
       m.name AS manager,  m.salary AS mgr_salary,
       e.salary - m.salary AS excess
FROM   users e
JOIN   users m ON m.user_id = e.manager_id
WHERE  e.salary > m.salary;
```

**Result: 0 rows.**

Verify — har employee vs manager:

| employee | salary | manager | mgr salary | more? |
|---|---|---|---|---|
| Ritik Chaturvedi | 90000 | Anil Mehta | 250000 | no |
| Sunita Rao | 110000 | Anil Mehta | 250000 | no |
| Kabir Singh | 60000 | Ritik Chaturvedi | 90000 | no |
| Meera Nair | 60000 | Ritik Chaturvedi | 90000 | no |
| Farhan Ali | 75000 | Sunita Rao | 110000 | no |

**Approach:** *"Classic self join — the same table aliased twice, joined on manager_id to user_id. Note it's an INNER JOIN deliberately: Anil has no manager, and he can't earn more than a manager who doesn't exist, so excluding him is correct. If the question were 'list everyone with their manager's salary', it would have to be a LEFT JOIN or Anil would silently vanish from the list.
> Zero rows is the honest answer on this data. If Farhan were promoted to a hundred and twenty thousand, he'd appear against Sunita's hundred and ten. When a query legitimately returns nothing, I say so and show why, rather than bending the question — and in a test suite I'd seed a known violating row so the check is proven to be capable of failing. A check that has never failed is a check you don't know works."*

**Ye aakhri line — "a check that has never failed is a check you don't know works" — interview mein bolna. Ye pure senior QA thinking hai.**

---

## Q24 — Median order amount (aur quartiles)  🔴

```sql
SELECT PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY total_amount) AS p25,
       PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY total_amount) AS median,
       PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY total_amount) AS p75,
       AVG(total_amount)                                           AS mean
FROM   orders
WHERE  is_deleted = false;
```

Sorted amounts (11 values): `0, 45000, 60000, 75000, 120000, 180000, 200000, 250000, 300000, 450000, 500000`

| p25 | median | p75 | mean |
|---|---|---|---|
| 67500.00 | 180000.00 | 275000.00 | 198181.82 |

**Bina PERCENTILE_CONT ke (portable, window function se):**

```sql
WITH ranked AS (
    SELECT total_amount,
           ROW_NUMBER() OVER (ORDER BY total_amount) AS rn,
           COUNT(*)     OVER ()                      AS n
    FROM   orders WHERE is_deleted = false
)
SELECT AVG(total_amount) AS median
FROM   ranked
WHERE  rn IN ((n+1)/2, (n+2)/2);   -- odd n: same index twice; even n: middle two
```

| median |
|---|
| 180000.00 |

**Approach:** *"PERCENTILE_CONT is an ordered-set aggregate — the WITHIN GROUP clause supplies the sort. CONT interpolates between values; PERCENTILE_DISC returns an actual value from the data. For money I'd usually want DISC, since an interpolated amount is a figure that never existed.
> The reason I'd volunteer the median at all: mean is 198,182 and median is 180,000, and that gap is entirely the two large orders pulling the mean up. Any report that quotes 'average order value' on a skewed distribution is misleading, and that's worth raising with the product owner. It's a testing observation as much as a statistics one — if a threshold or alert is based on the mean, a couple of outliers can move it."*

---

## Q25 — Material-wise summary (qty, value, distinct suppliers)  🟡

```sql
SELECT i.material,
       SUM(i.quantity)                    AS total_qty,
       SUM(i.quantity * i.unit_price)     AS total_value,
       COUNT(DISTINCT i.order_id)         AS order_count,
       COUNT(DISTINCT o.supplier_id)      AS supplier_count,
       ROUND(AVG(i.unit_price), 2)        AS avg_unit_price
FROM   order_items i
JOIN   orders o ON o.order_id = i.order_id
WHERE  o.is_deleted = false
GROUP  BY i.material
ORDER  BY total_value DESC;
```

| material | total_qty | total_value | order_count | supplier_count | avg_unit_price |
|---|---|---|---|---|---|
| TMT Bar 12mm | 15000.00 | 975000.00 | 3 | 1 | 65.00 |
| Cement OPC 53 | 1125.00 | 450000.00 | 4 | 2 | 400.00 |
| Bricks | 40000.00 | 200000.00 | 1 | 1 | 5.00 |
| CP Fittings | 300.00 | 180000.00 | 1 | 1 | 600.00 |
| TMT Bar 16mm | 2000.00 | 125000.00 | 1 | 1 | 62.50 |
| Sand | 225.00 | 90000.00 | 2 | 1 | 400.00 |
| Plywood | 150.00 | 75000.00 | 1 | 1 | 500.00 |
| Binding Wire | 500.00 | 40000.00 | 1 | 1 | 40.00 → 80.00 |

**Total value: ₹21,35,000** — check karo: ye deleted order 11 (₹90,000) aur item-less orders 7, 12 ko chhod kar sab hai. 2,180,000 − 45,000 (order 12 ka phantom total) = 2,135,000 ✅

**Approach:** *"COUNT DISTINCT on two different columns in one pass — orders and suppliers per material. The business insight this surfaces immediately: TMT Bar 12mm is our largest spend and it comes from exactly one supplier. That's single-supplier concentration risk on the biggest line item, which is a genuine finding for a procurement product.
> A caution on `AVG(unit_price)`: that's the unweighted average of the price on each line, not the average price actually paid. If one order bought a tonne at sixty rupees and another bought a kilo at ninety, the simple average of seventy-five is meaningless. The correct figure is total value divided by total quantity. Reporting bugs of this exact shape — averaging an average, or averaging without weighting — are extremely common and almost never caught, because the number looks plausible."*

---

## Q26 — Consecutive orders ke beech 30 din se zyada ka gap  🔴

```sql
WITH gaps AS (
    SELECT o.project_id, p.name AS project, o.order_id, o.order_date,
           LAG(o.order_date) OVER (PARTITION BY o.project_id ORDER BY o.order_date) AS prev_date,
           o.order_date - LAG(o.order_date) OVER (PARTITION BY o.project_id ORDER BY o.order_date) AS gap_days
    FROM   orders o JOIN projects p ON p.project_id = o.project_id
    WHERE  o.is_deleted = false
)
SELECT project, order_id, prev_date, order_date, gap_days
FROM   gaps
WHERE  gap_days > 30
ORDER  BY gap_days DESC;
```

| project | order_id | prev_date | order_date | gap_days |
|---|---|---|---|---|
| Skyline Towers | 5 | 2025-01-18 | 2025-02-27 | 40 |
| Metro Depot | 10 | 2025-03-11 | 2025-04-15 | 35 |
| Skyline Towers | 9 | 2025-02-27 | 2025-04-02 | 34 |
| Skyline Towers | 12 | 2025-04-02 | 2025-05-06 | 34 |

**Approach:** *"LAG within a partition gives each row the previous row's date, and subtracting dates in Postgres gives an integer number of days. The first order of each project has a NULL previous date, so gap_days is NULL and the WHERE correctly excludes it — that's the NULL semantics working in our favour for once.
> Two things I'd raise. First, this only sees gaps between orders that exist; if a project stopped ordering entirely in April, there's no row to flag it. Detecting 'silence since' needs a comparison against today's date, not against the next row. That's a common blind spot in monitoring queries — they detect anomalies in the data present, not the absence of data. Second, if this becomes a real alert, thirty days is an arbitrary threshold that should be configurable, and I'd want to see what the normal distribution of gaps actually looks like before hard-coding it."*

---

## Q27 — Duplicate invoices delete karo, sabse purani rakho  🔴

**Pehle hamesha SELECT — dekho kya delete hoga:**

```sql
WITH ranked AS (
    SELECT invoice_id, invoice_no, order_id, amount, status,
           ROW_NUMBER() OVER (PARTITION BY invoice_no ORDER BY invoice_id) AS rn
    FROM   invoices
)
SELECT * FROM ranked WHERE rn > 1;   -- ← ye rows delete hongi
```

| invoice_id | invoice_no | order_id | amount | status | rn |
|---|---|---|---|---|---|
| 7 | INV-2025-002 | 2 | 450000.00 | PENDING | 2 |

**Ab delete — transaction ke andar:**

```sql
BEGIN;

WITH ranked AS (
    SELECT invoice_id,
           ROW_NUMBER() OVER (PARTITION BY invoice_no ORDER BY invoice_id) AS rn
    FROM   invoices
)
DELETE FROM invoices
WHERE invoice_id IN (SELECT invoice_id FROM ranked WHERE rn > 1)
RETURNING invoice_id, invoice_no;
-- → DELETE 1,  returned: (7, 'INV-2025-002')

-- Ab constraint lagao taaki dobara na ho
ALTER TABLE invoices ADD CONSTRAINT uq_invoice_no UNIQUE (invoice_no);

COMMIT;
```

**Alternative — Postgres `ctid` trick (jab koi id column hi na ho):**

```sql
DELETE FROM invoices a
USING  invoices b
WHERE  a.invoice_no = b.invoice_no
  AND  a.ctid > b.ctid;
```

**Approach — poora process bolna, sirf query nahi:**
> *"I'd never run this as a single statement. The sequence is: first run it as a SELECT to see exactly which rows go, and eyeball them — here it's one row, invoice seven. Then wrap the DELETE in an explicit transaction and use RETURNING, so I can confirm the count matches what the SELECT showed before I commit. If it says one, I commit; if it says fifty, I roll back and go find out why.
> The third step is the one people skip and it's the most important: add the unique constraint in the same transaction. Cleaning duplicates without adding the constraint means you'll be back doing this again next quarter, because whatever created them is still running.
> One caveat before deleting anything: check what references these rows. If payments pointed at invoice seven, deleting it either fails on the foreign key or cascades and destroys the payments. In a real cleanup I'd first reassign children to the surviving row, then delete the duplicate."*

---

## Q28 — Suppliers jinka average order value overall average se zyada hai  🟡

```sql
SELECT s.name AS supplier,
       COUNT(*) AS orders,
       ROUND(AVG(o.total_amount), 2) AS supplier_avg,
       (SELECT ROUND(AVG(total_amount), 2) FROM orders WHERE is_deleted = false) AS overall_avg
FROM   orders o
JOIN   suppliers s ON s.supplier_id = o.supplier_id
WHERE  o.is_deleted = false
GROUP  BY s.supplier_id, s.name
HAVING AVG(o.total_amount) > (SELECT AVG(total_amount) FROM orders WHERE is_deleted = false)
ORDER  BY supplier_avg DESC;
```

Har supplier ka average:

| supplier | orders | avg |
|---|---|---|
| Verma Steels | 3 | 416666.67 |
| Gupta Sanitary | 1 | 180000.00 |
| Sharma Cement Co | 3 | 143333.33 |
| Nakoda Traders | 2 | 122500.00 |
| Bombay Hardware | 1 | 75000.00 |

Overall average = **198181.82**. Result:

| supplier | orders | supplier_avg | overall_avg |
|---|---|---|---|
| Verma Steels | 3 | 416666.67 | 198181.82 |

**Approach:** *"A scalar subquery in HAVING, compared against the group's aggregate. The subtlety is what 'overall average' means: the average over all orders including the ones with no supplier — order 7 has a NULL supplier and an amount of zero, and it's in the denominator of the overall average but in nobody's group average. That's a real definitional question, not pedantry, and it'll change the answer. I'd confirm which one the business wants before writing it, and whichever way it goes I'd put the definition in a comment, because the next person will assume the other one.
> Also worth flagging: Gupta Sanitary's 'average' is one order. Any average over a single data point is noise, so a real ranking query needs a minimum-count filter — `HAVING COUNT(*) >= 3` — or the leaderboard is dominated by suppliers with one lucky order."*

---

## Q29 — Orphan / referential integrity checks  🟡

**Ye wo queries hain jo QA ko nightly chalani chahiye.**

```sql
-- 1. Order items pointing at a non-existent order
SELECT i.item_id, i.order_id
FROM   order_items i
LEFT   JOIN orders o ON o.order_id = i.order_id
WHERE  o.order_id IS NULL;

-- 2. Invoices pointing at a non-existent order
SELECT inv.invoice_id, inv.order_id
FROM   invoices inv
LEFT   JOIN orders o ON o.order_id = inv.order_id
WHERE  o.order_id IS NULL;

-- 3. Payments pointing at a non-existent invoice
SELECT p.payment_id, p.invoice_id
FROM   payments p
LEFT   JOIN invoices i ON i.invoice_id = p.invoice_id
WHERE  i.invoice_id IS NULL;

-- 4. Active children under a soft-deleted parent  ← FK ise NAHI pakadta
SELECT i.item_id, i.order_id, o.is_deleted
FROM   order_items i
JOIN   orders o ON o.order_id = i.order_id
WHERE  o.is_deleted = true;

-- 5. Negative ya zero amounts jahan nahi hone chahiye
SELECT 'order_items' AS src, item_id AS id FROM order_items WHERE quantity <= 0 OR unit_price < 0
UNION ALL
SELECT 'payments', payment_id FROM payments WHERE amount <= 0
UNION ALL
SELECT 'invoices', invoice_id FROM invoices WHERE amount < 0;

-- 6. Overpaid invoices
SELECT i.invoice_id, i.amount, SUM(p.amount) AS paid
FROM   invoices i JOIN payments p ON p.invoice_id = i.invoice_id
GROUP  BY i.invoice_id, i.amount
HAVING SUM(p.amount) > i.amount;

-- 7. Timestamp ordering violations
SELECT p.payment_id, p.paid_on, i.invoice_date
FROM   payments p JOIN invoices i ON i.invoice_id = p.invoice_id
WHERE  p.paid_on < i.invoice_date;      -- paid before invoiced?!
```

**Results:** 1, 2, 3, 5, 6, 7 → **0 rows** (schema mein FKs hain, CHECK constraints hain). **Check 4 → 1 row: item 15 under deleted order 11.**

| item_id | order_id | is_deleted |
|---|---|---|
| 15 | 11 | true |

**Approach:** *"Checks one to three return nothing precisely because we have foreign keys — so on this schema they're redundant, and I'd say that out loud rather than pretending they found something. Where they earn their place is a system without foreign keys: microservices with separate databases, sharded tables, or MongoDB, where nothing at the storage layer prevents an orphan. In our Mongo system there are no foreign keys at all, so these are the only thing standing between us and dangling references.
> Check four is the interesting one and it's the one people miss. A foreign key protects against a hard delete, but soft delete is just an UPDATE — the FK is perfectly happy, and now you have live child rows under a dead parent. Whether that's a bug depends on whether the soft delete is supposed to cascade, but either way nothing in the schema is enforcing it, so it has to be a query.
> Checks six and seven are business invariants rather than structural ones — you can't pay more than you were invoiced, and you can't pay before you were invoiced. Those cross tables, so no constraint can express them; a query is the only option."*

---

## Q30 — Pagination correctness check  🔴

**OFFSET pagination:**

```sql
SELECT order_id, order_date FROM orders WHERE is_deleted=false
ORDER BY order_date, order_id LIMIT 3 OFFSET 3;
```

| order_id | order_date |
|---|---|
| 4 | 2025-02-14 |
| 5 | 2025-02-27 |
| 6 | 2025-03-03 |

**Keyset pagination (same page, stable):**

```sql
-- page 1 ka last row tha: (2025-02-02, order_id 3)
SELECT order_id, order_date FROM orders
WHERE  is_deleted = false
  AND  (order_date, order_id) > (DATE '2025-02-02', 3)
ORDER  BY order_date, order_id
LIMIT  3;
```

Same 3 rows — par ye **index seek** hai, aur beech mein insert hone par bhi rows skip ya repeat nahi hongi.

**QA ka test — saare pages jodo aur verify karo:**

```python
def test_pagination_returns_every_row_exactly_once(api, db):
    expected_total = db.scalar("SELECT COUNT(*) FROM orders WHERE is_deleted=false")

    seen, page, guard = [], 1, 0
    while True:
        guard += 1
        assert guard < 1000, "pagination did not terminate"
        rows = api.get(f"/orders?page={page}&size=3").json()["items"]
        if not rows:
            break
        seen.extend(r["orderId"] for r in rows)
        page += 1

    assert len(seen) == len(set(seen)), \
        f"Duplicates across pages: {[x for x in set(seen) if seen.count(x) > 1]}"
    assert len(seen) == expected_total, \
        f"Paged {len(seen)} rows but table has {expected_total} — rows were skipped"
```

**Approach:** *"Two assertions, and both are needed. No duplicates catches rows being served twice; the count matching the total catches rows being skipped entirely — and skipped rows are the dangerous one, because nothing on screen looks wrong. A user just never sees an order.
> The guard counter matters too: if the API has an off-by-one and never returns an empty page, the loop runs forever and the test hangs rather than failing, which in CI reads as an infrastructure problem instead of a bug.
> The stronger version of this test runs the pagination loop while a second thread inserts rows, which is what exposes OFFSET drift. With offset pagination that test fails; with keyset pagination it passes. I'd use it as the evidence when arguing for keyset, rather than arguing from theory."*

---

## Practice questions — quick index

| # | Question | Concept | Level |
|---|---|---|---|
| 1 | Orders above amount, newest first | WHERE + ORDER BY | 🟢 |
| 2 | Suppliers with no rating | IS NULL | 🟢 |
| 3 | Count by status | GROUP BY | 🟢 |
| 4 | Projects above total | GROUP BY + HAVING | 🟢 |
| 5 | All suppliers incl. zero orders | LEFT JOIN + COUNT(col) | 🟡 |
| 6 | Orders with project + supplier | INNER vs LEFT in one query | 🟢 |
| 7 | **Find duplicates** | GROUP BY HAVING / window | 🟡 |
| 8 | **Second highest** — 3 ways | DENSE_RANK / MAX / OFFSET | 🟡 |
| 9 | **Top N per group** | ROW_NUMBER + PARTITION BY | 🟡 |
| 10 | **Running total** | SUM OVER with ROWS frame | 🟡 |
| 11 | **Month-over-month change** | LAG + NULLIF | 🔴 |
| 12 | **Gaps in sequence** | generate_series / LEAD | 🔴 |
| 13 | **Orders with no items** | anti-join, 3 ways | 🟡 |
| 14 | **Supplied every material** | relational division | 🔴 |
| 15 | **First and last per group** | double ROW_NUMBER | 🟡 |
| 16 | Unpaid invoices + outstanding | LEFT JOIN + COALESCE | 🟡 |
| 17 | Status vs payments mismatch | derived-data drift check | 🔴 |
| 18 | Invoice on non-delivered order | cross-table state rule | 🟡 |
| 19 | Stored total vs items sum | integrity check | 🔴 |
| 20 | Rank suppliers | RANK over GROUP BY | 🟡 |
| 21 | % of grand total | SUM(SUM()) OVER () | 🟡 |
| 22 | Above role average | window vs correlated | 🟡 |
| 23 | Earn more than manager | self join | 🟡 |
| 24 | Median and quartiles | PERCENTILE_CONT | 🔴 |
| 25 | Material summary | COUNT DISTINCT, weighted avg trap | 🟡 |
| 26 | Gaps between orders | LAG on dates | 🔴 |
| 27 | Delete duplicates safely | CTE + DELETE + constraint | 🔴 |
| 28 | Above overall average | scalar subquery in HAVING | 🟡 |
| 29 | Orphan / integrity checks | anti-joins, invariants | 🟡 |
| 30 | Pagination correctness | keyset vs offset | 🔴 |

---
---

# PART C — MONGODB

> **Yahan tum sabse strong ho.** SQL tumne padh kar seekha hai; Mongo tumne **production mein dekha hai**. Interview mein jab Mongo ka sawaal aaye, **definition mat do — apna example do**. "Our purchase order collection has a compound index on orgId, isDeleted and status, and here's why the order is that way" — ye ek line 20 minute ki theory se bhaari hai.

---

# 27 — Document model vs relational

## 27.1 Fundamental difference

**Relational:** data ko **normalized tables** mein todo, query time par **JOIN** se jodo.
**Document:** jo cheezein **saath padhi jaati hain, unhe saath store karo** — ek document mein.

**Same purchase order, dono models mein:**

```
RELATIONAL — 2 tables, join on read
┌──────────────────────────────────────┐   ┌────────────────────────────────────┐
│ orders                               │   │ order_items                        │
│ order_id | project_id | total_amount │   │ item_id|order_id|material |qty|price│
│    1     |     1      |   120000     │◄──│   1    |   1    |Cement   |200|400 │
└──────────────────────────────────────┘   │   2    |   1    |Sand     |100|400 │
                                            └────────────────────────────────────┘

DOCUMENT — 1 document, no join
{
  _id: ObjectId("665a1f0c8d2e4b0012a3c101"),
  poNumber: "PO-2025-0001",
  totalAmount: NumberDecimal("120000.00"),
  lineItems: [                                    ← embedded array
    { material: "Cement OPC 53", quantity: 200, unitPrice: NumberDecimal("400.00") },
    { material: "Sand",          quantity: 100, unitPrice: NumberDecimal("400.00") }
  ]
}
```

## 27.2 Merlin ka actual document shape

> **[REAL]** Tumhare codebase mein purchase order kuch aisa dikhta hai:

```javascript
{
  _id:        ObjectId("665a1f0c8d2e4b0012a3c101"),
  org:        { $ref: "organisations", $id: ObjectId("64f0aa11bb22cc33dd44ee55") },  // DBRef
  orgId:      ObjectId("64f0aa11bb22cc33dd44ee55"),          // ← DENORMALIZED scalar
  poNumber:   "PO-2025-0001",
  projectId:  ObjectId("64f0bb22cc33dd44ee55ff66"),
  supplierId: ObjectId("64f0cc33dd44ee55ff66aa77"),
  status:     "DELIVERED",
  orderDate:  ISODate("2025-01-05T00:00:00Z"),
  currency:   "INR",
  totalAmount: NumberDecimal("120000.00"),
  lineItems: [
    { material: "Cement OPC 53", quantity: 200, unitPrice: NumberDecimal("400.00") },
    { material: "Sand",          quantity: 100, unitPrice: NumberDecimal("400.00") }
  ],
  isDeleted:  false,
  createdAt:  ISODate("2025-01-05T09:14:22Z"),
  updatedAt:  ISODate("2025-01-20T11:02:41Z")
}
```

**Do cheezein har interview mein bolne layak:**

1. **`org` DBRef hai, aur `orgId` uska denormalized scalar copy hai.** Kyun? Kyunki **index `orgId` par lagta hai**, DBRef par efficiently nahi. Aur multi-tenant system mein **har single query** `orgId` par filter karti hai — toh wo har compound index ka **pehla column** hona chahiye.
2. **`isDeleted` — soft delete.** Har query mein `orgId + isDeleted` dono chahiye. Ek query mein bhoolna = deleted data leak ya cross-tenant leak.

## 27.3 Embed vs Reference — decision framework

| Embed karo jab | Reference karo jab |
|---|---|
| Child hamesha parent ke saath padha jaata hai | Child independently query hota hai |
| Child ka parent ke bina wajood nahi (line items) | Child kai parents se share hota hai (supplier) |
| Array **bounded** hai (10s, 100s — hazaaron nahi) | Array **unbounded** grow karta hai (audit log) |
| Ek saath update hote hain (atomicity free mil jaati hai) | Alag-alag update hote hain |
| Document 16 MB limit ke andar rahega | Document limit cross kar sakta hai |

**16 MB document limit** — ye Mongo ki hard limit hai, aur ye **design constraint** hai. "Har order ke andar uske saare audit events embed kar do" ek din 16 MB pe crash karega. **Ye QA ka test hai:** unbounded array wale fields par volume test karo.

**Anti-pattern jo QA ko pakadna chahiye: the massive array.**
```javascript
// ❌ Har din grow hota rahega, kabhi rukega nahi
{ _id: ..., projectName: "Skyline", auditLog: [ /* 40,000 entries */ ] }
```
Test: 10,000 events push karke dekho — document size, update latency, aur index behaviour.

## 27.4 Kab kaunsa model sahi hai

| Scenario | Better fit | Kyun |
|---|---|---|
| Har request ek "aggregate" padhta hai (order + items) | **Document** | ek read, koi join nahi |
| Bahut saare ad-hoc analytical joins | **Relational** | joins hi kaam hai |
| Schema tez badal raha hai / per-tenant variation | **Document** | flexible schema |
| Multi-row financial invariants (ledger balancing) | **Relational** | transactions + constraints native |
| Very high write throughput, horizontally sharded | **Document** | sharding built-in |
| Strong referential integrity zaroori | **Relational** | Mongo mein FK hai hi nahi |
| Complex reporting / BI | **Relational** | SQL + window functions |

> **Interview answer:**
> "The relational model normalises data into tables and reassembles it at query time with joins. The document model stores together what's read together — so a purchase order and its line items are one document, and reading the whole order is a single read with no join.
> Our system is MongoDB, and the trade-off shows up concretely. Line items are embedded, because they're bounded, they're always read with the order, and updating an order and its items is atomic for free — a single document write is atomic in MongoDB regardless of transactions. Suppliers are referenced, not embedded, because a supplier is shared across many orders and is queried on its own.
> The two costs I'd name honestly. There are no foreign keys, so nothing at the storage layer stops a dangling reference — that has to be a nightly integrity check instead. And there's a sixteen-megabyte document limit, so any embedded array that grows without bound is a future outage. I test that specifically by pushing a large volume of child entries and watching document size and update latency, rather than assuming the design holds."

> **Cross-question: "Mongo 'schemaless' hai — matlab schema hai hi nahi?"**
> **Schemaless nahi, schema-flexible.** Schema application code mein hai (Kotlin data classes, Spring Data documents), sirf DB enforce nahi karta. Practically ek collection mein **kai schema versions ek saath** rehte hain — purane documents mein naya field hai hi nahi. **Ye QA ka bada test area hai:** naya field add hua, purane documents par `null` aayega — kya code handle karta hai? Mongo `$jsonSchema` validators support karta hai, aur main strongly recommend karunga ki core collections par lage.

---

# 28 — MongoDB CRUD

## 28.1 Create

```javascript
db.purchaseOrders.insertOne({
  orgId: ObjectId("64f0aa11bb22cc33dd44ee55"),
  poNumber: "PO-2025-0013",
  projectId: ObjectId("64f0bb22cc33dd44ee55ff66"),
  status: "DRAFT",
  orderDate: ISODate("2025-06-01"),
  totalAmount: NumberDecimal("0.00"),
  lineItems: [],
  isDeleted: false,
  createdAt: new Date()
})
// → { acknowledged: true, insertedId: ObjectId("6660aa...") }

db.purchaseOrders.insertMany([ {...}, {...} ], { ordered: false })
```

`ordered: false` — ek document fail ho toh baaki continue karo (default `true` = pehle error par ruk jao). **Bulk import testing mein ye difference important hai.**

## 28.2 Read

```javascript
// Sab (org-scoped — hamesha!)
db.purchaseOrders.find({ orgId: ORG, isDeleted: false })

// Ek
db.purchaseOrders.findOne({ _id: ObjectId("665a1f0c8d2e4b0012a3c101") })

// Projection — sirf ye fields (1 = include, 0 = exclude)
db.purchaseOrders.find(
  { orgId: ORG, isDeleted: false, status: "DELIVERED" },
  { poNumber: 1, totalAmount: 1, orderDate: 1, _id: 0 }
)

// Sort + pagination
db.purchaseOrders.find({ orgId: ORG, isDeleted: false })
                 .sort({ orderDate: -1, _id: -1 })
                 .skip(20).limit(10)

// Count
db.purchaseOrders.countDocuments({ orgId: ORG, isDeleted: false })   // accurate, respects filter
db.purchaseOrders.estimatedDocumentCount()                          // fast, metadata se, filter nahi
```

**`countDocuments` vs `estimatedDocumentCount`** — ye poocha jaata hai. Pehla actual scan/index count karta hai aur filter leta hai; doosra collection metadata se aata hai, **instant hai par filter nahi le sakta aur sharded cluster par galat ho sakta hai**. Dashboard ke totals ke liye `countDocuments`.

**`skip` bilkul SQL `OFFSET` jaisa hai — same scale problem.** Bade collections par cursor-based pagination (`_id` ya `createdAt` par `$gt`) use karo.

## 28.3 Update

```javascript
// $set — field set karo (ya naya banao)
db.purchaseOrders.updateOne(
  { _id: PO_ID, orgId: ORG },
  { $set: { status: "APPROVED", updatedAt: new Date() } }
)
// → { matchedCount: 1, modifiedCount: 1 }

// $inc — atomic increment (read-modify-write se bacho!)
db.purchaseOrders.updateOne(
  { _id: PO_ID, orgId: ORG },
  { $inc: { revisionCount: 1 } }
)

// $push — array mein add karo
db.purchaseOrders.updateOne(
  { _id: PO_ID, orgId: ORG },
  { $push: { lineItems: { material: "Bricks", quantity: 5000, unitPrice: NumberDecimal("5.00") } } }
)

// $pull — array se hatao
db.purchaseOrders.updateOne({ _id: PO_ID }, { $pull: { lineItems: { material: "Sand" } } })

// $addToSet — add karo agar already nahi hai
db.purchaseOrders.updateOne({ _id: PO_ID }, { $addToSet: { tags: "URGENT" } })

// $unset — field hatao
db.purchaseOrders.updateOne({ _id: PO_ID }, { $unset: { supplierId: "" } })

// updateMany
db.purchaseOrders.updateMany(
  { orgId: ORG, status: "DRAFT", orderDate: { $lt: ISODate("2025-01-01") } },
  { $set: { status: "EXPIRED" } }
)

// Positional operator — matched array element update karo
db.purchaseOrders.updateOne(
  { _id: PO_ID, "lineItems.material": "Cement OPC 53" },
  { $set: { "lineItems.$.unitPrice": NumberDecimal("420.00") } }
)

// Upsert
db.suppliers.updateOne(
  { orgId: ORG, gstin: "08AAACS1234F1Z5" },
  { $set: { name: "Sharma Cement Co", city: "Jaipur" },
    $setOnInsert: { createdAt: new Date() } },
  { upsert: true }
)
```

**`matchedCount` vs `modifiedCount` — ye difference QA ke liye critical hai:**

| Case | matched | modified | Matlab |
|---|---|---|---|
| Document mila, value badli | 1 | 1 | normal update |
| Document mila, value **already same thi** | 1 | **0** | no-op — bug nahi |
| **Document mila hi nahi** | **0** | 0 | **filter galat, ya kisi aur org ka doc** |

**Assertion `modifiedCount == 1` likhna galat hai** — idempotent retry par wo 0 dega aur test flaky ho jayega. **`matchedCount == 1` assert karo.**

## 28.4 Delete

```javascript
db.purchaseOrders.deleteOne({ _id: PO_ID, orgId: ORG })
db.purchaseOrders.deleteMany({ orgId: ORG, status: "DRAFT", isDeleted: true })

// Soft delete — Merlin ka actual pattern
db.purchaseOrders.updateOne(
  { _id: PO_ID, orgId: ORG },
  { $set: { isDeleted: true, deletedAt: new Date(), deletedBy: USER_ID } }
)
```

## 28.5 Findings for QA — har write query mein `orgId` hona chahiye

> **[REAL]** Ye Merlin ka **sabse important security test** hai:

```javascript
// ❌ Cross-tenant write ka darwaza khula hai
db.purchaseOrders.updateOne({ _id: PO_ID }, { $set: { status: "APPROVED" } })

// ✅ orgId filter mein — doosre org ka _id bhejo toh matchedCount = 0
db.purchaseOrders.updateOne({ _id: PO_ID, orgId: currentOrgId }, { $set: {...} })
```

**Test likhna:**

```python
def test_cannot_update_other_orgs_purchase_order(api, org_a_token, org_b_po_id):
    r = api.patch(f"/purchase-orders/{org_b_po_id}",
                  json={"status": "APPROVED"},
                  headers={"Authorization": org_a_token})
    assert r.status_code in (403, 404), \
        f"Cross-tenant write allowed! Got {r.status_code}"
    # Aur DB se confirm karo ki sach mein kuch nahi badla
    doc = mongo.purchaseOrders.find_one({"_id": ObjectId(org_b_po_id)})
    assert doc["status"] != "APPROVED", "Data changed despite error response"
```

**Doosri assertion — "error mila lekin data phir bhi badla" — bahut kam log check karte hain.** Ye interview mein bolne layak hai.

> **Interview answer:**
> "MongoDB CRUD is insertOne and insertMany, find and findOne, updateOne and updateMany with operators like $set, $inc, $push and $pull, and deleteOne and deleteMany.
> The operator choice matters for correctness, not just style: $inc is an atomic server-side increment, so it can't lose an update the way a read-modify-write in application code can. That's the same lost-update problem as in SQL, and $inc is the MongoDB answer to it.
> Two things I check as a tester. First, the result object distinguishes matchedCount from modifiedCount — matched zero means the filter didn't find anything, which in a multi-tenant system usually means you're pointing at another tenant's document, while matched one and modified zero just means the value was already what you set. I assert on matchedCount, because asserting on modifiedCount makes an idempotent retry look like a failure. Second, in our multi-tenant system every read and every write must carry the tenant ID in the filter, not just the document ID — otherwise knowing an ObjectId is enough to modify another customer's data. I test that by taking a document ID from tenant B and calling the API as tenant A, and I assert both on the response code and on the document actually being unchanged."

---

# 29 — MongoDB query operators

## 29.1 Reference table

| Operator | Matlab | Example |
|---|---|---|
| `$eq` | equal (implicit) | `{status: "DRAFT"}` = `{status: {$eq: "DRAFT"}}` |
| `$ne` | not equal | `{status: {$ne: "CANCELLED"}}` |
| `$gt` `$gte` `$lt` `$lte` | comparison | `{totalAmount: {$gte: 100000}}` |
| `$in` | list mein se koi | `{status: {$in: ["APPROVED","DELIVERED"]}}` |
| `$nin` | list mein nahi | `{status: {$nin: ["DRAFT"]}}` |
| `$exists` | field hai ya nahi | `{supplierId: {$exists: false}}` |
| `$type` | BSON type check | `{orgId: {$type: "objectId"}}` |
| `$regex` | pattern | `{poNumber: {$regex: "^PO-2025-"}}` |
| `$and` `$or` `$nor` | logical | `{$or: [{status:"DRAFT"}, {totalAmount: 0}]}` |
| `$not` | negate ek operator | `{rating: {$not: {$gte: 4}}}` |
| `$expr` | do fields ko compare karo | `{$expr: {$gt: ["$paidAmount", "$totalAmount"]}}` |
| `$elemMatch` | array element **saari** conditions match kare | neeche |
| `$size` | array ki length | `{lineItems: {$size: 0}}` |
| `$all` | array mein ye saare values hon | `{tags: {$all: ["URGENT","CIVIL"]}}` |

## 29.2 Worked examples

```javascript
// Delivered orders above 1 lakh, is org ke
db.purchaseOrders.find({
  orgId: ORG, isDeleted: false,
  status: "DELIVERED",
  totalAmount: { $gte: NumberDecimal("100000") }
})
// → PO-2025-0001 (120000), 0002 (450000), 0006 (200000), 0009 (500000), 0010 (250000)
//   PO-2025-0004 (75000) nahi aaya — 1 lakh se kam
```

```javascript
// Date range — half-open, hamesha
db.purchaseOrders.find({
  orgId: ORG, isDeleted: false,
  orderDate: { $gte: ISODate("2025-02-01"), $lt: ISODate("2025-03-01") }
})
// → PO-0003, PO-0004, PO-0005  (February ke 3 orders)
```

```javascript
// Draft ya zero-amount orders
db.purchaseOrders.find({
  orgId: ORG,
  $or: [ { status: "DRAFT" }, { totalAmount: NumberDecimal("0") } ]
})
```

```javascript
// PO number prefix — indexed prefix regex (anchored), efficient
db.purchaseOrders.find({ orgId: ORG, poNumber: { $regex: "^PO-2025-" } })

// ⚠️ Un-anchored regex — index use NAHI hoga, collection scan
db.purchaseOrders.find({ poNumber: { $regex: "2025" } })
```

## 29.3 `$exists` vs `null` — Mongo ka NULL trap

**SQL ke NULL trap ka Mongo version — aur ye zyada khatarnak hai kyunki Mongo mein "field hi nahi hai" ek teesri state hai.**

```javascript
// Teen alag documents:
{ _id: 1, supplierId: ObjectId("...") }   // field hai, value hai
{ _id: 2, supplierId: null }               // field hai, value null
{ _id: 3 }                                 // field hai hi nahi
```

| Query | Match karega |
|---|---|
| `{supplierId: null}` | **{_id:2} AUR {_id:3}** ← dono! |
| `{supplierId: {$exists: false}}` | sirf {_id:3} |
| `{supplierId: {$exists: true, $eq: null}}` | sirf {_id:2} |
| `{supplierId: {$ne: null}}` | sirf {_id:1} |
| `{supplierId: {$ne: SOME_ID}}` | **{_id:2} aur {_id:3} bhi** ← missing fields include! |

**`{field: null}` dono cases match karta hai** — ye Mongo ki sabse confusing behaviour hai aur interview mein poochi jaati hai.

**QA ke liye kyun matter karta hai:** schema evolve hota hai. Naya field `currency` add hua — **purane 50,000 documents mein wo field hai hi nahi**. Ab:
- `{currency: {$ne: "USD"}}` un purane documents ko **include** karega
- `{currency: "INR"}` unhe **exclude** karega
- Report ke do numbers alag aayenge, aur dono "sahi" lagenge

**Test:** migration ke baad `db.collection.countDocuments({newField: {$exists: false}})` chalao — zero hona chahiye, warna backfill adhoora hai.

## 29.4 `$elemMatch` — array ka trap

```javascript
// Documents:
// A: lineItems: [{material:"Cement", quantity: 200}, {material:"Sand", quantity: 50}]
// B: lineItems: [{material:"Cement", quantity: 50},  {material:"Sand", quantity: 200}]

// ❌ Galat: alag-alag elements match kar sakte hain!
db.purchaseOrders.find({ "lineItems.material": "Cement", "lineItems.quantity": { $gt: 100 } })
// → A aur B DONO. B mein "Cement" hai (qty 50) aur koi element qty>100 hai (Sand 200).
//   Conditions alag elements par match ho gayi.

// ✅ Sahi: EK HI element dono conditions satisfy kare
db.purchaseOrders.find({
  lineItems: { $elemMatch: { material: "Cement", quantity: { $gt: 100 } } }
})
// → sirf A
```

**Ye ek asli production bug pattern hai** aur bahut kam QA log ise jaante hain. Interview mein bolna:

> *"Without $elemMatch, multiple conditions on an array field can be satisfied by different elements of the array. So a filter meaning 'has a cement line over a hundred units' silently matches an order that has a small cement line and a large sand line. $elemMatch forces all the conditions onto a single element. Any time I see two conditions on the same array path in a query, I check whether it should be $elemMatch — it's a quiet correctness bug that returns extra rows rather than an error."*

## 29.5 `$expr` — do fields compare karna

Normal Mongo query ek field ko ek **constant** se compare karti hai. Do fields compare karne ke liye `$expr` chahiye:

```javascript
// Overpaid invoices — paidAmount > totalAmount
db.invoices.find({ orgId: ORG, $expr: { $gt: ["$paidAmount", "$totalAmount"] } })

// orgId aur DBRef ka drift check — humara [REAL] wala
db.purchaseOrders.find({
  $expr: { $ne: ["$orgId", "$org.$id"] }
})
```

`$expr` **index use nahi kar sakta** (zyadatar cases mein) — collection scan hoga. Isliye ye **audit/nightly check** ke liye theek hai, hot path ke liye nahi.

> **Interview answer:**
> "The query operators map fairly directly to SQL — $eq, $ne, comparison operators, $in and $nin, $regex for LIKE, $and and $or. Two are specific to the document model and both cause real bugs.
> First, $exists. In MongoDB a field can be present with a null value, or absent entirely, and those are different states — but a query for `field: null` matches both. So after a schema change, documents written before the new field existed behave differently from documents where it was explicitly set to null. My standard post-migration check is a count of documents where the new field doesn't exist; it should be zero, and if it isn't, the backfill is incomplete.
> Second, $elemMatch. If you put two conditions on the same array path without it, they can be satisfied by two different elements of the array — so a filter that reads as 'a cement line over a hundred units' will also match an order with a small cement line and a large sand line. It returns too many documents, silently. Whenever I see two predicates on one array field, I check whether it should be $elemMatch."

---

# 30 — Aggregation pipeline

## 30.1 Kya hai

Pipeline = **stages ki list**, jahan har stage ka output agle stage ka input hota hai. Ye Mongo ka `GROUP BY` + `JOIN` + `ORDER BY` + sab kuch hai.

```
documents ──► $match ──► $group ──► $sort ──► $limit ──► result
              filter     aggregate   sort      cut
```

## 30.2 Core stages

| Stage | SQL equivalent | Kaam |
|---|---|---|
| `$match` | `WHERE` | filter |
| `$group` | `GROUP BY` | aggregate |
| `$project` | `SELECT` | fields choose/compute karo |
| `$addFields` / `$set` | computed column | field add karo, baaki rakho |
| `$sort` | `ORDER BY` | sort |
| `$limit` / `$skip` | `LIMIT` / `OFFSET` | cut |
| `$lookup` | `LEFT JOIN` | doosri collection se jodo |
| `$unwind` | array ko rows mein todo | (SQL mein direct nahi) |
| `$count` | `COUNT(*)` | ginti |
| `$facet` | ek pass mein kai pipelines | sub-queries parallel |
| `$out` / `$merge` | `CREATE TABLE AS` | result likh do |

## 30.3 Worked example 1 — project-wise spend

```javascript
db.purchaseOrders.aggregate([
  { $match: { orgId: ORG, isDeleted: false } },                    // ← FIRST, hamesha
  { $group: {
      _id: "$projectId",
      orderCount: { $sum: 1 },
      totalValue: { $sum: "$totalAmount" },
      avgValue:   { $avg: "$totalAmount" },
      lastOrder:  { $max: "$orderDate" }
  }},
  { $sort: { totalValue: -1 } }
])
```

**Result (humare data ke barabar):**

```javascript
[
  { _id: ObjectId("...p1"), orderCount: 5, totalValue: 1175000, avgValue: 235000, lastOrder: ISODate("2025-05-06") },
  { _id: ObjectId("...p2"), orderCount: 4, totalValue:  625000, avgValue: 156250, lastOrder: ISODate("2025-04-15") },
  { _id: ObjectId("...p3"), orderCount: 1, totalValue:  200000, avgValue: 200000, lastOrder: ISODate("2025-03-03") },
  { _id: ObjectId("...p4"), orderCount: 1, totalValue:  180000, avgValue: 180000, lastOrder: ISODate("2025-03-19") }
]
```

**`$group` ke andar `_id` hi grouping key hai.** `_id: null` = poori collection ek group.

```javascript
// Grand total
db.purchaseOrders.aggregate([
  { $match: { orgId: ORG, isDeleted: false } },
  { $group: { _id: null, total: { $sum: "$totalAmount" }, n: { $sum: 1 } } }
])
// → [{ _id: null, total: 2180000, n: 11 }]
```

**Multi-field group:**

```javascript
{ $group: { _id: { project: "$projectId", status: "$status" }, count: { $sum: 1 } } }
```

## 30.4 Worked example 2 — `$unwind` aur THE PERFORMANCE RULE

`$unwind` array ko todkar **har element ki alag document** bana deta hai.

```
Pehle $unwind:
{ _id: 1, poNumber: "PO-0001", lineItems: [ {material:"Cement", qty:200}, {material:"Sand", qty:100} ] }

$unwind: "$lineItems" ke baad — 2 documents:
{ _id: 1, poNumber: "PO-0001", lineItems: {material:"Cement", qty:200} }
{ _id: 1, poNumber: "PO-0001", lineItems: {material:"Sand",   qty:100} }
```

**Material-wise spend:**

```javascript
db.purchaseOrders.aggregate([
  { $match: { orgId: ORG, isDeleted: false } },              // ← 1. FILTER FIRST
  { $unwind: "$lineItems" },                                  // ← 2. phir explode
  { $group: {
      _id: "$lineItems.material",
      totalQty:   { $sum: "$lineItems.quantity" },
      totalValue: { $sum: { $multiply: ["$lineItems.quantity", "$lineItems.unitPrice"] } },
      orderCount: { $addToSet: "$_id" }
  }},
  { $addFields: { orderCount: { $size: "$orderCount" } } },
  { $sort: { totalValue: -1 } }
])
```

**Result:**

```javascript
[
  { _id: "TMT Bar 12mm",  totalQty: 15000, totalValue: 975000, orderCount: 3 },
  { _id: "Cement OPC 53", totalQty:  1125, totalValue: 450000, orderCount: 4 },
  { _id: "Bricks",        totalQty: 40000, totalValue: 200000, orderCount: 1 },
  { _id: "CP Fittings",   totalQty:   300, totalValue: 180000, orderCount: 1 },
  { _id: "TMT Bar 16mm",  totalQty:  2000, totalValue: 125000, orderCount: 1 },
  { _id: "Sand",          totalQty:   225, totalValue:  90000, orderCount: 2 },
  { _id: "Plywood",       totalQty:   150, totalValue:  75000, orderCount: 1 },
  { _id: "Binding Wire",  totalQty:   500, totalValue:  40000, orderCount: 1 }
]
```

## 30.5 ⚠️ THE RULE — `$match` BEFORE `$unwind`

**Ye interview ka sabse important Mongo performance point hai.**

```javascript
// ❌ GALAT — pehle explode, phir filter
db.purchaseOrders.aggregate([
  { $unwind: "$lineItems" },                    // 1,000,000 docs × avg 8 items = 8,000,000 docs
  { $match: { orgId: ORG, isDeleted: false } }  // ab 8 million mein se filter — 88 bachi
])

// ✅ SAHI — pehle filter, phir explode
db.purchaseOrders.aggregate([
  { $match: { orgId: ORG, isDeleted: false } }, // 1,000,000 → 11 docs (INDEX use hota hai!)
  { $unwind: "$lineItems" }                     // 11 × 8 = 88 docs
])
```

**Do alag-alag reasons, dono bolna:**

1. **Working set explosion.** `$unwind` document count ko **array ki average length se multiply** kar deta hai. Ek million orders, average 8 line items = **8 million intermediate documents** memory mein. 100 MB per-stage limit cross ho jayega (`allowDiskUse` ke bina error, uske saath disk spill = bahut slow).

2. **Index use khatam ho jaata hai.** **Sirf pipeline ke shuru mein wala `$match`** index use kar sakta hai. `$unwind` ke baad documents synthetic hain — unka koi index nahi hai. Toh baad wala `$match` **hamesha** full in-memory scan hai.

```
Pipeline position:   1st stage        after any transform stage
Index available:     ✅ YES           ❌ NO — in-memory only
```

**General rule (ye ek line yaad rakho):** *"Filter early, project early, sort late — and $match must be the first stage so it can use an index."*

> **[REAL]** Merlin ke aggregation pipelines exactly ye rule follow karte hain: `$match` par `{orgId, isDeleted}` hamesha **pehli stage** hai, taaki `{orgId: 1, isDeleted: 1, status: 1}` compound index hit ho. Agar koi `$lookup` ya `$unwind` pehle aa jaye, toh us org ka filter index se nahi lag payega aur pipeline poori collection scan karegi — **multi-tenant system mein ye ek tenant ka slow query poore cluster ko affect kar deta hai** (noisy neighbour). Ye ek badhiya interview line hai.

## 30.6 `$lookup` — Mongo ka LEFT JOIN

```javascript
db.purchaseOrders.aggregate([
  { $match: { orgId: ORG, isDeleted: false, status: "DELIVERED" } },
  { $lookup: {
      from: "suppliers",
      localField: "supplierId",
      foreignField: "_id",
      as: "supplier"                    // ← hamesha ARRAY milta hai
  }},
  { $unwind: { path: "$supplier", preserveNullAndEmptyArrays: true } },  // array → object
  { $project: {
      _id: 0,
      poNumber: 1,
      totalAmount: 1,
      supplierName: "$supplier.name",
      supplierCity: "$supplier.city"
  }},
  { $sort: { totalAmount: -1 } }
])
```

**Result:**

```javascript
[
  { poNumber: "PO-2025-0009", totalAmount: 500000, supplierName: "Verma Steels",     supplierCity: "Delhi"  },
  { poNumber: "PO-2025-0002", totalAmount: 450000, supplierName: "Verma Steels",     supplierCity: "Delhi"  },
  { poNumber: "PO-2025-0010", totalAmount: 250000, supplierName: "Sharma Cement Co", supplierCity: "Jaipur" },
  { poNumber: "PO-2025-0006", totalAmount: 200000, supplierName: "Nakoda Traders",   supplierCity: "Jaipur" },
  { poNumber: "PO-2025-0001", totalAmount: 120000, supplierName: "Sharma Cement Co", supplierCity: "Jaipur" },
  { poNumber: "PO-2025-0004", totalAmount:  75000, supplierName: "Bombay Hardware",  supplierCity: "Mumbai" }
]
```

**`$lookup` ke baare mein 4 cheezein jo QA ko pata honi chahiye:**

1. **Hamesha LEFT JOIN hai.** Match na mile toh `as` field ek **empty array** `[]` hoga — document gayab nahi hoga.
2. **Result array hai, object nahi.** `$unwind` ke bina `supplier.name` kaam nahi karega. Aur `$unwind` par **`preserveNullAndEmptyArrays: true` bhoolna = LEFT JOIN INNER JOIN ban jaata hai** — bilkul SQL wala `ON` vs `WHERE` trap.
3. **Foreign collection ke `foreignField` par index hona chahiye**, warna har input document ke liye full scan. `_id` par index automatic hai, baaki par manually banao.
4. **Sharded collections par `$lookup` ki limitations hain** aur ye aksar sabse mehnga stage hota hai.

**Pipeline form of `$lookup` — foreign side par bhi filter lagane ke liye (ye zyada correct hai multi-tenant mein):**

```javascript
{ $lookup: {
    from: "suppliers",
    let: { sid: "$supplierId", org: "$orgId" },
    pipeline: [
      { $match: { $expr: { $and: [
          { $eq: ["$_id",   "$$sid"] },
          { $eq: ["$orgId", "$$org"] },      // ← tenant scoping join ke ANDAR
          { $eq: ["$isDeleted", false] }
      ]}}},
      { $project: { name: 1, city: 1 } }      // ← sirf zaroori fields
    ],
    as: "supplier"
}}
```

> **[REAL] Ye Merlin ke liye ek asli security point hai:** agar `$lookup` mein tenant filter na ho, toh theoretically ek org ka order doosre org ke supplier se join ho sakta hai (agar `supplierId` kabhi cross-org point kar de). Isliye **tenant filter join ke andar hona chahiye, baad wale `$match` mein nahi**. Interview mein bolna: *"I push the tenant filter into the lookup's own pipeline rather than filtering after the join, for the same reason you put a condition in ON rather than WHERE in a SQL outer join — filtering afterwards changes the join semantics and, in a multi-tenant system, doing it afterwards means the cross-tenant read actually happened before you filtered it out."*

## 30.7 `$facet` — ek pass mein kai results

`$facet` **ek hi input** par **kai independent pipelines** chalata hai. Sabse common use: **paginated list + total count, ek round trip mein**.

```javascript
db.purchaseOrders.aggregate([
  { $match: { orgId: ORG, isDeleted: false } },       // ← ek baar filter (index use)
  { $facet: {
      page: [
        { $sort: { orderDate: -1, _id: -1 } },
        { $skip: 0 },
        { $limit: 3 },
        { $project: { poNumber: 1, totalAmount: 1, status: 1, _id: 0 } }
      ],
      totalCount: [ { $count: "count" } ],
      byStatus: [
        { $group: { _id: "$status", n: { $sum: 1 }, value: { $sum: "$totalAmount" } } },
        { $sort: { value: -1 } }
      ],
      amountStats: [
        { $group: { _id: null, min: { $min: "$totalAmount" },
                    max: { $max: "$totalAmount" }, avg: { $avg: "$totalAmount" } } }
      ]
  }}
])
```

**Result:**

```javascript
[{
  page: [
    { poNumber: "PO-2025-0012", totalAmount:  45000, status: "DRAFT" },
    { poNumber: "PO-2025-0010", totalAmount: 250000, status: "DELIVERED" },
    { poNumber: "PO-2025-0009", totalAmount: 500000, status: "DELIVERED" }
  ],
  totalCount: [ { count: 11 } ],
  byStatus: [
    { _id: "DELIVERED", n: 6, value: 1595000 },
    { _id: "APPROVED",  n: 2, value:  480000 },
    { _id: "CANCELLED", n: 1, value:   60000 },
    { _id: "DRAFT",     n: 2, value:   45000 }
  ],
  amountStats: [ { _id: null, min: 0, max: 500000, avg: 198181.8181... } ]
}]
```

**QA ke liye `$facet` ka fayda:** list aur count **ek hi snapshot** se aate hain. Do alag queries mein count aur list ke beech mein data badal sakta hai → "10 of 47 results" jabki 48 hain. Ye ek real, hard-to-reproduce bug hai jo `$facet` khatam kar deta hai.

**`$facet` ka catch:** iske andar wali sub-pipelines **index use nahi kar sakti** — isliye `$match` `$facet` se **pehle** hona chahiye (phir wahi rule).

## 30.8 `$project` vs `$addFields`

```javascript
// $project — SIRF ye fields (whitelist)
{ $project: { poNumber: 1, totalAmount: 1, _id: 0 } }

// $addFields ($set) — ye field add karo, BAAKI SAB RAKHO
{ $addFields: {
    itemCount: { $size: "$lineItems" },
    computedTotal: { $sum: { $map: {
        input: "$lineItems", as: "li",
        in: { $multiply: ["$$li.quantity", "$$li.unitPrice"] }
    }}}
}}
```

**QA integrity check — stored total vs computed total (Q19 ka Mongo version):**

```javascript
db.purchaseOrders.aggregate([
  { $match: { orgId: ORG, isDeleted: false } },
  { $addFields: {
      computedTotal: { $sum: { $map: {
          input: "$lineItems", as: "li",
          in: { $multiply: ["$$li.quantity", "$$li.unitPrice"] } } } }
  }},
  { $match: { $expr: { $ne: ["$totalAmount", "$computedTotal"] } } },
  { $project: { poNumber: 1, totalAmount: 1, computedTotal: 1,
                drift: { $subtract: ["$totalAmount", "$computedTotal"] } } }
])
```

**Result:**

```javascript
[ { _id: ..., poNumber: "PO-2025-0012", totalAmount: 45000, computedTotal: 0, drift: 45000 } ]
```

**Ye query interview mein likh ke dikha dena — `$map` + `$sum` + `$expr` ek saath, aur ek real QA purpose. Bahut strong.**

## 30.9 Aggregation performance checklist

```javascript
// Explain
db.purchaseOrders.aggregate([...], { explain: true })
// ya
db.purchaseOrders.explain("executionStats").aggregate([...])
```

**Kya dekhna hai:**

| Signal | Matlab |
|---|---|
| `IXSCAN` | index use ho raha hai ✅ |
| `COLLSCAN` | poori collection scan ❌ |
| `totalDocsExamined` vs `nReturned` | ratio 1:1 ke paas hona chahiye; 10000:5 = index problem |
| `$cursor` stage ke andar `winningPlan` | pehla `$match` index par gaya ya nahi |
| `executionTimeMillis` | actual time |
| `hasSortStage: true` | in-memory sort ho raha hai — 100 MB limit ka khatra |

**Rules:**
1. `$match` **pehli stage**, aur indexed fields par.
2. `$sort` ko `$match` ke turant baad rakho **agar** compound index dono cover karta hai — tab sort free hai.
3. `$project` se **jaldi** unnecessary fields hatao — kam data agli stages mein.
4. `$unwind` ko **jitna der ho sake** utna baad mein.
5. `$lookup` sabse mehnga hai — foreign field indexed ho, aur `pipeline` form se foreign side pe filter/project karo.
6. `allowDiskUse: true` ek **escape hatch hai, solution nahi** — agar chahiye pad raha hai toh pipeline galat hai.

> **Interview answer (aggregation — the big one):**
> "The aggregation pipeline is a sequence of stages where each stage's output feeds the next — $match is WHERE, $group is GROUP BY, $project is SELECT, $sort and $limit are what they sound like, $lookup is a left join, and $unwind flattens an array into one document per element, which has no direct SQL equivalent.
> The single most important performance rule is that $match must be the first stage. Two separate reasons, and I'd give both. First, only a $match at the start of the pipeline can use an index — once any stage has transformed the documents, they're synthetic and there's no index to use, so every later $match is a full in-memory scan. Second, $unwind multiplies the document count by the array length, so unwinding a million orders with eight line items each produces eight million intermediate documents, which blows past the hundred-megabyte per-stage memory limit. Filtering first turns that into a handful of documents before the explosion.
> In our multi-tenant system that's not just about speed. Every pipeline starts with a $match on the tenant ID and the soft-delete flag, so it hits the compound index that leads with tenant. If a $lookup or $unwind came first, one tenant's report would scan the whole collection and become a noisy neighbour for everyone else.
> Two $lookup details I check: the result is always an array, so you need $unwind with preserveNullAndEmptyArrays to keep it a left join — without that flag it silently becomes an inner join and drops rows. And I push the tenant filter into the lookup's own pipeline rather than filtering after the join, for exactly the same reason you use ON rather than WHERE in a SQL outer join."

> **Cross-question: "`$facet` kab use karoge?"**
> Jab ek hi filtered dataset se **kai alag summaries** chahiye — sabse common: paginated list + total count + status breakdown, ek round trip mein. Fayda sirf latency nahi, **consistency** hai: sab kuch ek hi snapshot se aata hai, isliye "showing 10 of 47" wala count list se kabhi disagree nahi karega. Catch: `$facet` ke andar index use nahi hota, isliye `$match` uske pehle hona chahiye.

---

# 31 — Mongo indexes

## 31.1 Basics

```javascript
// Single field
db.purchaseOrders.createIndex({ poNumber: 1 })          // 1 = asc, -1 = desc

// Compound — ORDER MATTERS
db.purchaseOrders.createIndex({ orgId: 1, isDeleted: 1, status: 1 })

// Unique
db.suppliers.createIndex({ orgId: 1, gstin: 1 }, { unique: true })

// Partial — sirf matching documents index karo
db.purchaseOrders.createIndex(
  { orgId: 1, status: 1 },
  { partialFilterExpression: { isDeleted: false } }
)

// TTL — documents apne aap delete
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 })

// Text search
db.suppliers.createIndex({ name: "text", address: "text" })

// Multikey — array field par (Mongo apne aap multikey banata hai)
db.purchaseOrders.createIndex({ "lineItems.material": 1 })

// Production-safe: background build (Mongo 4.2+ default rolling)
db.purchaseOrders.createIndex({ createdAt: -1 }, { background: true })

// Dekho kaunse indexes hain
db.purchaseOrders.getIndexes()

// Kaunse USE ho rahe hain — QA ka favourite
db.purchaseOrders.aggregate([{ $indexStats: {} }])
```

## 31.2 Compound index key order — leftmost prefix, Mongo mein bhi

**SQL wala rule bilkul same hai.** Index `{orgId: 1, isDeleted: 1, status: 1}`:

| Query | Index use? |
|---|---|
| `{orgId: X}` | ✅ prefix |
| `{orgId: X, isDeleted: false}` | ✅ prefix |
| `{orgId: X, isDeleted: false, status: "DRAFT"}` | ✅ full |
| `{orgId: X, status: "DRAFT"}` | ⚠️ partial — `orgId` se seek, `status` in-memory filter (`isDeleted` skip hua) |
| `{isDeleted: false, status: "DRAFT"}` | ❌ leading field missing — COLLSCAN |
| `{status: "DRAFT"}` | ❌ COLLSCAN |
| `{orgId: X}` + `.sort({isDeleted: 1, status: 1})` | ✅ sort bhi free |

**Note:** Mongo mein **query ke fields ka order matter nahi karta** — `{isDeleted: false, orgId: X}` aur `{orgId: X, isDeleted: false}` same hain. Sirf **index ke fields ka order** matter karta hai.

> **[REAL]** Merlin ka `{orgId: 1, isDeleted: 1, status: 1}` bilkul is rule se justified hai:
> - `orgId` **pehla** kyunki **100% queries** us par filter karti hain (multi-tenancy).
> - `isDeleted` **doosra** kyunki wo bhi lagbhag har query mein hai (soft delete).
> - `status` **teesra** kyunki wo **optional** filter hai.
>
> Agar order `{status: 1, orgId: 1}` hota, toh saari "is org ke saare orders" wali queries — jo sabse common hain — index use hi nahi kar paatin.
>
> **Interview mein bolna:** *"The ordering rule is: fields that appear in every query go leftmost, then the ones that appear sometimes. Ours leads with the tenant ID because every single query filters on it — there is no query in the system that doesn't. Then the soft-delete flag, then status, which is optional. A common instinct is to put the most selective field first, and that's right when all the fields are always present, but a highly selective field that's only in half the queries makes the index useless for the other half."*

## 31.3 Low-cardinality field ko index karna — hai kya matlab?

`isDeleted` ke sirf 2 values hain. Akele us par index bekaar hai (50% documents match karenge). **Lekin compound index ke andar** wo super useful hai, aur **partial index ke filter mein** aur bhi zyada.

```javascript
// Partial index — sirf live documents index mein aate hain
db.purchaseOrders.createIndex(
  { orgId: 1, status: 1, orderDate: -1 },
  { partialFilterExpression: { isDeleted: false } }
)
```

**Fayda:** index **chhota** ho jaata hai (deleted documents usme hain hi nahi), toh zyada memory mein fit hota hai, aur writes bhi sasta.
**Catch:** query mein `isDeleted: false` **literally** hona chahiye, warna Mongo partial index use nahi karega (usko prove karna padta hai ki query ka scope index ke scope ke andar hai).

## 31.4 ⭐ Unique PARTIAL index — business rule enforcement

**Ye is poore document ka sabse valuable Merlin example hai.**

**Business rule: "Ek scope sirf ek baar sold ho sakta hai."**

```javascript
db.scopeSales.createIndex(
  { orgId: 1, scopeId: 1 },
  {
    unique: true,
    partialFilterExpression: { isDeleted: false, status: "SOLD" },
    name: "uq_scope_sold_once"
  }
)
```

**Ye kya karta hai:**
- Sirf un documents par unique enforce karta hai jo `isDeleted: false` **aur** `status: "SOLD"` hain.
- Matlab: ek scope ke liye **ek hi live SOLD record** ho sakta hai.
- Lekin `CANCELLED` ya soft-deleted records kitne bhi ho sakte hain — **history preserve rehti hai**.

**Partial kyun, plain unique kyun nahi?**
> *"A plain unique index on (orgId, scopeId) would also block the historical rows — you could never cancel a sale and re-sell the scope, because the cancelled document would still occupy the unique slot. The partial filter narrows uniqueness to exactly the states where the rule applies: live and sold. So the invariant is enforced without destroying history."*

**Concurrency ke waqt kya hota hai:**

```
 TIME   Request A (User 1)                   Request B (User 2)
 ────   ─────────────────────────────────    ─────────────────────────────────
  t1    POST /scopes/S1/sell                 POST /scopes/S1/sell
  t2    (app checks: koi SOLD record hai?)   (app checks: koi SOLD record hai?)
  t3    → nahi, aage badho                   → nahi, aage badho     ← DONO ne pass kiya!
  t4    insert {scopeId:S1, status:"SOLD"}
        → index slot claim kiya ✅
  t5                                         insert {scopeId:S1, status:"SOLD"}
                                             → E11000 duplicate key error ❌
  t6    HTTP 201 Created                     HTTP 409 Conflict
```

**Yahi asli point hai:** application ka `if (alreadySold) throw` check **t2–t3 ke beech ki race window** ko rok hi nahi sakta. Ye **TOCTOU** (Time Of Check to Time Of Use) hai. **Sirf unique index** — jo storage engine mein lock ke andar enforce hota hai — ise rok sakta hai.

**Error ka shape:**
```
E11000 duplicate key error collection: merlin.scopeSales
index: uq_scope_sold_once dup key: { orgId: ObjectId('...'), scopeId: ObjectId('...') }
```
Spring Data ise `DuplicateKeyException` mein wrap karta hai, jo exception handler mein **HTTP 409 Conflict** ban jaata hai. **409 hi sahi status hai** — 400 nahi (input galat nahi tha), 500 nahi (server crash nahi hua, business rule cleanly enforce hua).

**QA ka test — ye poora likhna aata hona chahiye:**

```python
import threading
from concurrent.futures import ThreadPoolExecutor

def test_scope_can_be_sold_only_once_under_concurrency(api, token, scope_id, mongo):
    """Do concurrent sell requests: exactly ek 201, ek 409, aur DB mein exactly ek SOLD doc."""
    barrier = threading.Barrier(2)

    def sell():
        barrier.wait()                      # dono exactly ek saath fire karein
        return api.post(f"/scopes/{scope_id}/sell",
                        json={"buyerId": "B1", "amount": 100000},
                        headers={"Authorization": token})

    with ThreadPoolExecutor(max_workers=2) as ex:
        r1, r2 = [f.result() for f in [ex.submit(sell), ex.submit(sell)]]

    codes = sorted([r1.status_code, r2.status_code])

    # 1. Exactly ek jeeta, ek ko clean conflict mila
    assert codes == [201, 409], f"Expected [201, 409], got {codes}"

    # 2. 500 kabhi nahi — duplicate key error leak nahi hona chahiye
    assert 500 not in codes, "Duplicate key error leaked as a 500"

    # 3. DB truth — exactly ek live SOLD document
    n = mongo.scopeSales.count_documents(
        {"scopeId": scope_id, "status": "SOLD", "isDeleted": False})
    assert n == 1, f"Expected exactly 1 SOLD record, found {n}"
```

**Teeno assertions ka alag matlab hai — teeno bolna:**
1. `[201, 409]` — race resolve hua, dono succeed nahi hue.
2. `500 not in codes` — duplicate-key error **handle** hua, leak nahi hua. Ye ek alag defect hai: rule sahi enforce ho raha ho par error 500 ke roop mein aaye toh client retry karega aur user ko "something went wrong" dikhega.
3. **DB count == 1** — API ne jo bola wo DB mein sach bhi hai. Sirf HTTP codes par assert karna adhoora test hai.

> **Interview answer (partial unique index — ye poora bolna):**
> "We have a rule that a scope can only be sold once. That's enforced by a unique partial index on tenant ID and scope ID, with a partial filter restricting it to documents that are not deleted and have status SOLD.
> It's partial rather than plain unique for a specific reason: a plain unique index would also block historical rows, so once a sale was cancelled you could never re-sell that scope. The partial filter narrows the uniqueness to exactly the states where the rule applies, so the invariant holds and history is preserved.
> The reason it's an index rather than an application check is concurrency. An application-level 'is it already sold' check has a race window between the check and the insert — two requests can both pass the check and both insert. A unique index is enforced by the storage engine under a lock, so one write wins and the other gets a duplicate key error, which we map to a 409 Conflict.
> I test it by firing two concurrent requests synchronised on a barrier, and I assert three things: exactly one 201 and one 409, no 500 — because a duplicate key error leaking as a 500 is a separate defect even when the rule is correctly enforced — and then I go to the database and assert there's exactly one live SOLD document. Asserting only on the HTTP codes isn't enough; the database is the source of truth."

## 31.5 Mongo index gotchas for QA

| Gotcha | Detail |
|---|---|
| **`_id` par index automatic** | banate nahi, hata bhi nahi sakte |
| **Multikey index limits** | ek compound index mein **sirf ek** array field ho sakti hai |
| **Index build ka impact** | bade collections par foreground build **poori DB block** kar sakta hai; 4.2+ mein default safer, par staging par pehle test karo |
| **Index size vs RAM** | working set (indexes + hot data) RAM mein fit hona chahiye, warna page faults |
| **Case sensitivity** | `{name: "sharma"}` `"Sharma"` match nahi karega. Case-insensitive ke liye **collation** wala index chahiye |
| **`$ne`, `$nin`, un-anchored `$regex`** | index se **poora fayda nahi** milta — aksar scan |
| **Sort + filter same index se** | equality fields pehle, phir sort field — warna in-memory sort (32 MB limit!) |
| **Unused indexes** | `$indexStats` se `accesses.ops: 0` dhoondo — pure write overhead |

```javascript
// QA check: koi index use hi nahi ho raha?
db.purchaseOrders.aggregate([{ $indexStats: {} }]).forEach(i =>
  print(i.name + " → " + i.accesses.ops + " ops since " + i.accesses.since))
```

---

# 32 — Transactions in MongoDB

## 32.1 Single-document atomicity — pehle ye samjho

**MongoDB mein ek single document ka write HAMESHA atomic hai** — chahe wo document 50 nested fields aur ek 200-element array update kar raha ho, aur chahe koi transaction na ho.

```javascript
// Ye poora update atomic hai — ya sab lagega ya kuch nahi
db.purchaseOrders.updateOne(
  { _id: PO_ID, orgId: ORG },
  { $set:  { status: "DELIVERED", updatedAt: new Date() },
    $inc:  { revisionCount: 1 },
    $push: { statusHistory: { from: "APPROVED", to: "DELIVERED", at: new Date() } } }
)
```

**Isliye document model mein embedding ka ek bada fayda ye hai:** order aur uske line items ek document mein hain, toh unka combined update **transaction ke bina bhi atomic** hai. **Ye interview mein bolne layak insight hai** — bahut log sochte hain Mongo mein atomicity hai hi nahi.

## 32.2 Multi-document transactions

Mongo 4.0 se replica sets par, 4.2 se sharded clusters par multi-document ACID transactions available hain.

```javascript
const session = db.getMongo().startSession();
session.startTransaction({
  readConcern:  { level: "snapshot" },
  writeConcern: { w: "majority" }
});

try {
  const po  = session.getDatabase("merlin").purchaseOrders;
  const inv = session.getDatabase("merlin").invoices;

  po.updateOne({ _id: PO_ID, orgId: ORG }, { $set: { status: "INVOICED" } }, { session });
  inv.insertOne({ orgId: ORG, orderId: PO_ID, amount: NumberDecimal("120000"),
                  status: "PENDING", createdAt: new Date() }, { session });

  session.commitTransaction();
} catch (e) {
  session.abortTransaction();
  throw e;
} finally {
  session.endSession();
}
```

**Python (pymongo):**

```python
with client.start_session() as session:
    with session.start_transaction():
        db.purchaseOrders.update_one({"_id": po_id, "orgId": org},
                                     {"$set": {"status": "INVOICED"}}, session=session)
        db.invoices.insert_one({"orgId": org, "orderId": po_id,
                                "amount": Decimal128("120000")}, session=session)
    # block clean exit kare toh auto-commit; exception par auto-abort
```

## 32.3 ⚠️ Replica set requirement — ye poocha jaata hai

**Multi-document transactions ke liye replica set (ya sharded cluster) ZAROORI hai. Standalone mongod par ye kaam hi nahi karenge.**

```
MongoServerError: Transaction numbers are only allowed on a replica set member or mongos
```

**Kyun:** transactions **oplog** aur `majority` read/write concern par depend karte hain, jo replication machinery ka hissa hain. Standalone mongod mein oplog hi nahi hota.

**QA ke liye ye ek asli practical problem hai:**
- Developer ke laptop par standalone Mongo → transaction wala code **local par error dega** par staging par chalega. Ya ulta.
- CI mein plain `mongo` Docker image standalone hai → transaction tests fail.
- **Fix:** CI mein single-node replica set chalao:

```bash
docker run -d --name mongo-rs -p 27017:27017 mongo:7 \
  mongod --replSet rs0 --bind_ip_all

docker exec mongo-rs mongosh --eval 'rs.initiate({
  _id: "rs0", members: [{ _id: 0, host: "localhost:27017" }]
})'
```

**Ya `mongodb-atlas-local` / testcontainers ka `MongoDBContainer`** (wo apne aap single-node replica set banata hai).

**Interview mein bolna:** *"Multi-document transactions require a replica set, because they're built on the oplog and majority read and write concerns. That's not a theoretical detail — it means a developer running standalone MongoDB locally cannot exercise the transaction path at all, so the code compiles, the unit tests pass, and it only breaks in an environment where transactions actually run. I make sure CI runs a single-node replica set for exactly this reason. It's a good example of an environment difference that silently disables a whole class of tests."*

## 32.4 Transaction limitations (aur ye bolna maturity dikhata hai)

| Limit | Detail |
|---|---|
| **60-second default timeout** | `transactionLifetimeLimitSeconds`. Lamba transaction apne aap abort |
| **16 MB oplog entry limit** | ek transaction bahut zyada documents nahi chhoo sakta |
| **Performance cost** | single-doc write se kaafi mehnga; sab kuch transaction mein daal dena anti-pattern hai |
| **`TransientTransactionError`** | retry karna **application ki zimmedari** hai — driver automatically nahi karta (retryable writes alag cheez hai) |
| **DDL nahi** | transaction ke andar collection create/drop nahi (4.4 se kuch cases allow) |
| **Sharded transactions mehnge** | cross-shard coordination |

**Sahi design philosophy (ye bolna):**
> *"In MongoDB the first tool for atomicity is the document model, not transactions. If two pieces of data must change together, the strongest design is usually to put them in the same document, because a single document write is atomic without any transaction machinery. Multi-document transactions are the right answer when the data genuinely belongs in different collections — writing an order and a ledger entry, for instance. Reaching for transactions to paper over a modelling problem gets you the cost of transactions without the benefit of the document model."*

## 32.5 Write concern aur read concern — durability ka control

```javascript
// Write concern — kitne nodes acknowledge karein
{ w: 1 }                              // primary ne likh liya (default) — fast, crash par loss possible
{ w: "majority" }                     // majority nodes — durable
{ w: "majority", j: true }            // + journal disk par — sabse durable
{ w: "majority", wtimeout: 5000 }     // timeout

// Read concern
{ level: "local" }        // jo primary ke paas hai — rollback ho sakta hai
{ level: "majority" }     // jo majority ne acknowledge kiya — rollback nahi hoga
{ level: "snapshot" }     // consistent point-in-time (transactions ke liye)
```

**QA ka sawaal jo poochna chahiye:** *"What's our write concern on the payment collection?"* Agar `w: 1` hai, toh primary crash hone par **acknowledged writes kho sakti hain**. Payments ke liye `w: "majority", j: true` hona chahiye. Ye ek **design-level bug report** hai jo bahut kam QA likhte hain.

**Stale read from secondary:**

```javascript
db.purchaseOrders.find({...}).readPref("secondaryPreferred")   // ← replication lag!
```

Agar app write ke turant baad secondary se padhe, toh **abhi likha hua data nahi milega**. Ye ek classic "API ne 200 diya par UI purana data dikha raha hai" ka karan hai (Section 35).

**Fix:** `readPreference: primary` for read-after-write, ya **causal consistency** (`session` ke saath `afterClusterTime`).

> **Interview answer:**
> "A single document write in MongoDB is always atomic, no transaction needed — even if it updates twenty fields and pushes to an array. That's actually the first tool for atomicity in the document model: if two things must change together, the strongest answer is usually to put them in the same document.
> Multi-document transactions exist from version 4.0, but they require a replica set, because they're built on the oplog and majority concerns. That has a practical consequence I've had to deal with: a standalone MongoDB — which is what you get from the default Docker image — can't run them at all, so transaction code passes locally and fails, or behaves differently, wherever transactions actually run. CI has to run a single-node replica set.
> The other thing I'd raise is write concern. The default acknowledges the write on the primary only, which means an acknowledged write can be lost if the primary fails over before it replicates. For a payments collection I'd expect majority with journaling. That's a question I ask in design review, because it's invisible in testing until the day there's a failover."

---

# 33 — SQL → MongoDB translation table

## 33.1 Concepts

| SQL | MongoDB |
|---|---|
| Database | Database |
| Table | Collection |
| Row | Document |
| Column | Field |
| Primary key | `_id` (auto-indexed, mandatory) |
| Foreign key | *(exist hi nahi karta — application ki zimmedari)* |
| Index | Index |
| JOIN | `$lookup` (sirf aggregation mein) |
| Transaction | Transaction (replica set chahiye) |
| View | View / on-demand materialized view (`$merge`) |
| Trigger | Change stream / Atlas Trigger |
| `NULL` | `null` **ya** field ka missing hona (do alag states!) |
| Schema (enforced) | `$jsonSchema` validator (optional) |

## 33.2 Queries

| SQL | MongoDB |
|---|---|
| `SELECT * FROM orders` | `db.orders.find({})` |
| `SELECT a, b FROM orders` | `db.orders.find({}, {a:1, b:1, _id:0})` |
| `WHERE status = 'DRAFT'` | `{status: "DRAFT"}` |
| `WHERE amount > 1000` | `{amount: {$gt: 1000}}` |
| `WHERE amount BETWEEN 1 AND 9` | `{amount: {$gte: 1, $lte: 9}}` |
| `WHERE status IN ('A','B')` | `{status: {$in: ["A","B"]}}` |
| `WHERE status <> 'A'` | `{status: {$ne: "A"}}` ⚠️ missing fields bhi match |
| `WHERE col IS NULL` | `{col: null}` ⚠️ missing bhi; ya `{col: {$exists:false}}` |
| `WHERE name LIKE 'Sh%'` | `{name: {$regex: "^Sh"}}` |
| `WHERE a=1 AND b=2` | `{a: 1, b: 2}` |
| `WHERE a=1 OR b=2` | `{$or: [{a:1},{b:2}]}` |
| `ORDER BY a ASC, b DESC` | `.sort({a: 1, b: -1})` |
| `LIMIT 10 OFFSET 20` | `.skip(20).limit(10)` |
| `SELECT DISTINCT city` | `db.c.distinct("city")` |
| `COUNT(*)` | `db.c.countDocuments({})` |
| `SELECT COUNT(*) ... GROUP BY x` | `[{$group: {_id: "$x", n: {$sum: 1}}}]` |
| `SUM(amt)` | `{$group: {_id: null, t: {$sum: "$amt"}}}` |
| `HAVING COUNT(*) > 1` | `$group` ke **baad** `{$match: {n: {$gt: 1}}}` |
| `INNER JOIN` | `$lookup` + `$unwind` (bina `preserveNull...`) |
| `LEFT JOIN` | `$lookup` + `$unwind` **with** `preserveNullAndEmptyArrays: true` |
| `UNION ALL` | `$unionWith` |
| `CASE WHEN` | `$cond` / `$switch` |
| `COALESCE(a, b)` | `$ifNull: ["$a", "$b"]` |
| Window function | `$setWindowFields` (Mongo 5.0+) |

## 33.3 Writes

| SQL | MongoDB |
|---|---|
| `INSERT INTO t VALUES (…)` | `db.t.insertOne({…})` |
| `INSERT` multiple | `db.t.insertMany([…])` |
| `UPDATE t SET a=1 WHERE id=5` | `db.t.updateOne({_id:5}, {$set: {a: 1}})` |
| `UPDATE t SET n = n + 1` | `db.t.updateOne({…}, {$inc: {n: 1}})` |
| `UPDATE` many | `db.t.updateMany(filter, {$set: {…}})` |
| `DELETE FROM t WHERE id=5` | `db.t.deleteOne({_id: 5})` |
| `TRUNCATE t` | `db.t.deleteMany({})` (ya `db.t.drop()`) |
| `INSERT … ON CONFLICT DO UPDATE` | `updateOne(filter, update, {upsert: true})` |
| — | `$setOnInsert` — sirf insert hone par set karo |

## 33.4 Side-by-side worked example

**"Har project ka delivered order count aur total, sirf 5 lakh se upar wale."**

```sql
SELECT project_id,
       COUNT(*)          AS order_count,
       SUM(total_amount) AS total_value
FROM   orders
WHERE  is_deleted = false AND status = 'DELIVERED'
GROUP  BY project_id
HAVING SUM(total_amount) > 500000
ORDER  BY total_value DESC
LIMIT  10;
```

```javascript
db.purchaseOrders.aggregate([
  { $match: { orgId: ORG, isDeleted: false, status: "DELIVERED" } },  // WHERE
  { $group: { _id: "$projectId",                                      // GROUP BY
              orderCount: { $sum: 1 },                                // COUNT(*)
              totalValue: { $sum: "$totalAmount" } } },               // SUM()
  { $match: { totalValue: { $gt: 500000 } } },                        // HAVING
  { $sort:  { totalValue: -1 } },                                     // ORDER BY
  { $limit: 10 }                                                      // LIMIT
])
```

**Dekho — pipeline ka order bilkul SQL ke logical execution order jaisa hai.** `$match` do baar hai: pehla `WHERE` (index use karta hai), doosra `HAVING` (grouped result par). **Ye samjha dena interview mein ek clean, memorable answer hai.**

**Result:**

```javascript
[
  { _id: ObjectId("...p1"), orderCount: 3, totalValue: 1070000 }
]
```

(Project 1 ke delivered: 120000+450000+500000 = 1,070,000. Project 2 ke delivered: 75000+250000 = 325,000 — 5 lakh se kam, filter ho gaya.)

> **Interview answer:**
> "The mapping is fairly clean: a table is a collection, a row is a document, a column is a field, and the primary key is always called _id and is always indexed. WHERE becomes $match, GROUP BY becomes $group, ORDER BY becomes $sort, LIMIT and OFFSET become $limit and $skip, and a join becomes $lookup in an aggregation pipeline.
> The neat part is that an aggregation pipeline written in the natural order looks exactly like SQL's logical execution order — match, group, match again for the HAVING, sort, limit. I find that a useful way to explain it to people coming from SQL, because the first $match is the WHERE and can use an index, and the second $match after $group is the HAVING and can't.
> The two things that genuinely don't translate: there are no foreign keys, so referential integrity is the application's job and needs its own checks; and NULL splits into two states, an explicit null and a missing field, which behave differently in queries."

---
---

# PART D — PRACTICAL QA

> **Ye section hi tumhe baaki candidates se alag karega.** SQL syntax har koi rat sakta hai. "UI par success dikha — DB mein kya verify karoge?" ka **structured, complete** jawab bahut kam log de paate hain. Ye do sections (34 aur 35) **ratt lo** — literally, structure ke saath.

---

# 34 — "UI shows payment successful" — what do you verify in the DB?

## 34.1 Sochne ka framework

Ye sawaal isliye poocha jaata hai ki dekha jaye tum **"green tick = done"** wale QA ho ya **"success ka matlab N invariants" wale**. Jawab **layers** mein do:

```
  1. THE RECORD ITSELF          — payment row exist karti hai, sahi values ke saath
  2. THE PARENT STATE           — invoice / order ka status aage badha
  3. THE MONEY                  — ledger / journal entries balance karti hain
  4. THE ABSENCE OF DUPLICATES  — ek hi charge, do nahi
  5. THE AUDIT TRAIL            — kisne, kab, kis se
  6. STRUCTURAL INTEGRITY       — FKs, orphans, no dangling refs
  7. TIME                       — timestamps ka order logical ho
  8. THE EXTERNAL WORLD         — gateway reference, reconciliation
```

## 34.2 Layer 1 — Payment row itself

```sql
SELECT payment_id, invoice_id, amount, currency, method,
       status, gateway_txn_id, idempotency_key, paid_on, created_at
FROM   payments
WHERE  invoice_id = :invoice_id
ORDER  BY created_at DESC;
```

**Kya assert karna hai:**

| Check | Kyun |
|---|---|
| Row **exist karti hai** | UI ne success bola — DB mein hai? |
| `status` = `SUCCESS`/`CAPTURED` (`PENDING`/`INITIATED` nahi) | UI aksar "initiated" ko "successful" dikha deta hai — ye ek real bug class hai |
| `amount` **exactly** wahi jo user ne dekha | rounding, tax, discount ke baad ka final |
| `amount` ka **type** — `NUMERIC`/`DECIMAL`, `FLOAT` nahi | float par 0.1+0.2 = 0.30000000000000004 |
| `currency` **explicitly set** hai, assume nahi | multi-currency system mein missing currency = wrong ledger |
| `method` sahi hai (UPI/NEFT/CARD) | user ne jo chuna |
| `gateway_txn_id` **non-null** | gateway se sach mein confirm hua? Ya sirf humne likh diya? |
| `payment_id` valid ObjectId/PK hai | |

**Sabse important check jo log bhoolte hain: `status`.** "Payment created" aur "payment succeeded" do alag cheezein hain. Agar row `status = 'INITIATED'` hai aur UI "Payment Successful" dikha raha hai, **wo ek P1 bug hai** — user ne samajh liya paisa chala gaya, par gateway ne confirm nahi kiya.

## 34.3 Layer 2 — Parent state

```sql
SELECT i.invoice_id, i.invoice_no, i.amount AS invoice_amount, i.status AS invoice_status,
       COALESCE(SUM(p.amount), 0) AS paid_total,
       i.amount - COALESCE(SUM(p.amount), 0) AS outstanding,
       o.order_id, o.status AS order_status
FROM   invoices i
JOIN   orders o   ON o.order_id = i.order_id
LEFT   JOIN payments p ON p.invoice_id = i.invoice_id AND p.status = 'SUCCESS'
WHERE  i.invoice_id = :invoice_id
GROUP  BY i.invoice_id, i.invoice_no, i.amount, i.status, o.order_id, o.status;
```

**Assert:**
- `invoice_status` = `PAID` agar `paid_total >= invoice_amount`, `PARTIALLY_PAID` agar beech mein
- `outstanding` = 0 (full payment ke case mein)
- `order_status` bhi aage badha ho, agar business rule hai (`PAID` → order `CONFIRMED`)
- **Aur negative outstanding na ho** — overpayment silently accept ho gaya?

**Ye ek do-step invariant hai:** payment likhna aur invoice status update karna alag writes hain. Agar ye ek transaction mein nahi hain, toh crash ke baad payment hai par invoice `PENDING` hai. **Test:** payment write ke baad, invoice update se pehle service ko kill karo.

## 34.4 Layer 3 — Money must balance (ledger / double-entry)

Agar system mein accounting ledger hai (construction ERP mein aksar hota hai):

```sql
-- Har transaction ke liye debits = credits — ye NON-NEGOTIABLE invariant hai
SELECT je.journal_id, je.reference_type, je.reference_id,
       SUM(CASE WHEN jl.direction = 'DEBIT'  THEN jl.amount ELSE 0 END) AS total_debit,
       SUM(CASE WHEN jl.direction = 'CREDIT' THEN jl.amount ELSE 0 END) AS total_credit
FROM   journal_entries je
JOIN   journal_lines  jl ON jl.journal_id = je.journal_id
WHERE  je.reference_type = 'PAYMENT' AND je.reference_id = :payment_id
GROUP  BY je.journal_id, je.reference_type, je.reference_id
HAVING SUM(CASE WHEN jl.direction='DEBIT'  THEN jl.amount ELSE 0 END)
    <> SUM(CASE WHEN jl.direction='CREDIT' THEN jl.amount ELSE 0 END);
-- Result HAMESHA 0 rows hona chahiye
```

**Aur:**
- Journal entry **exist** karti hai payment ke liye (sirf balanced hona kaafi nahi — hona bhi chahiye)
- Sahi accounts hit hue: bank/cash account **debit**, accounts receivable **credit**
- Amount ledger mein aur payment row mein **same** hai
- Ledger entry ki `posting_date` sahi accounting period mein hai (closed period mein posting nahi)

**Ye layer bolna hi tumhe alag banata hai.** Zyadatar QA candidates payment row tak ruk jaate hain. "Debits equal credits" bolna dikhata hai ki tum **domain** samajhte ho, sirf CRUD nahi.

## 34.5 Layer 4 — No duplicate charge

```sql
-- (a) Same idempotency key se do payments?
SELECT idempotency_key, COUNT(*) AS n, ARRAY_AGG(payment_id) AS ids
FROM   payments
WHERE  idempotency_key IS NOT NULL
GROUP  BY idempotency_key
HAVING COUNT(*) > 1;

-- (b) Same gateway transaction do baar record hua?
SELECT gateway_txn_id, COUNT(*) FROM payments
WHERE gateway_txn_id IS NOT NULL
GROUP BY gateway_txn_id HAVING COUNT(*) > 1;

-- (c) Overpayment — invoice se zyada paisa liya?
SELECT i.invoice_id, i.amount, SUM(p.amount) AS paid
FROM   invoices i JOIN payments p ON p.invoice_id = i.invoice_id AND p.status='SUCCESS'
GROUP  BY i.invoice_id, i.amount
HAVING SUM(p.amount) > i.amount;

-- (d) Suspicious near-duplicate — same amount, same invoice, 60 second ke andar
SELECT a.payment_id, b.payment_id, a.amount, a.created_at, b.created_at
FROM   payments a JOIN payments b
       ON b.invoice_id = a.invoice_id AND b.amount = a.amount
      AND b.payment_id > a.payment_id
      AND b.created_at - a.created_at < INTERVAL '60 seconds';
```

**Aur test bhi karo, sirf query nahi:**

```python
def test_payment_is_idempotent(api, invoice_id):
    key = str(uuid.uuid4())
    body = {"invoiceId": invoice_id, "amount": "120000.00",
            "currency": "INR", "method": "NEFT"}
    headers = {"Idempotency-Key": key}

    r1 = api.post("/payments", json=body, headers=headers)
    r2 = api.post("/payments", json=body, headers=headers)     # exact same call, dobara

    assert r1.status_code in (200, 201)
    assert r2.status_code in (200, 201, 409)
    # Dono ne SAME payment id return kiya ho — naya nahi banaya
    if r2.status_code in (200, 201):
        assert r2.json()["paymentId"] == r1.json()["paymentId"]

    # DB truth
    n = db.scalar("SELECT COUNT(*) FROM payments WHERE idempotency_key = %s", (key,))
    assert n == 1, f"Idempotency broken: {n} payment rows for one key"
```

**Idempotency test hamesha DB count se end hona chahiye.** HTTP response same dikh sakta hai jabki DB mein do rows ban gayi hon.

## 34.6 Layer 5 — Audit trail

```sql
SELECT audit_id, entity_type, entity_id, action, actor_user_id,
       old_value, new_value, created_at, ip_address
FROM   audit_log
WHERE  entity_type = 'PAYMENT' AND entity_id = :payment_id
ORDER  BY created_at;
```

**Assert:**
- Audit row **exist** karti hai
- `actor_user_id` **sahi user** hai (system user nahi, jab user ne kiya ho)
- `old_value` → `new_value` transition sahi hai
- **Exactly ek** audit row (do = double-write bug, jaise trigger + app dono likh rahe hain)
- Audit row **immutable** hai — koi update path nahi hona chahiye

## 34.7 Layer 6 — Structural integrity

```sql
-- Payment ka invoice exist karta hai aur soft-deleted nahi hai
SELECT p.payment_id, p.invoice_id, i.invoice_id AS found, i.is_deleted
FROM   payments p LEFT JOIN invoices i ON i.invoice_id = p.invoice_id
WHERE  p.payment_id = :payment_id;
-- found NULL nahi hona chahiye; is_deleted true nahi hona chahiye

-- Tenant scoping — payment, invoice, order sab SAME org ke hain?
SELECT p.payment_id, p.org_id AS pay_org, i.org_id AS inv_org, o.org_id AS ord_org
FROM   payments p
JOIN   invoices i ON i.invoice_id = p.invoice_id
JOIN   orders   o ON o.order_id   = i.order_id
WHERE  p.payment_id = :payment_id
  AND  NOT (p.org_id = i.org_id AND i.org_id = o.org_id);
-- 0 rows hona chahiye
```

**Ye tenant check Merlin ke liye khaas important hai** — multi-tenant system mein cross-org linkage ek **security incident** hai, data bug nahi.

## 34.8 Layer 7 — Timestamps ka order

```sql
SELECT p.payment_id,
       o.created_at AS order_created,
       i.invoice_date,
       p.created_at AS payment_created,
       p.paid_on,
       a.created_at AS audit_created
FROM   payments p
JOIN   invoices i ON i.invoice_id = p.invoice_id
JOIN   orders   o ON o.order_id   = i.order_id
LEFT   JOIN audit_log a ON a.entity_id = p.payment_id AND a.entity_type='PAYMENT'
WHERE  p.payment_id = :payment_id;
```

**Assert karo ki ye order hold karta hai:**

```
order.created_at  ≤  invoice.invoice_date  ≤  payment.created_at  ≤  audit.created_at  ≤  now()
```

**Aur timezone check:** DB mein `TIMESTAMPTZ` (UTC) store ho raha hai ya naive `TIMESTAMP`? Agar naive hai, toh **server ke timezone par depend karta hai** — do servers, do alag values. Ye Section 35 ka bada RCA cause hai.

```sql
-- Future-dated payments — clock skew ya timezone bug ka signal
SELECT payment_id, paid_on, created_at FROM payments WHERE created_at > now();
```

## 34.9 Layer 8 — External reconciliation

- `gateway_txn_id` **gateway ke pass sach mein exist karta hai?** (test mein gateway sandbox se verify karo, ya at least non-null aur expected format)
- Gateway ka webhook aaya? `webhook_events` table mein entry hai?
- Webhook ka `status` payment ke `status` se **match** karta hai? (Webhook ne `FAILED` bola par humne `SUCCESS` likh diya = disaster)
- Webhook **do baar** aaya toh dobara process nahi hua? (webhooks at-least-once hote hain — idempotency yahan bhi chahiye)

> **Interview answer (ye poora, structured bolna — ye is document ka sabse important jawab hai):**
> "A green tick on the UI only tells me the API returned a success code. What I actually verify is that a set of invariants holds in the database, and I'd go through it in layers.
> First, the payment record itself: it exists, and critically its status is the terminal success state, not 'initiated' or 'pending' — showing 'payment successful' for a record that's only been initiated is a real bug class, because the user believes the money moved. Then the amount matches exactly what the user was shown, the currency is explicitly set rather than assumed, and the amount column is a decimal type, not a float.
> Second, the parent state moved: the invoice is marked paid or partially paid, the outstanding balance recomputes to zero, and the order status advanced if the business rule says it should. Writing the payment and updating the invoice are two writes, so I'd also check they're in one transaction by killing the process between them and confirming there's no payment with a still-pending invoice.
> Third, the money balances. If there's a ledger, I assert that the journal entry for this payment exists and that its debits equal its credits, and that the correct accounts were hit — bank debited, receivables credited. That invariant should never be violated, so it's a good permanent assertion.
> Fourth, no duplicate charge. I check that the idempotency key appears exactly once, that the gateway transaction ID isn't recorded twice, that the total paid doesn't exceed the invoice, and I actively test it by posting the same request twice with the same idempotency key and asserting the database has one payment row, not two — the HTTP responses can look identical while two rows were created.
> Fifth, the audit trail exists, attributes the action to the right user, and has exactly one row — two rows usually means both the application and a trigger are writing it.
> Sixth, structural integrity: the payment's invoice exists and isn't soft-deleted, and — this matters in a multi-tenant system — the payment, invoice and order all belong to the same tenant. A cross-tenant link there is a security incident, not a data bug.
> Seventh, the timestamps are in a sensible order: order created before invoice, invoice before payment, payment before audit, and nothing in the future. Future-dated rows usually mean a timezone or clock-skew problem.
> And finally, reconciliation with the outside world — the gateway transaction ID is real, the webhook arrived, and the webhook's status agrees with what we stored. Webhooks are at-least-once, so I also check that receiving the same one twice doesn't create a second payment."

---

# 35 — "API returned 200 but UI shows wrong data" — RCA

## 35.1 The method — bisect the pipeline

Ye ek **debugging methodology** ka sawaal hai, database ka nahi. Interviewer dekh raha hai ki tum **randomly guess karte ho ya systematically halve karte ho**.

```
   DB row  ──►  ORM/Repository  ──►  Service  ──►  DTO/Serializer  ──►  HTTP
       │              │                 │               │                │
       └──────────────┴─────────────────┴───────────────┴────────────────┘
                              ▲                                 │
                              │                                 ▼
                        Cache layer                       Network / CDN
                              │                                 │
                              └────────────► Browser ◄──────────┘
                                                │
                                         Frontend state
                                                │
                                          Rendered DOM
```

**Rule: pehle establish karo ki galat data KAHAN se shuru hota hai.** Har layer par same question: *"Is this layer's output already wrong, or still right?"* Jahan sahi se galat mein badla — **wahi bug hai**.

## 35.2 Step-by-step

**Step 0 — "Wrong" ko define karo.**
Kaunsa field? Expected kya tha, actual kya hai? Screenshot + exact values. "Wrong data" se debugging start nahi hoti. Aur poocho: **hamesha galat hai ya kabhi-kabhi?** Intermittent = caching/replica/race. Consistent = mapping/logic bug.

**Step 1 — DB row padho (source of truth).**

```sql
SELECT * FROM orders WHERE order_id = 9;
```
- Agar **DB hi galat hai** → problem read path mein nahi, **write path** mein hai. Ab poora RCA ulta ho gaya: kis request ne ye likha? Audit log dekho.
- Agar **DB sahi hai** → aage badho.

**Step 2 — API response body dekho (rendered UI nahi).**

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
     "https://staging.merlin.co/api/orders/9" | jq .
```
- **Response galat** → bug backend mein hai (steps 3–6)
- **Response sahi hai par UI galat** → bug frontend mein hai (steps 7–9)

**Ye ek command poore problem space ko aadha kar deti hai. Ye sabse important step hai.**

**Step 3 — Caching.**

| Cache type | Kaise pakdo |
|---|---|
| HTTP / CDN | `curl -I` se `Cache-Control`, `Age`, `X-Cache: HIT`, `ETag` headers dekho |
| Application (Redis/Caffeine) | cache key manually inspect karo; cache invalidate karke retry |
| ORM second-level / session | new session/connection se query chalao |
| Browser | hard reload / incognito / disable cache |

**Test:** cache clear karo, **turant** dobara call karo. Agar ab sahi aaya → **cache invalidation bug**. Ye sabse common cause hai.

**Step 4 — Stale read from a replica.**

Agar system read replicas / Mongo secondaries use karta hai:
- Write primary par gaya, read secondary se aaya, **replication lag** ke andar → purana data
- Symptom: **update ke turant baad** galat, refresh karne par sahi. **Ye signature hai.**

```sql
-- Postgres: replication lag
SELECT client_addr, state, sent_lsn, replay_lsn,
       pg_wal_lsn_diff(sent_lsn, replay_lsn) AS lag_bytes
FROM   pg_stat_replication;
```
```javascript
// Mongo: replica set lag
rs.printSecondaryReplicationInfo()
```

**Fix:** read-after-write ke liye primary se padho, ya causal consistency / sticky-read session use karo.

> **[REAL]** Mongo mein agar koi read `readPreference: secondaryPreferred` se ja rahi hai (performance ke liye), toh POST ke turant baad ka GET **purana document** de sakta hai. Ye Merlin jaisi app mein "maine save kiya par dikha nahi raha" wali complaint ka classic karan hai.

**Step 5 — Timezone conversion.**

**Sabse common "wrong data" ka karan, aur sabse zyada under-diagnosed.**

| Layer | Kya galat ho sakta hai |
|---|---|
| DB column type | `TIMESTAMP` (naive) vs `TIMESTAMPTZ` — naive column server ke TZ par depend karta hai |
| DB server TZ | `SHOW timezone;` |
| App/JVM TZ | `TZ` env var; container mein aksar UTC, dev machine par IST |
| Serialization | ISO-8601 mein offset hai (`+05:30`/`Z`) ya nahi? Offset ke bina client guess karega |
| Frontend | browser local TZ mein render karta hai |

**Signature:** date **5 hours 30 minutes** ya **ek pura din** off hai. "Order 5 May ka hai par 4 May dikha raha hai" = **IST ka midnight UTC mein pichhla din hai**.

```sql
SHOW timezone;
SELECT order_date, order_date AT TIME ZONE 'UTC' AT TIME ZONE 'Asia/Kolkata' FROM orders WHERE order_id=12;
```

**Rule jo bolna hai:** *"Store in UTC, convert only at the presentation layer, and always serialise with an explicit offset."*

**Step 6 — DTO / serialization mapping bug.**

- Field naam mismatch — DTO mein `totalAmount`, JSON mein `total_amount`; frontend `undefined` padh raha hai aur `0` render kar raha hai
- Galat field map ho gaya — `amount` ki jagah `taxAmount`
- Nested object flatten nahi hua — `supplier.name` ki jagah `[object Object]`
- Enum mapping — DB mein `DELIVERED`, DTO mein `Delivered`, frontend `DELIVERED` expect kar raha hai
- **Type coercion** — `BigDecimal` JSON number ban gaya aur JS ne precision kho di
- `null` vs missing field — frontend `??` operator se default laga raha hai

```java
// Ek acha check: raw DB value aur DTO value ko ek saath log karo
log.debug("db={} dto={}", order.getTotalAmount(), dto.getTotalAmount());
```

**Step 7 — Rounding aur precision.**

| Problem | Example |
|---|---|
| Float use hua | `0.1 + 0.2 = 0.30000000000000004` |
| JS `Number` 53-bit hai | bade paise (paise mein store) precision kho dete hain |
| Rounding jagah galat | har line item round kiya vs total round kiya — sum alag aayega |
| Rounding mode | HALF_UP vs HALF_EVEN — 2.5 → 3 ya 2? |
| Currency minor units | rupees vs paise — 100x error |
| Percentage | `SUM(ROUND(x))` ≠ `ROUND(SUM(x))` |

```sql
-- Rounding drift check
SELECT o.order_id,
       ROUND(SUM(i.quantity * i.unit_price), 2)     AS rounded_at_end,
       SUM(ROUND(i.quantity * i.unit_price, 2))     AS rounded_per_line,
       ROUND(SUM(i.quantity*i.unit_price),2) - SUM(ROUND(i.quantity*i.unit_price,2)) AS drift
FROM   orders o JOIN order_items i ON i.order_id=o.order_id
GROUP  BY o.order_id
HAVING ROUND(SUM(i.quantity*i.unit_price),2) <> SUM(ROUND(i.quantity*i.unit_price,2));
```

**Step 8 — Agar API sahi hai par UI galat: frontend.**

- Browser devtools → **Network tab** — actual response body kya aaya (proxy/interceptor ne badla toh nahi?)
- **Stale frontend state** — component ne refetch nahi kiya, Redux/store purana hai
- **Optimistic update** jo server response se reconcile nahi hua
- **Locale formatting** — `1,00,000` (Indian) vs `100,000` — number sahi hai, format alag
- **Wrong field bind** — template mein `order.amount` likha jab `order.totalAmount` hai
- **Race** — do parallel requests, jo baad mein aayi wo pehle wali ko overwrite kar gayi

**Step 9 — RCA document karo.**

```
Symptom:   Order 9 ki UI par date "2025-04-01" dikh rahi hai, DB mein "2025-04-02"
Layer:     DB=correct  API=correct(2025-04-02T00:00:00+05:30)  UI=wrong
Root cause: Frontend `new Date(str).toLocaleDateString()` browser TZ (UTC) mein
            convert kar raha hai; IST midnight UTC mein pichhla din hai
Fix:       Date-only fields ko timezone conversion ke bina render karo
Prevention: Ek regression test jo IST midnight-boundary date par assert kare;
            aur ek lint rule date-only fields par toLocaleDateString ke against
```

**"Prevention" line hamesha likhna.** Bug fix karna developer ka kaam hai; **wo dobara na ho** ye QA ka kaam hai.

> **Interview answer (structured, bolne layak):**
> "A 200 only means the request completed, so my first job is to find which layer the data goes wrong at, and I do that by bisecting rather than guessing.
> Step one is the database row — is the stored value actually correct? If it isn't, the read path is fine and this is really a write-path bug, which flips the whole investigation; I'd go to the audit log and find which request wrote it.
> Step two, if the row is right, is to curl the API directly and look at the raw JSON, not the rendered page. That single step halves the problem: if the JSON is already wrong the bug is in the backend, and if the JSON is right the bug is in the frontend.
> On the backend side I'd check, roughly in order of how often they're the cause: caching — clear the cache and see whether it comes back correct, which points at an invalidation bug; a stale read from a replica, which has a distinctive signature of being wrong immediately after an update and correct after a refresh; timezone conversion, where the give-away is being off by exactly the offset or by one whole day, because midnight in IST is the previous day in UTC; DTO mapping, where a renamed or mis-mapped field silently serialises as null and the frontend renders a zero; and rounding, especially rounding per line versus rounding the total, or a float used where a decimal belongs.
> On the frontend side: the network tab to confirm what actually arrived, stale component state that never refetched, an optimistic update that wasn't reconciled, or locale formatting, where the number is right and only the presentation differs.
> Then I write it up with the layer where correct became incorrect, the root cause, and — the part I care most about — the prevention: what test now exists so this can't come back. For a timezone bug that means a regression test pinned to a midnight boundary, because that's the case that would have caught it and clearly didn't exist."

> **Cross-question: "Sabse pehla step kya hoga?"**
> Reproduce karna aur "wrong" ko exactly define karna — kaunsa field, expected vs actual, aur **consistent hai ya intermittent**. Intermittent almost hamesha caching, replica lag ya race hota hai; consistent almost hamesha mapping ya logic hota hai. Ye ek sawaal aadha search space kaat deta hai.

---

# 36 — Test data setup & cleanup

## 36.1 Teen strategies

| Strategy | Kaise | Pros | Cons |
|---|---|---|---|
| **Transaction rollback** | test transaction mein chalao, end mein rollback | fastest, perfect isolation | jo code khud commit karta hai wo test nahi hoga; API tests (alag process) mein kaam nahi karta |
| **Truncate + reseed** | har test ke pehle tables clear karo | deterministic, simple | slow, parallel tests mein clash |
| **Unique data per test** | har test apna prefixed data banaye | parallel-safe, prod-jaisa | cleanup discipline chahiye, leaks ho sakte hain |

**API/E2E tests ke liye teesra hi practical hai.** Transaction rollback sirf tab kaam karta hai jab test aur code **same connection** par hon.

## 36.2 Setup — factory pattern

```python
# conftest.py
import uuid, pytest
from decimal import Decimal

RUN_ID = uuid.uuid4().hex[:8]      # is poore run ke liye unique tag

@pytest.fixture
def make_supplier(db):
    created = []
    def _make(name=None, city="Jaipur", rating=4):
        name = name or f"QA-{RUN_ID}-supplier-{uuid.uuid4().hex[:6]}"
        row = db.query_one("""
            INSERT INTO suppliers (name, city, rating)
            VALUES (%s, %s, %s) RETURNING supplier_id, name
        """, (name, city, rating))
        created.append(row["supplier_id"])
        return row
    yield _make
    # teardown — reverse order, FK-safe
    for sid in reversed(created):
        db.execute("DELETE FROM suppliers WHERE supplier_id = %s", (sid,))

@pytest.fixture
def make_order(db, make_supplier):
    created = []
    def _make(project_id=1, status="DRAFT", items=None):
        supplier = make_supplier()
        order = db.query_one("""
            INSERT INTO orders (project_id, supplier_id, order_date, status, total_amount)
            VALUES (%s, %s, CURRENT_DATE, %s, 0) RETURNING order_id
        """, (project_id, supplier["supplier_id"], status))
        oid = order["order_id"]
        created.append(oid)
        total = Decimal(0)
        for it in (items or [("Cement OPC 53", 10, "400.00")]):
            material, qty, price = it
            db.execute("""INSERT INTO order_items (order_id, material, quantity, unit_price)
                          VALUES (%s,%s,%s,%s)""", (oid, material, qty, price))
            total += Decimal(qty) * Decimal(price)
        db.execute("UPDATE orders SET total_amount=%s WHERE order_id=%s", (total, oid))
        return oid
    yield _make
    for oid in reversed(created):
        db.execute("DELETE FROM order_items WHERE order_id=%s", (oid,))
        db.execute("DELETE FROM orders WHERE order_id=%s", (oid,))
```

**Factory ke 5 rules:**
1. **Har test apna data banaye.** Shared fixtures = order-dependent, flaky tests.
2. **Unique naming with a run tag** — `QA-<run>-<random>`. Parallel-safe, aur leak hone par identify karna aasan.
3. **Sensible defaults, override optional** — `make_order()` kaam kare, `make_order(status="APPROVED")` bhi.
4. **Created IDs track karo**, cleanup ke liye.
5. **Reverse order mein delete karo** — children pehle, parents baad mein (FK).

## 36.3 Cleanup — defence in depth

```python
# Layer 1: fixture teardown (upar) — normal case

# Layer 2: session-level sweeper — jab test crash ho jaye
@pytest.fixture(scope="session", autouse=True)
def sweep_leftovers(db):
    yield
    db.execute("DELETE FROM order_items WHERE order_id IN "
               "(SELECT order_id FROM orders WHERE created_by_tag LIKE %s)", (f"QA-{RUN_ID}-%",))
    db.execute("DELETE FROM orders    WHERE created_by_tag LIKE %s", (f"QA-{RUN_ID}-%",))
    db.execute("DELETE FROM suppliers WHERE name LIKE %s", (f"QA-{RUN_ID}-%",))

# Layer 3: nightly janitor — purana orphaned QA data
# DELETE FROM suppliers WHERE name LIKE 'QA-%' AND created_at < now() - INTERVAL '3 days';
```

**Layer 2 aur 3 ka zikr karna interview mein maturity dikhata hai.** Test crash hoga — teardown nahi chalega. Uske liye plan hona chahiye.

## 36.4 Transaction-rollback fixture (unit/integration tests ke liye)

```python
@pytest.fixture
def db_txn(connection):
    """Test ke andar sab kuch rollback ho jayega — DB bilkul saaf."""
    txn = connection.begin()
    try:
        yield connection
    finally:
        txn.rollback()
```

**Kab kaam karta hai:** repository-layer tests, jahan test aur code same connection share karte hain.
**Kab nahi:** API tests jahan server alag process/connection hai — wahan tumhara rollback server ke commit ko affect nahi karega.

## 36.5 Bilkul mat karna

| ❌ Anti-pattern | Kyun bura hai |
|---|---|
| `DELETE FROM orders;` (bina WHERE) | shared staging DB par baaki sabka kaam uda diya |
| Production ka dump staging mein bina masking | GDPR/PII violation. Emails/phones mask karo |
| Hard-coded IDs (`order_id = 1`) | dusra test us row ko badal dega — flaky |
| Test data manually UI se banana | reproducible nahi, slow, version-controlled nahi |
| Shared "golden" record jo saare tests use karein | ek test usko modify karega, baaki toot jayenge |
| Cleanup ke bina chhod dena | staging DB dheere-dheere zeher ban jaati hai |

> **Interview answer:**
> "For unit and repository tests I run each test inside a transaction and roll it back, which is the fastest and gives perfect isolation. That doesn't work for API or end-to-end tests, because the server commits on its own connection — so there I use factory fixtures: each test creates exactly the data it needs, tagged with a unique run identifier, and the fixture tracks what it created and deletes it in reverse dependency order in teardown.
> I treat cleanup as defence in depth, because a crashed test never runs its teardown. So there's the fixture teardown, then a session-level sweeper that removes anything tagged with this run's identifier, and then a nightly job that removes QA-tagged rows older than a few days. Without those second and third layers a shared staging database slowly fills with orphaned test data, and eventually somebody's test starts failing for reasons that have nothing to do with their change.
> Two rules I hold to: never hard-code IDs, because another test will modify that row and you get an intermittent failure that's really a data collision; and never restore a production dump into a shared environment without masking personal data first."

---

# 37 — Data integrity checks QA should automate

## 37.1 Kyun

Zyadatar QA **feature** test karta hai: "PO banao, dekho ban gaya". Bahut kam QA **invariants** test karta hai: *"is there any purchase order in the entire database whose stored total doesn't match its line items?"*

Difference ye hai:
- **Feature test** ek known path ko verify karta hai.
- **Invariant check** poore data par ek rule check karta hai — jo **anjaan paths** se aayi corruption bhi pakadta hai: buggy migrations, manual DB fixes, race conditions, partial failures, purane code ke chhode hue nishan.

**Interview mein bolne ki line:** *"Feature tests verify the paths we thought of. Invariant checks catch the corruption that came from the paths we didn't."*

## 37.2 The 6 categories

### Category 1 — Orphan / referential integrity

```sql
-- Child bina parent ke
SELECT 'orphan_order_items' AS check_name, COUNT(*) AS violations
FROM   order_items i LEFT JOIN orders o ON o.order_id = i.order_id
WHERE  o.order_id IS NULL
UNION ALL
SELECT 'orphan_invoices', COUNT(*)
FROM   invoices inv LEFT JOIN orders o ON o.order_id = inv.order_id
WHERE  o.order_id IS NULL
UNION ALL
SELECT 'orphan_payments', COUNT(*)
FROM   payments p LEFT JOIN invoices i ON i.invoice_id = p.invoice_id
WHERE  i.invoice_id IS NULL
UNION ALL
-- Soft-delete orphan: FK ise nahi pakadta!
SELECT 'live_items_under_deleted_order', COUNT(*)
FROM   order_items i JOIN orders o ON o.order_id = i.order_id
WHERE  o.is_deleted = true;
```

**Result:**

| check_name | violations |
|---|---|
| orphan_order_items | 0 |
| orphan_invoices | 0 |
| orphan_payments | 0 |
| **live_items_under_deleted_order** | **1** |

**Ye MongoDB mein critical hai** — wahan FK hai hi nahi, toh ye **ekmatra** protection hai.

```javascript
// Mongo: purchase orders jinka project exist nahi karta
db.purchaseOrders.aggregate([
  { $match: { orgId: ORG, isDeleted: false } },
  { $lookup: { from: "projects", localField: "projectId", foreignField: "_id", as: "p" } },
  { $match: { p: { $size: 0 } } },
  { $project: { poNumber: 1, projectId: 1 } }
])
// Expected: 0 documents
```

### Category 2 — Impossible values

```sql
SELECT 'negative_quantity'   AS check_name, COUNT(*) FROM order_items WHERE quantity <= 0
UNION ALL SELECT 'negative_price',     COUNT(*) FROM order_items WHERE unit_price < 0
UNION ALL SELECT 'negative_payment',   COUNT(*) FROM payments     WHERE amount <= 0
UNION ALL SELECT 'negative_invoice',   COUNT(*) FROM invoices     WHERE amount < 0
UNION ALL SELECT 'negative_order',     COUNT(*) FROM orders       WHERE total_amount < 0
UNION ALL SELECT 'future_payment',     COUNT(*) FROM payments     WHERE paid_on > CURRENT_DATE
UNION ALL SELECT 'future_order',       COUNT(*) FROM orders       WHERE order_date > CURRENT_DATE
UNION ALL SELECT 'invalid_status',     COUNT(*) FROM orders
          WHERE status NOT IN ('DRAFT','APPROVED','DELIVERED','CANCELLED')
UNION ALL SELECT 'null_required_field',COUNT(*) FROM orders WHERE project_id IS NULL;
```

Sab **0** expected. Aur jo bhi 0 nahi hai — **wo constraint DB mein honi chahiye thi**. Ye check ka result hamesha ek schema recommendation deta hai.

### Category 3 — Derived data drift

```sql
-- (a) Order total vs items sum
SELECT o.order_id, o.total_amount, COALESCE(SUM(i.quantity*i.unit_price),0) AS computed
FROM   orders o LEFT JOIN order_items i ON i.order_id=o.order_id
WHERE  o.is_deleted = false
GROUP  BY o.order_id, o.total_amount
HAVING o.total_amount <> COALESCE(SUM(i.quantity*i.unit_price),0);
-- → order 12: 45000 vs 0   ← REAL DEFECT

-- (b) Invoice status vs payments
-- (Section Q17 wali query)

-- (c) Denormalized counter drift
SELECT p.project_id, p.cached_order_count, COUNT(o.order_id) AS actual
FROM   projects p LEFT JOIN orders o ON o.project_id=p.project_id AND o.is_deleted=false
GROUP  BY p.project_id, p.cached_order_count
HAVING p.cached_order_count <> COUNT(o.order_id);
```

> **[REAL] Mongo mein Merlin ka sabse important drift check:**
> ```javascript
> // orgId scalar aur org DBRef ka $id — hamesha barabar hone chahiye
> db.purchaseOrders.aggregate([
>   { $addFields: { refOrgId: "$org.$id" } },
>   { $match: { $expr: { $ne: ["$orgId", "$refOrgId"] } } },
>   { $project: { _id: 1, poNumber: 1, orgId: 1, refOrgId: 1 } }
> ])
> // Expected: 0 documents. Non-zero = TENANT ISOLATION BROKEN = security incident.
> ```
> **Interview mein bolna:** *"We denormalise the tenant ID as a scalar next to the reference so it can lead the compound index. The cost of any denormalisation is drift, so I assert nightly that the two always agree. And I'd escalate a mismatch as a security issue, not a data issue — tenant scoping is what isolates one customer's data from another's."*

### Category 4 — Duplicate business keys

```sql
SELECT 'dup_invoice_no' AS check_name, invoice_no AS value, COUNT(*) AS n
FROM   invoices GROUP BY invoice_no HAVING COUNT(*) > 1
UNION ALL
SELECT 'dup_idempotency_key', idempotency_key, COUNT(*)
FROM   payments WHERE idempotency_key IS NOT NULL
GROUP  BY idempotency_key HAVING COUNT(*) > 1
UNION ALL
SELECT 'dup_user_email', email, COUNT(*)
FROM   users GROUP BY email HAVING COUNT(*) > 1;
```

**Result:**

| check_name | value | n |
|---|---|---|
| dup_invoice_no | INV-2025-002 | 2 |

**Har duplicate ka fix ek missing unique constraint hai.** Report mein wo constraint likh kar do.

### Category 5 — State machine violations

```sql
-- Invoice bina delivery ke
SELECT 'invoice_on_undelivered_order' AS check_name, i.invoice_id, o.status
FROM   invoices i JOIN orders o ON o.order_id=i.order_id
WHERE  o.status <> 'DELIVERED';
-- → invoice 8 on order 8 (APPROVED)

-- Payment bina invoice ke
-- Cancelled order par payment
SELECT p.payment_id, o.order_id, o.status
FROM   payments p JOIN invoices i ON i.invoice_id=p.invoice_id
       JOIN orders o ON o.order_id=i.order_id
WHERE  o.status = 'CANCELLED';

-- Overpayment
SELECT i.invoice_id, i.amount, SUM(p.amount) AS paid
FROM   invoices i JOIN payments p ON p.invoice_id=i.invoice_id
GROUP  BY i.invoice_id, i.amount HAVING SUM(p.amount) > i.amount;
```

### Category 6 — Temporal / audit consistency

```sql
SELECT 'payment_before_invoice' AS check_name, COUNT(*) AS violations
FROM   payments p JOIN invoices i ON i.invoice_id=p.invoice_id
WHERE  p.paid_on < i.invoice_date
UNION ALL
SELECT 'invoice_before_order', COUNT(*)
FROM   invoices i JOIN orders o ON o.order_id=i.order_id
WHERE  i.invoice_date < o.order_date
UNION ALL
SELECT 'updated_before_created', COUNT(*)
FROM   orders WHERE updated_at < created_at
UNION ALL
SELECT 'status_change_without_audit', COUNT(*)
FROM   orders o
WHERE  o.status <> 'DRAFT'
  AND  NOT EXISTS (SELECT 1 FROM audit_log a
                   WHERE a.entity_type='ORDER' AND a.entity_id=o.order_id);
```

## 37.3 Isko ek automated test suite banao

```python
# tests/integrity/test_data_integrity.py
import pytest, yaml, pathlib

CHECKS = yaml.safe_load(pathlib.Path("tests/integrity/checks.yaml").read_text())

@pytest.mark.integrity
@pytest.mark.parametrize("check", CHECKS, ids=lambda c: c["name"])
def test_integrity_invariant(db, check):
    rows = db.query(check["sql"])
    assert rows == [], (
        f"\nINVARIANT VIOLATED: {check['name']}"
        f"\n  Rule:      {check['description']}"
        f"\n  Severity:  {check['severity']}"
        f"\n  Violations: {len(rows)} (showing first 5)"
        f"\n  {rows[:5]}"
        f"\n  Suggested fix: {check.get('fix', 'investigate')}"
    )
```

```yaml
# tests/integrity/checks.yaml
- name: order_total_matches_line_items
  severity: HIGH
  description: An order's stored total must equal the sum of its line items
  fix: Recompute totals; ensure order and item writes share a transaction
  sql: |
    SELECT o.order_id, o.total_amount, COALESCE(SUM(i.quantity*i.unit_price),0) AS computed
    FROM orders o LEFT JOIN order_items i ON i.order_id=o.order_id
    WHERE o.is_deleted = false
    GROUP BY o.order_id, o.total_amount
    HAVING o.total_amount <> COALESCE(SUM(i.quantity*i.unit_price),0)

- name: no_duplicate_invoice_numbers
  severity: CRITICAL
  description: invoice_no must be unique
  fix: "ALTER TABLE invoices ADD CONSTRAINT uq_invoice_no UNIQUE (invoice_no)"
  sql: |
    SELECT invoice_no, COUNT(*) FROM invoices GROUP BY invoice_no HAVING COUNT(*) > 1

- name: no_orphan_payments
  severity: CRITICAL
  description: Every payment must reference an existing invoice
  fix: Add a foreign key; investigate the write path that created the orphan
  sql: |
    SELECT p.payment_id FROM payments p
    LEFT JOIN invoices i ON i.invoice_id = p.invoice_id
    WHERE i.invoice_id IS NULL
```

**YAML-driven design ke 3 fayde (interview mein bolna):**
1. Naya check add karna = **ek YAML block**, koi Python nahi. Toh developers aur BAs bhi contribute kar sakte hain.
2. Failure message mein **rule, severity aur suggested fix** hai — bug report khud likh jaata hai.
3. Same file **nightly job** aur **CI** dono mein reuse hoti hai.

## 37.4 Kahan chalao

| Kahan | Kaunse checks | Failure par |
|---|---|---|
| **CI (har PR)** | fast structural checks seed data par | build fail |
| **Post-deploy smoke** | critical invariants | rollback trigger |
| **Nightly, staging** | poora suite | Slack alert + ticket |
| **Nightly, production (read-only)** | poora suite | page on-call for CRITICAL |
| **Migration ke baad** | poora suite, before + after | migration rollback |

> **Interview answer:**
> "Feature tests verify the paths we thought of. Invariant checks catch corruption that arrived through the paths we didn't — a bad migration, a manual production fix, a race condition, a partial failure. So alongside functional tests I maintain a suite of database invariants that run nightly.
> I group them into six categories: orphans and referential integrity — which matter far more in MongoDB, where there are no foreign keys at all, so a query is the only thing standing between you and dangling references; impossible values like negative amounts or future dates; drift between derived data and its source, such as a stored order total that no longer matches its line items; duplicate business keys, where every hit tells you a unique constraint is missing; state-machine violations like an invoice against an order that was never delivered; and temporal consistency, like a payment dated before its invoice.
> I implement them as data rather than code — a YAML file of named checks with a description, a severity and a suggested fix, driven by one parametrised test. That means a developer or a BA can add a rule without touching Python, and when one fails the message already contains the rule, the severity, sample violating rows and the recommended fix, so the bug report writes itself.
> The one I'd single out from our system: we denormalise the tenant ID next to a reference so it can lead the compound index, and I assert nightly that the two always agree. If they ever diverge, one tenant's data could surface for another — so I'd escalate that as a security incident, not a data quality issue."

---

# 38 — Verifying migrations

## 38.1 Migration kya-kya ho sakti hai

| Type | Example | Main risk |
|---|---|---|
| **Schema** | column add, rename, type change | lock, downtime, breaking old code |
| **Data** | backfill, transform, cleanup | wrong transform, partial run |
| **System** | MySQL → Postgres, monolith → services | everything |
| **Version** | Postgres 13 → 16, Mongo 5 → 7 | behaviour changes |

## 38.2 The 5-phase framework

**Phase 1 — BEFORE: baseline capture karo**

Ye phase log skip karte hain, aur phir prove hi nahi kar paate ki kya toota.

```sql
-- Row counts
SELECT 'orders' AS t, COUNT(*) FROM orders
UNION ALL SELECT 'order_items', COUNT(*) FROM order_items
UNION ALL SELECT 'invoices', COUNT(*) FROM invoices
UNION ALL SELECT 'payments', COUNT(*) FROM payments;

-- Financial control totals — SABSE IMPORTANT
SELECT SUM(total_amount) AS orders_total,
       (SELECT SUM(amount) FROM invoices) AS invoices_total,
       (SELECT SUM(amount) FROM payments) AS payments_total,
       (SELECT SUM(quantity*unit_price) FROM order_items) AS items_total
FROM   orders;

-- Distribution — sirf total nahi, shape bhi
SELECT status, COUNT(*), SUM(total_amount) FROM orders GROUP BY status ORDER BY status;

-- Checksum — har row ka fingerprint
SELECT MD5(STRING_AGG(order_id || '|' || project_id || '|' || status || '|' || total_amount,
                      ',' ORDER BY order_id)) AS checksum
FROM orders;

-- Boundary samples — pehli, aakhri, sabse badi, NULL wali rows alag se save karo
```

**Phase 2 — DRY RUN on a production-sized copy**

- Prod ka masked copy lo, migration chalao
- **Duration measure karo** — 10-minute lock production mein acceptable hai?
- **Rollback test karo** — ye asli test hai. "Migration chal gayi" kaafi nahi; **"migration wapas ho sakti hai"** chahiye.

**Phase 3 — AFTER: reconcile**

```sql
-- 1. Row counts match?
-- 2. Control totals match? (paisa ka ek rupya bhi idhar-udhar nahi)
-- 3. Symmetric difference — the strongest check
(SELECT order_id, project_id, status, total_amount FROM orders_old
 EXCEPT
 SELECT order_id, project_id, status, total_amount FROM orders)
UNION ALL
(SELECT order_id, project_id, status, total_amount FROM orders
 EXCEPT
 SELECT order_id, project_id, status, total_amount FROM orders_old);
-- 0 rows = provably identical, DONO directions mein
```

**Symmetric difference kyun dono directions mein?** Sirf ek direction "missing rows" pakadta hai. Doosri direction **"extra rows"** pakadta hai — jaise duplicate ban gaye. Dono ke bina proof adhoora hai.

```sql
-- 4. Nayi columns backfill hui?
SELECT COUNT(*) FROM orders WHERE new_column IS NULL;   -- expected 0

-- 5. Type conversion ne precision toh nahi khoyi?
SELECT order_id, old_amount, new_amount, old_amount - new_amount AS loss
FROM   migration_audit WHERE old_amount <> new_amount;

-- 6. Encoding/truncation
SELECT supplier_id, name FROM suppliers WHERE LENGTH(name) = 50;  -- max length pe atke?
SELECT * FROM suppliers WHERE name ~ '[^\x00-\x7F]';              -- non-ASCII intact?

-- 7. Constraints/indexes wapas aaye?
SELECT conname, contype FROM pg_constraint WHERE conrelid='orders'::regclass;
SELECT indexname FROM pg_indexes WHERE tablename='orders';

-- 8. Sequences reset hui?
SELECT last_value FROM orders_order_id_seq;   -- MAX(order_id) se zyada honi chahiye
```

**Point 8 bahut miss hota hai:** data copy ke baad sequence purani value par reh jaati hai → agla insert **duplicate key error**. Ye migration ke baad ka classic P1 hai.

**Phase 4 — Functional & non-functional**

- Critical user journeys chalao (create PO → invoice → payment)
- Reports ke numbers migration se **pehle** wale se compare karo
- **Query performance** — nayi table par plans same hain? `ANALYZE` chalaya?
- Sab **integrity checks** (Section 37) dobara chalao

**Phase 5 — Monitoring after cutover**

- Error rate, latency (24–48 ghante)
- Wo **control totals daily** compare karo dual-write period mein
- Rollback plan **ready aur tested** rakho

## 38.3 Mongo-specific migration checks

```javascript
// 1. Naya field har document mein pahuncha?
db.purchaseOrders.countDocuments({ currency: { $exists: false } })     // → 0

// 2. Type consistency — kuch documents mein string, kuch mein number?
db.purchaseOrders.aggregate([
  { $group: { _id: { $type: "$totalAmount" }, n: { $sum: 1 } } }
])
// → [{ _id: "decimal", n: 12 }]   ← ek hi type hona chahiye

// 3. Purani field bachi hui hai?
db.purchaseOrders.countDocuments({ oldFieldName: { $exists: true } })  // → 0

// 4. Indexes wapas bane?
db.purchaseOrders.getIndexes()

// 5. Control total
db.purchaseOrders.aggregate([{ $group: { _id: null, t: { $sum: "$totalAmount" } } }])
```

**Type consistency check Mongo mein khaas important hai** — flexible schema ka matlab hai ki migration script agar aadhi chali toh ek collection mein **do alag types** ho sakte hain, aur queries silently kuch documents miss karengi.

## 38.4 Automated migration test

```python
@pytest.mark.migration
def test_migration_preserves_financial_totals(db_old, db_new):
    for table, col in [("orders","total_amount"), ("invoices","amount"),
                       ("payments","amount")]:
        old = db_old.scalar(f"SELECT COALESCE(SUM({col}),0) FROM {table}")
        new = db_new.scalar(f"SELECT COALESCE(SUM({col}),0) FROM {table}")
        assert old == new, f"{table}.{col}: {old} → {new}, diff {new-old}"

@pytest.mark.migration
def test_migration_row_sets_identical(db):
    diff = db.query(SYMMETRIC_DIFFERENCE_SQL)
    assert diff == [], f"{len(diff)} rows differ. First 5: {diff[:5]}"

@pytest.mark.migration
def test_sequences_advanced_past_max_id(db):
    for table, seq, pk in [("orders","orders_order_id_seq","order_id")]:
        max_id = db.scalar(f"SELECT MAX({pk}) FROM {table}")
        last   = db.scalar(f"SELECT last_value FROM {seq}")
        assert last >= max_id, f"{seq} at {last} but max {pk} is {max_id} — next insert will collide"
```

> **Interview answer:**
> "I treat migration verification in five phases, and the first one is the one people skip: capture a baseline before you touch anything. Row counts per table, control totals for every money column, the distribution by status — not just the grand total, because a total can be preserved while the shape is wrong — and a checksum over the sorted rows.
> Then a dry run against a production-sized masked copy, where I measure how long it takes, because a lock that's fine on ten thousand rows isn't fine on ten million, and — critically — I test the rollback. 'The migration ran' isn't the bar; 'the migration can be undone' is.
> After the migration, the strongest single check is a symmetric difference: old EXCEPT new, unioned with new EXCEPT old. Zero rows proves the two sets are identical in both directions. One direction only finds missing rows; you need the other to find duplicates and extras. Alongside that: no NULLs in backfilled columns, no precision lost in type conversions, no truncation at the column length boundary, non-ASCII characters intact, and constraints and indexes recreated — those get dropped for speed during a load and then forgotten.
> The one that has genuinely bitten teams I'd mention is sequences. If you copy data with explicit IDs, the sequence stays at its old value and the very first insert after cutover fails with a duplicate key. So I assert the sequence is past the max ID.
> Then functional journeys, a rerun of the whole integrity suite, and monitoring the error rate and control totals daily through the dual-write period, with a tested rollback still available."

---

# 39 — DB access from Python tests

## 39.1 Golden rule pehle

> **QA ko shared/production database par write access nahi chahiye. Read-only credentials maango.**

**Kyun (ye poora bolna interview mein):**

| Reason | Detail |
|---|---|
| **Blast radius** | ek galat `UPDATE` bina `WHERE` = poori staging DB corrupt, poori team blocked |
| **Test pollution** | manually likha hua data doosron ke tests ko break karta hai, aur baad mein "prod bug" lagta hai |
| **Reproducibility** | manual DB fixes se test "pass" ho jaata hai — bug fix nahi hua, chhup gaya |
| **Auditability** | shared credentials se pata nahi chalta kisne kya kiya |
| **Blame** | koi aur cheez toote toh QA par shak jaata hai. Read-only access **tumhari suraksha** bhi hai |
| **The right path** | data **API se** banao. Wahi user ka path hai — aur usko test karna hi tumhara kaam hai |

**Exception:** ephemeral, per-test, disposable databases (Docker/testcontainers) par full access bilkul theek hai. Rule **shared** environments ke liye hai.

```sql
-- QA ke liye read-only role
CREATE ROLE qa_readonly LOGIN PASSWORD '…';
GRANT CONNECT ON DATABASE merlin TO qa_readonly;
GRANT USAGE ON SCHEMA public TO qa_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO qa_readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO qa_readonly;
-- Aur safety net:
ALTER ROLE qa_readonly SET default_transaction_read_only = on;
ALTER ROLE qa_readonly SET statement_timeout = '30s';
```

`default_transaction_read_only = on` — ab galti se bhi `UPDATE` chal hi nahi sakta. **Ye line interview mein bolna** — ye dikhata hai ki tum policy ko technically enforce karna jaante ho.

## 39.2 Postgres — reusable fixture (psycopg2)

```python
# conftest.py
import os, pytest, psycopg2
from psycopg2.extras import RealDictCursor
from contextlib import contextmanager

@pytest.fixture(scope="session")
def db_config():
    return dict(
        host     = os.environ["DB_HOST"],
        port     = int(os.getenv("DB_PORT", 5432)),
        dbname   = os.environ["DB_NAME"],
        user     = os.environ["DB_USER"],          # read-only user
        password = os.environ["DB_PASSWORD"],      # NEVER hard-code / commit
        connect_timeout = 5,
        options  = "-c statement_timeout=30000",   # 30s — runaway query se bacho
    )

class Db:
    """Chhota read-only helper. Deliberately koi execute() nahi hai."""
    def __init__(self, conn):
        self._conn = conn

    def query(self, sql, params=None):
        with self._conn.cursor(cursor_factory=RealDictCursor) as cur:
            cur.execute(sql, params)
            return [dict(r) for r in cur.fetchall()]

    def query_one(self, sql, params=None):
        rows = self.query(sql, params)
        assert len(rows) <= 1, f"Expected at most 1 row, got {len(rows)}"
        return rows[0] if rows else None

    def scalar(self, sql, params=None):
        row = self.query_one(sql, params)
        return next(iter(row.values())) if row else None

    def exists(self, sql, params=None):
        return self.scalar(f"SELECT EXISTS({sql})", params)

@pytest.fixture(scope="session")
def _connection(db_config):
    conn = psycopg2.connect(**db_config)
    conn.set_session(readonly=True, autocommit=True)   # ← belt and braces
    yield conn
    conn.close()

@pytest.fixture
def db(_connection):
    """Har test ko fresh view mile — autocommit hai, isliye koi stale snapshot nahi."""
    return Db(_connection)
```

**Use:**

```python
def test_order_is_persisted_correctly(api, db):
    resp = api.post("/orders", json={"projectId": 1, "supplierId": 2,
                                     "items": [{"material": "Sand",
                                                "quantity": 100, "unitPrice": "400.00"}]})
    assert resp.status_code == 201
    order_id = resp.json()["orderId"]

    row = db.query_one("""
        SELECT order_id, project_id, supplier_id, status, total_amount, is_deleted
        FROM   orders WHERE order_id = %s
    """, (order_id,))

    assert row is not None,                 "API returned 201 but no row exists"
    assert row["status"] == "DRAFT"
    assert row["total_amount"] == Decimal("40000.00")
    assert row["is_deleted"] is False

    items = db.query("SELECT material, quantity, unit_price FROM order_items "
                     "WHERE order_id=%s ORDER BY item_id", (order_id,))
    assert len(items) == 1
    assert items[0]["material"] == "Sand"
```

**Fixture design ke 5 points (interview mein bolna):**
1. **Connection session-scoped** — har test par nayi connection banana slow hai.
2. **`readonly=True` connection par** — code-level guarantee, sirf policy nahi.
3. **`statement_timeout`** — ek galat query poori CI ko nahi rok sakti.
4. **`RealDictCursor`** — `row["status"]` padhne mein `row[3]` se kahin behtar; column order badalne par test nahi tootega.
5. **Parameterised queries hamesha** (`%s`, f-string nahi) — SQL injection se bachav aur type handling sahi.

## 39.3 MongoDB — reusable fixture (pymongo)

```python
# conftest.py
import os, pytest
from pymongo import MongoClient, ReadPreference
from bson import ObjectId

@pytest.fixture(scope="session")
def mongo_client():
    client = MongoClient(
        os.environ["MONGO_URI"],              # read-only user ke credentials
        serverSelectionTimeoutMS=5000,
        connectTimeoutMS=5000,
        readPreference="primary",             # ← read-after-write consistency ke liye
    )
    client.admin.command("ping")              # fail fast agar connection nahi bana
    yield client
    client.close()

@pytest.fixture
def mongo(mongo_client):
    return mongo_client[os.environ["MONGO_DB"]]

@pytest.fixture
def org_id():
    return ObjectId(os.environ["TEST_ORG_ID"])
```

**Use — Merlin-shaped assertions:**

```python
def test_purchase_order_document_shape(api, mongo, org_id, token):
    resp = api.post("/purchase-orders",
                    json={"projectId": PROJECT_ID, "items": [
                        {"material": "Cement OPC 53", "quantity": 200, "unitPrice": "400.00"}]},
                    headers={"Authorization": token})
    assert resp.status_code == 201
    po_id = ObjectId(resp.json()["id"])

    doc = mongo.purchaseOrders.find_one({"_id": po_id})
    assert doc is not None, "API returned 201 but no document exists"

    # Tenant scoping — Merlin ka core invariant
    assert doc["orgId"] == org_id,          "Document not scoped to caller's org"
    assert doc["org"]["$id"] == org_id,     "DBRef and scalar orgId disagree"

    # Soft delete default
    assert doc["isDeleted"] is False

    # Embedded items
    assert len(doc["lineItems"]) == 1
    assert doc["lineItems"][0]["material"] == "Cement OPC 53"

    # Derived total consistent with items
    computed = sum(li["quantity"] * li["unitPrice"] for li in doc["lineItems"])
    assert doc["totalAmount"] == computed, \
        f"Stored total {doc['totalAmount']} != computed {computed}"

    # Timestamps
    assert doc["createdAt"] <= doc["updatedAt"]


def test_query_uses_the_compound_index(mongo, org_id):
    """Performance ko test mein lock karo — regression pakdo."""
    plan = mongo.command("explain", {
        "find": "purchaseOrders",
        "filter": {"orgId": org_id, "isDeleted": False, "status": "DELIVERED"}
    }, verbosity="executionStats")

    stats = plan["executionStats"]
    stage = plan["queryPlanner"]["winningPlan"]

    assert "COLLSCAN" not in str(stage), f"Query fell back to a collection scan:\n{stage}"
    examined, returned = stats["totalDocsExamined"], stats["nReturned"]
    assert examined <= max(returned * 3, 100), \
        f"Examined {examined} docs to return {returned} — index is not selective enough"
```

**`test_query_uses_the_compound_index` — ye interview mein bolne layak hai.** Zyadatar QA functional correctness test karta hai; **plan** ko assert karna ek alag level hai. Ye wo regression pakadta hai jahan koi developer query mein ek field add kar deta hai aur index chup-chaap use hona band ho jaata hai — functionally sab pass, production mein sab slow.

## 39.4 Credentials handling

```python
# ❌ KABHI NAHI
DB_PASSWORD = "prod_pass_123"          # commit ho gaya = incident

# ✅ Environment variables, secrets manager se inject
password = os.environ["DB_PASSWORD"]

# ✅ CI mein: GitHub Actions secrets / Vault / AWS Secrets Manager
# ✅ Local mein: .env file jo .gitignore mein hai
# ✅ Assertion messages mein credentials kabhi print mat karo
```

```python
# Aur ek safety check — galti se prod se connect toh nahi ho rahe?
@pytest.fixture(scope="session", autouse=True)
def assert_not_production(db_config):
    host = db_config["host"].lower()
    forbidden = ("prod", "production", "live")
    assert not any(f in host for f in forbidden), \
        f"REFUSING TO RUN: host '{host}' looks like production"
```

**Ye guard fixture bolna interview mein.** Ek line, aur wo poori team ko ek accident se bacha sakti hai.

> **Interview answer:**
> "For database checks in tests I use psycopg2 for Postgres and pymongo for MongoDB, wrapped in a session-scoped connection fixture and a thin helper exposing query, query_one and scalar. Three things I build into the fixture rather than leaving to discipline: the connection is opened read-only, there's a statement timeout so one bad query can't hang CI, and rows come back as dictionaries so assertions read by column name and don't break when column order changes. And I add an autouse fixture that refuses to run if the host name looks like production.
> On access: I ask for read-only credentials on any shared environment, and I'd argue for it even if offered write access. The reason isn't only blast radius — it's that if I can fix data by hand, I will, and then a test passes while the bug is still there. Test data should be created through the API, because that's the path the user takes and it's the path I'm supposed to be testing. On disposable per-test containers, full access is fine; the rule is about shared environments.
> The check I'd highlight is asserting on the query plan, not just the data. I run explain and assert the plan isn't a collection scan and that the documents examined are in the same ballpark as the documents returned. That catches a specific regression class: someone adds a field to a query filter, the compound index silently stops applying, every functional test still passes, and it only shows up as a production slowdown weeks later."

> **Cross-question: "Har API test mein DB check karoge?"**
> Nahi. DB assertions **coupling** badhati hain — schema badla toh test toota, chahe behaviour sahi ho. Main DB check tab karta hoon jab: (a) **side effects** verify karne hon jo API return nahi karta — audit rows, status transitions, ledger entries; (b) **API ke jhooth bolne ka shak** ho — 200 aaya par sach mein likha? (c) **negative test** — error ke baad data **badla toh nahi**; (d) **integrity invariants**. Simple round-trip ke liye API se hi read karke assert karna behtar hai — kam brittle, aur wo bhi ek real code path exercise karta hai.

---

# PART E — SENIOR

---

# 40 — Senior scenario questions

> Ye 5 sawaal senior SDET interviews mein **almost guaranteed** hain. Har ek ka jawab **structure** ke saath hai — pehle framework, phir detail. Interview mein pehle 3–4 line ka structure bolo ("I'd approach it in four phases..."), phir detail mein jao. Isse interviewer ko pata chal jaata hai ki tumhare paas method hai, list nahi.

---

## 40.1 — "How do you verify a data migration is correct?"

**Structure: baseline → dry run → reconcile → functional → monitor.** (Detail Section 38 mein hai; yahan interview-ready compressed answer.)

> **Interview answer:**
> "I verify a migration in five phases, and the first one is the one that usually gets skipped — capturing a baseline *before* anything is touched. Row counts per table, control totals for every money column, the distribution by status, and a checksum over the sorted rows. Without that you can't prove afterwards what changed; you can only argue about it.
> Second, a dry run against a production-sized masked copy. Two things I measure there: how long it takes, because a lock that's fine on ten thousand rows is an outage on ten million; and whether the rollback actually works. 'The migration ran' isn't the acceptance bar — 'the migration can be undone' is.
> Third, reconciliation. Row counts and control totals must match exactly. The strongest single check is a symmetric difference — old EXCEPT new, unioned with new EXCEPT old. Zero rows proves the sets are identical in *both* directions; one direction alone only finds missing rows, and you need the other to catch duplicates and extras. Alongside that I check that backfilled columns have no NULLs, that type conversions didn't lose precision, that nothing was truncated at a column-length boundary, that non-ASCII characters survived the encoding, and that constraints and indexes were recreated — those get dropped for load speed and then forgotten.
> The specific one I always check because it has bitten teams: sequences. If you copy rows with explicit IDs, the sequence stays at its old value and the very first insert after cutover fails with a duplicate key. So I assert the sequence is past the max ID.
> Fourth, functional verification — run the critical journeys end to end, compare the key reports against their pre-migration numbers, and re-run the whole data-integrity suite.
> Fifth, monitoring for forty-eight hours after cutover: error rate, latency, and the control totals compared daily through any dual-write period, with the tested rollback still available.
> And one thing I'd say about scope: on anything large I'd push for an expand-and-contract migration rather than a big-bang — add the new column, dual-write, backfill, switch reads, then remove the old column in a later release. Each step is individually reversible, which is worth far more than doing it in one go."

> **Cross-question: "Migration 8 ghante lagegi aur downtime allowed nahi hai. Kya karoge?"**
> Expand-and-contract. (1) Nayi column/table **nullable** add karo — instant, no lock. (2) **Dual-write** — application dono jagah likhe. (3) Purana data **batches mein backfill** karo (1000 rows at a time, sleep ke saath), taaki replication lag na bane. (4) Har batch ke baad reconcile karo. (5) Reads ko naye column par **feature flag ke peeche** switch karo. (6) Ek release baad purani column drop karo. Har step alag se reversible hai — aur ye QA ke liye behtar hai kyunki har phase alag se testable hai.

---

## 40.2 — "A report shows the wrong total. How do you find where it broke?"

**Structure: reproduce → bracket → bisect the pipeline → classify the cause.**

> **Interview answer:**
> "First I nail down what 'wrong' means: which number, what value is shown, what value is expected, and where the expected number comes from — because half the time the 'correct' figure is someone's spreadsheet with a different definition, and there's no bug at all. I also ask whether it's wrong by a small amount or a large one, because that alone narrows the cause: a rounding-sized error and a doubled total have completely different root causes.
> Then I bracket it. Is it wrong for every filter, or only for a particular project, date range or tenant? Is it wrong today only, or historically? A total that's correct until March and wrong from April onwards points at a data or code change at that boundary, and that's often enough to find it on its own.
> Then I bisect the pipeline. A reported number goes through roughly five stages — the source rows, the aggregation query, any cached or materialized layer, the API serialization, and the UI rendering. I compute the number independently at each stage and find where correct becomes incorrect.
> I start by computing the total directly from the base tables with a deliberately naive query — no joins, no cleverness — and compare it to what the report query produces. If those two disagree, the bug is inside the report query and there are four classic causes I check in order.
> One: fan-out from a join. Joining orders to line items multiplies each order row once per item, so a SUM over the order total double-counts. That's the single most common cause of a total that's too high, and it shows up as a total that's a suspiciously clean multiple. The fix is to aggregate the child table first, then join.
> Two: an inner join that should be a left join, or a right-table condition sitting in WHERE instead of ON — both silently drop rows and make the total too low. The tell is that the count on the summary doesn't match the count in the detail list.
> Three: NULL handling. A filter like `status <> 'CANCELLED'` excludes rows where status is NULL, and SUM over an empty set returns NULL rather than zero.
> Four: a filter mismatch — the report applies the soft-delete filter and the detail view doesn't, or the date range is inclusive on one side and half-open on the other.
> If the report query is correct, I move up: is there a materialized view or cache serving a stale number? I check when it last refreshed and whether the refresh job has been failing silently — that's extremely common, because a failed refresh doesn't break anything visibly, it just keeps serving yesterday. Then serialization: a decimal going through a JSON float loses precision, and a big total in JavaScript can exceed the safe integer range. Then rendering: rounding each row and summing gives a different answer from summing and rounding once, and locale formatting can make a correct number look wrong.
> Once I've found it, I write the regression test at the layer where it broke — and for a totals bug that means asserting on a fixed dataset with a known expected total, including the edge cases that caused it: an order with multiple line items, an order with none, and a NULL in the filtered column."

> **Cross-question: "Total sahi hai par ek particular project ka galat hai. Ab?"**
> Ye almost hamesha **data-specific** hai, code-specific nahi. Us project ki rows nikaal kar dekho ki wahan kya alag hai: NULL supplier, soft-deleted parent, duplicate row, ek order jiske items nahi, ek different currency, ya ek status jo enum mein hai hi nahi. Ek "poison row" dhoondo — phir wo bug reproduce karne ka minimal test case ban jaati hai.

---

## 40.3 — "How would you test a system where the same record can be updated by two users at once?"

**Structure: pehle expected behaviour define karo → phir race reproduce karo → phir invariant assert karo.**

> **Interview answer:**
> "The first thing I do is not testing at all — it's establishing what the *expected* behaviour is, because 'two users edit the same record' has three legitimate designs and they're tested differently. Either last-write-wins is acceptable, or the second writer should be rejected with a conflict, or the two edits should merge field by field. If nobody has decided which one it is, that's the first defect, and it's a requirements defect.
> Assuming the intended behaviour is optimistic locking — the common choice for a REST API — the failure mode I'm hunting is the lost update: both users read the same version, both compute from it, and the second write silently overwrites the first. The user who lost gets no error at all, which is what makes it so damaging.
> To reproduce it I need the two operations to genuinely overlap, which means threads synchronised on a barrier — without that they run staggered and the race never happens, and the test passes while the bug is real. Then I assert three separate things. First, the HTTP outcomes: with optimistic locking I expect one success and one 409 Conflict, not two successes. Second — and this is the one people skip — the database state, because two 200s and two 409s can both look plausible while the data is wrong; if the operation is an increment of five thousand and three thousand, the final value must be either 128,000 if both applied, or one of the intermediate values if one was rejected, and anything else means an update was lost. Third, no 500s: a serialization failure or a duplicate key error leaking as a 500 is a separate defect even when the invariant held, because the client can't tell a conflict from an outage.
> Beyond the happy race I'd test: the conflicting write arriving after a long think-time, so the version is genuinely stale rather than microseconds old; whether the client can even recover — does the API return enough information to show the user what changed; concurrent delete versus update, which is a case people forget; and I'd run the same scenario at higher concurrency, ten or twenty writers, and grep the database log for deadlocks afterwards, because deadlocks are invisible in the application if there's a retry and surface as 500s if there isn't.
> On the design side, I'd push back on any place where the fix is a read-modify-write in application code. An atomic increment in SQL, or `$inc` in MongoDB, has no race window at all. And for a uniqueness rule I'd want a unique index rather than an application-level existence check — we do exactly that for a rule that a scope can only be sold once, and the index means one writer wins and the other gets a duplicate key error we map to a 409, with no retry logic and no code path that can bypass it."

**Code jo interview mein likh ke dikha sakte ho:**

```python
import threading
from concurrent.futures import ThreadPoolExecutor

def test_concurrent_update_does_not_lose_data(api, db, order_id):
    db.execute("UPDATE orders SET total_amount=120000, version=1 WHERE order_id=%s", (order_id,))
    barrier = threading.Barrier(2)

    def bump(delta):
        barrier.wait()                     # dono exactly ek saath — race guarantee
        return api.patch(f"/orders/{order_id}", json={"amountDelta": delta})

    with ThreadPoolExecutor(2) as ex:
        codes = sorted(f.result().status_code for f in
                       [ex.submit(bump, 5000), ex.submit(bump, 3000)])

    final = db.scalar("SELECT total_amount FROM orders WHERE order_id=%s", (order_id,))

    assert 500 not in codes, f"Conflict leaked as a 500: {codes}"
    assert (codes == [200, 200] and final == 128000) \
        or (codes == [200, 409] and final in (125000, 123000)), \
        f"Lost update: codes={codes} final={final}"
```

> **Cross-question: "Optimistic aur pessimistic locking mein kaunsa chunoge?"**
> **Optimistic** (version column) — web/REST ke liye default. User ke think-time ke दौरान koi lock held nahi hota, koi blocking nahi. Cost: conflict par user ka kaam dobara karna pad sakta hai — isliye UI ko conflict gracefully handle karna chahiye. **Pessimistic** (`SELECT … FOR UPDATE`) — jab conflict **aam** ho aur retry mehnga ho (inventory allocation, seat booking). Cost: locks, contention, deadlock risk, aur transaction lamba nahi rakh sakte. Rule: *"Optimistic when conflicts are rare, pessimistic when they're the norm — and never hold a pessimistic lock across a user interaction or a network call."*

---

## 40.4 — "How do you test soft delete?"

**Structure: soft delete ek cross-cutting concern hai — har read path, har uniqueness rule, aur har downstream system pe asar daalta hai.**

> **Interview answer:**
> "Soft delete looks like a one-field change and it's actually one of the most leak-prone features in a system, because it turns 'delete' into 'update' and then every single read path has to remember to filter. Missing the filter in one query is enough.
> I test it in six areas.
> First, the delete itself: the row is still physically there, the flag is set, and there's a deleted-at timestamp and a deleted-by user. If those two aren't captured, the audit story is incomplete and that's worth raising.
> Second — and this is where the bugs actually are — *every* read path. Not just the list endpoint that the ticket mentions. The list, the detail-by-ID fetch, search, filters, exports, reports and dashboards, the aggregate counts, any autocomplete or picker, the public API, and any downstream consumer reading the same collection. The most common real bug is that the list is filtered but the count on the dashboard isn't, so the UI says 'showing 10 of 11'. And detail-by-ID is very often unfiltered — the item vanishes from the list, but the direct link still returns it with a 200. My assertion for that path is a 404, not a 200 with the data.
> Third, uniqueness. If email has a unique constraint and a user is soft-deleted, can I register that email again? Both answers are defensible, but the system has to have decided. And if the intent is 'yes, reuse is allowed', a plain unique constraint won't do it — you need a partial unique index restricted to non-deleted rows. That's exactly what we do for a rule in our system, and it's the cleanest way to express 'unique among live records'.
> Fourth, the relationship rules. Deleting a parent — do the children go too? Our data has an order that's soft-deleted while its line items are still live, which a foreign key can't catch, because soft delete is an UPDATE and the FK is perfectly satisfied. Whether that's a bug depends on the cascade rule, but nothing in the schema is enforcing it either way, so it has to be a query. And the reverse: can you attach a new child to a deleted parent? Can you create an invoice against a deleted order? Usually that should be rejected.
> Fifth, restore, if the feature exists — does the record come back complete, do its relationships still resolve, and does the unique constraint now conflict with something created while it was deleted? That last one is the interesting case and it's usually untested.
> Sixth, the long-term concerns: the table only ever grows, so I'd check that queries still use the right index once most rows are deleted — a partial index on the non-deleted rows is usually the right answer — and I'd ask whether there's a hard-delete retention job, because 'soft delete' and a GDPR right-to-erasure request are in direct conflict. If a user asks to be deleted and we only flip a flag, we haven't complied.
> The check I'd automate rather than test case by case is a query that finds live children under a deleted parent, and a comparison of the filtered count against the unfiltered count on every list endpoint. Those two catch the leaks that individual test cases miss."

**Concrete test set:**

```python
def test_soft_deleted_order_is_gone_from_every_read_path(api, db, order_id):
    api.delete(f"/orders/{order_id}")

    # DB: row still there, flag set, audit fields populated
    row = db.query_one("SELECT is_deleted, deleted_at, deleted_by FROM orders WHERE order_id=%s",
                       (order_id,))
    assert row is not None,            "Soft delete performed a hard delete"
    assert row["is_deleted"] is True
    assert row["deleted_at"] is not None
    assert row["deleted_by"] is not None

    # Every read path
    assert order_id not in [o["orderId"] for o in api.get("/orders").json()["items"]]
    assert api.get(f"/orders/{order_id}").status_code == 404, \
        "Direct fetch still returns a deleted order"
    assert order_id not in [o["orderId"] for o in api.get("/orders/search?q=").json()["items"]]

    # Count and list must agree
    listed = len(api.get("/orders?size=1000").json()["items"])
    counted = api.get("/orders/count").json()["count"]
    assert listed == counted, f"List shows {listed} but count says {counted}"

    # Aggregates exclude it
    total = api.get("/reports/order-totals").json()["grandTotal"]
    assert Decimal(str(total)) == db.scalar(
        "SELECT COALESCE(SUM(total_amount),0) FROM orders WHERE is_deleted=false")

    # Cannot attach new children to a deleted parent
    assert api.post(f"/orders/{order_id}/items",
                    json={"material": "Sand", "quantity": 1, "unitPrice": "400.00"}
                    ).status_code in (400, 404, 409)
```

> **Cross-question: "Soft delete vs hard delete + archive table — kaunsa better?"**
> **Soft delete**: simple, restore aasan; par har query pollute karti hai, table grow karti rehti hai, aur ek missed filter = data leak. **Hard delete + archive table**: main table saaf aur chhoti rehti hai, queries simple, aur delete sach mein delete hai (GDPR-friendly); par restore complex hai aur delete ke waqt do writes ek transaction mein chahiye. **Mera default:** user-facing entities jinhe accidentally delete kar sakte hain → soft delete. High-volume transient data (events, logs, sessions) → hard delete + TTL/archive. Aur dono cases mein **retention policy** honi chahiye — warna "soft delete" ka matlab "never delete" ban jaata hai.

---

## 40.5 — "What DB checks would you add to a nightly job?"

**Structure: 4 tiers — integrity, business invariants, health/performance, freshness.**

> **Interview answer:**
> "I'd organise a nightly database job into four tiers, each with a different severity and a different owner.
> Tier one is structural integrity — the checks that say the data is well-formed. Orphan rows in every child table; live children under a soft-deleted parent, which a foreign key can't catch because soft delete is an update; duplicate business keys like invoice number or a tenant-plus-email pair; impossible values such as negative quantities, negative payment amounts or future-dated records; and required fields that are null. Every one of these that fires tells me a constraint is missing from the schema, so the ticket writes itself — the fix is usually one DDL statement plus a cleanup.
> Tier two is business invariants — the rules that no constraint can express because they span tables. An order's stored total must equal the sum of its line items; an invoice's status must agree with the payments actually recorded against it; payments must never exceed the invoice; the ledger's debits must equal its credits for every journal entry; timestamps must be in a sensible order, so a payment is never dated before its invoice; and status transitions must be legal — no invoice against an order that was never delivered. In our multi-tenant system I'd add the most important one: every document's denormalised tenant ID must equal the tenant on its reference, because if those ever diverge one customer's data can surface for another. I'd wire that one to page someone, because it's a security issue rather than a data-quality issue.
> Tier three is health and performance, which is about tomorrow's incidents rather than today's. Table and index growth rate — a table growing ten percent a night has a capacity date. Index bloat, and unused indexes, which are pure write overhead. Slowest queries from pg_stat_statements or the profiler, with a diff against last night so a regression is visible. Replication lag. Connection pool saturation. Long-running or idle-in-transaction sessions, because in Postgres one forgotten open transaction blocks vacuum and bloats the whole database. And a count of deadlocks from the log, since those are invisible in the application when there's a retry.
> Tier four is freshness and job health — the meta layer. Did the materialized views refresh, and when? Did last night's ETL land? Is any scheduled job silently failing? This tier matters more than people expect, because a silent refresh failure doesn't break anything visibly — the dashboard just keeps serving yesterday's numbers, and nobody notices for a week.
> On implementation: I'd define the checks as data — a YAML file with a name, description, severity, the SQL, and a suggested fix — driven by one parametrised test, so a developer or a BA can add a rule without touching code, and a failure message already contains everything needed to raise the bug. Routing by severity: critical pages on-call, high opens a ticket automatically, medium goes to a Slack channel, and everything is written to a trend table so I can see whether a violation count is growing or was a one-off.
> Two things I'd insist on. Every check runs read-only with a statement timeout, because a nightly job should never be the thing that takes production down. And every check gets a seeded negative test proving it can actually fail — a check that has never fired is a check you don't know works."

**Skeleton:**

```yaml
# nightly-checks.yaml
- name: tenant_id_matches_reference
  tier: business_invariant
  severity: CRITICAL          # → page on-call, security impact
  description: Denormalised orgId must equal the org DBRef's $id
  fix: Investigate the write path; a mismatch can leak data across tenants
  mongo: |
    [{ "$addFields": { "refOrgId": "$org.$id" } },
     { "$match": { "$expr": { "$ne": ["$orgId", "$refOrgId"] } } },
     { "$limit": 20 }]

- name: order_total_matches_line_items
  tier: business_invariant
  severity: HIGH              # → auto-create ticket
  description: An order's stored total must equal the sum of its line items
  fix: Recompute totals; ensure order and item writes share one transaction

- name: materialized_view_freshness
  tier: freshness
  severity: MEDIUM            # → Slack
  description: mv_project_spend must have refreshed within the last 26 hours
  sql: |
    SELECT last_refresh FROM mv_refresh_log
    WHERE view_name='mv_project_spend' AND last_refresh < now() - INTERVAL '26 hours'
```

> **Cross-question: "Nightly job production par chalega — koi risk?"**
> Haan, aur usko explicitly manage karna chahiye. (1) **Read-only replica par chalao**, primary par nahi — analytics queries primary ka cache poison kar sakti hain. (2) Har query par **`statement_timeout`** lagao. (3) **Off-peak** window chuno aur staggered chalao, sab ek saath nahi. (4) **`EXPLAIN` pehle** run karo — koi check accidentally 10-table cross join na ho. (5) Job ka apna **timeout aur alert** ho — job hang ho gaya toh pata chalna chahiye. (6) Results ek **trend table** mein likho taaki "3 violations" vs "3 violations jo pichhle hafte 0 the" ka farak dikhe.

---

# 41 — Red flags — ye jawab mat dena

> Ye wo jawab hain jo **turant** junior ka signal dete hain — chahe baaki interview acha gaya ho. Har ek ke saath **kya bolna chahiye** bhi diya hai.

## 41.1 Honesty ke bare mein

| ❌ Mat bolna | ✅ Ye bolna |
|---|---|
| "Haan mujhe SQL achhi aati hai" (jab nahi aati) | *"I work with MongoDB day to day, so my aggregation pipelines are strong. I've deliberately learned relational SQL because the concepts carry over — indexes, transactions, isolation, normalisation — and I've been practising query writing. Ask me anything and if I don't know I'll tell you."* |
| "Maine wo kabhi use nahi kiya" (full stop) | *"I haven't used it in production. Here's what I understand it does, and here's the closest thing I have used…"* — hamesha **bridge** banao |
| Jhooth bolna ki tumne kuch kiya hai | Kabhi nahi. Ek follow-up question mein pakde jaoge, aur phir **poora** interview shaq ke daayre mein aa jaata hai |
| "Mujhe nahi pata" (aur chup) | *"I don't know that one. My guess would be X because of Y — is that close?"* Reasoning dikhana hi asli test hai |

**Sabse important:** tumhara project MongoDB par hai. **Ye chhupana mat.** Ye tumhari **taakat** hai, kamzori nahi. *"Our primary datastore is MongoDB"* bol kar phir Mongo ke real examples dena tumhe honest aur specific dikhata hai. Jo banda SQL ka rata hua jawab deta hai par project ka koi example nahi de paata, wo weaker lagta hai.

## 41.2 Technical red flags

| ❌ Mat bolna | Kyun galat hai | ✅ Ye bolna |
|---|---|---|
| "Index laga do, query fast ho jayegi" | Trade-off ignore kiya | *"An index would help the read, but it makes every write slower and adds storage — so I'd check the write path too, and confirm with EXPLAIN that it's actually used."* |
| "`SELECT *` theek hai" | Production mein nahi | *"Fine for debugging. In application code I name the columns, because adding a column silently changes the payload and breaks contracts."* |
| "NoSQL SQL se better hai" (ya ulta) | Religion, engineering nahi | *"They optimise for different access patterns. Documents win when you read an aggregate together; relational wins when you need cross-entity joins and multi-row invariants."* |
| "Normalization se performance kharab hoti hai" | Blanket, aur aksar ulta sach hai | *"Normalised tables are smaller, so more rows per page and better cache hits — writes usually get faster. I denormalise a specific read path when I've measured it, and then I add a drift check."* |
| "`COUNT(1)` `COUNT(*)` se fast hai" | Ek purani myth | *"They're identical; optimisers treat them the same. The real difference is COUNT of a column, which skips NULLs."* |
| "`NOT IN` aur `NOT EXISTS` same hain" | NULL par bilkul nahi | *"NOT IN returns zero rows if the subquery yields a NULL — silently. NOT EXISTS is NULL-safe, so I default to it."* |
| "`WHERE x <> 'A'` saare non-A rows dega" | NULLs miss ho jaate hain | *"It drops rows where x is NULL, because the comparison is UNKNOWN. I'd write `IS DISTINCT FROM` or add an explicit OR IS NULL."* |
| "Transaction laga do, race fix ho jayega" | Isolation level batayaa hi nahi | *"A transaction alone doesn't stop a lost update at READ COMMITTED. I'd use an atomic update, a version column, or a unique index depending on the invariant."* |
| "Mongo mein transactions nahi hote" | Galat, 4.0 se hain | *"Multi-document transactions exist since 4.0 but need a replica set. And a single document write is already atomic, which is often the better design."* |
| "Mongo schemaless hai" | Schema code mein hai | *"Schema-flexible, not schemaless. The schema lives in the application, so in practice a collection holds several schema versions at once — which is a real test area."* |
| "DELETE aur TRUNCATE same hain" | | *"TRUNCATE is DDL — no WHERE, no triggers, and in MySQL and Oracle it can't be rolled back."* |
| "Deadlock ka matlab DB hang ho gaya" | | *"No — the detector finds the cycle and aborts one transaction. The application needs retry logic; the fix is consistent lock ordering."* |
| "Views performance improve karte hain" | Normal views nahi karte | *"A plain view is just a stored query — same cost. A materialized view stores the result, which is faster but stale until refreshed."* |
| "Primary key hamesha auto-increment integer hona chahiye" | | *"Usually a surrogate key, but the type depends: sequential integers index better but are enumerable, so I'd want IDOR tests; UUIDs are safer for distributed writes but hurt index locality."* |

## 41.3 QA-mindset red flags — ye sabse zyada nuksan karte hain

| ❌ Mat bolna | ✅ Ye bolna |
|---|---|
| "API ne 200 diya toh pass" | *"A 200 means the request completed. I verify the side effects — the row exists, its status is the terminal one, the parent state advanced, and there's exactly one audit row."* |
| "Maine DB mein data theek kar diya, ab test pass ho raha hai" | **Ye sabse bada red flag hai.** *"I'd never fix data by hand to make a test pass — that hides the defect. I reproduce it, file it, and add the assertion that would have caught it."* |
| "Concurrency test karna mushkil hai" | *"It needs threads synchronised on a barrier, otherwise they run staggered and the race never fires. Then I assert on the HTTP codes *and* the final database state."* |
| "Wo edge case hai, low priority" | *"Let me quantify the impact first — if it's a money or tenant-isolation edge case, it's high severity regardless of how rare it is."* |
| "Developer ne bola ye by design hai" | *"Then I'd ask for it to be documented as such, and I'd add a test asserting the documented behaviour — so if it changes later, we know."* |
| "Test data prod se copy kar leta hoon" | *"Not without masking. Copying production PII into a shared environment is a compliance problem before it's a testing one."* |
| "Staging DB mein manually update kar deta hoon" | *"I create data through the API, because that's the path the user takes and it's the path I'm meant to be testing. On shared environments I ask for read-only credentials."* |
| "Automation ke liye time nahi mila" | *"I prioritised — here's what I automated first and why, and here's what's still manual with the risk that carries."* |

## 41.4 Communication red flags

| ❌ | ✅ |
|---|---|
| Definition ratt ke bolna, example ke bina | Har concept ke saath **apne project ka** example |
| Sawaal ka jawab dene se pehle clarify na karna | *"Before I write this — with ties, do you want the second distinct value or the second row?"* |
| Trade-off na batana | Har technical choice ke saath uska **cost** |
| Sirf "kya" batana, "kyun" nahi | *"…and the reason that matters is…"* |
| Interviewer ko interrupt karna / defensive hona | Sun lo, phir *"That's a fair point — in that case I'd…"* |
| 5 minute non-stop bolna | 60–90 second ka structured answer, phir *"Want me to go deeper on any of those?"* |

## 41.5 Do lines jo tumhe turant senior dikhati hain

Interview mein natural jagah par ye do lines daal dena:

> **"A check that has never failed is a check you don't know works."**
> — jab bhi automated validation/monitoring ki baat ho.

> **"Feature tests verify the paths we thought of. Invariant checks catch the corruption that came through the paths we didn't."**
> — jab bhi test strategy ki baat ho.

Aur ek teesri, Merlin-specific:

> **"We enforce that rule with a unique partial index rather than an application-level existence check, because a check-then-insert has a race window that the index doesn't."**
> — jab bhi uniqueness, concurrency, ya 409 ki baat ho.

---

# 42 — Quick revision cheatsheet

> **Interview se ek raat pehle sirf ye section padhna.** Baaki document seekhne ke liye hai; ye yaad karne ke liye.

## 42.1 SQL syntax — one-liners

| Chahiye | SQL |
|---|---|
| Filter | `WHERE col = x` / `<>` / `>` / `BETWEEN a AND b` / `IN (…)` / `LIKE 'a%'` / `IS NULL` |
| NULL-safe not-equal | `WHERE col IS DISTINCT FROM x` |
| Sort | `ORDER BY a ASC, b DESC NULLS LAST` |
| Page | `LIMIT 10 OFFSET 20` (deep pages ke liye keyset use karo) |
| Unique values | `SELECT DISTINCT col` |
| Group | `GROUP BY a, b` |
| Group filter | `HAVING COUNT(*) > 1` |
| Duplicates | `GROUP BY key HAVING COUNT(*) > 1` |
| Anti-join | `LEFT JOIN b ON … WHERE b.id IS NULL` **ya** `NOT EXISTS (…)` |
| 2nd highest | `DENSE_RANK() OVER (ORDER BY x DESC)` → `WHERE dr = 2` |
| Top N per group | `ROW_NUMBER() OVER (PARTITION BY g ORDER BY x DESC)` → `WHERE rn <= N` |
| Running total | `SUM(x) OVER (ORDER BY d ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)` |
| Previous / next row | `LAG(x) OVER (…)` / `LEAD(x) OVER (…)` |
| % of group total | `100.0 * x / SUM(x) OVER (PARTITION BY g)` |
| % of grand total (after GROUP BY) | `100.0 * SUM(x) / SUM(SUM(x)) OVER ()` |
| Conditional count (pivot) | `SUM(CASE WHEN s='A' THEN 1 ELSE 0 END)` / `COUNT(*) FILTER (WHERE s='A')` |
| Null default | `COALESCE(x, 0)` |
| Div-by-zero guard | `x / NULLIF(y, 0)` |
| Diff two result sets | `(A EXCEPT B) UNION ALL (B EXCEPT A)` → 0 rows = identical |
| Upsert | `INSERT … ON CONFLICT (k) DO UPDATE SET c = EXCLUDED.c` |
| Median | `PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY x)` |
| Hierarchy | `WITH RECURSIVE t AS (anchor UNION ALL recursive)` |
| Row lock | `SELECT … FOR UPDATE` (`NOWAIT` / `SKIP LOCKED`) |
| Plan | `EXPLAIN ANALYZE <query>` |

## 42.2 The facts most likely to be asked

| Topic | The answer in one line |
|---|---|
| **Logical execution order** | FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT |
| **WHERE vs HAVING** | WHERE filters rows before grouping; HAVING filters groups after — aggregates only legal in HAVING |
| **Alias in WHERE?** | Nahi — SELECT step 6 par chalta hai, WHERE step 3 par. ORDER BY mein chalta hai |
| **COUNT(\*) vs COUNT(col) vs COUNT(DISTINCT col)** | all rows / non-NULL rows / unique non-NULL values. `COUNT(1)` = `COUNT(*)`, faster nahi |
| **SUM on empty set** | `NULL`, **0 nahi**. COUNT deta hai 0. Isliye `COALESCE(SUM(x),0)` |
| **NULL comparison** | Har comparison `UNKNOWN`; WHERE sirf TRUE paas karta hai. Par GROUP BY/DISTINCT/UNION NULLs ko equal maante hain |
| **NOT IN danger** | Subquery mein ek NULL = **zero rows**, silently. `NOT EXISTS` use karo |
| **ON vs WHERE in outer join** | Right-table condition WHERE mein daalna LEFT JOIN ko INNER JOIN bana deta hai |
| **DISTINCT after JOIN** | Fan-out ka smell — join galat hai, DISTINCT usko chhupa raha hai |
| **ROW_NUMBER / RANK / DENSE_RANK** | 1,2,3,4 (no ties) / 1,2,2,**4** (gap) / 1,2,2,**3** (no gap) |
| **Window in WHERE?** | Nahi — CTE mein wrap karke bahar filter karo |
| **Default window frame** | `ORDER BY` ke saath `RANGE` hota hai (ties lump ho jaate hain) — running total ke liye `ROWS` likho |
| **LAST_VALUE trap** | Default frame current row par khatam — `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` chahiye |
| **UNION vs UNION ALL** | UNION duplicates hatata hai (sort/hash cost); UNION ALL sirf jodta hai — disjoint ho toh yahi use karo |
| **Leftmost prefix rule** | Index `(A,B,C)` → `A`, `(A,B)`, `(A,B,C)` par kaam karta hai. `B` akele par **nahi** |
| **Composite index order** | Equality columns pehle, range column last; har query mein aane wala column leftmost |
| **Index kab use NAHI hota** | `LIKE '%x'`, `UPPER(col) = …`, `col + 1 = 5`, `<>`, low selectivity |
| **Covering index** | Saare needed columns index mein → `Index Only Scan`, table chhua hi nahi |
| **Index ka cost** | Har write slow, storage, bloat. Read fast |
| **EXPLAIN — kya dekhna** | `Seq Scan` on big table + big `Rows Removed by Filter`; estimated vs actual rows ka 10x gap = stale stats → `ANALYZE` |
| **ACID** | Atomicity (all-or-nothing), Consistency (constraints hold), Isolation (concurrent txns don't interfere), Durability (commit = permanent via WAL+fsync) |
| **ACID-C ≠ CAP-C** | ACID-C = constraints; CAP-C = all nodes see same data |
| **Dirty read** | Uncommitted data padh liya → only READ UNCOMMITTED |
| **Non-repeatable read** | Same **row** dobara padhi, value badli → READ COMMITTED tak possible |
| **Phantom read** | Same **query** dobara, nayi **rows** aayi → SERIALIZABLE rokta hai |
| **Lost update** | Dono ne read kiya, doosra write pehle wale ko overwrite — fix: atomic update / `FOR UPDATE` / version column → 409 |
| **Defaults** | Postgres/Oracle/SQL Server: READ COMMITTED. MySQL InnoDB: REPEATABLE READ |
| **SERIALIZABLE cost (PG)** | Block nahi karta — abort karta hai (`40001`). **Application ko retry chahiye** |
| **MVCC** | Har update nayi row version banata hai; readers writers ko block nahi karte. Cost: VACUUM, bloat |
| **Deadlock** | Wait ka cycle; DB victim chun kar abort karta hai. Fix: **consistent lock order** (IDs sort karke lock lo) + retry |
| **1NF/2NF/3NF** | Atomic values / no partial dependency on composite key / no transitive dependency. *"the key, the whole key, and nothing but the key"* |
| **BCNF** | Har determinant candidate key ho — 3NF se strict, sirf overlapping candidate keys par farak |
| **Denormalize kab** | Measured slow read path, historical snapshot (invoice par us waqt ka address), tenant scoping — **hamesha drift check ke saath** |
| **View vs Materialized view** | Stored query (always fresh, no storage, no index) vs stored result (fast, indexable, **stale till refresh**) |
| **Trigger — QA ka concern** | Invisible side effect: test cleanup fail, extra rows, slow bulk updates, double writes. `information_schema.triggers` se dekho |
| **DELETE vs TRUNCATE** | DML with WHERE, fires triggers, rollbackable / DDL, no WHERE, no triggers, MySQL-Oracle mein no rollback |
| **Surrogate vs natural key** | Surrogate PK use karo **+ natural key par UNIQUE constraint** — dono ke fayde |
| **FK par index** | PK par automatic, **FK par nahi** — manually banao, warna parent delete full-scan karega |

## 42.3 SQL → MongoDB

| SQL | MongoDB |
|---|---|
| table / row / column / PK | collection / document / field / `_id` |
| `WHERE a=1 AND b>2` | `{a: 1, b: {$gt: 2}}` |
| `IN` / `NOT IN` | `{$in: […]}` / `{$nin: […]}` |
| `LIKE 'Sh%'` | `{$regex: "^Sh"}` (anchored = index; un-anchored = scan) |
| `IS NULL` | `{f: null}` ⚠️ missing bhi match; exact ke liye `{f: {$exists: false}}` |
| `ORDER BY a, b DESC` | `.sort({a: 1, b: -1})` |
| `LIMIT/OFFSET` | `.limit(n).skip(m)` |
| `COUNT(*)` | `countDocuments(filter)` (`estimatedDocumentCount` filter nahi leta) |
| `GROUP BY` | `{$group: {_id: "$f", n: {$sum: 1}}}` |
| `HAVING` | `$group` ke **baad** ek aur `$match` |
| `LEFT JOIN` | `$lookup` + `$unwind` **with** `preserveNullAndEmptyArrays: true` |
| `INNER JOIN` | `$lookup` + `$unwind` bina us flag ke |
| `UNION ALL` | `$unionWith` |
| `CASE WHEN` | `$cond` / `$switch` |
| `COALESCE` | `$ifNull` |
| Window functions | `$setWindowFields` (5.0+) |
| `UPDATE SET n=n+1` | `{$inc: {n: 1}}` — **atomic, race-free** |
| `INSERT … ON CONFLICT` | `updateOne(f, u, {upsert: true})` + `$setOnInsert` |
| Foreign key | **exist hi nahi karta** — nightly orphan check hi protection hai |
| Trigger | change stream / Atlas Trigger |

**Mongo-specific yaad rakhne layak:**

| Point | Detail |
|---|---|
| **`$match` FIRST** | Sirf pehla stage index use kar sakta hai; `$unwind` document count ko array length se multiply karta hai (100 MB stage limit) |
| **`$elemMatch`** | Bina iske, ek array par do conditions **alag-alag elements** se satisfy ho sakti hain |
| **Missing vs null** | Do alag states; `{f: null}` dono match karta hai; `{f: {$ne: x}}` missing fields bhi deta hai |
| **Single doc write = atomic** | Transaction ke bina bhi, chahe 20 fields + array push ho |
| **Transactions** | 4.0+, par **replica set zaroori** — standalone Docker Mongo par chalengi hi nahi |
| **Write concern** | `w: 1` default — failover par acknowledged write kho sakti hai. Payments ke liye `w: "majority", j: true` |
| **Stale read** | `secondaryPreferred` + replication lag = "maine save kiya par dikha nahi raha" |
| **Partial unique index** | `{unique: true, partialFilterExpression: {isDeleted: false, status: "SOLD"}}` → business rule, history preserved |
| **16 MB doc limit** | Unbounded embedded array = future outage. Volume test karo |
| **`matchedCount` vs `modifiedCount`** | `matched: 0` = filter galat (aksar cross-tenant). `matched:1, modified:0` = value already same, bug nahi |
| **Compound index order** | SQL jaisa hi leftmost prefix. Merlin: `{orgId: 1, isDeleted: 1, status: 1}` |
| **`$indexStats`** | `accesses.ops = 0` → unused index, pure write overhead |

## 42.4 QA-practical checks — the one-page version

**"UI ne success bola" — 8 layers:**

| # | Layer | Assert |
|---|---|---|
| 1 | Payment row | exists; `status` = terminal SUCCESS (INITIATED nahi); amount exact; currency explicit; decimal type; gateway txn id non-null |
| 2 | Parent state | invoice `PAID`/`PARTIALLY_PAID`; outstanding = 0; order status advanced; **negative outstanding nahi** |
| 3 | Ledger | journal entry exists; **debits = credits**; correct accounts; open period |
| 4 | No duplicate | idempotency key exactly once; gateway txn id once; paid ≤ invoice; same call dobara = same payment id |
| 5 | Audit | row exists; correct actor; old→new correct; **exactly one** row |
| 6 | Integrity | invoice exists & not soft-deleted; payment+invoice+order **same tenant** |
| 7 | Time | order ≤ invoice ≤ payment ≤ audit ≤ now(); koi future date nahi; TZ-aware column |
| 8 | External | webhook aaya; webhook status = stored status; duplicate webhook = no second payment |

**"200 aaya par UI galat" — bisect order:**

```
1. Define "wrong" + consistent ya intermittent?
2. DB row           → galat? bug WRITE path mein hai, read mein nahi
3. curl the API     → JSON galat? backend. JSON sahi? frontend.   ← problem aadha
4. Cache            → clear + retry. Sahi aaya? invalidation bug
5. Replica lag      → update ke turant baad galat, refresh par sahi = signature
6. Timezone         → off by 5:30 ya poora ek din = IST/UTC boundary
7. DTO mapping      → field rename, enum case, nested flatten, null→0
8. Rounding         → float vs decimal; ROUND(SUM) vs SUM(ROUND); paise vs rupees
9. Frontend         → stale state, optimistic update, locale format, wrong field bind
10. RCA + PREVENTION test
```

**Nightly job — 4 tiers:**

| Tier | Checks | Severity routing |
|---|---|---|
| **Integrity** | orphans; live children under deleted parent; duplicate business keys; negative/future values; null required fields | CRITICAL → page |
| **Business invariants** | order total = Σ items; invoice status = payments; paid ≤ invoiced; debits = credits; timestamp order; illegal state transitions; **orgId = org.$id** | CRITICAL/HIGH |
| **Health** | table+index growth; bloat; unused indexes; slowest queries diff; replication lag; long/idle-in-transaction; **deadlock count in logs** | MEDIUM |
| **Freshness** | materialized view refresh age; ETL landed; scheduled job failures | MEDIUM → Slack |

**Golden rules:**
- Checks ko **data** banao (YAML), code nahi — failure message mein rule + severity + suggested fix
- Read-only replica par, `statement_timeout` ke saath
- **Har check ka ek seeded negative test** ho — *"a check that has never failed is a check you don't know works"*

## 42.5 Python test snippets — memory jog

```python
# Read-only DB fixture
conn.set_session(readonly=True, autocommit=True)
options = "-c statement_timeout=30000"
cursor_factory = RealDictCursor            # row["col"], not row[3]

# Prod guard
assert not any(f in host for f in ("prod","production","live"))

# Concurrency — barrier is mandatory
barrier = threading.Barrier(2)
def op(): barrier.wait(); return api.post(...)
codes = sorted(f.result().status_code for f in [ex.submit(op), ex.submit(op)])
assert codes == [201, 409]                 # ek jeeta, ek clean conflict
assert 500 not in codes                    # duplicate-key leak nahi hua
assert db_count == 1                       # DB is the source of truth

# Idempotency
assert db.scalar("SELECT COUNT(*) FROM payments WHERE idempotency_key=%s", (k,)) == 1

# Pagination
assert len(seen) == len(set(seen))         # no duplicates across pages
assert len(seen) == expected_total         # no skipped rows

# Mongo plan assertion — catches silent index regressions
assert "COLLSCAN" not in str(plan["queryPlanner"]["winningPlan"])
assert stats["totalDocsExamined"] <= max(stats["nReturned"] * 3, 100)

# Audit — "exactly one", never "at least one"
assert after - before == 1
```

## 42.6 Aakhri 5 line — interview se pehle padhna

1. **Honest raho:** *"Our primary datastore is MongoDB, so my pipelines and index work are strong; I've learned relational SQL because the concepts carry over."*
2. **Har concept ke saath Merlin ka example do** — `{orgId, isDeleted, status}` index, unique partial index → 409, `orgId` vs DBRef drift, `findByOrgAndEmailAddress` wala duplicate bug.
3. **Har technical choice ke saath uska cost bolo.** Index = fast read, slow write. Denormalize = fast read, drift risk.
4. **Query likhne se pehle clarify karo.** *"With ties, do you want the second distinct value or the second row?"*
5. **Har jawab ko prevention par khatam karo.** *"…and the test I'd add so it can't come back is…"*

---

> **Bas. Ab practice karo — padho mat.**
> Docker mein Postgres uthao, Section 0 ka data daalo, aur 30 questions apne haath se likho. Jo query tumne khud chalayi hai, wahi interview mein confidently nikalti hai.
>
> Aur Mongo section ke liye: apne Merlin ke staging cluster par ek `explain` chala kar dekho ki `{orgId, isDeleted, status}` index sach mein hit ho raha hai ya nahi. Us ek observation se tumhare paas ek asli story ban jayegi — aur asli story hi interview jeet-ti hai.

---
