# Tablespaces in Multitenant Architecture

## What is a Tablespace?
A **tablespace** is a logical storage unit in an Oracle database that stores database objects such as tables and indexes. The actual data is stored in **datafiles**, which are physical storage units.

In a **Multitenant Architecture**, tablespaces exist at two different levels:

1. **Container Database (CDB)**
2. **Pluggable Database (PDB)**

Each PDB has its own set of tablespaces and datafiles.

---

# Types of Tablespaces in Multitenant Architecture

There are two types of tablespaces:

1. Common Tablespace
2. Local Tablespace

---

## 1. Common Tablespace

### Definition
A **Common Tablespace** belongs to the **Container Database (CDB)** and is shared by all Pluggable Databases (PDBs) for Oracle internal operations.

### Characteristics
- Created in the Root Container (CDB$ROOT).
- Used for Oracle internal components.
- Shared among all PDBs.
- Not intended for application data.

### Check Datafiles

```sql
SELECT file_id,
       tablespace_name,
       file_name,
       ROUND(bytes/1024/1024,2) AS size_mb,
       autoextensible,
       ROUND(maxbytes/1024/1024,2) AS max_size_mb
FROM dba_data_files
ORDER BY file_id;
```
<img width="979" height="388" alt="image" src="https://github.com/user-attachments/assets/980f9018-f970-4e23-9fb9-49c83f5855b7" />

---

## 2. Local Tablespace

### Definition
A **Local Tablespace** belongs to a specific **Pluggable Database (PDB)** and stores only that PDB's data and objects.

### Characteristics
- Exists only within one PDB.
- Cannot be accessed from another PDB.
- Used for user tables, indexes, LOBs, and application data.

### Create a Local Tablespace

```sql
CREATE TABLESPACE sha
DATAFILE '/u01/app/oracle/oradata/CDB/pdb1/sha01.dbf'
SIZE 300M
AUTOEXTEND ON;
```
<img width="979" height="203" alt="image" src="https://github.com/user-attachments/assets/2f635132-7b03-4595-9042-770113991d30" />

---
# Create Tablespace inside PDB
A Local Tablespace is created inside a Pluggable Database (PDB) to store that PDB's user data, such as tables, indexes, and other database objects. It exists only within the specific PDB and cannot be accessed by other PDBs.

create tablespace sha datafile '/u01/app/oracle/oradata/CDB/pdb1/sha01.dbf' size 300m autoextend on;

<img width="979" height="206" alt="image" src="https://github.com/user-attachments/assets/291906b5-fb29-4035-bf3f-82dbc515b7bf" />


# Common Tablespace vs Local Tablespace

| Feature | Common Tablespace | Local Tablespace |
|---------|-------------------|------------------|
| Scope | Entire CDB | Single PDB |
| Created In | CDB$ROOT | Individual PDB |
| Accessible From | All PDBs | Only the owning PDB |
| Purpose | Oracle internal operations | User/Application data |
| Datafiles | Stored at CDB level | Stored inside the PDB |

---
