# Oracle ASM Practical Lab Manual

## Objective

This lab demonstrates the basic administration tasks in Oracle Automatic Storage Management (ASM), including:

- Starting and stopping the ASM instance
- Starting and stopping the database
- Verifying ASM and database status
- Checking Oracle Clusterware resources
- Viewing configured databases
- Identifying mandatory ASM processes
- Listing ASM Disk Groups
- Creating tablespaces in ASM
- Moving datafiles from the file system to ASM
- Verifying successful migration

---

# Lab Environment

- Oracle Grid Infrastructure
- Oracle ASM
- Oracle RAC/Oracle Restart
- Database Name: PROD
- ASM Instance: +ASM

---

# Exercise 1 – Start Oracle ASM

## Purpose

Start the Oracle High Availability Services (OHAS), which also starts the ASM instance.

### Command

```bash
crsctl start has
```
<img width="975" height="216" alt="image" src="https://github.com/user-attachments/assets/a9997b91-d141-4b87-b44e-a38cb27b6c4b" />

### Verify ASM Process

```bash
ps -ef | grep pmon
```
Expected output should include:

```
asm_pmon_+ASM
```
<img width="1111" height="49" alt="image" src="https://github.com/user-attachments/assets/f51ee75f-674f-4d87-96a8-589886d77b32" />

### Verify Grid Processes

```bash
ps -ef | grep d.bin
```

Important processes include:

- ohasd.bin
- evmd.bin
- ocssd.bin

<img width="975" height="146" alt="image" src="https://github.com/user-attachments/assets/fc1a6202-3320-4a7f-8b2b-461407351419" />

---

# Exercise 2 – Start the Database

Start the database managed by Clusterware.

```bash
srvctl start database -d prod
```

Verify status:

```bash
srvctl status database -d prod
```

Expected result:

```
Database is running.
```
<img width="975" height="79" alt="image" src="https://github.com/user-attachments/assets/5c39c80d-2deb-423f-86df-7086e9727273" />

---

# Exercise 3 – Verify ASM Instance

Login as SYSASM.

```sql
sqlplus / as sysasm
```

Check ASM instance status.

```sql
SELECT instance_name,
       status
FROM v$instance;
```

Expected status:

```
STARTED
```
<img width="975" height="104" alt="image" src="https://github.com/user-attachments/assets/2da52409-b0ea-4614-bddb-8ce676568823" />

---

# Exercise 4 – Verify Database

Login as SYSDBA.

```sql
sqlplus / as sysdba
```

Check database status.

```sql
SELECT name,
       open_mode
FROM v$database;
```

Expected:

```
READ WRITE
```
<img width="975" height="101" alt="image" src="https://github.com/user-attachments/assets/e59b3302-9a35-4e4f-bd0e-3dfb3f8de69c" />

---

# Exercise 5 – Check Cluster Resources

Display all Oracle Clusterware resources.

```bash
crsctl stat res -t
```
<img width="975" height="419" alt="image" src="https://github.com/user-attachments/assets/bf435a56-79e2-4526-a5b6-c9c5bfd0afa1" />

Verify:

- ASM resource
- Database resource
- Listener
- Disk Groups
- Cluster Services

All required resources should display ONLINE.

---

# Exercise 6 – View Configured Databases

Display all databases registered with Clusterware.

```bash
srvctl config database
```

To display complete configuration:

<img width="975" height="46" alt="image" src="https://github.com/user-attachments/assets/aa313097-28d2-4caf-a5a7-590f5974155e" />

```bash
srvctl config database -d prod
```
<img width="975" height="265" alt="image" src="https://github.com/user-attachments/assets/81ada217-e7c9-432d-9af3-049a0f7a909f" />

Typical information displayed:

- Oracle Home
- Database Name
- SPFILE
- Password File
- Start Options
- Stop Options
- Database Role
- Disk Groups
- Management Policy

---

# Exercise 7 – Verify ASM Processes

Stop only the database.

```bash
srvctl stop database -d prod
```

Leave ASM running.

Check Oracle processes.

```bash
ps -ef | grep pmon
```

Verify Grid Infrastructure processes.

```bash
ps -ef | grep d.bin
```
<img width="975" height="146" alt="image" src="https://github.com/user-attachments/assets/b542d1ca-07c2-4783-95dd-c552ef9b317a" />


Even after stopping the database, ASM-related processes should continue running.

Mandatory Grid processes include:

- ohasd.bin
- evmd.bin
- ocssd.bin

---

# Exercise 8 – Stop ASM

Shutdown Oracle High Availability Services.

```bash
crsctl stop has
```
<img width="975" height="257" alt="image" src="https://github.com/user-attachments/assets/d35f0c31-2c9b-4720-9a9e-84cb2c69d63e" />

Verify ASM has stopped.

```bash
ps -ef | grep pmon
```

```bash
ps -ef | grep d.bin
```

The ASM process should no longer appear.

---

# Exercise 9 – Restart ASM

Start ASM again.

```bash
crsctl start has
```

Verify:

```bash
ps -ef | grep pmon
```

Expected:

```
asm_pmon_+ASM
```

Verify Grid services.

```bash
ps -ef | grep d.bin
```

---

# Exercise 10 – List ASM Disk Groups

Connect as SYSASM.

```sql
sqlplus / as sysasm
```

List available disk groups.

```sql
lsdg
```
<img width="975" height="119" alt="image" src="https://github.com/user-attachments/assets/06881b07-29fb-4082-b60e-9d35b775d82d" />

Typical information includes:

- Disk Group Name
- State
- Redundancy
- Total Space
- Free Space
- Allocation Unit Size

---

# Exercise 11 – Create a Tablespace in ASM

Create a new tablespace.

```sql
CREATE TABLESPACE RAM
DATAFILE '+DATA'
SIZE 30M;
```
<img width="975" height="53" alt="image" src="https://github.com/user-attachments/assets/ea4b6955-6434-4f61-83ef-41a188c31ca3" />

<img width="975" height="163" alt="image" src="https://github.com/user-attachments/assets/8f414d67-ed0b-4e7d-adfb-f5c3dd092b13" />


**Important**

When using ASM storage, the datafile location must begin with a `+`.

Example:

```
+DATA
```

If the plus sign is omitted, Oracle creates the file in the default filesystem location (typically the `dbs` directory).
<img width="975" height="57" alt="image" src="https://github.com/user-attachments/assets/b271d976-0146-4ce0-8025-e3bd8f5a87a3" />

<img width="975" height="164" alt="image" src="https://github.com/user-attachments/assets/9a1cf0e3-878d-423a-bbff-0b68b1cf9389" />


---

# Exercise 12 – Verify Datafile Location

Check where the datafile is stored.

```sql
SELECT
    file_id,
    tablespace_name,
    file_name
FROM dba_data_files;
```

Verify that the ASM-managed file begins with:

```
+DATA/
```

---

# Exercise 13 – Identify Files Stored in the File System

Run the same query.

```sql
SELECT
    file_name
FROM dba_data_files;
```

If a file path begins with:

```
/u01/app/...
```

then it resides on the operating system file system rather than ASM.

---

# Exercise 14 – Move a Datafile to ASM

Move the existing datafile into ASM.

Example:

```sql
ALTER DATABASE MOVE DATAFILE
'/u01/app/oracle/product/19.3.0/dbhome_1/dbs/DATA'
TO '+DATA';
```
<img width="975" height="51" alt="image" src="https://github.com/user-attachments/assets/a7171cdc-a1c0-48d3-9074-55a7edc974dd" />

<img width="975" height="164" alt="image" src="https://github.com/user-attachments/assets/2be4f126-a3b6-400d-95a9-e98786f735ce" />


Oracle copies the file into ASM and updates the control file automatically.

---

# Exercise 15 – Verify the Migration

Run:

```sql
SELECT
    file_id,
    tablespace_name,
    file_name
FROM dba_data_files;
```

The moved datafile should now display a path beginning with:

```
+DATA/
```

This confirms that the migration completed successfully.

---

# Common Verification Commands

## ASM Instance

```sql
SELECT * FROM v$instance;
```

## Database

```sql
SELECT name, open_mode FROM v$database;
```

## Disk Groups

```sql
SELECT
    name,
    state,
    total_mb,
    free_mb
FROM v$asm_diskgroup;
```

## ASM Disks

```sql
SELECT
    path,
    state
FROM v$asm_disk;
```

---

# Useful Linux Commands

List ASM processes

```bash
ps -ef | grep asm
```

Check PMON

```bash
ps -ef | grep pmon
```

Check Grid services

```bash
ps -ef | grep d.bin
```

Display Cluster resources

```bash
crsctl stat res -t
```

---
