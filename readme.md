AiCore project on Azure Database Migration (ADM), involving building and implementing a cloud-based database system architecture using Microsoft Azure. This README file contains an overview of the progress made, split into the revelant 'milestones'. The relevant UML diagram can be found as a .png file in this repo alongside the readme.md file, which outlines the project.

Milestone 2 - Setting up the Production Environment
- A Windows 10 Pro VM called 'adm-vm' was provisioned under the 'ADM' resource group in Azure, with network settings configured to allow RDP traffic. This VM is the primary machine for the production environment.
- A connection to the VM was established using RDP. Database infrastructure tools were installed, namely SQL Server and SSMS.
- The local 'AdvetureWorks' database was restored from the backup file on the VM, ready for migration.

Milestone 3 - Azure SQL Database Migration
- A production database called 'adm-db' was created in Azure using SQL authentication credentials.
- Azure Data Studio was installed on the production VM and a successful connection was established to 'adm-db'.
- The SQL Server Schema Compare extension was utilised within Azure Data Studio to compare and migrate the schema of the restored local 'AdventureWorks' database to the newly created Azure 'adm-db' database. No migration issues were encountered according to the Azure Data Studio automated checks.
- The Azure SQL Migration extension was utilised to migrate the data from the local database to 'adm-db'. This involved creating an Azure Database Migration Service under the 'ADM' resource group, as well as installing the integration runtime to facilitate the data migration.
- The data migration proved successful as identified by the migration extension. The local database and 'adm-db' tables were compared to ensure all have been migrated successfully. The entry counts in all tables were also compared to ensure that all tables in the new database contained the same amount of data as in the original database. 

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
- The SQL server 'Restore' feature was used to deploy a restored 'point-in-time' database version (named 'adm-db-restored') that was created prior to the data loss.
- After connecting to this DB in Azure Data Studio, the database was queried to ensure restoration success. The 'Person.EmailAddress' table contained 19972 rows and the top 100 entries of the 'PasswordHash' column in the 'Person.Password' table were password hashes, not '1' values. This validates that the data restoration was successful.
- The original 'adm-db' database was deleted, since it lacks critical data. The 'adm-db-restored' database is now the primary production database.

Milestone 6 - Geo-Replication and Failover
- The 'adm-db-restored' production database was geo-replicated and hosted on the 'adm-replica' SQL server (US East location). This was confirmed on Azure portal.
- A failover group for the primary database was created. A manual failover was initiated and proved successful, making the 'adm-replica' server (US East) the primary and the 'adm-server' (UK South) the secondary. Refreshing the database on Azure Data Studio ensured a successful connection. Running queries on the data ensured that all data was present and no data loss occurred.
- Another failover procedure was initiated to act as the tailback, transitioning operations back to the primary copy. This proved successful, with the 'adm-replica' being set back as the secondary, and 'adm-server' as the primary. Refreshing the connection and querying the data both proved successful - no data loss was identified.

Milestone 7 - Configuring Entra ID
- Entra ID was enabled and an Entra ID admin was assigned for the 'adm-server' primary server using the Azure portal. A new connection was successfully established to this server in Azure Data Studio using the Entra ID credentials.
- An Entra ID user was created (named 'db_reader_nr') using the Azure portal. In Azure Data Studio, a query was executed to grant "db_datareader" permissions to this 'db_reader_nr' user.
- A new connection to the production database was established using the "db_reader_nr" user. To test this user's privileges, a SELECT query was run on one of the tables and proved successful. However, running a DELETE query on the same table proved unsuccessful, returning the error message: "The DELETE permission was denied on the object 'EmailAddress', database 'ADM-DB-RESTORED', schema 'Person'". This shows that the user indeed has only read-access and cannot perform changes to the data.
