# Oracle ASM Background Processes - Quick Study Guide

## What is Oracle ASM?

Oracle Automatic Storage Management (ASM) is Oracle's integrated storage management solution. It manages Oracle database files by organizing physical disks into **Disk Groups** and automatically handling:

* File placement
* Striping
* Mirroring
* Disk management
* Online storage expansion
* Automatic rebalancing

Unlike a database instance, an ASM instance **does not store application data**. It stores **metadata** and manages storage.

---

# ASM Architecture

```text
Database Instance(s)
        │
     ASMB Process
        │
   ASM Instance
        │
   Disk Groups
        │
 Physical Disks
```

---

# Why ASM Uses Background Processes

Oracle ASM divides storage management into specialized background processes to improve:

* Performance
* Parallelism
* Availability
* Automatic recovery
* Scalability

Some processes run continuously, while others start only when needed.

---

# Major ASM Background Processes

| Process  | Purpose                                      | Always Running |
| -------- | -------------------------------------------- | -------------- |
| **RBAL** | Coordinates rebalance operations             | Yes            |
| **ARBx** | Performs rebalance work                      | No (On-demand) |
| **ASMB** | Communication between Database and ASM       | Yes            |
| **GMON** | Monitors and manages Disk Groups             | Yes            |
| **PMON** | Cleans failed processes                      | Yes            |
| **SMON** | Performs instance recovery                   | Yes            |
| **CKPT** | Updates checkpoint information               | Yes            |
| **DBWR** | Writes dirty buffers to disk                 | Yes            |
| **LGWR** | Writes redo entries                          | Yes            |
| **VKTM** | Maintains system time                        | Yes            |
| **PSP0** | Spawns Oracle background processes           | Yes            |
| **PING** | Synchronizes Cluster/RAC communication       | RAC            |
| **MARK** | Coordinates advanced ASM metadata operations | Internal       |
| **KATE** | Internal ASM maintenance tasks               | Internal       |
| **PZ9x** | Parallel server/RAC helper processes         | RAC            |

---

# Important Processes

## RBAL (Rebalance Coordinator)

* Detects Disk Group changes.
* Decides whether rebalancing is required.
* Starts ARBx worker processes.
* Monitors rebalance progress.

> RBAL **coordinates** the operation but **does not move data**.

---

## ARBx (Rebalance Workers)

* Moves extents between disks.
* Redistributes data after:

  * Adding disks
  * Dropping disks
  * Replacing disks
  * Resizing disks

Example:

```sql
ALTER DISKGROUP DATA REBALANCE POWER 8;
```

Higher POWER → Faster rebalance → More CPU & I/O usage.

---

## ASMB

The bridge between the **Database Instance** and **ASM Instance**.

Responsibilities:

* Requests file locations
* Retrieves ASM metadata
* Coordinates storage operations

Without ASMB, the database cannot access ASM-managed files.

---

## GMON

Monitors ASM Disk Groups by:

* Detecting disk failures
* Updating metadata
* Monitoring mount status
* Coordinating recovery

---

## PMON

* Cleans abandoned resources
* Removes failed sessions
* Registers the instance with listeners

---

## SMON

Responsible for:

* Instance recovery
* Temporary segment cleanup
* Space maintenance

---

## CKPT

Updates checkpoint information in:

* Control files
* Datafile headers

Works closely with DBWR.

---

## DBWR

Writes modified database blocks from memory to datafiles.

---

## LGWR

Writes redo information from the Redo Log Buffer to Online Redo Logs, ensuring transaction durability.

---

## VKTM

Maintains high-resolution system time used by Oracle scheduling and timing mechanisms.

---

## PSP0

Creates and manages Oracle background processes during instance startup.

---

# Rebalance Workflow

```mermaid
flowchart LR

A[Disk Added/Dropped]
-->B[RBAL Detects Change]
-->C[Starts ARBx]
-->D[ARBx Moves Extents]
-->E[GMON Updates Metadata]
-->F[Rebalance Complete]
```

---

# Useful Dynamic Performance Views

| View            | Purpose                    |
| --------------- | -------------------------- |
| V$ASM_DISKGROUP | Disk Group information     |
| V$ASM_DISK      | ASM Disk status            |
| V$ASM_OPERATION | Rebalance progress         |
| V$BGPROCESS     | Background processes       |
| V$SESSION       | Active background sessions |

---

# Common SQL Queries

### Disk Groups

```sql
SELECT name, state, total_mb, free_mb
FROM V$ASM_DISKGROUP;
```

### ASM Disks

```sql
SELECT name, header_status, state
FROM V$ASM_DISK;
```

### Rebalance Progress

```sql
SELECT *
FROM V$ASM_OPERATION;
```

### Background Processes

```sql
SELECT pname, description
FROM V$BGPROCESS;
```

---

# Linux Commands

List ASM processes:

```bash
ps -ef | grep asm
```

Check RBAL:

```bash
ps -ef | grep rbal
```

Check ARBx:

```bash
ps -ef | grep arb
```

---

# Common Rebalance Triggers

* Add Disk
* Drop Disk
* Resize Disk
* Replace Failed Disk
* Change Rebalance Power

---

# Troubleshooting

| Problem             | Possible Cause                      |
| ------------------- | ----------------------------------- |
| Slow rebalance      | Low POWER, heavy I/O                |
| Disk not mounting   | Disk failure or metadata issue      |
| ASM startup failure | Missing disks or corrupted metadata |
| No ARBx process     | No rebalance in progress            |

---

| ASM Instance         | Database Instance       |
| -------------------- | ----------------------- |
| Manages storage      | Manages user data       |
| Stores metadata      | Stores application data |
| Manages Disk Groups  | Executes SQL            |
| Performs rebalancing | Processes transactions  |

---

# Quick Revision

* **RBAL** → Rebalance Coordinator
* **ARBx** → Rebalance Worker
* **ASMB** → Database ↔ ASM Communication
* **GMON** → Disk Group Monitor
* **PMON** → Process Cleanup
* **SMON** → Instance Recovery
* **CKPT** → Checkpoint Updates
* **DBWR** → Writes Data Blocks
* **LGWR** → Writes Redo Logs
* **VKTM** → Time Management
* **PSP0** → Process Spawner
* **PING / PZ9x** → RAC Communication
* **MARK / KATE** → Internal ASM Maintenance

---
