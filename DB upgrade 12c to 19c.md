# Oracle Database 12cR1 to 19c Upgrade Using AutoUpgrade

## Overview

This document explains how to upgrade an Oracle Database from **12c Release 1 (12.1.0.2)** to **Oracle Database 19c** using the **AutoUpgrade** utility.

---

# Prerequisites

- Oracle Database 12cR1 (12.1.0.2)
- Oracle 19c software installed in a new ORACLE_HOME
- JDK 8 or later installed
- Full RMAN backup (recommended)
- Sufficient FRA (Fast Recovery Area) space
- SYSDBA privileges

---

# Step 1: Verify Database Version

```sql
SELECT name, version FROM v$instance;
```

---

# Step 2: Create Test Table (Optional)

```sql
CREATE TABLE scott_test
(
    id NUMBER,
    description VARCHAR2(50)
);

INSERT /*+ APPEND */
INTO scott_test
SELECT level,
       'Description for ' || level
FROM dual
CONNECT BY level <= 10000;

COMMIT;

SELECT COUNT(*) FROM scott_test;
```

Expected Output

```
10000
```

---

# Step 3: Take Full Backup

It is recommended to take a complete RMAN backup before starting the upgrade.

---

# Step 4: Install Oracle 19c Software

Install only the Oracle 19c software into a **new ORACLE_HOME**.

Example

```
/u02/app/oracle/product/19.0.0/dbhome_1
```

Do not overwrite the existing 12c ORACLE_HOME.

---

# Step 5: Pre-Upgrade Tasks

## Recompile Invalid Objects

```sql
@$ORACLE_HOME/rdbms/admin/utlrp.sql
```

Check invalid objects

```sql
SELECT COUNT(*)
FROM dba_objects
WHERE status='INVALID';
```

Expected Output

```
0
```

---

## Gather Dictionary Statistics

```sql
EXEC DBMS_STATS.GATHER_DICTIONARY_STATS;
```

---

## Purge Recycle Bin

```sql
PURGE DBA_RECYCLEBIN;
```

Verify

```sql
SELECT COUNT(*)
FROM DBA_RECYCLEBIN;
```

Expected Output

```
0
```

---

# Step 6: Create Upgrade Directory

```bash
cd /u02
mkdir upgrade_to_19c
cd upgrade_to_19c
```

---

# Step 7: Verify Java Installation

```bash
java -version
```

Install JDK if Java is unavailable.

---

# Step 8: Generate Sample AutoUpgrade Configuration

```bash
java -jar $ORACLE_HOME/rdbms/admin/autoupgrade.jar \
-create_sample_file config
```

---

# Step 9: Configure AutoUpgrade

Rename the sample configuration.

```bash
mv sample_config.cfg sha_config.cfg
```

Example configuration

```text
global.autoupg_log_dir=/u02/upgrade_to_19c/global_logs

upg1.dbname=sha
upg1.start_time=NOW
upg1.source_home=/u01/app/oracle/product/12.1.0/dbhome_1
upg1.target_home=/u02/app/oracle/product/19.0.0/dbhome_1
upg1.sid=sha
upg1.log_dir=/u02/upgrade_to_19c/sha_upgrade_logs
upg1.target_version=19
```

---

# Step 10: AutoUpgrade Modes

| Mode | Description |
|------|-------------|
| ANALYZE | Performs prerequisite checks only |
| FIXUPS | Fixes pre-upgrade issues |
| DEPLOY | Performs the complete upgrade |
| UPGRADE | Used when only the target ORACLE_HOME is available |

---

# Step 11: Run Analyze Mode

```bash
java -jar \
/u02/app/oracle/product/19.0.0/dbhome_1/rdbms/admin/autoupgrade.jar \
-config sha_config.cfg \
-mode ANALYZE
```

Useful monitoring command

```
lsj
```

Review error log

```
/u02/upgrade_to_19c/sha_upgrade_logs/100/autoupgrade_err.log
```

---

# Step 12: Run Deploy Mode

```bash
java -jar \
/u02/app/oracle/product/19.0.0/dbhome_1/rdbms/admin/autoupgrade.jar \
-config sha_config.cfg \
-mode DEPLOY
```

Useful commands

```
lsj

status -job <job_number>

tasks
```

---

# Step 13: Resolve FRA Space Error (If Required)

Increase FRA size.

Example

```sql
ALTER SYSTEM SET db_recovery_file_dest_size=16G;
```

Move previous logs before rerunning.

```bash
mv global_logs /u01
mv sha_upgrade_logs /u01
```

Run DEPLOY again.

---

# Step 14: AutoUpgrade Creates Restore Point

AutoUpgrade automatically creates a **Guaranteed Restore Point** before the upgrade.

This allows rollback if the upgrade fails.

---

# Step 15: Post Upgrade File Movement

Copy the following to the new ORACLE_HOME.

- Password file
- SPFILE
- listener.ora
- tnsnames.ora
- sqlnet.ora

---

# Step 16: Set New ORACLE_HOME

```bash
export ORACLE_HOME=/u02/app/oracle/product/19.0.0/dbhome_1

export PATH=$ORACLE_HOME/bin:$PATH
```

Verify

```bash
echo $ORACLE_HOME
```

---

# Step 17: Post Upgrade Tasks

Recompile invalid objects.

```sql
@$ORACLE_HOME/rdbms/admin/utlrp.sql
```

Gather dictionary statistics.

```sql
EXEC DBMS_STATS.GATHER_DICTIONARY_STATS;
```

---

# Step 18: Check Time Zone

```sql
SELECT version
FROM v$timezone_file;

SELECT DBMS_DST.get_latest_timezone_version
FROM dual;
```

If both values match, no action is required.

---

# Step 19: Verify Database Version

```sql
SELECT
    name,
    host_name,
    version,
    open_mode
FROM v$database,
     v$instance;
```

---

# Step 20: Check Compatible Parameter

```sql
SHOW PARAMETER compatible;
```

Initially it may still display

```
12.1.0.2.0
```

---

# Step 21: Drop Restore Point

```sql
DROP RESTORE POINT AUTOUPGRADE_<restore_point_name>;
```

---

# Step 22: Set Compatibility to 19c

```sql
ALTER SYSTEM SET compatible='19.0.0'
SCOPE=SPFILE;
```

Restart the database.

---

# Step 23: Verify Upgrade

Database status

```sql
SELECT name,
       open_mode
FROM v$database;
```

Version

```sql
SELECT banner
FROM v$version;
```

Expected

```
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0
```

---

# Step 24: Verify Application Data

```sql
CONN scott/tiger

SELECT COUNT(*)
FROM scott_test;
```

Expected Output

```
10000
```

---

# Upgrade Completed Successfully

## Summary

- Installed Oracle 19c software
- Performed pre-upgrade checks
- Gathered dictionary statistics
- Purged recycle bin
- Generated AutoUpgrade configuration
- Executed ANALYZE mode
- Executed DEPLOY mode
- Resolved FRA issue (if encountered)
- Verified database version
- Recompiled invalid objects
- Updated COMPATIBLE parameter
- Confirmed application data integrity
