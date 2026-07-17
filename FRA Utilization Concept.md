# Oracle FRA (Fast Recovery Area) - Monitoring and Verification Guide

## Overview

The **Fast Recovery Area (FRA)** is a centralized storage location managed by Oracle Database to store recovery-related files. It simplifies backup and recovery by automatically managing the space allocated for archived redo logs, flashback logs, RMAN backups, control file autobackups, and other recovery files.

---

# Check the FRA Location

To identify the configured Fast Recovery Area location, query the Oracle initialization parameters.

```sql
SHOW PARAMETER db_recovery_file_dest;
```

or

```sql
SHOW PARAMETER reco;
```

### Sample Output

<img width="1059" height="335" alt="image" src="https://github.com/user-attachments/assets/fe1454e4-cecb-4f0d-955c-9fd5ffcdb00f" />

---

# Check the FRA Size

To view the maximum space allocated for the Fast Recovery Area:

```sql
SHOW PARAMETER db_recovery_file_dest_size;
```

Example Output:

```
db_recovery_file_dest_size = 12732M
```
<img width="979" height="110" alt="image" src="https://github.com/user-attachments/assets/c14720f5-3316-4870-aa24-659407d6185f" />

---

# Check FRA Utilization (Detailed Report)

The following query displays the FRA location, total allocated space, used space, available space, and percentage of utilization.

```sql
SELECT
    NAME,
    ROUND(SPACE_LIMIT / 1048576 / 1024) AS SPACE_LIMIT_GB,
    ROUND(SPACE_USED / 1048576 / 1024) AS SPACE_USED_GB,
    ROUND(SPACE_LIMIT / 1048576 / 1024) -
    ROUND(SPACE_USED / 1048576 / 1024) AS SPACE_AVAILABLE_GB,
    CEIL(((SPACE_USED / 1048576) * 100) /
    (SPACE_LIMIT / 1048576)) AS PERCENT_USED
FROM V$RECOVERY_FILE_DEST;
```

### Output Description
<img width="1038" height="170" alt="image" src="https://github.com/user-attachments/assets/1ae23a1d-3f96-473c-ae38-8f4561db5586" />

---

# Check FRA Space in GB

For a simplified view showing only total and used space:

```sql
SELECT
    SPACE_USED / 1024 / 1024 / 1024 AS "SPACE_USED(GB)",
    SPACE_LIMIT / 1024 / 1024 / 1024 AS "SPACE_LIMIT(GB)"
FROM V$RECOVERY_FILE_DEST;
```

### Example Output
<img width="1064" height="82" alt="image" src="https://github.com/user-attachments/assets/53a7ccd2-b1b9-48f8-8c78-3a77a350f71d" />

---

# Check FRA Utilization in MB

If you prefer the output in megabytes, use the following query:

```sql
SELECT
    NAME,
    CEIL(SPACE_LIMIT / 1024 / 1024) AS SIZE_MB,
    CEIL(SPACE_USED / 1024 / 1024) AS USED_MB,
    DECODE(
        NVL(SPACE_USED, 0),
        0,
        0,
        CEIL((SPACE_USED / SPACE_LIMIT) * 100)
    ) AS PCT_USED
FROM V$RECOVERY_FILE_DEST;
```

### Output Description
<img width="1041" height="164" alt="image" src="https://github.com/user-attachments/assets/bf67c92f-5ace-48bb-8941-a8bad58ba5a0" />

---

# Check Recovery File Types Stored in FRA

To see how the FRA space is being utilized by different recovery file types:

```sql
SELECT
    FILE_TYPE,
    PERCENT_SPACE_USED,
    PERCENT_SPACE_RECLAIMABLE,
    NUMBER_OF_FILES
FROM V$FLASH_RECOVERY_AREA_USAGE;
```

Typical file types include:

- Archived Redo Logs
- Backup Pieces
- Image Copies
- Flashback Logs
- Control Files
- Foreign Archived Logs

---

# Verify Archive Log Destination

If archive logging uses the FRA, verify the destination with:

```sql
ARCHIVE LOG LIST;
```

or

```sql
SHOW PARAMETER log_archive_dest;
```

---

# Useful Dynamic Performance Views

| View | Purpose |
|------|----------|
| V$RECOVERY_FILE_DEST | FRA size and utilization |
| V$FLASH_RECOVERY_AREA_USAGE | Recovery file usage by file type |
| V$ARCHIVED_LOG | Archived redo log information |
| V$RMAN_BACKUP_JOB_DETAILS | RMAN backup history |

---

# Troubleshooting

| Problem | Possible Cause | Solution |
|----------|----------------|----------|
| FRA Full | Archived logs not deleted | Delete obsolete backups or increase FRA size |
| ORA-19809 | Recovery area exceeded | Increase `db_recovery_file_dest_size` |
| ORA-19815 | FRA almost full | Remove unnecessary recovery files |
| Backup Failure | Insufficient FRA space | Free space or expand the FRA |

---

# Quick Revision

| Task | Command |
|------|---------|
| Check FRA Location | `SHOW PARAMETER reco;` |
| Check FRA Size | `SHOW PARAMETER db_recovery_file_dest_size;` |
| Check FRA Utilization | `SELECT * FROM V$RECOVERY_FILE_DEST;` |
| Check Recovery File Usage | `SELECT * FROM V$FLASH_RECOVERY_AREA_USAGE;` |
| Check Archive Destination | `ARCHIVE LOG LIST;` |

---
