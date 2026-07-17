# Automating Oracle RMAN Backups Using a Shell Script

## Overview

In production environments, Oracle database backups are typically automated to ensure consistency and eliminate the need for manual intervention. Oracle Recovery Manager (RMAN) can be integrated with a shell script and scheduled using **cron** to perform regular backups.

This guide explains how to:

- Create a backup directory
- Develop an RMAN shell script
- Configure RMAN backup operations
- Assign execution permissions
- Schedule automatic backups using Cron

---

# Prerequisites

Before configuring automated RMAN backups, ensure that:

- Oracle Database is installed.
- RMAN is available.
- Oracle environment variables are configured correctly.
- The database is running in **ARCHIVELOG** mode.
- The operating system user has permission to execute RMAN.

---

# Step 1 – Create a Backup Directory

Create a dedicated directory to store:

- RMAN backup pieces
- Backup logs
- Shell scripts

Using a separate directory simplifies backup management and troubleshooting.

```bash
mkdir -p /u01/rman
```

Verify the directory:

```bash
ls -ld /u01/rman
```

---

# Step 2 – Create the RMAN Backup Script

Create a shell script that will execute the RMAN backup.

```bash
vi /u01/rman/full_backup.sh
```

Add the following content:

```bash
#!/bin/bash

export ORACLE_SID=asrblg
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/19/db
export PATH=/usr/sbin:$PATH
export PATH=$ORACLE_HOME/bin:$PATH

rman target / nocatalog log=/u01/rman/RMAN_backup.log <<EOF

RUN
{
    ALLOCATE CHANNEL ch1 DEVICE TYPE DISK;
    ALLOCATE CHANNEL ch2 DEVICE TYPE DISK;
    ALLOCATE CHANNEL ch3 DEVICE TYPE DISK;
    ALLOCATE CHANNEL ch4 DEVICE TYPE DISK;

    SQL 'ALTER SYSTEM ARCHIVE LOG CURRENT';

    CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 5 DAYS;

    CONFIGURE DEVICE TYPE DISK
    BACKUP TYPE TO COMPRESSED BACKUPSET
    PARALLELISM 4;

    CROSSCHECK BACKUP DEVICE TYPE DISK;

    CROSSCHECK ARCHIVELOG ALL;

    DELETE NOPROMPT ARCHIVELOG ALL
    BACKED UP 1 TIMES TO DEVICE TYPE DISK;

    BACKUP
    INCREMENTAL LEVEL 0
    AS COMPRESSED BACKUPSET
    DATABASE
    ARCHIVELOG ALL
    TAG 'WEEKLY_FULL_BACKUP'
    DELETE INPUT;

    DELETE NOPROMPT OBSOLETE;

    DELETE NOPROMPT EXPIRED BACKUP;

    RELEASE CHANNEL ch1;
    RELEASE CHANNEL ch2;
    RELEASE CHANNEL ch3;
    RELEASE CHANNEL ch4;
}

EOF
```

Save and exit the editor.

---

# Step 3 – Understanding the Script

## Oracle Environment Variables

```bash
export ORACLE_SID=asrblg
```

Specifies the Oracle database instance.

```bash
export ORACLE_BASE=/u01/app/oracle
```

Defines the Oracle base directory.

```bash
export ORACLE_HOME=$ORACLE_BASE/product/19/db
```

Specifies the Oracle software installation path.

```bash
export PATH=$ORACLE_HOME/bin:$PATH
```

Adds Oracle binaries to the system PATH.

---

## RMAN Connection

```bash
rman target / nocatalog
```

Connects RMAN to the target database without using a recovery catalog.

The backup output is written to:

```
/u01/rman/RMAN_backup.log
```

---

# Step 4 – RMAN Backup Operations

The script performs the following operations:

### Allocate Channels

```rman
ALLOCATE CHANNEL ch1 DEVICE TYPE DISK;
```

Multiple channels enable parallel backup operations, improving performance.

---

### Force Archive Log Switch

```rman
SQL 'ALTER SYSTEM ARCHIVE LOG CURRENT';
```

Archives the current online redo log before the backup begins.

---

### Configure Retention Policy

```rman
CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 5 DAYS;
```

Retains backups required to recover the database within the last five days.

---

### Configure Backup Type

```rman
CONFIGURE DEVICE TYPE DISK
BACKUP TYPE TO COMPRESSED BACKUPSET
PARALLELISM 4;
```

This configuration:

- Uses compressed backup sets.
- Creates four parallel backup streams.

---

### Crosscheck Existing Backups

```rman
CROSSCHECK BACKUP DEVICE TYPE DISK;
```

Verifies that backup pieces recorded in the RMAN repository still exist.

---

### Crosscheck Archive Logs

```rman
CROSSCHECK ARCHIVELOG ALL;
```

Validates archived redo log records.

---

### Delete Archived Logs

```rman
DELETE NOPROMPT ARCHIVELOG ALL
BACKED UP 1 TIMES TO DEVICE TYPE DISK;
```

Deletes archive logs that have already been backed up at least once.

---

### Perform Database Backup

```rman
BACKUP
INCREMENTAL LEVEL 0
AS COMPRESSED BACKUPSET
DATABASE
ARCHIVELOG ALL
TAG 'WEEKLY_FULL_BACKUP'
DELETE INPUT;
```

This command:

- Performs a Level 0 incremental (full) backup.
- Includes archived redo logs.
- Compresses backup sets.
- Tags the backup.
- Removes archived logs after successful backup.

---

### Delete Obsolete Backups

```rman
DELETE NOPROMPT OBSOLETE;
```

Removes backups no longer required by the retention policy.

---

### Delete Expired Backups

```rman
DELETE NOPROMPT EXPIRED BACKUP;
```

Removes metadata for missing or unavailable backups.

---

### Release Channels

```rman
RELEASE CHANNEL ch1;
```

Frees the allocated RMAN channels after the backup completes.

---

# Step 5 – Assign Execute Permission

Grant execute permission to the shell script.

```bash
chmod 775 /u01/rman/full_backup.sh
```

Verify the permissions.

```bash
ls -l /u01/rman/full_backup.sh
```

---

# Step 6 – Test the Script

Run the script manually to verify that it executes successfully.

```bash
/u01/rman/full_backup.sh
```

Monitor the RMAN log.

```bash
tail -f /u01/rman/RMAN_backup.log
```

---

# Step 7 – Schedule the Backup Using Cron

Edit the crontab.

```bash
crontab -e
```

Schedule the backup to run every day at **3:00 AM** and **11:00 PM**.

```cron
00 03,23 * * * /u01/rman/full_backup.sh
```

### Cron Schedule Breakdown

| Field | Value | Description |
|-------|------|-------------|
| Minute | 00 | At the beginning of the hour |
| Hour | 03,23 | 3:00 AM and 11:00 PM |
| Day | * | Every day |
| Month | * | Every month |
| Weekday | * | Every day of the week |

---

# Verify Cron Jobs

Display scheduled cron jobs.

```bash
crontab -l
```

---

# Verify RMAN Backups

Connect to RMAN.

```bash
rman target /
```

List all backups.

```rman
LIST BACKUP;
```

Display backup summary.

```rman
LIST BACKUP SUMMARY;
```

---

# Troubleshooting

| Issue | Possible Cause | Solution |
|--------|----------------|----------|
| RMAN not found | Incorrect ORACLE_HOME | Verify environment variables |
| Backup failed | Database unavailable | Ensure the database is running |
| Cron job not executing | Cron service stopped | Check cron service status |
| Archive logs not deleted | Backup incomplete | Verify RMAN backup completion |
| Permission denied | Script not executable | Run `chmod 775` on the script |


