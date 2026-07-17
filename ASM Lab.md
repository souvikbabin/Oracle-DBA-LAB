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

### Verify ASM Process

```bash
ps -ef | grep pmon
```

Expected output should include:

```
asm_pmon_+ASM
```

### Verify Grid Processes

```bash
ps -ef | grep d.bin
```

Important processes include:

- ohasd.bin
- evmd.bin
- ocssd.bin

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

---

# Exercise 5 – Check Cluster Resources

Display all Oracle Clusterware resources.

```bash
crsctl stat res -t
```

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

```bash
srvctl config database -d prod
```

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

**Important**

When using ASM storage, the datafile location must begin with a `+`.

Example:

```
+DATA
```

If the plus sign is omitted, Oracle creates the file in the default filesystem location (typically the `dbs` directory).

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
