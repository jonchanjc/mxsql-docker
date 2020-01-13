# Description

To run latest MSSQL 2017 in Linux container using Docker Compose for fast local development and testing on Windows Machine.

# Requirements

1. Latest Docker Toolbox (Win 7) or Docker Desktop (Win 10)
2. Enable Virtualization in BIOS
3. At least 2GB of Ram in your Docker VM

# To start the SQL instance

```
cd .\mxsql-docker
docker-compose up -d
```
Once the container is up you can connect using the information below:

```
host: <Your docker machine ip>
port: 1403
login: sa
pass: big64&Mac
```

# To restore a database to the instance

1. Copy the DB backup file to the container

```docker
docker cp <DB backup file> mxsql:/var/opt/mssql/backups/<DB backup file>
```

2. Launch SSMS and open up a new query window

3. Check the datafile and logfile location of the DB before restore

```SQL
RESTORE FILELISTONLY 
FROM DISK='/var/opt/mssql/backups/<DB backup file>'
GO
```

4. Restore the DB

```SQL
RESTORE DATABASE <DBName>
FROM DISK='/var/opt/mssql/backups/<DB backup file>'
WITH 
MOVE '<DataFile Logical Name>' TO '/var/opt/mssql/data/<Datafile Physical Name>'
,MOVE '<LogFile Logical Name>' TO '/var/opt/mssql/data/<Logfile Physical Name>'
,STATS=5
, REPLACE -- use if overwriting existing DB
```

# Data Persistence

Data is presisted in 2 container volumes to preserve the changes in the sqldata volume and be able to re-mount the backup volume to multiple container instances. 

1. sqldata => /var/opt/mssql
2. sqlbackups => /var/opt/mssql/backups

# Custom configurations

## There are 2 configuration files

### 1. docker-compose.yml

* Defines the SQL service like *Docker image, Port and mount volumes*

### 2. mssql_config.env

* Defines the Environment Variables in the container instance like *SA Password*

See [Here](https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-configure-environment-variables?view=sql-server-ver15)
for full list of SQL Environment Variables.

# Upgrade/Downgrade between SQL Server CU versions

The power of container can let you upgrade or downgrade your SQL instance to test for new features and compatiblity by just changing one line of code in the config file.

## To switch between different version of CU 

1. Change the docker image tag in **docker-compose.yml** to a different CU#.

All databases in the instance will be automatically upgraded/downgraded

See full list of tags [HERE](https://hub.docker.com/_/microsoft-mssql-server)

```yaml
mssql01:
      #image: "mcr.microsoft.com/mssql/server:2017-latest"
      image: "mcr.microsoft.com/mssql/server:2017-CU17-ubuntu"
```
## To upgrade from SQL 2017 to SQL 2019

1. Before you stop the SQL 2017 container fix the permission as SQL 2019 is running under mssql instead of root

```docker
docker exec -it mxsql "bash"
##bash commands
chgrp -R 0 /var/opt/mssql
chmod -R g=u /var/opt/mssql
```

2. Change the docker image tag in **docker-compose.yml** to SQL 2019

```yaml
mssql01:
      #image: "mcr.microsoft.com/mssql/server:2017-latest"
      image: "mcr.microsoft.com/mssql/server:2019-latest"
```
3. Stop the SQL 2017 container

```docker
docker-compose down
```

4. Start the SQL 2019 container

```docker
docker-compose up -d
```
### **WARNING: You won't be able to do in-place downgrade across major versions**

## To restore back to SQL 2017 you'll either

### 1. Create a new SQL 2017 container and transfer your work over with no dataloss
### 2. Drop your sqldata volume, update docker-compose.yml to use 2017 image and docker-compose up to start the container in SQL 2017 with dataloss

```docker
docker-compose down
docker volume rm mxsql-docker_sqldata
docker-compose up -d
```








