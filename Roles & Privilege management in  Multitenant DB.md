# Roles & Privilege Management in Oracle Multitenant Database

## Overview

In an Oracle Multitenant Database, privileges can be granted to both **Common Users** and **Local Users**. Privileges are categorized into:

- **System Privileges** – Allow users to perform database-level operations.
- **Object Privileges** – Allow users to perform operations on specific database objects such as tables, views, or procedures.

---

# Common User Privileges

> **Note:** A common user must be connected to **CDB$ROOT** and the username must start with the prefix `C##`.

## Verify Current Container

```sql
SHOW CON_NAME;
```

Expected Output

```text
CON_NAME
---------
CDB$ROOT
```

---

## Grant System Privileges to a Common User

### Command

```sql
GRANT CREATE SESSION, CREATE TABLE TO C##SHA;
```

### Description

Grants the following system privileges:

- `CREATE SESSION` – Allows the user to connect to the database.
- `CREATE TABLE` – Allows the user to create tables in their schema.

### Expected Output
<img width="979" height="171" alt="image" src="https://github.com/user-attachments/assets/23f775a1-01d7-4837-84cc-e1137c65c1fb" />

```text
Grant succeeded.
```

---

## Grant Object Privileges to a Common User

### Command

```sql
GRANT SELECT, INSERT, UPDATE
ON C##SHA.T20
TO C##SHA;
```

### Description

Grants object-level privileges on the table `T20`.

| Privilege | Description |
|-----------|-------------|
| SELECT | Read data from the table |
| INSERT | Insert new rows |
| UPDATE | Modify existing rows |

### Expected Output
<img width="979" height="67" alt="image" src="https://github.com/user-attachments/assets/f7fd2412-0ab4-46bc-a75f-d029e1c20fe0" />

```text
Grant succeeded.
```
<img width="979" height="181" alt="image" src="https://github.com/user-attachments/assets/881f2252-363f-4889-aedb-a4e46f25a176" />

---

# Local User Privileges

> **Note:** A local user exists only within a specific Pluggable Database (PDB).

## Connect to the PDB

```sql
ALTER SESSION SET CONTAINER = PDB1;
```

Verify:

```sql
SHOW CON_NAME;
```

Expected Output

```text
CON_NAME
---------
PDB1
```

---

## Create a Local User

### Command

```sql
CREATE USER HRIS
IDENTIFIED BY hris
QUOTA 1G ON users;
```

### Description

Creates a local user named `HRIS` with a 1 GB quota on the `USERS` tablespace.

### Expected Output

```text
User created.
```
<img width="979" height="251" alt="image" src="https://github.com/user-attachments/assets/b0c189d2-fb29-44e8-b92d-dc8128fed288" />

---

## Grant System Privileges to a Local User

### Command

```sql
GRANT CREATE SESSION, CREATE TABLE TO HRIS;
```

### Description

Allows the local user to:

- Connect to the database.
- Create tables in the user's schema.

### Expected Output

```text
Grant succeeded.
```
<img width="979" height="160" alt="image" src="https://github.com/user-attachments/assets/fa082ea1-eb2b-4841-a760-995c6c9cd600" />

---

## Grant Object Privileges to a Local User

### Command

```sql
GRANT SELECT, INSERT, UPDATE
ON HRIS.TEST
TO HRIS;
```

### Description

Grants read and DML privileges on the `TEST` table.

### Expected Output

```text
Grant succeeded.
```
<img width="979" height="191" alt="image" src="https://github.com/user-attachments/assets/4b7e0863-e293-43bf-862a-ce693825aab3" />

<img width="979" height="213" alt="image" src="https://github.com/user-attachments/assets/dffa6d19-7e45-4455-8c71-35731c75d4ac" />


---

# Verify System Privileges

```sql
SELECT grantee, privilege
FROM dba_sys_privs
WHERE grantee IN ('C##SHA', 'HRIS');
```

---

# Verify Object Privileges

```sql
SELECT grantee,
       owner,
       table_name,
       privilege
FROM dba_tab_privs
WHERE grantee IN ('C##SHA', 'HRIS');
```

---

# Difference Between System and Object Privileges

| System Privilege | Object Privilege |
|------------------|------------------|
| Applies to database operations | Applies to a specific object |
| Example: `CREATE SESSION` | Example: `SELECT` |
| Granted using `GRANT CREATE TABLE` | Granted using `GRANT SELECT ON table_name` |

---

# Summary

- **System Privileges** control database-level actions, such as connecting to the database or creating tables.
- **Object Privileges** control access to specific database objects, such as tables and views.
- **Common Users (`C##`)** are created in `CDB$ROOT` and can be granted privileges across the multitenant environment.
- **Local Users** are created inside a specific PDB and their privileges are limited to that PDB.
- Use `DBA_SYS_PRIVS` to verify system privileges and `DBA_TAB_PRIVS` to verify object privileges.
