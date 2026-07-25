# Pluggable Database Backup and Restore Activity

## Objective
Perform a physical backup of an Oracle Pluggable Database (PDB), simulate datafile loss, and restore and recover the database using RMAN.

---

## Step 1: Connect to the Pluggable Database

```sql
SQL> conn
Enter user-name: sys/oracle@pdb1 as sysdba
Connected.
```

---

## Step 2: Open the Pluggable Database

```sql
SQL> ALTER PLUGGABLE DATABASE pdb1 OPEN;
```

Output:

```text
Pluggable database altered.
```

---

## Step 3: Verify the Datafiles

```sql
SQL> SELECT name FROM v$datafile;
```

Verify that all PDB datafiles are available.

---

## Step 4: Take a Physical Backup Using RMAN

Connect to RMAN:

```bash
$ rman target sys/oracle@pdb1
```

RMAN Output:

```text
connected to target database: CDB:PDB1
```

Take the backup:

```rman
RMAN> BACKUP DATABASE;
```

---

## Step 5: Verify the Backup

```rman
RMAN> LIST BACKUP;
```

Ensure the backup completed successfully.

---

## Step 6: Simulate Datafile Loss

Delete the required PDB datafile (for example, `sha01.dbf`) from the operating system.

> **Note:** Perform this step only in a test or lab environment.

---

## Step 7: Restart the Database

Attempt to restart the database.

Since the required datafile is missing, the PDB will not open successfully.

---

## Step 8: Restore the Database

```rman
RMAN> RESTORE DATABASE;
```

---

## Step 9: Recover the Database

```rman
RMAN> RECOVER DATABASE;
```

---

## Step 10: Open the Pluggable Database

```rman
RMAN> ALTER PLUGGABLE DATABASE pdb1 OPEN;
```

Output:

```text
Statement processed.
```

---

## Step 11: Verify the Recovery

### Check at the OS Level

Confirm that the deleted datafile has been restored to its original location.

### Check at the Database Level

```sql
SQL> SELECT name FROM v$datafile;
```

Verify that all datafiles are online and accessible.

---

## Summary

This activity demonstrated how to:

- Connect to a Pluggable Database (PDB)
- Open the PDB
- Verify datafiles
- Perform an RMAN physical backup
- Validate the backup
- Simulate datafile loss
- Restore the database
- Recover the database
- Reopen the PDB
- Verify successful recovery
