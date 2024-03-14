
Milestone 4 - Data Backup and Restore:
- Full production database backup of AdventureWorks was created using SSMS and stored locally as a .bak file on the ADM-VM virtual machine at the default location.
- An Azure Blob Storage account was created and named 'admblob'. The .bak file was uploaded into the 'adm-prod-backup' container for redundancy.
- Windows VM provisioned ('adm-vm-development') to host the development environment, with SQL Server and SSMS installed.
- Backup manually restored to the development VM from 'admblob'.
- A maintenance plan was created in SSMS using SQL Server Agent to schedule weekly backups of the AdventureWorks database to the Blob Storage Account. A dedicated development
storage container was created to store these backups (named 'adm-dev-backup'). The plan was created by generating an SQL Server Credential using an T-SQL query to hold
the blob details, namely the access key. This SQL Credential was then used to create and execute the maintenance plan, which has been scheduled to backup at weekly
intervals on Wednesdays at 5:05 PM. Execution was verified on the Azure portal by checking the dedicated 'adm-dev-backup' blob container where the .bak files are stored.

Milestone 5 - Disaster Recovery Simulation
- To simulate a disaster recovery procedure, data from two tables in the AdventureWorks database were intentionally corrupted. Before corruption, the 'Person.EmailAddress'
table contained 19972 rows. The top 100 entries of this table were removed, leaving the table with 19872 rows. The top 100 entries of the 'Person.Password' table had
the 'PasswordHash' column entries set to a value of '1' to simulate corruption.
- The SQL server 'Restore' feature was used to deploy a restored 'point-in-time' database version that was created prior to the data loss.
