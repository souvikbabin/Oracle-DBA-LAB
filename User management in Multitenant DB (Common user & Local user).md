# User Management in Oracle Multitenant Database (Common User & Local User)

## Overview

Oracle Multitenant architecture consists of one **Container Database (CDB)** and one or more **Pluggable Databases (PDBs)**. User management depends on whether you are connected to the **CDB** or a **PDB**.

---

# Step 1: Check the Current Container

## Command

```sql
SHOW CON_NAME;
```
<img width="979" height="144" alt="image" src="https://github.com/user-attachments/assets/bc60a2ee-6da3-4adf-8dab-589f0421b8b2" />

### Description

Displays the name of the container to which you are currently connected.

### Expected Output

```text
CON_NAME
--------------------
CDB$ROOT
```

or

```text
CON_NAME
--------------------
PDB1
```

---

# Step 2: List All Pluggable Databases

## Command

```sql
SHOW PDBS;
```

### Description

Displays all Pluggable Databases along with their current open mode.

<img width="979" height="130" alt="image" src="https://github.com/user-attachments/assets/56d4682d-2aeb-4778-ba61-9ce3c3ac08d9" />

### Example Output

```text
CON_ID  CON_NAME   OPEN MODE
------  ---------  ----------
2       PDB$SEED   READ ONLY
3       PDB1       READ WRITE
4       PDB2       MOUNTED
```

---

# Step 3: Connect to a Pluggable Database

## Command

```sql
CONN sys/oracle@pdb1 AS SYSDBA
```

### Description

Connects directly to the specified Pluggable Database using SYSDBA privileges.

### Verify the Connection

```sql
SHOW CON_NAME;
```

Expected Output

```text
CON_NAME
--------------------
PDB1
```
<img width="979" height="95" alt="image" src="https://github.com/user-attachments/assets/1a245eb1-d749-43dc-b9c1-7ed78409532c" />

---

# Step 4: Close a Pluggable Database

## Command

```sql
ALTER PLUGGABLE DATABASE pdb2 CLOSE IMMEDIATE;
```

### Description

Closes the specified Pluggable Database immediately.

### Verify

```sql
SHOW PDBS;
```

Expected Output

```text
PDB2    MOUNTED
```
<img width="979" height="163" alt="image" src="https://github.com/user-attachments/assets/dbc4718e-3345-41df-9838-24900fa37c31" />

---

# Step 5: Open a Pluggable Database

## Command

```sql
ALTER PLUGGABLE DATABASE pdb2 OPEN;
```

### Verify

```sql
SHOW PDBS;
```

Expected Output

```text
PDB2    READ WRITE
```

---

# Step 6: Create a Common User

> **Note:** Common users must be created while connected to **CDB$ROOT**.

## Verify Current Container

```sql
SHOW CON_NAME;
```

It should display:

```text
CDB$ROOT
```

## Create Common User

```sql
CREATE USER C##sha
IDENTIFIED BY sha
QUOTA 1G ON users;
```
<img width="979" height="300" alt="image" src="https://github.com/user-attachments/assets/a256f1d8-cc89-4937-822d-883c33beb14d" />

## Grant Required Privileges

```sql
GRANT CREATE SESSION TO C##sha;
```

### Description

A common user is available in every Pluggable Database within the Container Database.

---

# Step 7: Create a Local User

> **Note:** Connect to the target PDB before creating a local user.

## Verify Current Container

```sql
SHOW CON_NAME;
```

Example

```text
PDB1
```

## Create Local User

```sql
CREATE USER pdb_sha
IDENTIFIED BY oracle
QUOTA 1G ON users;
```
<img width="979" height="352" alt="image" src="https://github.com/user-attachments/assets/75de1e2c-f2c5-47d2-9114-9b804b27a2c9" />

## Grant Required Privileges

```sql
GRANT CREATE SESSION TO pdb_sha;
```

### Description

A local user exists only inside the current Pluggable Database.

---

# Difference Between Common User and Local User

| Feature | Common User | Local User |
|----------|-------------|------------|
| Created In | CDB$ROOT | PDB |
| Prefix | Must begin with `C##` | Any valid name |
| Accessible From | All PDBs | Only the current PDB |
| Example | `C##SHA` | `PDB_SHA` |

---

# Useful Commands

## Check Current Container

```sql
SHOW CON_NAME;
```

## Display All PDBs

```sql
SHOW PDBS;
```



## Switch to Root Container

```sql
ALTER SESSION SET CONTAINER = CDB$ROOT;
```

## Switch to a PDB

```sql
ALTER SESSION SET CONTAINER = PDB1;
```

## Check Existing Users

```sql
SELECT username, common
FROM dba_users
ORDER BY username;
```
