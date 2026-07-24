# Oracle 19c Multitenant Database Installation Guide

## Overview

This document describes the step-by-step process for creating an Oracle 19c Multitenant Database using the Database Configuration Assistant (DBCA). Follow each step in sequence to successfully create a Container Database (CDB) with a Pluggable Database (PDB).

---

# Step 1: Launch Database Configuration Assistant (DBCA)
[oracle@localhost ~]$ 
[oracle@localhost ~]$ dbca
After that we follow GUI prompt step by step:
<img width="1029" height="506" alt="image" src="https://github.com/user-attachments/assets/6d0baf24-4ccd-44e5-bc08-41a12d2171ca" />


### Description
- Open the Oracle Database Configuration Assistant (DBCA).
- Select **Create Database**.
- Click **Next**.

### Purpose
Starts the Oracle database creation wizard.

### Expected Result
The database creation options page is displayed.

---

# Step 2: Select Database Creation Mode

Click the advance configuration option.
<img width="1027" height="481" alt="image" src="https://github.com/user-attachments/assets/3c708ce7-1697-473a-abe5-eeffe72df61f" />


### Description
- Choose **Advanced Configuration**.
- Click **Next**.

### Purpose
Allows customization of the database configuration.

### Expected Result
The deployment type selection page appears.

---

# Step 3: Select Deployment Type
General purpose or transaction processing.
<img width="1018" height="455" alt="image" src="https://github.com/user-attachments/assets/3c7d0013-ae7d-48dd-8c52-73ade35d4ad9" />


### Description
- Select **Oracle Single Instance Database**.
- Click **Next**.

### Purpose
Creates a standalone Oracle database.

### Expected Result
The database type selection page opens.

---

# Step 4: Select Database Type
Here have to check create as a container database, pdb name also select as pdb.
<img width="1021" height="510" alt="image" src="https://github.com/user-attachments/assets/3f5d7e36-7ffa-479e-8f1c-edc8a77a23f6" />

### Description
- Select **General Purpose** (or the required template).
- Click **Next**.

### Purpose
Defines the initial database configuration.

### Expected Result
The database identification page appears.

---

# Step 5: Enter Database Identification

<img width="1009" height="528" alt="image" src="https://github.com/user-attachments/assets/5dd5e9d6-4c6f-4ee4-8378-27111cc39932" />


### Description
- Enter the **Global Database Name**.
- Enter the **SID**.
- Enable **Create as Container Database**.

### Purpose
Configures the database name and enables the multitenant architecture.

### Expected Result
The Pluggable Database configuration page appears.

---

# Step 6: Configure Pluggable Database (PDB)

Set the FRA location size.

<img width="1012" height="511" alt="image" src="https://github.com/user-attachments/assets/a4183ff3-9168-43db-b65d-567f8a07a836" />


### Description
- Select **Create a Pluggable Database**.
- Enter the **PDB Name**.
- Specify the administrator password.

### Purpose
Creates the first Pluggable Database inside the Container Database.

### Expected Result
The storage configuration page opens.

---

# Step 7: Configure Storage

<img width="1009" height="524" alt="image" src="https://github.com/user-attachments/assets/2a0057c7-6864-49e3-9395-a9769689ba6b" />


### Description
- Select the database storage location.
- Choose the file system or ASM disk group.
- Configure Fast Recovery Area if required.

### Purpose
Defines where Oracle database files will be stored.

### Expected Result
The initialization parameters page appears.

---

# Step 8: Configure Initialization Parameters

<img width="1007" height="447" alt="image" src="https://github.com/user-attachments/assets/f771ac3b-58d6-4ffa-bbbd-989b30507a65" />


### Description
- Configure memory allocation.
- Set character set.
- Configure process limits if required.

### Purpose
Defines database performance and language settings.

### Expected Result
The management configuration page appears.

---

# Step 9: Configure Administrative Passwords

<img width="1001" height="515" alt="image" src="https://github.com/user-attachments/assets/93d6c9b8-6c73-42af-a705-d92491029078" />


### Description
- Enter passwords for SYS and SYSTEM users.
- Confirm the password.

### Purpose
Creates administrative accounts.

### Expected Result
The summary page is displayed.

---

# Step 10: Review Summary

This is the summary page. Checking the all parameter & size. After all checking clickon the install button.

<img width="999" height="516" alt="image" src="https://github.com/user-attachments/assets/7103606b-9325-4452-951b-a9599659cf49" />


### Description
- Review all selected configuration options.
- Click **Finish** to begin database creation.

### Purpose
Verifies the configuration before installation starts.

### Expected Result
Oracle begins creating the Container Database and Pluggable Database.

---

# Step 11: Database Creation Progress

<img width="996" height="571" alt="image" src="https://github.com/user-attachments/assets/faf79534-3257-43a1-a17c-2f8dfd8cad6d" />


### Description
- Wait while Oracle creates the database.
- Monitor the progress window.

### Purpose
Tracks the status of the database creation process.

### Expected Result
Database creation completes successfully.

---

# Step 12: Database Creation Completed

The container database is created successfully. 

<img width="979" height="213" alt="image" src="https://github.com/user-attachments/assets/d6e1ceb7-c3be-421f-85ca-48d2cf074b31" />


### Description
- Review the completion message.
- Verify that the Container Database (CDB) and Pluggable Database (PDB) were created successfully.

### Purpose
Confirms successful installation of the Oracle Multitenant Database.

### Expected Result

```
The container database is created successfully.
```

---

# Verification

After installation, verify the database using SQL*Plus.

```sql
sqlplus / as sysdba
```

Check the database name:

```sql
SELECT name FROM v$database;
```

Check the container database:

```sql
SHOW CON_NAME;
```

List all pluggable databases:

```sql
SHOW PDBS;
```

Or:

```sql
SELECT name, open_mode
FROM v$pdbs;
```

---

# Conclusion

The Oracle 19c Multitenant Database installation has been completed successfully. The Container Database (CDB) and Pluggable Database (PDB) are now available and ready for administrative tasks and application deployment.
