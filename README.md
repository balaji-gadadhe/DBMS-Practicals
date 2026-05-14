# DBMS Revised Practical — Complete Solutions with Explanations
> Last-minute prep guide | Copy-paste ready | With beginner comments & terminal steps
> 
> Covers: MySQL (DDL/DML/Joins) | Oracle PL/SQL (Procedures/Functions/Triggers/Unnamed Block) | MongoDB (CRUD/Aggregation)

---

# ⚡ STARTUP COMMANDS (Ubuntu Terminal)

## MySQL — How to start

```bash
# Start MySQL service
sudo service mysql start

# Login to MySQL (default root, no password in lab)
mysql -u root -p
# Press Enter if no password set

# If a password was set (common in lab)
mysql -u root -p'root'

# Once inside MySQL, create and select a database
CREATE DATABASE dbms_lab;
USE dbms_lab;

# To exit MySQL
exit;
```

---

## Oracle SQL*Plus (PL/SQL) — How to start

```bash
# Start Oracle listener (if not running)
sudo service oracle-xe start

# Connect to Oracle (lab default credentials)
sqlplus system/manager

# OR connect as sysdba if above fails
sqlplus / as sysdba

# Once inside SQL*Plus, enable output (ALWAYS run this first)
SET SERVEROUTPUT ON;

# To exit SQL*Plus
EXIT;
```

> 💡 **Tip:** Always type `SET SERVEROUTPUT ON;` before running any PL/SQL block — otherwise DBMS_OUTPUT.PUT_LINE won't show output.

---

## MongoDB — How to start

```bash
# Start MongoDB service
sudo service mongod start

# Launch MongoDB shell
mongosh
# OR older systems use:
mongo

# Once inside shell, create/switch to a database
use dbms_lab

# To see all collections
show collections

# To exit MongoDB shell
exit
```

> 💡 **Tip:** In MongoDB you don't need to explicitly "create" a database — just `use dbms_lab` and the moment you insert data, it gets created automatically.

---
---

# PROBLEM STATEMENT 1 — DDL Using MySQL

> **What is DDL?** DDL = Data Definition Language. Commands that *define structure* — CREATE, ALTER, DROP, INDEX, VIEW. Not for inserting data, just building the skeleton.

## Step 1 — Create the Database and Tables

```sql
-- Always start by selecting your database
USE dbms_lab;

-- Customer table — stores customer info
CREATE TABLE Customer (
    CustID       INT PRIMARY KEY,         -- Unique ID for each customer
    Name         VARCHAR(50),
    Cust_Address VARCHAR(100),
    Phone_no     VARCHAR(15),
    Email_ID     VARCHAR(50),
    Age          INT
);

-- Branch table — stores branch info
CREATE TABLE Branch (
    Branch_ID   INT PRIMARY KEY,
    Branch_Name VARCHAR(50),
    Address     VARCHAR(100)
);

-- Account table — links Customer and Branch using Foreign Keys
-- FOREIGN KEY = referential integrity (ensures no orphan records)
CREATE TABLE Account (
    Account_no   INT PRIMARY KEY,
    Branch_ID    INT,
    CustID       INT,
    date_open    DATE,
    Account_type VARCHAR(30),
    Balance      DECIMAL(12,2),
    FOREIGN KEY (Branch_ID) REFERENCES Branch(Branch_ID),   -- Branch must exist
    FOREIGN KEY (CustID)    REFERENCES Customer(CustID)     -- Customer must exist
);
```

## Step 2 — Insert Sample Data

```sql
INSERT INTO Customer VALUES (101, 'Amit',  'Pune',   '9876543210', 'amit@mail.com',  28);
INSERT INTO Customer VALUES (102, 'Priya', 'Mumbai', '9123456780', 'priya@mail.com', 35);
INSERT INTO Customer VALUES (103, 'Raj',   'Pune',   '9012345678', 'raj@mail.com',   44);
INSERT INTO Customer VALUES (104, 'Neha',  'Delhi',  '9988776655', 'neha@mail.com',  50);

INSERT INTO Branch VALUES (1, 'Pune Main', 'FC Road Pune');
INSERT INTO Branch VALUES (2, 'Mumbai',    'Andheri Mumbai');

INSERT INTO Account VALUES (1001, 1, 101, '2017-04-16', 'Saving Account',  75000);
INSERT INTO Account VALUES (1002, 2, 102, '2019-06-20', 'Current Account', 120000);
INSERT INTO Account VALUES (1003, 1, 103, '2021-03-10', 'Saving Account',  45000);
INSERT INTO Account VALUES (1004, 2, 104, '2022-08-05', 'Saving Account',  30000);
```

## ER Diagram (Text Representation)

```
Customer(CustID PK, Name, Cust_Address, Phone_no, Email_ID, Age)
    |
    | 1:M (one customer, many accounts)
    |
Account(Account_no PK, Branch_ID FK, CustID FK, date_open, Account_type, Balance)
    |
    | M:1 (many accounts, one branch)
    |
Branch(Branch_ID PK, Branch_Name, Address)
```

## Query 3 — Index on primary key of Account

```sql
-- INDEX speeds up SELECT queries on that column
-- Primary keys already have an index, but this makes it explicit
CREATE INDEX idx_account_pk ON Account(Account_no);
```

## Query 4 — View: Customer_Info for age < 45

```sql
-- VIEW = a saved SELECT query. Acts like a virtual table.
-- Use it to restrict data shown to users.
CREATE VIEW Customer_Info AS
SELECT * FROM Customer WHERE Age < 45;

-- Test it:
SELECT * FROM Customer_Info;
```

## Query 5 — Update View with open_date = 16/4/2017

```sql
-- Views on single tables are updatable in MySQL
-- We update the underlying Account table
UPDATE Account SET date_open = '2017-04-16'
WHERE CustID IN (SELECT CustID FROM Customer_Info);
```

## Query 6 — Sequence on Branch table

```sql
-- MySQL doesn't have SEQUENCE like Oracle.
-- Equivalent: AUTO_INCREMENT on the primary key
ALTER TABLE Branch MODIFY Branch_ID INT AUTO_INCREMENT;

-- Test: Insert without specifying ID — MySQL auto-assigns it
INSERT INTO Branch (Branch_Name, Address) VALUES ('Nagpur', 'Civil Lines Nagpur');
```

## Query 7 — Synonym 'Branch_info' for Branch table

```sql
-- MySQL doesn't support synonyms. Workaround: create a VIEW with that name
CREATE VIEW Branch_info AS SELECT * FROM Branch;

-- Now Branch_info behaves like Branch
SELECT * FROM Branch_info;
```

---

# PROBLEM STATEMENT 2 — DDL Using MySQL

> Same schema as PS1 but with `open_date` instead of `date_open` and different view requirements.

## Step 1 — Create Tables (with referential integrity)

```sql
USE dbms_lab;

CREATE TABLE Customer (
    CustID       INT PRIMARY KEY,
    Name         VARCHAR(50),
    Cust_Address VARCHAR(100),
    Phone_no     VARCHAR(15),
    Email_ID     VARCHAR(50),
    Age          INT
);

CREATE TABLE Branch (
    Branch_ID   INT PRIMARY KEY,
    Branch_Name VARCHAR(50),
    Address     VARCHAR(100)
);

CREATE TABLE Account (
    Account_no   INT PRIMARY KEY,
    Branch_ID    INT,
    CustID       INT,
    open_date    DATE,             -- Note: "open_date" here, not "date_open"
    Account_type VARCHAR(30),
    Balance      DECIMAL(12,2),
    FOREIGN KEY (Branch_ID) REFERENCES Branch(Branch_ID),
    FOREIGN KEY (CustID)    REFERENCES Customer(CustID)
);
```

## Step 2 — Insert Sample Data

```sql
INSERT INTO Customer VALUES (101, 'Amit',  'Pune',   '9876543210', 'amit@mail.com',  28);
INSERT INTO Customer VALUES (102, 'Priya', 'Mumbai', '9123456780', 'priya@mail.com', 35);
INSERT INTO Customer VALUES (103, 'Raj',   'Pune',   '9012345678', 'raj@mail.com',   44);

INSERT INTO Branch VALUES (1, 'Pune Main', 'FC Road Pune');
INSERT INTO Branch VALUES (2, 'Mumbai',    'Andheri Mumbai');

INSERT INTO Account VALUES (1001, 1, 101, '2018-08-16', 'Saving Account',  80000);
INSERT INTO Account VALUES (1002, 2, 102, '2018-02-16', 'Loan Account',    150000);
INSERT INTO Account VALUES (1003, 1, 103, '2018-08-16', 'Saving Account',  50000);
```

## Query 3 — View: Saving Account with open_date 16/8/2018

```sql
CREATE VIEW Saving_account AS
SELECT c.*
FROM Customer c
JOIN Account a ON c.CustID = a.CustID
WHERE a.Account_type = 'Saving Account'
  AND a.open_date = '2018-08-16';

SELECT * FROM Saving_account;
```

## Query 4 — Update View: Set Cust_Address = Pune for CustID = 103

```sql
-- Update the base Customer table directly (view on JOIN isn't directly updatable)
UPDATE Customer SET Cust_Address = 'Pune' WHERE CustID = 103;
```

## Query 5 — View: Loan Account with open_date 16/2/2018

```sql
CREATE VIEW Loan_account AS
SELECT c.*
FROM Customer c
JOIN Account a ON c.CustID = a.CustID
WHERE a.Account_type = 'Loan Account'
  AND a.open_date = '2018-02-16';

SELECT * FROM Loan_account;
```

## Query 6 — Index on primary key of Customer

```sql
CREATE INDEX idx_customer_pk ON Customer(CustID);
```

## Query 7 — Index on primary key of Branch

```sql
CREATE INDEX idx_branch_pk ON Branch(Branch_ID);
```

## Query 8 — Sequence on Customer table

```sql
ALTER TABLE Customer MODIFY CustID INT AUTO_INCREMENT;
```

## Query 9 — Synonym 'Cust_info' for Branch table

```sql
CREATE VIEW Cust_info AS SELECT * FROM Branch;
SELECT * FROM Cust_info;
```

---

# PROBLEM STATEMENT 3 — DML Using MySQL

> **What is DML?** DML = Data Manipulation Language. Commands that *work with data* — INSERT, UPDATE, DELETE, SELECT, ALTER (column changes).

## Step 1 — Create Tables

```sql
-- Schema has NO Email column initially (we'll add it in Query 1)
CREATE TABLE Customer (
    CustID       INT PRIMARY KEY,
    Name         VARCHAR(50),
    Cust_Address VARCHAR(100),
    Phone_no     VARCHAR(15),
    Age          INT
);

CREATE TABLE Branch (
    Branch_ID   INT PRIMARY KEY,
    Branch_Name VARCHAR(50),
    Address     VARCHAR(100)
);

CREATE TABLE Account (
    Account_no   INT PRIMARY KEY,
    Branch_ID    INT,
    CustID       INT,
    date_open    DATE,
    Account_type VARCHAR(30),
    Balance      DECIMAL(12,2),
    FOREIGN KEY (Branch_ID) REFERENCES Branch(Branch_ID),
    FOREIGN KEY (CustID)    REFERENCES Customer(CustID)
);

INSERT INTO Customer VALUES (101, 'Amit',  'Pune',   '9876543210', 28);
INSERT INTO Customer VALUES (102, 'Alok',  'Mumbai', '9123456780', 35);
INSERT INTO Customer VALUES (103, 'Priya', 'Pune',   '9012345678', 42);
INSERT INTO Customer VALUES (104, 'Anil',  'Delhi',  '9988776655', 50);

INSERT INTO Branch VALUES (1, 'Pune Main', 'FC Road');
INSERT INTO Branch VALUES (2, 'Mumbai',    'Andheri');

INSERT INTO Account VALUES (1001, 1, 101, '2020-01-15', 'Saving Account',  75000);
INSERT INTO Account VALUES (1002, 2, 102, '2019-06-20', 'Current Account', 120000);
INSERT INTO Account VALUES (1003, 1, 103, '2021-03-10', 'Saving Account',  45000);
INSERT INTO Account VALUES (1004, 2, 104, '2022-08-05', 'Saving Account',  15000);
```

## Query 1 — Add column Email_Address

```sql
-- ALTER TABLE ADD = adds a new column to existing table
ALTER TABLE Customer ADD Email_Address VARCHAR(50);
```

## Query 2 — Rename Email_Address to Email_ID

```sql
-- RENAME COLUMN (MySQL 8.0+)
ALTER TABLE Customer RENAME COLUMN Email_Address TO Email_ID;
```

## Query 3 — Customer with highest balance

```sql
-- Subquery finds the max balance, outer query finds the customer
SELECT c.*, a.Balance
FROM Customer c
JOIN Account a ON c.CustID = a.CustID
WHERE a.Balance = (SELECT MAX(Balance) FROM Account);
```

## Query 4 — Customer with lowest balance for Saving Account

```sql
SELECT c.*, a.Balance
FROM Customer c
JOIN Account a ON c.CustID = a.CustID
WHERE a.Account_type = 'Saving Account'
  AND a.Balance = (
      SELECT MIN(Balance) FROM Account WHERE Account_type = 'Saving Account'
  );
```

## Query 5 — Customers from Pune AND age > 35

```sql
-- AND means BOTH conditions must be true
SELECT * FROM Customer
WHERE Cust_Address LIKE '%Pune%' AND Age > 35;
```

## Query 6 — CustID, Name, Age in ascending order

```sql
-- ORDER BY ASC = smallest to largest (default)
SELECT CustID, Name, Age FROM Customer ORDER BY Age ASC;
```

## Query 7 — Name, Branch_ID grouped by Account_type

```sql
-- GROUP BY groups rows with same Account_type together
SELECT c.Name, a.Branch_ID, a.Account_type
FROM Customer c
JOIN Account a ON c.CustID = a.CustID
GROUP BY a.Account_type, c.Name, a.Branch_ID;
```

---

# PROBLEM STATEMENT 4 — DML Using MySQL

> Same schema as PS3 but Email_ID already exists. Queries focus on ALTER + aggregate functions.

## Step 1 — Create Tables

```sql
CREATE TABLE Customer (
    CustID       INT PRIMARY KEY,
    Name         VARCHAR(50),
    Cust_Address VARCHAR(100),
    Phone_no     VARCHAR(15),
    Email_ID     VARCHAR(50),      -- Email already present here
    Age          INT
);

CREATE TABLE Branch (
    Branch_ID   INT PRIMARY KEY,
    Branch_Name VARCHAR(50),
    Address     VARCHAR(100)
);

CREATE TABLE Account (
    Account_no   INT PRIMARY KEY,
    Branch_ID    INT,
    CustID       INT,
    date_open    DATE,
    Account_type VARCHAR(30),
    Balance      DECIMAL(12,2),
    FOREIGN KEY (Branch_ID) REFERENCES Branch(Branch_ID),
    FOREIGN KEY (CustID)    REFERENCES Customer(CustID)
);

INSERT INTO Customer VALUES (101, 'Amit',  'Pune',   '9876543210', 'amit@mail.com',  28);
INSERT INTO Customer VALUES (102, 'Alok',  'Mumbai', '9123456780', 'alok@mail.com',  35);
INSERT INTO Customer VALUES (103, 'Priya', 'Pune',   '9012345678', 'priya@mail.com', 42);
INSERT INTO Customer VALUES (104, 'Anil',  'Delhi',  '9988776655', 'anil@mail.com',  25);

INSERT INTO Branch VALUES (1, 'Pune Main', 'FC Road');
INSERT INTO Branch VALUES (2, 'Mumbai',    'Andheri');

INSERT INTO Account VALUES (1001, 1, 101, '2020-01-15', 'Saving Account',  75000);
INSERT INTO Account VALUES (1002, 2, 102, '2019-06-20', 'Current Account', 120000);
INSERT INTO Account VALUES (1003, 1, 103, '2021-03-10', 'Saving Account',  45000);
INSERT INTO Account VALUES (1004, 2, 104, '2022-08-05', 'Saving Account',  30000);
```

## Query 1 — Modify size of Email_ID to 20

```sql
-- MODIFY changes column definition (type/size/constraints)
ALTER TABLE Customer MODIFY Email_ID VARCHAR(20);
```

## Query 2 — Change Email_ID to NOT NULL

```sql
-- NOT NULL means the column cannot be left empty
ALTER TABLE Customer MODIFY Email_ID VARCHAR(20) NOT NULL;
```

## Query 3 — Total customers with balance > 50,000

```sql
-- COUNT(*) counts the number of matching rows
SELECT COUNT(*) AS Total_Customers
FROM Customer c
JOIN Account a ON c.CustID = a.CustID
WHERE a.Balance > 50000;
```

## Query 4 — Average balance for Saving Account

```sql
-- AVG() calculates the mean of a numeric column
SELECT AVG(Balance) AS Avg_Balance
FROM Account
WHERE Account_type = 'Saving Account';
```

## Query 5 — Customers from Pune OR name starts with 'A'

```sql
-- OR means EITHER condition is enough
-- LIKE 'A%' means starts with A (% = wildcard, any characters after)
SELECT * FROM Customer
WHERE Cust_Address LIKE '%Pune%'
   OR Name LIKE 'A%';
```

## Query 6 — Create Saving_Account table from Account

```sql
-- CREATE TABLE ... AS SELECT = copies structure + filtered data
CREATE TABLE Saving_Account AS
SELECT Account_no, Branch_ID, CustID, date_open, Balance
FROM Account
WHERE Account_type = 'Saving Account';

-- Verify:
SELECT * FROM Saving_Account;
```

## Query 7 — Customer details age-wise with balance >= 20,000

```sql
SELECT c.*, a.Balance
FROM Customer c
JOIN Account a ON c.CustID = a.CustID
WHERE a.Balance >= 20000
ORDER BY c.Age;
```

---

# PROBLEM STATEMENT 5 — JOINs & Subqueries (MySQL)

> **JOIN** = combine rows from two tables based on a related column.  
> **Subquery** = a SELECT inside another SELECT (inner query runs first).

## Step 1 — Create Tables with Referential Integrity

```sql
-- Location first (no dependencies)
CREATE TABLE Locations (
    Location_id    INT PRIMARY KEY,
    Street_address VARCHAR(100),
    Postal_code    VARCHAR(10),
    City           VARCHAR(50),
    State          VARCHAR(50),
    Country_id     VARCHAR(10)     -- 'IN' = India, 'US' = United States
);

-- Departments depends on Locations
CREATE TABLE Departments (
    Department_id   INT PRIMARY KEY,
    Department_name VARCHAR(50),
    Manager_id      INT,
    Location_id     INT,
    FOREIGN KEY (Location_id) REFERENCES Locations(Location_id)
);

-- Manager table
CREATE TABLE Manager (
    Manager_id   INT PRIMARY KEY,
    Manager_name VARCHAR(50)
);

-- Employee depends on Manager and Departments
CREATE TABLE Employee (
    Employee_id   INT PRIMARY KEY,
    First_name    VARCHAR(50),
    Last_name     VARCHAR(50),
    Hire_date     DATE,
    Salary        DECIMAL(10,2),
    Job_title     VARCHAR(50),
    Manager_id    INT,
    Department_id INT,
    FOREIGN KEY (Manager_id)    REFERENCES Manager(Manager_id),
    FOREIGN KEY (Department_id) REFERENCES Departments(Department_id)
);
```

## Step 2 — Insert Sample Data

```sql
INSERT INTO Locations VALUES (1, 'MG Road',   '411001', 'Pune',     'MH',  'IN');
INSERT INTO Locations VALUES (2, '5th Ave',   '10001',  'New York', 'NY',  'US');
INSERT INTO Locations VALUES (3, 'Baker St',  'SW1',    'London',   'ENG', 'UK');

INSERT INTO Departments VALUES (10, 'IT',      1, 1);
INSERT INTO Departments VALUES (20, 'HR',      2, 2);
INSERT INTO Departments VALUES (30, 'Finance', 3, 3);

INSERT INTO Manager VALUES (1, 'Ramesh');
INSERT INTO Manager VALUES (2, 'Suresh');
INSERT INTO Manager VALUES (3, 'Mahesh');

INSERT INTO Employee VALUES (1, 'Amit',  'Sharma', '2005-03-10', 70000, 'Developer',    1, 10);
INSERT INTO Employee VALUES (2, 'Priya', 'Singh',  '2008-06-15', 45000, 'HR Executive', 2, 20);
INSERT INTO Employee VALUES (3, 'Raj',   'Kumar',  '2010-01-20', 80000, 'Senior Dev',   1, 10);
INSERT INTO Employee VALUES (4, 'Neha',  'Gupta',  '2015-07-30', 55000, 'Analyst',      3, 30);
INSERT INTO Employee VALUES (5, 'Vikas', 'Mehta',  '2018-11-05', 40000, 'Intern',       2, 20);
```

## Query 1 — Employees with salary higher than Singh's

```sql
-- Subquery: find Singh's salary first, then compare
SELECT First_name, Last_name, Salary
FROM Employee
WHERE Salary > (
    SELECT Salary FROM Employee WHERE Last_name = 'Singh' LIMIT 1
);
```

## Query 2 — Employees with a manager, working in US department

```sql
-- JOIN three tables: Employee → Departments → Locations
-- Filter: Manager_id IS NOT NULL AND Country = US
SELECT e.First_name, e.Last_name
FROM Employee e
JOIN Departments d ON e.Department_id = d.Department_id
JOIN Locations   l ON d.Location_id   = l.Location_id
WHERE e.Manager_id IS NOT NULL
  AND l.Country_id = 'US';
```

## Query 3 — Employees with salary > average salary

```sql
-- AVG() inside subquery gives average; outer query compares
SELECT First_name, Last_name, Salary
FROM Employee
WHERE Salary > (SELECT AVG(Salary) FROM Employee);
```

---

# PROBLEM STATEMENT 6 — Procedures / Functions (Oracle PL/SQL)

> **Remember:** Launch SQL*Plus first, then run `SET SERVEROUTPUT ON;`

## Schema Setup

```sql
-- Create Employee table
CREATE TABLE Employee (
    emp_id INT PRIMARY KEY,
    dept_id INT,
    emp_name VARCHAR2(50),
    DoJ DATE,
    salary NUMBER,
    commission NUMBER,
    job_title VARCHAR2(50)
);

-- Create Departments table (needed for the function)
CREATE TABLE Departments (
    dept_id INT PRIMARY KEY,
    manager_name VARCHAR2(50)
);

-- Insert dummy data so we have something to calculate
INSERT INTO Employee VALUES (1, 10, 'Amit', TO_DATE('2010-01-01', 'YYYY-MM-DD'), 12000, NULL, 'Manager');
INSERT INTO Employee VALUES (2, 20, 'Rahul', TO_DATE('2020-10-20', 'YYYY-MM-DD'), 2500, NULL, 'Intern');
INSERT INTO Departments VALUES (10, 'Ramesh');
INSERT INTO Departments VALUES (20, 'Suresh');
COMMIT;
```

## Query 1 — Stored Procedure: Calculate Commission

```sql
CREATE OR REPLACE PROCEDURE set_commission IS
BEGIN
    UPDATE Employee
    SET commission = CASE
        WHEN salary > 10000 THEN salary * 0.004
        WHEN salary < 10000 AND (SYSDATE - DoJ) / 365 > 10 THEN salary * 0.0035
        WHEN salary < 3000 THEN salary * 0.0025
        ELSE salary * 0.0015
    END;
    COMMIT;
END;
/

-- Run the procedure:
EXEC calc_commission;

-- Verify:
-- Run the procedure
BEGIN
    set_commission;
END;
/

-- Check the result (You should see numbers in the commission column now)
SELECT * FROM Employee;

```

## Query 2 — Function: Return Manager name for a Department

```sql
-- FUNCTION = like a procedure but RETURNS a value
-- Takes dept_id as input, returns manager name as output
CREATE OR REPLACE FUNCTION get_manager(did NUMBER)
RETURN VARCHAR2 IS
    mname VARCHAR2(30);
BEGIN
    SELECT manager_name INTO mname
    FROM Departments
    WHERE dept_id = did;
    
    RETURN mname;
END;
/

Test the Function (The Output)

```

SET SERVEROUTPUT ON;
DECLARE
    res VARCHAR2(30);
BEGIN
    res := get_manager(10); -- 10 is the Department ID
    DBMS_OUTPUT.PUT_LINE('Manager Name is: ' || res);
END;
/
```
```

---

# PROBLEM STATEMENT 7 — Unnamed Block (Oracle PL/SQL)

> **Unnamed Block** = a PL/SQL block with no name. Runs once, not stored. Structure: `DECLARE → BEGIN → EXCEPTION → END`

## Schema Setup

```sql
-- Create the main table
CREATE TABLE Employee (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR2(50),
    DoJ DATE,
    salary NUMBER
);

-- Create the table where results will be stored
CREATE TABLE Salary_Increment (
    emp_id INT,
    new_salary NUMBER
);

-- Insert a test employee with ID 115
INSERT INTO Employee VALUES (115, 'John Doe', TO_DATE('2012-01-01', 'YYYY-MM-DD'), 5000);
COMMIT;
```

## The Unnamed PL/SQL Block

```sql
SET SERVEROUTPUT ON;

DECLARE
    -- These are your local variables to hold data temporarily
    eid NUMBER;
    yrs NUMBER;
    sal NUMBER;
    newsal NUMBER;

BEGIN
    -- 1. Accept input from user (The & creates the pop-up)
    eid := &emp_id;

    -- 2. Fetch current salary and calculate years of experience
    SELECT salary, (SYSDATE - DoJ) / 365
    INTO sal, yrs
    FROM Employee
    WHERE emp_id = eid;

    -- 3. Logic for Increment
    IF yrs > 10 THEN
        newsal := sal * 1.20; -- 20% increase
    ELSIF yrs > 5 THEN
        newsal := sal * 1.10; -- 10% increase
    ELSE
        newsal := sal * 1.05; -- 5% increase
    END IF;

    -- 4. Store the result in the second table
    INSERT INTO Salary_Increment (emp_id, new_salary)
    VALUES (eid, newsal);

    DBMS_OUTPUT.PUT_LINE('Salary processed for Employee ID: ' || eid);
    COMMIT;

EXCEPTION
    -- 5. Exception Handling (If ID 115 doesn't exist)
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Error: Employee ID ' || eid || ' does not exist.');
    
    -- Catch-all for any other weird errors
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('An unexpected error occurred.');

END;
/
```
Step 3: Verify the Result

```
SELECT * FROM Salary_Increment;
```

---

# PROBLEM STATEMENT 8 — Triggers (Oracle PL/SQL)

> **Trigger** = automatic PL/SQL code that fires BEFORE or AFTER an event (INSERT/UPDATE/DELETE) on a table. You don't call it manually — it fires on its own.

## Schema Setup

```sql
-- Main Table
CREATE TABLE Employee (
    emp_id INT PRIMARY KEY,
    dept_id INT,
    emp_name VARCHAR2(50),
    DoJ DATE,
    salary NUMBER,
    job_title VARCHAR2(50)
);

-- History Table (For Requirement 2)
CREATE TABLE job_history (
    emp_id INT,
    old_job_title VARCHAR2(50),
    old_dept_id INT,
    start_date DATE,
    end_date DATE
);

-- Insert a test employee
INSERT INTO Employee VALUES (101, 10, 'Amit', TO_DATE('2020-01-01', 'YYYY-MM-DD'), 5000, 'Junior Dev');
COMMIT;
```
Step 2: Trigger 1 (The Security Guard)
This ensures salary never goes down. It is a BEFORE trigger because we want to stop the change before it hits the database.

```
CREATE OR REPLACE TRIGGER trg_no_salary_decrease
BEFORE UPDATE OF salary ON Employee
FOR EACH ROW
BEGIN
    -- Check if the new salary is lower than the old one
    IF :NEW.salary < :OLD.salary THEN
        -- -20002 is a custom error code you can choose
        RAISE_APPLICATION_ERROR(-20002, 'Salary cannot decrease!');
    END IF;
END;
/
```
How to test it:
Try to lower the salary: UPDATE Employee SET salary = 4000 WHERE emp_id = 101;
(It should fail with your custom error message.)

-- Test:
UPDATE Employee SET salary = 60000 WHERE emp_id = 1;  -- OK
UPDATE Employee SET salary = 30000 WHERE emp_id = 1;  -- Will fail with error
```

## Trigger 2 — Log job title change to job_history

```sql
-- Fires BEFORE UPDATE on job_title column only
-- :OLD = values before update, :NEW = values after update
CREATE OR REPLACE TRIGGER trg_job_history
AFTER UPDATE OF job_title ON Employee
FOR EACH ROW
BEGIN
    -- Insert the OLD details into history table
    INSERT INTO job_history VALUES (
        :OLD.emp_id,
        :OLD.job_title,
        :OLD.dept_id,
        :OLD.DoJ,
        SYSDATE -- The day the job changed
    );
END;
/
-- Test:
UPDATE Employee SET job_title = 'Senior Analyst' WHERE emp_id = 101;
SELECT * FROM job_history;
```

---

# PROBLEM STATEMENT 9 — CRUD Using MongoDB

> **CRUD** = Create, Read, Update, Delete. Basic operations on any database.
> In MongoDB: `insertMany()` = Create, `find()` = Read, `updateOne()` = Update, `deleteOne()` = Delete

## Step 1 — Start MongoDB and Create Collection

```bash
# In Ubuntu terminal:
sudo service mongod start
mongosh
```

```javascript
// Switch to our database (auto-created on first insert)
use dbms_lab

// MongoDB is schema-less — no need to define structure before inserting
```

## Step 2 — Insert 20 Documents

```javascript
// insertMany() = insert multiple documents at once
// Friends_List and Interests are arrays (square brackets)

db.Social_Media.insertMany([
    { User_Id: 1,  User_Name: "Aryan",   No_of_Posts: 120, No_of_Friends: 8,  Friends_List: ["Riya","Karan","Ansh"],       Interests: ["Gaming","Music"] },
    { User_Id: 2,  User_Name: "Riya",    No_of_Posts: 80,  No_of_Friends: 5,  Friends_List: ["Aryan","Priya"],             Interests: ["Travel","Food"] },
    { User_Id: 3,  User_Name: "Karan",   No_of_Posts: 200, No_of_Friends: 12, Friends_List: ["Aryan","Neha","Sam","Raj"],  Interests: ["Cricket","Movies"] },
    { User_Id: 4,  User_Name: "Priya",   No_of_Posts: 45,  No_of_Friends: 3,  Friends_List: ["Riya","Ansh"],               Interests: ["Art","Books"] },
    { User_Id: 5,  User_Name: "Neha",    No_of_Posts: 300, No_of_Friends: 15, Friends_List: ["Karan","Sam","Ansh","Jay"],  Interests: ["Fashion","Travel"] },
    { User_Id: 6,  User_Name: "Sam",     No_of_Posts: 90,  No_of_Friends: 6,  Friends_List: ["Karan","Neha"],              Interests: ["Tech","Gaming"] },
    { User_Id: 7,  User_Name: "Jay",     No_of_Posts: 150, No_of_Friends: 9,  Friends_List: ["Neha","Raj","Tina"],         Interests: ["Fitness","Music"] },
    { User_Id: 8,  User_Name: "Tina",    No_of_Posts: 60,  No_of_Friends: 4,  Friends_List: ["Jay","Mia"],                 Interests: ["Movies","Art"] },
    { User_Id: 9,  User_Name: "Raj",     No_of_Posts: 110, No_of_Friends: 7,  Friends_List: ["Karan","Jay","Ansh"],        Interests: ["Cricket","Food"] },
    { User_Id: 10, User_Name: "Mia",     No_of_Posts: 25,  No_of_Friends: 2,  Friends_List: ["Tina"],                      Interests: ["Books","Travel"] },
    { User_Id: 11, User_Name: "Ansh",    No_of_Posts: 175, No_of_Friends: 10, Friends_List: ["Aryan","Priya","Raj"],       Interests: ["Gaming","Tech"] },
    { User_Id: 12, User_Name: "Pooja",   No_of_Posts: 130, No_of_Friends: 8,  Friends_List: ["Simran","Asha"],             Interests: ["Fashion","Food"] },
    { User_Id: 13, User_Name: "Simran",  No_of_Posts: 55,  No_of_Friends: 3,  Friends_List: ["Pooja"],                     Interests: ["Music","Art"] },
    { User_Id: 14, User_Name: "Asha",    No_of_Posts: 220, No_of_Friends: 13, Friends_List: ["Pooja","Dev","Rohan"],       Interests: ["Travel","Books"] },
    { User_Id: 15, User_Name: "Dev",     No_of_Posts: 95,  No_of_Friends: 5,  Friends_List: ["Asha","Rohan"],              Interests: ["Tech","Gaming"] },
    { User_Id: 16, User_Name: "Rohan",   No_of_Posts: 140, No_of_Friends: 9,  Friends_List: ["Dev","Asha","Vikram"],       Interests: ["Cricket","Fitness"] },
    { User_Id: 17, User_Name: "Vikram",  No_of_Posts: 35,  No_of_Friends: 2,  Friends_List: ["Rohan"],                     Interests: ["Movies","Music"] },
    { User_Id: 18, User_Name: "Meera",   No_of_Posts: 260, No_of_Friends: 16, Friends_List: ["Sona","Nitu","Ritu","Piya"], Interests: ["Food","Travel"] },
    { User_Id: 19, User_Name: "Sona",    No_of_Posts: 70,  No_of_Friends: 4,  Friends_List: ["Meera","Nitu"],              Interests: ["Art","Fitness"] },
    { User_Id: 20, User_Name: "Nitu",    No_of_Posts: 105, No_of_Friends: 6,  Friends_List: ["Meera","Sona","Ritu"],       Interests: ["Books","Gaming"] }
])
```

## Query 1 — List all users in formatted manner

```javascript
// .pretty() = nicely formatted JSON output (readable)
db.Social_Media.find().pretty()
```

## Query 2 — Users with No_of_Posts > 100

```javascript
// $gt = greater than operator in MongoDB
db.Social_Media.find({ No_of_Posts: { $gt: 100 } }).pretty()
```

## Query 3 — User names and their Friends_List

```javascript
// Second argument in find() = projection (1 = show, 0 = hide)
// _id: 0 hides the auto-generated MongoDB ID
db.Social_Media.find({}, { User_Name: 1, Friends_List: 1, _id: 0 }).pretty()
```

## Query 4 — User IDs and Friends_List where No_of_Friends > 5

```javascript
db.Social_Media.find(
    { No_of_Friends: { $gt: 5 } },          // Filter condition
    { User_Id: 1, Friends_List: 1, _id: 0 } // Show only these fields
).pretty()
```

## Query 5 — All users sorted by No_of_Posts descending

```javascript
// .sort({ field: -1 }) = descending order
// .sort({ field:  1 }) = ascending order
db.Social_Media.find().sort({ No_of_Posts: -1 }).pretty()
```

---

# PROBLEM STATEMENT 10 — Aggregation & Indexing (MongoDB)

> **Aggregation** = processing multiple documents to compute results (like GROUP BY in SQL).  
> The `aggregate()` function takes a **pipeline** — an array of stages, each transforming the data.

## Step 1 — Create Collection and Insert Data

```javascript
use dbms_lab

db.Movies_Data.insertMany([
    { Movie_ID: 1, Movie_Name: "RRR",       Director: "Rajamouli",       Genre: "Action", BoxOfficeCollection: 1200 },
    { Movie_ID: 2, Movie_Name: "KGF",       Director: "Prashant Neel",   Genre: "Action", BoxOfficeCollection: 1000 },
    { Movie_ID: 3, Movie_Name: "Bahubali",  Director: "Rajamouli",       Genre: "Drama",  BoxOfficeCollection: 900  },
    { Movie_ID: 4, Movie_Name: "Dangal",    Director: "Nitesh Tiwari",   Genre: "Drama",  BoxOfficeCollection: 800  },
    { Movie_ID: 5, Movie_Name: "3 Idiots",  Director: "Rajkumar Hirani", Genre: "Comedy", BoxOfficeCollection: 600  },
    { Movie_ID: 6, Movie_Name: "PK",        Director: "Rajkumar Hirani", Genre: "Comedy", BoxOfficeCollection: 700  },
    { Movie_ID: 7, Movie_Name: "Pushpa",    Director: "Sukumar",         Genre: "Action", BoxOfficeCollection: 850  },
    { Movie_ID: 8, Movie_Name: "Dil Dhadakne Do", Director: "Zoya Akhtar", Genre: "Drama", BoxOfficeCollection: 500 }
])
```

## Query 1 — How many movies directed by each Director

```javascript
// $group: groups documents by a field
// _id: "$Director" = group by Director field
// $sum: 1 = count +1 for each document in group

db.Movies_Data.aggregate([
    {
        $group: {
            _id: "$Director",
            TotalMovies: { $sum: 1 }
        }
    }
])
```

## Query 2 — Highest BoxOfficeCollection in each Genre

```javascript
// Stage 1: $sort — sort by BoxOfficeCollection descending
// Stage 2: $group — group by Genre, pick the first (which is highest)

db.Movies_Data.aggregate([
    { $sort: { BoxOfficeCollection: -1 } },    // Sort desc first
    {
        $group: {
            _id: "$Genre",
            Movie_Name:    { $first: "$Movie_Name" },     // First after sort = highest
            MaxCollection: { $max:   "$BoxOfficeCollection" }
        }
    }
])
```

## Query 3 — Highest BoxOfficeCollection per Genre, ascending order

```javascript
// Same as Q2 but add another $sort at the end (ascending)

db.Movies_Data.aggregate([
    { $sort: { BoxOfficeCollection: -1 } },
    {
        $group: {
            _id: "$Genre",
            Movie_Name:    { $first: "$Movie_Name" },
            MaxCollection: { $max:   "$BoxOfficeCollection" }
        }
    },
    { $sort: { MaxCollection: 1 } }    // Final sort: ascending by collection
])
```

## Query 4 — Create index on Movie_ID

```javascript
// createIndex({ field: 1 }) = ascending index
// Index = like a book index — speeds up search
db.Movies_Data.createIndex({ Movie_ID: 1 })
```

## Query 5 — Create compound index on Movie_Name and Director

```javascript
// Compound index = index on two fields together
db.Movies_Data.createIndex({ Movie_Name: 1, Director: 1 })
```

## Query 6 — Drop index on Movie_ID

```javascript
// dropIndex removes the index (not the data)
db.Movies_Data.dropIndex({ Movie_ID: 1 })
```

## Query 7 — Drop compound index on Movie_Name and Director

```javascript
db.Movies_Data.dropIndex({ Movie_Name: 1, Director: 1 })
```

---

# 🔥 Quick Reference Cheat Sheet

## MySQL — Commonly Used Commands
| Task | Command |
|------|---------|
| Create DB | `CREATE DATABASE name;` |
| Use DB | `USE name;` |
| See tables | `SHOW TABLES;` |
| Describe table | `DESC tablename;` |
| Show all rows | `SELECT * FROM table;` |

## Oracle SQL*Plus — Key Reminders
| Task | Command |
|------|---------|
| Enable output | `SET SERVEROUTPUT ON;` |
| Run a file | `@filename.sql` |
| Commit changes | `COMMIT;` |
| See tables | `SELECT * FROM TAB;` |
| Current date | `SELECT SYSDATE FROM DUAL;` |

## MongoDB Shell — Quick Commands
| Task | Command |
|------|---------|
| List all DBs | `show dbs` |
| Switch DB | `use dbname` |
| List collections | `show collections` |
| Count documents | `db.collection.countDocuments()` |
| Delete all docs | `db.collection.deleteMany({})` |
| Drop collection | `db.collection.drop()` |

---

*Solutions prepared for SPPU B.Tech DBMS Practical Exam — I²IT, Pune.*  
*Oracle PL/SQL for Procedures/Functions/Triggers/Blocks | MySQL for DDL/DML/Joins | MongoDB for CRUD/Aggregation*
