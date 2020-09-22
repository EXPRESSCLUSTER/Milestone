# Milestone XProtect VMS Products 2020 R2 on Windows EXPRESSCLUSTER X Quick Start Guide
This article shows how to setup SQL Server 2019 Cluster with EXPRESSCLUSTER X Mirror Disk configuratoin.

## Reference

### EXPRESSCLUSTER X
- https://www.nec.com/en/global/prod/expresscluster/en/support/manuals.html

### EXPRESSCLUSTER X HA/DR solution with Milestone Xprotect
- https://www.milestonesys.com/marketplace/nec/nec-expresscluster-x-ha--dr-solution/


## System configuration
- Servers: 2 node with Mirror Disk
- OS: Windows Server 2019
- SW:
	- SQL Server 2019 Standard
	- Milestone Xprotect2020 R2
	- EXPRESSCLUSTER X 4.0/4.1/4.2

```bat
<LAN>
 |
 |  +----------------------------+
 +--| Primary Server             |
 |  | - Windows Server 2019      |
 |  | - SQL Server 2019          |
 |  | - EXPRESSCLUSTER X 4       |
 |  +----------------------------+
 |                                
 |  +----------------------------+
 +--| Secondary Server           |
 |  | - Windows Server 2019      |
 |  | - SQL Server 2019          |
 |  | - EXPRESSCLUSTER X 4       |
 |  +----------------------------+
 |
```

## EXPRESSCLUSTER X Setup
This procedure shows how to setup SQL Server cluster by mirroring both SQL Server master database and user database.

### EXPRESSCLUSTER X Setup a basic cluster
Please refer [Basic Cluster Setup](https://github.com/EXPRESSCLUSTER/BasicCluster/blob/master/X41/Win/2nodesMirror.md)

### Install SQL Server

#### On Primary Server and Secondary Server
1. Start SQL Server Installer and select as follows:
	- Installation  
		Select "New SQL Server stand-alone installation or add features to an existing installaion"
	- Microsift Update  
		Default or as you like
	- Product Updates  
		Default or as you like
	- Product Key  
		Enter license key
	- License Terms  
		Accept
	- Feature Selection
		- Database Engine Service: Check
		- Shared Features: As you like
			- **Note** We recommend to install SQL Client Connectivity SDK to enable sql command for maintenance.
	- Instance Configuration  
		Default or as you like
	- Server Configuration
		- Service Accounts
			- SQL Server Agent:	Manual
			- SQL Server Database Engine:	Manual
			- SQL Server Browser:	As you like
	- Database Engine Configuration
		- Server Coonfiguration
			- As you like
				- **Note** We recommend to set SA Acount with Mixed Mode or add Domain Account for Windows authentication because the database should be accessible from both Primary and Secondary Servers.
		- Data Directories
            - C:\Program Files\Microsoft SQL Server\
			- User database directory:	C:\Program Files\Microsoft SQL Server\MSSQL15.TEST\MSSQL\Data
			- User database log directory:	C:\Program Files\Microsoft SQL Server\MSSQL15.TEST\MSSQL\Data
			- Backup directory:	C:\Program Files\Microsoft SQL Server\MSSQL15.TEST\MSSQL\Backup
	- Ready to install  
		Install

### Milestone Mileston Xprotect2020 R2 Installation on Windows
- https://doc.milestonesys.com/2020r2/en-US/standard_features/sf_mc/sf_installation/mc_installthesystem.htm?TocPath=XProtect%20VMS%20products%7CXProtect%20VMS%20administrator%20manual%7CInstallation%7C_____1#InstallyoursystemCustomoption 

#### Data Directories configuration On Primary
1. Confirm that the failover group is active on the server
1. Create a folder on Mirror Disk  
   
	```bat
	e.g.) E:\SQL
	```

2. Start SQL Server Configuration Manager
3. Select [SQL Server Services] at the left tree
4. Right click [SQL Server (<instance name>)] and select [Properties]
5. Goto [Setup Parameters] tab and edit existing parameters as follow:
	- Before:
		- -dC:\Program Files\Microsoft SQL Server\MSSQL15.TEST\MSSQL\DATA\master.md
		- -lC:\Program Files\Microsoft SQL Server\MSSQL15.TEST\MSSQL\DATA\mastlog.ld
	- After:
		- -dE:\SQL\MSSQL15.TEST\MSSQL\DATA\master.md
		- -lE:\SQL\MSSQL15.TEST\MSSQL\DATA\mastlog.ld

1. Check SQL Server is installed normally.
	1. Start Windows Service Manager and start SQL Server service.
	1. Confirm that SQL Server service status becomes running.
	1. Stop SQL Server service

#### Data Directories configuration On Secondary Server
1. Confirm that the failover group is active on the server.
1. Start SQL Server Configuration Manager.
1. Select [SQL Server Services] at the left tree.
1. Right click [SQL Server (<instance name>)] and select [Properties]
1. Goto [Setup Parameters] tab and edit existing parameters as follow:
	- Before:
		- -dC:\Program Files\Microsoft SQL Server\MSSQL15.TEST\MSSQL\DATA\master.md
		- -lC:\Program Files\Microsoft SQL Server\MSSQL15.TEST\MSSQL\DATA\mastlog.ld
	- After:
		- -dE:\SQL\MSSQL15.TEST\MSSQL\DATA\master.md
		- -lE:\SQL\MSSQL15.TEST\MSSQL\DATA\mastlog.ld
1. Check SQL Server is installed normally.
	1. Start Windows Service Manager and start SQL Server service.
	1. Confirm that SQL Server service status becomes running.
	1. Stop SQL Server service. 
 
## Set the DB and application Services to Manual 
After the SQL Server setup has completed on both servers, set all of the SQL Server and Milestone Services to manual, and make sure that they should stop.

## SQL and Milestone Services Setup in Cluster
1. Right click on failover and click Add Resource in builder window.
1. Choose service resource.
1. Type a service name to the resource (Ex: MSSQLSERVER) and add optional comments if required.
1. Click Next.
1. Click on Connect and select the service MSSQLSERVER from the drop down.
1. Click OK.
1. lick Next (for default values) to learn more about parameters please refer the Express Cluster Reference Guide. Click Next.
1. Click Finish.
1. ight click on failover and click Add Resource in builder window.
1. Choose service resource.
1. Type a service name to the resource (Ex: Milestone Data service) and add optional comments if required.
1. Click Next.
1. Click on Connect and select the service Milestone XProtect Data Collector Server from the drop down.
1. Click OK.
1. Click Next (for default values) to learn more about parameters please refer the Express Cluster Reference Guide. Click Next.
1. Click Finish.
1. Right click on failover and click Add Resource in builder window.
1. Choose service resource.
1. Type a service name to the resource (Ex: Milestone XProtect Management Server) and add optional comments if required.
1. Click Next.
1. Click on Connect and select the service Milestone XProtect Management Server from the drop down.
1. Click OK.
1. Click Next (for default values) to learn more about parameters please refer the Express Cluster Reference Guide. Click Next.
1. Click Finish.
1. Right click on failover and click Add Resource in builder window.
1. Choose service resource.
1. Type a service name to the resource (Ex: Milestone XProtect Event Server) and add optional comments if required.
1. Click Next.
1. Click on Connect and select the service Milestone XProtect Event Server from the drop down.
1. Click OK.
1. Click Next (for default values) to learn more about parameters please refer the Express Cluster Reference Guide. Click Next.
1. Click Finish.
1. Select File and then Upload the Configuration File.
1. Click OK, then Navigate back to Cluster WebUI and select Start Cluster.

#### Configure Milestone Database in Mirror drive
1. Using SQL management studio.
1. Right click the milestone database name
    - tasks
      - detach and click OK.on the database detach window.
1. Right click databases
    - attach
      - add and point to the mdf file. It will automatically take the ldf file to the attach databases window.

### Check Milestone Cluster
#### On Primary Server
1. Confirm that the failover group is active normally on the server
1. Connect to SQL Server
   	```bat
	> sqlcmd -S localhost -U <username> -P <password>
	```
2. Create a test database and table and inser a value to it
	```bat
	1> create database testdb
	2> go
	1> use testdb
	2> go
	Changed database context to 'testdb'.
	1> create table testtb(
	2>  id int,
	3>  name varchar(20)
	4> );
	5> go
	1> insert into testtb (id, name) values(0, "Milestone");
	2> go
	```
1. Confirm the value is inserted
	```bat
	1> select * from testtb
	2> go
	id          name
	----------- --------------------
          0 Milestone

	(1 rows affected)
	```
1. Exit from the database
	```bat
	1> quit
1. Move the failover group to Secondary Server
	```

#### On Secondary Server
1. Confirm that the failover group is active normally on the server
1. Connect to SQL Server
	```bat
	> sqlcmd -S localhost -U SA -P <password>
	```
1. Confirm that the database, table and its value is replicated
	```bat
	1> use testdb
	2> go
	Changed database context to 'testdb'.
	1> select * from testtb
	2> go
	id          name
	----------- --------------------
	          0 Milestone
	
	(1 rows affected)
	```
1. Exit from the database
	```bat
	1> quit
	```
1. Move the failover group to the Primary Sarver.
