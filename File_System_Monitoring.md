# Oracle DBA - File System and Storage Monitoring

## Overview

File system monitoring is one of the routine responsibilities of an Oracle DBA. Regular monitoring helps prevent disk space issues that can affect database performance and availability. This document explains how to monitor file systems, Oracle directories, trace files, Fast Recovery Area (FRA), alert logs, and tablespaces.

---

# 1. Monitor the File System

To check all file systems at the operating system level, use the following command:

```bash
df -h
```

### Purpose

The `df -h` command displays:

- Total file system size
- Used space
- Available space
- Percentage of space utilized
- Mount point

### Example Output

```bash
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3        50G   22G   28G  45% /
/dev/sda2       100G   65G   35G  65% /u01
```
<img width="975" height="198" alt="image" src="https://github.com/user-attachments/assets/94592091-723a-4b95-bd06-68104b022879" />

### Note

If the root (`/`) file system reaches a high utilization percentage, monitor it proactively. Remove unnecessary files if possible to maintain sufficient free space and avoid storage-related issues.

---

# 2. Check Oracle Directory Disk Usage

After checking the overall file system usage, navigate to the Oracle directories to identify which directories are consuming more space.

```bash
cd /u01/app/oracle

du -sh *
```

### Purpose

The `du -sh *` command displays the size of each directory in a human-readable format.

### Sample Output

```bash
4.5G admin
25G oradata
1.2G diag
8G FRA
```

This helps identify directories that require cleanup or further investigation.

---

# 3. Monitor Trace Files

Oracle generates trace files for background processes, user sessions, and error diagnostics.

Navigate to the trace directory:

```bash
cd $ORACLE_BASE/diag/rdbms/<DB_NAME>/<INSTANCE_NAME>/trace
```

Check the trace file sizes:

```bash
du -sh *
```

or

```bash
ls -lh
```

Large or old trace files can consume significant disk space over time.

---

# 4. Delete Old Trace Files

Delete trace files older than two days using:
<img width="975" height="240" alt="image" src="https://github.com/user-attachments/assets/c5377fae-c22b-475b-b031-ed80b25e0f4b" />

```bash
find . -name "*.trc" -mtime +2 -exec rm -f {} \;
```

### Explanation

- `find` → Searches for files
- `-name "*.trc"` → Finds trace files
- `-mtime +2` → Files older than 2 days
- `-exec rm -f {} \;` → Deletes the matching files

### Verify

Check the directory size again:
<img width="975" height="233" alt="image" src="https://github.com/user-attachments/assets/5b92553f-39e6-46e3-83df-4338425bd668" />

```bash
du -sh .
```

or

```bash
ls -lh
```

> **Best Practice:** Delete only old trace files after confirming they are no longer required for troubleshooting.

---

# 5. Monitor Fast Recovery Area (FRA)

The Fast Recovery Area stores:

- Archived redo logs
- Flashback logs
- RMAN backups
- Control file autobackups

Monitoring FRA usage helps prevent database issues caused by insufficient recovery space.

## Check FRA Configuration

```sql
SHOW PARAMETER db_recovery_file_dest;
```
<img width="975" height="152" alt="image" src="https://github.com/user-attachments/assets/c4255c96-6c9a-4f7f-a169-e8a298fc3339" />

## Check FRA Usage

```sql
SELECT
    NAME,
    SPACE_LIMIT,
    SPACE_USED,
    SPACE_RECLAIMABLE,
    NUMBER_OF_FILES
FROM
    V$RECOVERY_FILE_DEST;
```

### Check Recovery Area Information
<img width="975" height="90" alt="image" src="https://github.com/user-attachments/assets/ba1c71ab-88fe-4273-a094-b7de1a22ef0e" />

```sql
SELECT *
FROM V$RECOVERY_FILE_DEST;
```

---

# 6. Locate the Alert Log

The alert log records important database events, startup, shutdown, errors, and warnings.

Find the alert log location using:

```sql
SELECT *
FROM V$DIAG_INFO;
```

Look for the row named:
<img width="975" height="279" alt="image" src="https://github.com/user-attachments/assets/2b7eb499-b989-4c55-9bbe-e878fc222f00" />

```
Diag Trace
```

or

```
Alert Log
```

This displays the complete path of the alert log file.

---

# 7. Monitor Tablespace Usage

To monitor current tablespace utilization, use the following query:

```sql
SELECT
    tablespace_name,
    used_space,
    tablespace_size,
    ROUND((used_space/tablespace_size)*100,2) AS used_percent
FROM
    dba_tablespace_usage_metrics;
```
<img width="975" height="136" alt="image" src="https://github.com/user-attachments/assets/b919f638-58c5-43f2-ac62-d1f15445af59" />

### Purpose

This view provides:

- Tablespace name
- Used space
- Total allocated space
- Percentage of utilization

Monitor tablespaces regularly to prevent them from reaching full capacity.

---

# 8. Check Datafile Details

Use the following query to view datafile information:

```sql
SELECT
    file_id,
    tablespace_name,
    file_name,
    ROUND(bytes/1024/1024,2) AS size_mb,
    ROUND(maxbytes/1024/1024/1024,2) AS max_size_gb,
    blocks
FROM
    dba_data_files
ORDER BY
    file_id;
```
<img width="975" height="143" alt="image" src="https://github.com/user-attachments/assets/64ad0519-1b8d-4066-9507-cd519dfedeaf" />

### Purpose

This query displays:

- File ID
- Tablespace name
- Datafile location
- Current size
- Maximum autoextend size
- Number of blocks

It is useful for capacity planning and monitoring datafile growth.

---

# Best Practices

- Monitor file system usage regularly using `df -h`.
- Check Oracle directory sizes using `du -sh *`.
- Remove obsolete trace files periodically.
- Monitor FRA usage to prevent archive log issues.
- Review the alert log daily for warnings and errors.
- Monitor tablespace utilization and extend tablespaces before they become full.
- Verify datafile growth and autoextend settings regularly.

---

