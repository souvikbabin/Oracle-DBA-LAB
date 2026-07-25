# Connection / Startup / Shutdown Process in Multitenant DB (CDB/PDB)

## How to Start the CDB

```sql
SQL> STARTUP;
```
<img width="979" height="348" alt="image" src="https://github.com/user-attachments/assets/e536fe97-0bd4-489c-b12c-35abdb4e60bb" />

<img width="979" height="169" alt="image" src="https://github.com/user-attachments/assets/01a71d9e-c78f-47f3-91c9-65e078d50094" />

---

## How to Start a PDB

```sql
SQL> CONN
Enter user-name: sys/oracle@pdb1 AS SYSDBA

Connected.

SQL> STARTUP;
```

**Output:**

```
Pluggable Database opened.
```
<img width="979" height="103" alt="image" src="https://github.com/user-attachments/assets/5eeb63d0-2bf5-4b63-a2a4-6934c9149ebf" />

---

## How to Shut Down a PDB

```sql
SQL> SHUTDOWN IMMEDIATE;
```

**Output:**

```
Pluggable Database closed.
```
<img width="979" height="208" alt="image" src="https://github.com/user-attachments/assets/2bdab34e-ac6e-452f-8b82-6a22c4e035f9" />

You can also use:

```sql
SQL> ALTER PLUGGABLE DATABASE pdb1 CLOSE IMMEDIATE;
```

---

## How to Shut Down the CDB

```sql
SQL> SHUTDOWN IMMEDIATE;
```

**Output:**

```
Database closed.
Database dismounted.
ORACLE instance shut down.
```
<img width="979" height="227" alt="image" src="https://github.com/user-attachments/assets/2c2f5c23-882d-449e-ad27-d434b29c0e8c" />
