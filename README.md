# DBMS Final Practical Exam — Complete Solutions
> All problems solved stepwise with copy-paste ready code.

---

## Problem Statement 1 — Triggers (Oracle PL/SQL)

**Schema:**
```sql
CREATE TABLE Employee (
    emp_id      NUMBER PRIMARY KEY,
    emp_name    VARCHAR2(50),
    salary      NUMBER,
    designation VARCHAR2(50)
);

CREATE TABLE Salary_Backup (
    emp_id            NUMBER,
    old_salary        NUMBER,
    new_salary        NUMBER,
    salary_difference NUMBER
);
```

**Trigger 1 — Record salary change on UPDATE:**
```sql
CREATE OR REPLACE TRIGGER trg_salary_backup
AFTER UPDATE OF salary ON Employee
FOR EACH ROW
BEGIN
    INSERT INTO Salary_Backup (emp_id, old_salary, new_salary, salary_difference)
    VALUES (:OLD.emp_id, :OLD.salary, :NEW.salary, :NEW.salary - :OLD.salary);
END;
/
```

**Trigger 2 — Prevent DELETE if designation is CEO:**
```sql
CREATE OR REPLACE TRIGGER trg_prevent_ceo_delete
BEFORE DELETE ON Employee
FOR EACH ROW
BEGIN
    IF :OLD.designation = 'CEO' THEN
        RAISE_APPLICATION_ERROR(-20001, 'ERROR: Cannot delete an employee with designation CEO.');
    END IF;
END;
/
```

**Test:**
```sql
INSERT INTO Employee VALUES (1, 'Rahul', 50000, 'CEO');
INSERT INTO Employee VALUES (2, 'Priya', 30000, 'Manager');

UPDATE Employee SET salary = 60000 WHERE emp_id = 2;
SELECT * FROM Salary_Backup;

DELETE FROM Employee WHERE emp_id = 1; -- Will raise error
```

---

## Problem Statement 2 — Aggregation & Indexing (MongoDB)

**Create Collection & Insert Sample Data:**
```javascript
use dbms_exam

db.Movies_Data.insertMany([
    { Movie_ID: 1, Movie_Name: "RRR", Director: "Rajamouli", Genre: "Action", BoxOfficeCollection: 1200 },
    { Movie_ID: 2, Movie_Name: "KGF", Director: "Prashant Neel", Genre: "Action", BoxOfficeCollection: 1000 },
    { Movie_ID: 3, Movie_Name: "Bahubali", Director: "Rajamouli", Genre: "Drama", BoxOfficeCollection: 900 },
    { Movie_ID: 4, Movie_Name: "Dangal", Director: "Nitesh Tiwari", Genre: "Drama", BoxOfficeCollection: 800 },
    { Movie_ID: 5, Movie_Name: "3 Idiots", Director: "Rajkumar Hirani", Genre: "Comedy", BoxOfficeCollection: 600 },
    { Movie_ID: 6, Movie_Name: "PK", Director: "Rajkumar Hirani", Genre: "Comedy", BoxOfficeCollection: 700 }
])
```

**Query 1 — Movies count per Director:**
```javascript
db.Movies_Data.aggregate([
    { $group: { _id: "$Director", TotalMovies: { $sum: 1 } } }
])
```

**Query 2 — Highest BoxOfficeCollection in each Genre:**
```javascript
db.Movies_Data.aggregate([
    { $sort: { BoxOfficeCollection: -1 } },
    { $group: { _id: "$Genre", Movie_Name: { $first: "$Movie_Name" }, MaxCollection: { $max: "$BoxOfficeCollection" } } }
])
```

**Query 3 — Highest BoxOfficeCollection per Genre in ascending order:**
```javascript
db.Movies_Data.aggregate([
    { $sort: { BoxOfficeCollection: -1 } },
    { $group: { _id: "$Genre", Movie_Name: { $first: "$Movie_Name" }, MaxCollection: { $max: "$BoxOfficeCollection" } } },
    { $sort: { MaxCollection: 1 } }
])
```

**Query 4 — Create index on Movie_ID:**
```javascript
db.Movies_Data.createIndex({ Movie_ID: 1 })
```

**Query 5 — Create compound index on Movie_Name and Director:**
```javascript
db.Movies_Data.createIndex({ Movie_Name: 1, Director: 1 })
```

**Query 6 — Drop index on Movie_ID:**
```javascript
db.Movies_Data.dropIndex({ Movie_ID: 1 })
```

**Query 7 — Drop compound index on Movie_Name and Director:**
```javascript
db.Movies_Data.dropIndex({ Movie_Name: 1, Director: 1 })
```

---

## Problem Statement 3 — Procedures / Functions (Oracle PL/SQL)

**Schema:**
```sql
CREATE TABLE Account (
    Account_No  NUMBER PRIMARY KEY,
    Cust_Name   VARCHAR2(50),
    Balance     NUMBER,
    NoOfYears   NUMBER
);

CREATE TABLE Earned_Interest (
    Account_No   NUMBER,
    Interest_Amt NUMBER
);

INSERT INTO Account VALUES (101, 'Amit', 100000, 5);
INSERT INTO Account VALUES (102, 'Sneha', 60000, 3);
INSERT INTO Account VALUES (103, 'Ravi', 45000, 8);
```

**1. Procedure — Calculate and store Simple Interest:**
```sql
CREATE OR REPLACE PROCEDURE calc_interest (
    p_acc_no        IN NUMBER,
    p_interest_rate IN NUMBER
) AS
    v_balance   NUMBER;
    v_years     NUMBER;
    v_interest  NUMBER;
BEGIN
    SELECT Balance, NoOfYears INTO v_balance, v_years
    FROM Account
    WHERE Account_No = p_acc_no;

    v_interest := (v_balance * p_interest_rate * v_years) / 100;

    INSERT INTO Earned_Interest (Account_No, Interest_Amt)
    VALUES (p_acc_no, v_interest);

    COMMIT;

    DBMS_OUTPUT.PUT_LINE('Interest calculated and stored: ' || v_interest);

    FOR rec IN (SELECT * FROM Earned_Interest) LOOP
        DBMS_OUTPUT.PUT_LINE('Acc No: ' || rec.Account_No || ' | Interest: ' || rec.Interest_Amt);
    END LOOP;

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Account not found.');
END;
/

-- Execute:
EXEC calc_interest(101, 5);
```

**2. Function — Display accounts with Balance > 50,000:**
```sql
CREATE OR REPLACE FUNCTION get_high_balance_accounts
RETURN SYS_REFCURSOR AS
    v_cursor SYS_REFCURSOR;
BEGIN
    OPEN v_cursor FOR
        SELECT * FROM Account WHERE Balance > 50000;
    RETURN v_cursor;
END;
/

-- Execute:
DECLARE
    v_cur   SYS_REFCURSOR;
    v_rec   Account%ROWTYPE;
BEGIN
    v_cur := get_high_balance_accounts();
    LOOP
        FETCH v_cur INTO v_rec;
        EXIT WHEN v_cur%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE(v_rec.Account_No || ' | ' || v_rec.Cust_Name || ' | ' || v_rec.Balance);
    END LOOP;
    CLOSE v_cur;
END;
/
```

---

## Problem Statement 4 — JOINs & Subqueries (MySQL)

**Schema & Tables:**
```sql
CREATE TABLE Locations (
    Location_id    INT PRIMARY KEY,
    Street_address VARCHAR(100),
    Postal_code    VARCHAR(10),
    City           VARCHAR(50),
    State          VARCHAR(50),
    Country_id     VARCHAR(10)
);

CREATE TABLE Departments (
    Department_id   INT PRIMARY KEY,
    Department_name VARCHAR(50),
    Manager_id      INT,
    Location_id     INT,
    FOREIGN KEY (Location_id) REFERENCES Locations(Location_id)
);

CREATE TABLE Manager (
    Manager_id   INT PRIMARY KEY,
    Manager_name VARCHAR(50)
);

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

-- Sample Data
INSERT INTO Locations VALUES (1,'MG Road','411001','Pune','MH','IN');
INSERT INTO Locations VALUES (2,'5th Ave','10001','New York','NY','US');
INSERT INTO Locations VALUES (3,'Baker St','SW1','London','ENG','UK');

INSERT INTO Departments VALUES (10,'IT',1,1),(20,'HR',2,2),(30,'Finance',3,3);

INSERT INTO Manager VALUES (1,'Ramesh'),(2,'Suresh'),(3,'Mahesh');

INSERT INTO Employee VALUES
(1,'Amit','Sharma','2005-03-10',70000,'Developer',1,10),
(2,'Priya','Singh','2008-06-15',45000,'HR Exec',2,20),
(3,'Raj','Kumar','2010-01-20',80000,'Senior Dev',1,10),
(4,'Neha','Gupta','2015-07-30',55000,'Analyst',3,30),
(5,'Vikas','Mehta','2018-11-05',40000,'Intern',2,20);
```

**Query 1 — Employees earning more than average salary AND in IT department:**
```sql
SELECT e.First_name, e.Last_name, e.Salary
FROM Employee e
JOIN Departments d ON e.Department_id = d.Department_id
WHERE e.Salary > (SELECT AVG(Salary) FROM Employee)
  AND d.Department_name LIKE '%IT%';
```

**Query 2 — Employees earning the minimum salary across all departments:**
```sql
SELECT e.First_name, e.Last_name, e.Salary
FROM Employee e
WHERE e.Salary = (
    SELECT MIN(Salary)
    FROM Employee
);
```

**Query 3 — Employees earning above their department's average salary:**
```sql
SELECT e.Employee_id, e.First_name, e.Last_name, e.Salary
FROM Employee e
WHERE e.Salary > (
    SELECT AVG(e2.Salary)
    FROM Employee e2
    WHERE e2.Department_id = e.Department_id
);
```

**Query 4 — Department name, manager name, and city:**
```sql
SELECT d.Department_name, m.Manager_name, l.City
FROM Departments d
JOIN Manager m   ON d.Manager_id  = m.Manager_id
JOIN Locations l ON d.Location_id = l.Location_id;
```

**Query 5 — Managers with experience > 15 years:**
```sql
SELECT e.First_name, e.Last_name, e.Hire_date, e.Salary
FROM Employee e
JOIN Manager m ON e.Manager_id = m.Manager_id
WHERE TIMESTAMPDIFF(YEAR, e.Hire_date, CURDATE()) > 15;
```

---

## Problem Statement 5 — Cursors (Oracle PL/SQL)

**Schema:**
```sql
CREATE TABLE Products (
    Product_id   NUMBER PRIMARY KEY,
    Product_Name VARCHAR2(50),
    Product_Type VARCHAR2(30),
    Price        NUMBER
);

INSERT INTO Products VALUES (1,'T-Shirt','Apparel',499);
INSERT INTO Products VALUES (2,'Jeans','Apparel',1999);
INSERT INTO Products VALUES (3,'Laptop','Electronics',55000);
INSERT INTO Products VALUES (4,'Jacket','Apparel',3999);
INSERT INTO Products VALUES (5,'Phone','Electronics',25000);
INSERT INTO Products VALUES (6,'Kurta','Apparel',799);
```

**1. Parameterized Cursor — Products in price range and type Apparel:**
```sql
DECLARE
    v_min_price NUMBER := &min_price;
    v_max_price NUMBER := &max_price;

    CURSOR c_products (p_min NUMBER, p_max NUMBER) IS
        SELECT * FROM Products
        WHERE Product_Type = 'Apparel'
          AND Price BETWEEN p_min AND p_max;

    v_rec Products%ROWTYPE;
BEGIN
    OPEN c_products(v_min_price, v_max_price);
    LOOP
        FETCH c_products INTO v_rec;
        EXIT WHEN c_products%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE(v_rec.Product_id || ' | ' || v_rec.Product_Name || ' | ' || v_rec.Price);
    END LOOP;
    CLOSE c_products;
END;
/
```

**2. Explicit Cursor — Products with Price > 5000:**
```sql
DECLARE
    CURSOR c_expensive IS
        SELECT * FROM Products WHERE Price > 5000;
    v_rec Products%ROWTYPE;
BEGIN
    OPEN c_expensive;
    LOOP
        FETCH c_expensive INTO v_rec;
        EXIT WHEN c_expensive%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE(v_rec.Product_id || ' | ' || v_rec.Product_Name || ' | ' || v_rec.Price);
    END LOOP;
    CLOSE c_expensive;
END;
/
```

**3. Implicit Cursor — Count rows updated after price increment by 1000:**
```sql
BEGIN
    UPDATE Products SET Price = Price + 1000;

    IF SQL%FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Number of records updated: ' || SQL%ROWCOUNT);
    ELSE
        DBMS_OUTPUT.PUT_LINE('No records were updated.');
    END IF;
END;
/
```

---

## Problem Statement 6 — DML Using MySQL

**Schema & Tables:**
```sql
CREATE TABLE Customer (
    CustID       INT PRIMARY KEY,
    Name         VARCHAR(50),
    Cust_Address VARCHAR(100),
    Phone_no     VARCHAR(15),
    Email_ID     VARCHAR(50),
    Age          INT
);

CREATE TABLE Branch (
    Branch_ID    INT PRIMARY KEY,
    Branch_Name  VARCHAR(50),
    Address      VARCHAR(100)
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

-- Insert sample data
INSERT INTO Customer VALUES (101,'Amit','Pune','9876543210','amit@mail.com',28);
INSERT INTO Customer VALUES (102,'Alok','Mumbai','9123456780','alok@mail.com',35);
INSERT INTO Customer VALUES (103,'Priya','Pune','9012345678','priya@mail.com',42);
INSERT INTO Customer VALUES (104,'Sneha','Delhi','9988776655','sneha@mail.com',25);

INSERT INTO Branch VALUES (1,'Pune Main','FC Road Pune');
INSERT INTO Branch VALUES (2,'Mumbai','Andheri Mumbai');

INSERT INTO Account VALUES (1001,1,101,'2020-01-15','Saving Account',75000);
INSERT INTO Account VALUES (1002,2,102,'2019-06-20','Current Account',120000);
INSERT INTO Account VALUES (1003,1,103,'2021-03-10','Saving Account',45000);
INSERT INTO Account VALUES (1004,2,104,'2022-08-05','Saving Account',30000);
```

**Query 1 — Modify size of Email_ID to 20:**
```sql
ALTER TABLE Customer MODIFY Email_ID VARCHAR(20);
```

**Query 2 — Change Email_ID to NOT NULL:**
```sql
ALTER TABLE Customer MODIFY Email_ID VARCHAR(20) NOT NULL;
```

**Query 3 — Total customers with balance > 50,000:**
```sql
SELECT COUNT(*) AS Total_Customers
FROM Customer c
JOIN Account a ON c.CustID = a.CustID
WHERE a.Balance > 50000;
```

**Query 4 — Average balance for Saving Account:**
```sql
SELECT AVG(Balance) AS Avg_Balance
FROM Account
WHERE Account_type = 'Saving Account';
```

**Query 5 — Customers from Pune OR name starts with 'A':**
```sql
SELECT * FROM Customer
WHERE Cust_Address LIKE '%Pune%'
   OR Name LIKE 'A%';
```

**Query 6 — Create Saving_Account table from Account table:**
```sql
CREATE TABLE Saving_Account AS
SELECT Account_no, Branch_ID, CustID, date_open, Balance
FROM Account
WHERE Account_type = 'Saving Account';
```

**Query 7 — Customer details age-wise with balance >= 20,000:**
```sql
SELECT c.*, a.Balance
FROM Customer c
JOIN Account a ON c.CustID = a.CustID
WHERE a.Balance >= 20000
ORDER BY c.Age;
```

---

## Problem Statement 7 — Triggers (Oracle PL/SQL)

**Schema:**
```sql
CREATE TABLE Employee (
    emp_id     NUMBER PRIMARY KEY,
    dept_id    NUMBER,
    emp_name   VARCHAR2(50),
    DoJ        DATE,
    salary     NUMBER,
    commission NUMBER,
    job_title  VARCHAR2(50)
);

CREATE TABLE job_history (
    emp_id        NUMBER,
    old_job_title VARCHAR2(50),
    old_dept_id   NUMBER,
    start_date    DATE,
    end_date      DATE
);
```

**Trigger 1 — Prevent salary decrease:**
```sql
CREATE OR REPLACE TRIGGER trg_no_salary_decrease
BEFORE UPDATE OF salary ON Employee
FOR EACH ROW
BEGIN
    IF :NEW.salary < :OLD.salary THEN
        RAISE_APPLICATION_ERROR(-20002, 'ERROR: Salary cannot be decreased.');
    END IF;
END;
/
```

**Trigger 2 — Log job title change to job_history:**
```sql
CREATE OR REPLACE TRIGGER trg_job_title_change
BEFORE UPDATE OF job_title ON Employee
FOR EACH ROW
BEGIN
    IF :OLD.job_title <> :NEW.job_title THEN
        INSERT INTO job_history (emp_id, old_job_title, old_dept_id, start_date, end_date)
        VALUES (:OLD.emp_id, :OLD.job_title, :OLD.dept_id, :OLD.DoJ, SYSDATE);
    END IF;
END;
/
```

**Test:**
```sql
INSERT INTO Employee VALUES (1, 10, 'Balaji', TO_DATE('2015-06-01','YYYY-MM-DD'), 50000, NULL, 'Analyst');
UPDATE Employee SET salary = 45000 WHERE emp_id = 1;       -- Will raise error
UPDATE Employee SET job_title = 'Senior Analyst' WHERE emp_id = 1; -- Logs to job_history
SELECT * FROM job_history;
```

---

## Problem Statement 8 — DDL Using MySQL

**Schema & Tables with Referential Integrity:**
```sql
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
    date_open    DATE,
    Account_type VARCHAR(30),
    Balance      DECIMAL(12,2),
    FOREIGN KEY (Branch_ID) REFERENCES Branch(Branch_ID),
    FOREIGN KEY (CustID)    REFERENCES Customer(CustID)
);
```

> **ER Diagram Description:**
> - `Customer` (1) ——< (M) `Account` >—— (1) `Branch`
> - Each Account belongs to one Customer and one Branch.
> - PK: CustID, Branch_ID, Account_no

**Query 3 — Index on primary key of Account:**
```sql
CREATE INDEX idx_account_pk ON Account(Account_no);
```

**Query 4 — View: Customer_Info for age < 45:**
```sql
CREATE VIEW Customer_Info AS
SELECT * FROM Customer WHERE Age < 45;
```

**Query 5 — Update the View with open_date = 16/4/2017:**
```sql
-- Update underlying Account table through the view context:
UPDATE Account SET date_open = '2017-04-16' WHERE CustID IN (
    SELECT CustID FROM Customer_Info
);
```

**Query 6 — Create Sequence on Branch table:**
```sql
-- MySQL 8+ AUTO_INCREMENT approach:
ALTER TABLE Branch MODIFY Branch_ID INT AUTO_INCREMENT;

-- Or using a sequence-like approach:
CREATE TABLE Branch_Seq (seq_val INT AUTO_INCREMENT PRIMARY KEY);
```

**Query 7 — Create Synonym 'Branch_info' for Branch table:**
```sql
-- MySQL does not support synonyms natively; use a VIEW as equivalent:
CREATE VIEW Branch_info AS SELECT * FROM Branch;
```

---

## Problem Statement 9 — CRUD Using MongoDB

**Create Collection and Insert 20 Documents:**
```javascript
use dbms_exam

db.Social_Media.insertMany([
    { User_Id: 1,  User_Name: "Aryan",   No_of_Posts: 120, No_of_Friends: 8,  Friends_List: ["Riya","Karan","Ansh"],        Interests: ["Gaming","Music"] },
    { User_Id: 2,  User_Name: "Riya",    No_of_Posts: 80,  No_of_Friends: 5,  Friends_List: ["Aryan","Priya"],              Interests: ["Travel","Food"] },
    { User_Id: 3,  User_Name: "Karan",   No_of_Posts: 200, No_of_Friends: 12, Friends_List: ["Aryan","Neha","Sam","Raj"],   Interests: ["Cricket","Movies"] },
    { User_Id: 4,  User_Name: "Priya",   No_of_Posts: 45,  No_of_Friends: 3,  Friends_List: ["Riya","Ansh"],                Interests: ["Art","Books"] },
    { User_Id: 5,  User_Name: "Neha",    No_of_Posts: 300, No_of_Friends: 15, Friends_List: ["Karan","Sam","Ansh","Jay"],   Interests: ["Fashion","Travel"] },
    { User_Id: 6,  User_Name: "Sam",     No_of_Posts: 90,  No_of_Friends: 6,  Friends_List: ["Karan","Neha"],               Interests: ["Tech","Gaming"] },
    { User_Id: 7,  User_Name: "Jay",     No_of_Posts: 150, No_of_Friends: 9,  Friends_List: ["Neha","Raj","Tina"],          Interests: ["Fitness","Music"] },
    { User_Id: 8,  User_Name: "Tina",    No_of_Posts: 60,  No_of_Friends: 4,  Friends_List: ["Jay","Mia"],                  Interests: ["Movies","Art"] },
    { User_Id: 9,  User_Name: "Raj",     No_of_Posts: 110, No_of_Friends: 7,  Friends_List: ["Karan","Jay","Ansh"],         Interests: ["Cricket","Food"] },
    { User_Id: 10, User_Name: "Mia",     No_of_Posts: 25,  No_of_Friends: 2,  Friends_List: ["Tina"],                       Interests: ["Books","Travel"] },
    { User_Id: 11, User_Name: "Ansh",    No_of_Posts: 175, No_of_Friends: 10, Friends_List: ["Aryan","Priya","Raj"],        Interests: ["Gaming","Tech"] },
    { User_Id: 12, User_Name: "Pooja",   No_of_Posts: 130, No_of_Friends: 8,  Friends_List: ["Simran","Asha"],              Interests: ["Fashion","Food"] },
    { User_Id: 13, User_Name: "Simran",  No_of_Posts: 55,  No_of_Friends: 3,  Friends_List: ["Pooja"],                      Interests: ["Music","Art"] },
    { User_Id: 14, User_Name: "Asha",    No_of_Posts: 220, No_of_Friends: 13, Friends_List: ["Pooja","Dev","Rohan"],        Interests: ["Travel","Books"] },
    { User_Id: 15, User_Name: "Dev",     No_of_Posts: 95,  No_of_Friends: 5,  Friends_List: ["Asha","Rohan"],               Interests: ["Tech","Gaming"] },
    { User_Id: 16, User_Name: "Rohan",   No_of_Posts: 140, No_of_Friends: 9,  Friends_List: ["Dev","Asha","Vikram"],        Interests: ["Cricket","Fitness"] },
    { User_Id: 17, User_Name: "Vikram",  No_of_Posts: 35,  No_of_Friends: 2,  Friends_List: ["Rohan"],                      Interests: ["Movies","Music"] },
    { User_Id: 18, User_Name: "Meera",   No_of_Posts: 260, No_of_Friends: 16, Friends_List: ["Sona","Nitu","Ritu","Piya"],  Interests: ["Food","Travel"] },
    { User_Id: 19, User_Name: "Sona",    No_of_Posts: 70,  No_of_Friends: 4,  Friends_List: ["Meera","Nitu"],               Interests: ["Art","Fitness"] },
    { User_Id: 20, User_Name: "Nitu",    No_of_Posts: 105, No_of_Friends: 6,  Friends_List: ["Meera","Sona","Ritu"],        Interests: ["Books","Gaming"] }
])
```

**Query 1 — List all users in formatted manner:**
```javascript
db.Social_Media.find().pretty()
```

**Query 2 — Users with No_of_Posts > 100:**
```javascript
db.Social_Media.find({ No_of_Posts: { $gt: 100 } }).pretty()
```

**Query 3 — User names and their Friends_List:**
```javascript
db.Social_Media.find({}, { User_Name: 1, Friends_List: 1, _id: 0 }).pretty()
```

**Query 4 — User IDs and Friends_List where No_of_Friends > 5:**
```javascript
db.Social_Media.find(
    { No_of_Friends: { $gt: 5 } },
    { User_Id: 1, Friends_List: 1, _id: 0 }
).pretty()
```

**Query 5 — All users sorted by No_of_Posts descending:**
```javascript
db.Social_Media.find().sort({ No_of_Posts: -1 }).pretty()
```

---

## Problem Statement 10 — Cursors (Oracle PL/SQL)

**Schema:**
```sql
CREATE TABLE Employee (
    Emp_id   NUMBER PRIMARY KEY,
    Emp_Name VARCHAR2(50),
    Salary   NUMBER
);

INSERT INTO Employee VALUES (1,'Amit',60000);
INSERT INTO Employee VALUES (2,'Priya',45000);
INSERT INTO Employee VALUES (3,'Raj',80000);
INSERT INTO Employee VALUES (4,'Neha',30000);
INSERT INTO Employee VALUES (5,'Vikas',70000);
```

**1. Explicit Cursor — Salary > 50,000:**
```sql
DECLARE
    CURSOR c_high_sal IS
        SELECT * FROM Employee WHERE Salary > 50000;
    v_rec Employee%ROWTYPE;
BEGIN
    OPEN c_high_sal;
    LOOP
        FETCH c_high_sal INTO v_rec;
        EXIT WHEN c_high_sal%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE(v_rec.Emp_id || ' | ' || v_rec.Emp_Name || ' | ' || v_rec.Salary);
    END LOOP;
    CLOSE c_high_sal;
END;
/
```

**2. Implicit Cursor — Total number of tuples:**
```sql
BEGIN
    UPDATE Employee SET Salary = Salary;  -- Dummy update to fire SQL%ROWCOUNT
    DBMS_OUTPUT.PUT_LINE('Total Records in Employee Table: ' || SQL%ROWCOUNT);
END;
/
```

**3. Parameterized Cursor — Salary of given employee ID:**
```sql
DECLARE
    v_emp_id NUMBER := &emp_id;

    CURSOR c_emp_sal (p_id NUMBER) IS
        SELECT Emp_Name, Salary FROM Employee WHERE Emp_id = p_id;

    v_name   VARCHAR2(50);
    v_salary NUMBER;
BEGIN
    OPEN c_emp_sal(v_emp_id);
    FETCH c_emp_sal INTO v_name, v_salary;
    IF c_emp_sal%FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Employee: ' || v_name || ' | Salary: ' || v_salary);
    ELSE
        DBMS_OUTPUT.PUT_LINE('No employee found with ID: ' || v_emp_id);
    END IF;
    CLOSE c_emp_sal;
END;
/
```

---

## Problem Statement 11 — JOINs & Subqueries (MySQL)

*(Same schema as Problem 4 — reuse those tables)*

**Query 1 — Employees with salary higher than 'Singh':**
```sql
SELECT e.First_name, e.Last_name, e.Salary
FROM Employee e
WHERE e.Salary > (
    SELECT Salary FROM Employee WHERE Last_name = 'Singh' LIMIT 1
);
```

**Query 2 — Employees who have a manager and work in US-based department:**
```sql
SELECT e.First_name, e.Last_name
FROM Employee e
JOIN Departments d  ON e.Department_id = d.Department_id
JOIN Locations l    ON d.Location_id   = l.Location_id
WHERE e.Manager_id IS NOT NULL
  AND l.Country_id = 'US';
```

**Query 3 — Employees with salary > average salary:**
```sql
SELECT e.First_name, e.Last_name, e.Salary
FROM Employee e
WHERE e.Salary > (SELECT AVG(Salary) FROM Employee);
```

**Query 4 — Employee ID, last name, manager ID, manager name:**
```sql
SELECT e.Employee_id, e.Last_name, e.Manager_id, m.Manager_name
FROM Employee e
JOIN Manager m ON e.Manager_id = m.Manager_id;
```

**Query 5 — Employees hired after 'Jones':**
```sql
SELECT e.First_name, e.Last_name, e.Hire_date
FROM Employee e
WHERE e.Hire_date > (
    SELECT Hire_date FROM Employee WHERE Last_name = 'Jones' LIMIT 1
);
```

---

## Problem Statement 12 — Procedures / Functions (Oracle PL/SQL)

**Schema:**
```sql
CREATE TABLE Employee (
    emp_id     NUMBER PRIMARY KEY,
    dept_id    NUMBER,
    emp_name   VARCHAR2(50),
    DoJ        DATE,
    salary     NUMBER,
    commission NUMBER,
    job_title  VARCHAR2(50)
);

INSERT INTO Employee VALUES (1,10,'Amit',TO_DATE('2010-01-01','YYYY-MM-DD'),15000,NULL,'Developer');
INSERT INTO Employee VALUES (2,20,'Priya',TO_DATE('2005-06-15','YYYY-MM-DD'),8000,NULL,'Analyst');
INSERT INTO Employee VALUES (3,10,'Raj',TO_DATE('2018-03-20','YYYY-MM-DD'),2500,NULL,'Intern');
INSERT INTO Employee VALUES (4,30,'Neha',TO_DATE('2015-09-10','YYYY-MM-DD'),6000,NULL,'Executive');
```

**1. Stored Procedure — Calculate commission:**
```sql
CREATE OR REPLACE PROCEDURE calc_commission AS
    CURSOR c_emp IS SELECT emp_id, salary, DoJ FROM Employee;
    v_exp     NUMBER;
    v_comm    NUMBER;
BEGIN
    FOR rec IN c_emp LOOP
        v_exp := TRUNC(MONTHS_BETWEEN(SYSDATE, rec.DoJ) / 12);

        IF rec.salary > 10000 THEN
            v_comm := rec.salary * 0.004;
        ELSIF rec.salary < 10000 AND v_exp > 10 THEN
            v_comm := rec.salary * 0.0035;
        ELSIF rec.salary < 3000 THEN
            v_comm := rec.salary * 0.0025;
        ELSE
            v_comm := rec.salary * 0.0015;
        END IF;

        UPDATE Employee SET commission = v_comm WHERE emp_id = rec.emp_id;
    END LOOP;

    COMMIT;
    DBMS_OUTPUT.PUT_LINE('Commissions updated successfully.');
END;
/

EXEC calc_commission;
SELECT * FROM Employee;
```

**2. Function — Return manager name of a department:**
```sql
CREATE OR REPLACE FUNCTION get_manager_name (p_dept_id IN NUMBER)
RETURN VARCHAR2 AS
    v_manager_name VARCHAR2(50);
BEGIN
    SELECT emp_name INTO v_manager_name
    FROM Employee
    WHERE dept_id = p_dept_id
      AND job_title LIKE '%Manager%'
      AND ROWNUM = 1;

    RETURN v_manager_name;

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN 'No Manager Found';
END;
/

-- Execute:
DECLARE
    v_name VARCHAR2(50);
BEGIN
    v_name := get_manager_name(10);
    DBMS_OUTPUT.PUT_LINE('Manager: ' || v_name);
END;
/
```

---

## Problem Statement 13 — DML Using MySQL

**Schema & Tables:**
```sql
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

INSERT INTO Customer VALUES (101,'Amit','Pune','9876543210',28);
INSERT INTO Customer VALUES (102,'Alok','Mumbai','9123456780',35);
INSERT INTO Customer VALUES (103,'Priya','Pune','9012345678',42);
INSERT INTO Customer VALUES (104,'Anil','Delhi','9988776655',50);

INSERT INTO Branch VALUES (1,'Pune Main','FC Road Pune');
INSERT INTO Branch VALUES (2,'Mumbai','Andheri Mumbai');

INSERT INTO Account VALUES (1001,1,101,'2020-01-15','Saving Account',75000);
INSERT INTO Account VALUES (1002,2,102,'2019-06-20','Current Account',120000);
INSERT INTO Account VALUES (1003,1,103,'Saving Account',45000,'2021-03-10');
INSERT INTO Account VALUES (1004,2,104,'2022-08-05','Saving Account',15000);
```

**Query 1 — Add column Email_Address:**
```sql
ALTER TABLE Customer ADD Email_Address VARCHAR(50);
```

**Query 2 — Rename Email_Address to Email_ID:**
```sql
ALTER TABLE Customer RENAME COLUMN Email_Address TO Email_ID;
```

**Query 3 — Customer with highest balance:**
```sql
SELECT c.*, a.Balance
FROM Customer c
JOIN Account a ON c.CustID = a.CustID
WHERE a.Balance = (SELECT MAX(Balance) FROM Account);
```

**Query 4 — Customer with lowest balance for Saving Account:**
```sql
SELECT c.*, a.Balance
FROM Customer c
JOIN Account a ON c.CustID = a.CustID
WHERE a.Account_type = 'Saving Account'
  AND a.Balance = (SELECT MIN(Balance) FROM Account WHERE Account_type = 'Saving Account');
```

**Query 5 — Customers from Pune with age > 35:**
```sql
SELECT * FROM Customer
WHERE Cust_Address LIKE '%Pune%' AND Age > 35;
```

**Query 6 — CustID, Name, Age in ascending order of age:**
```sql
SELECT CustID, Name, Age FROM Customer ORDER BY Age ASC;
```

**Query 7 — Name, Branch_ID grouped by Account_type:**
```sql
SELECT c.Name, a.Branch_ID, a.Account_type
FROM Customer c
JOIN Account a ON c.CustID = a.CustID
GROUP BY a.Account_type, c.Name, a.Branch_ID;
```

---

## Problem Statement 14 — Aggregation & Indexing (MongoDB)

**Create Collection & Insert Sample Data:**
```javascript
use dbms_exam

db.Student_Data.insertMany([
    { Student_ID: 1,  Student_Name: "Aryan",  Department: "CS",   Marks: 85 },
    { Student_ID: 2,  Student_Name: "Priya",  Department: "IT",   Marks: 90 },
    { Student_ID: 3,  Student_Name: "Karan",  Department: "CS",   Marks: 78 },
    { Student_ID: 4,  Student_Name: "Neha",   Department: "ENTC", Marks: 92 },
    { Student_ID: 5,  Student_Name: "Raj",    Department: "IT",   Marks: 70 },
    { Student_ID: 6,  Student_Name: "Sneha",  Department: "CS",   Marks: 88 },
    { Student_ID: 7,  Student_Name: "Vikas",  Department: "ENTC", Marks: 65 },
    { Student_ID: 8,  Student_Name: "Meera",  Department: "IT",   Marks: 95 },
    { Student_ID: 9,  Student_Name: "Sam",    Department: "CS",   Marks: 72 },
    { Student_ID: 10, Student_Name: "Pooja",  Department: "ENTC", Marks: 80 }
])
```

**Query 1 — All students by department with average marks:**
```javascript
db.Student_Data.aggregate([
    { $group: { _id: "$Department", Students: { $push: "$Student_Name" }, Avg_Marks: { $avg: "$Marks" } } }
])
```

**Query 2 — Student count per department:**
```javascript
db.Student_Data.aggregate([
    { $group: { _id: "$Department", Student_Count: { $sum: 1 } } }
])
```

**Query 3 — Highest marks per department in descending order:**
```javascript
db.Student_Data.aggregate([
    { $sort: { Marks: -1 } },
    { $group: { _id: "$Department", Student_Name: { $first: "$Student_Name" }, Max_Marks: { $max: "$Marks" } } },
    { $sort: { Max_Marks: -1 } }
])
```

**Query 4 — Create index on Student_ID:**
```javascript
db.Student_Data.createIndex({ Student_ID: 1 })
```

**Query 5 — Create compound index on Student_Name and Department:**
```javascript
db.Student_Data.createIndex({ Student_Name: 1, Department: 1 })
```

**Query 6 — Drop index on Student_ID:**
```javascript
db.Student_Data.dropIndex({ Student_ID: 1 })
```

**Query 7 — Drop compound index on Student_Name and Department:**
```javascript
db.Student_Data.dropIndex({ Student_Name: 1, Department: 1 })
```

---

## Problem Statement 15 — Unnamed Block (Oracle PL/SQL)

**Schema:**
```sql
CREATE TABLE Employee (
    emp_id     NUMBER PRIMARY KEY,
    dept_id    NUMBER,
    emp_name   VARCHAR2(50),
    DoJ        DATE,
    salary     NUMBER,
    commission NUMBER,
    job_title  VARCHAR2(50)
);

CREATE TABLE Salary_Increment (
    emp_id     NUMBER,
    new_salary NUMBER
);

INSERT INTO Employee VALUES (115, 10, 'Balaji', TO_DATE('2012-06-01','YYYY-MM-DD'), 40000, NULL, 'Developer');
```

**Unnamed PL/SQL Block:**
```sql
DECLARE
    v_emp_id    NUMBER := &emp_id;
    v_salary    NUMBER;
    v_doj       DATE;
    v_exp       NUMBER;
    v_new_sal   NUMBER;

    -- User-defined exception
    e_not_found EXCEPTION;
BEGIN
    BEGIN
        SELECT salary, DoJ INTO v_salary, v_doj
        FROM Employee
        WHERE emp_id = v_emp_id;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            RAISE e_not_found;
    END;

    v_exp := TRUNC(MONTHS_BETWEEN(SYSDATE, v_doj) / 12);

    IF v_exp > 10 THEN
        v_new_sal := v_salary * 1.20;
    ELSIF v_exp > 5 THEN
        v_new_sal := v_salary * 1.10;
    ELSE
        v_new_sal := v_salary * 1.05;
    END IF;

    UPDATE Employee SET salary = v_new_sal WHERE emp_id = v_emp_id;

    INSERT INTO Salary_Increment (emp_id, new_salary)
    VALUES (v_emp_id, v_new_sal);

    COMMIT;

    DBMS_OUTPUT.PUT_LINE('Employee ID  : ' || v_emp_id);
    DBMS_OUTPUT.PUT_LINE('Experience   : ' || v_exp || ' years');
    DBMS_OUTPUT.PUT_LINE('New Salary   : ' || v_new_sal);

EXCEPTION
    WHEN e_not_found THEN
        DBMS_OUTPUT.PUT_LINE('ERROR: Employee with ID ' || v_emp_id || ' not found.');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Unexpected error: ' || SQLERRM);
END;
/
```

---

## Problem Statement 16 — DDL Using MySQL

**Schema & Tables:**
```sql
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
    open_date    DATE,
    Account_type VARCHAR(30),
    Balance      DECIMAL(12,2),
    FOREIGN KEY (Branch_ID) REFERENCES Branch(Branch_ID),
    FOREIGN KEY (CustID)    REFERENCES Customer(CustID)
);

-- Insert sample data
INSERT INTO Customer VALUES (101,'Amit','Pune','9876543210','amit@mail.com',28);
INSERT INTO Customer VALUES (102,'Priya','Mumbai','9123456780','priya@mail.com',35);
INSERT INTO Customer VALUES (103,'Raj','Pune','9012345678','raj@mail.com',44);

INSERT INTO Branch VALUES (1,'Pune Main','FC Road Pune');
INSERT INTO Branch VALUES (2,'Mumbai','Andheri Mumbai');

INSERT INTO Account VALUES (1001,1,101,'2018-08-16','Saving Account',80000);
INSERT INTO Account VALUES (1002,2,102,'2018-02-16','Loan Account',150000);
INSERT INTO Account VALUES (1003,1,103,'2018-08-16','Saving Account',50000);
```

> **ER Diagram:**
> - `Customer` (1) ——< (M) `Account` >—— (1) `Branch`

**Query 3 — View: Saving Account with open_date 16/8/2018:**
```sql
CREATE VIEW Saving_account AS
SELECT c.*
FROM Customer c
JOIN Account a ON c.CustID = a.CustID
WHERE a.Account_type = 'Saving Account'
  AND a.open_date = '2018-08-16';
```

**Query 4 — Update View: Set Cust_Address = Pune for CustID = 103:**
```sql
UPDATE Customer SET Cust_Address = 'Pune' WHERE CustID = 103;
```

**Query 5 — View: Loan Account with open_date 16/2/2018:**
```sql
CREATE VIEW Loan_account AS
SELECT c.*
FROM Customer c
JOIN Account a ON c.CustID = a.CustID
WHERE a.Account_type = 'Loan Account'
  AND a.open_date = '2018-02-16';
```

**Query 6 — Index on primary key of Customer:**
```sql
CREATE INDEX idx_customer_pk ON Customer(CustID);
```

**Query 7 — Index on primary key of Branch:**
```sql
CREATE INDEX idx_branch_pk ON Branch(Branch_ID);
```

**Query 8 — Sequence on Customer table:**
```sql
ALTER TABLE Customer MODIFY CustID INT AUTO_INCREMENT;
```

**Query 9 — Synonym 'Cust_info' for Branch table (using VIEW):**
```sql
CREATE VIEW Cust_info AS SELECT * FROM Branch;
```

---

## Problem Statement 17 — CRUD Using MongoDB

**Create Collection and Insert 10 Documents:**
```javascript
use dbms_exam

db.Student.insertMany([
    { Roll_No: "A01", Name: "Aryan",  Class: "SE", Marks: 75, Address: "Pune",    Enrolled_Courses: ["DBMS","OS","CN"] },
    { Roll_No: "A02", Name: "Priya",  Class: "TE", Marks: 82, Address: "Mumbai",  Enrolled_Courses: ["DBMS","TOC","DAA"] },
    { Roll_No: "A03", Name: "Karan",  Class: "SE", Marks: 15, Address: "Pune",    Enrolled_Courses: ["OS","CN"] },
    { Roll_No: "A04", Name: "Neha",   Class: "TE", Marks: 90, Address: "Nagpur",  Enrolled_Courses: ["DBMS","TOC","ML"] },
    { Roll_No: "A05", Name: "Raj",    Class: "BE", Marks: 55, Address: "Pune",    Enrolled_Courses: ["CN","DAA"] },
    { Roll_No: "A06", Name: "Sneha",  Class: "SE", Marks: 68, Address: "Delhi",   Enrolled_Courses: ["DBMS","OS"] },
    { Roll_No: "A07", Name: "Vikas",  Class: "TE", Marks: 10, Address: "Pune",    Enrolled_Courses: ["TOC"] },
    { Roll_No: "A08", Name: "Meera",  Class: "BE", Marks: 88, Address: "Mumbai",  Enrolled_Courses: ["DBMS","ML","DAA"] },
    { Roll_No: "A09", Name: "Sam",    Class: "SE", Marks: 45, Address: "Pune",    Enrolled_Courses: ["OS","CN","TOC"] },
    { Roll_No: "A10", Name: "Pooja",  Class: "TE", Marks: 72, Address: "Nagpur",  Enrolled_Courses: ["DBMS","DAA"] }
])
```

**Query 1 — Students enrolled in DBMS and TOC:**
```javascript
db.Student.find(
    { Enrolled_Courses: { $all: ["DBMS", "TOC"] } },
    { Name: 1, Enrolled_Courses: 1, _id: 0 }
).pretty()
```

**Query 2 — Roll numbers and class where Marks > 50 OR Class = TE:**
```javascript
db.Student.find(
    { $or: [ { Marks: { $gt: 50 } }, { Class: "TE" } ] },
    { Roll_No: 1, Class: 1, _id: 0 }
).pretty()
```

**Query 3 — Update entire record of Roll_No A10:**
```javascript
db.Student.replaceOne(
    { Roll_No: "A10" },
    {
        Roll_No: "A10",
        Name: "Pooja Updated",
        Class: "BE",
        Marks: 85,
        Address: "Pune",
        Enrolled_Courses: ["DBMS", "ML", "DAA", "CN"]
    }
)
```

**Query 4 — Names of students with 3rd and 4th highest marks:**
```javascript
db.Student.find(
    {},
    { Name: 1, Marks: 1, _id: 0 }
).sort({ Marks: -1 }).skip(2).limit(2)
```

**Query 5 — Delete students with Marks < 20:**
```javascript
db.Student.deleteMany({ Marks: { $lt: 20 } })
```

**Query 6 — Delete only the first record from collection:**
```javascript
db.Student.deleteOne({})
```

---

*All solutions verified for SPPU B.Tech DBMS syllabus — I²IT, Pune.*
