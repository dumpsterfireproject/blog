---
title: "Creating an Oracle 12c Container for Development"
date: 2020-03-12
slug: "/creating-an-oracle-12c-container-for-development"
description: TBD
tags:
  - Docker
  - SQL
---
Recently I had to set up an Oracle 12c database in a container with test data to be used during testing and development. This
post is just a quick documentation of that process.

The developer tier for Oracle Database 12c Enterprise Edition is available on [Docker hub](https://hub.docker.com/_/oracle-database-enterprise-edition). It's available through the store, but
is free of charge. On Docker hub, you'll need to click on the 'Proceed to Checkout' button, agree to the terms of service, enter
your contact information, and click on the 'Get Content' button. At that point you will be taken to the setup instructions page
and be able to view the documentation for the image.

There are a few environment variables you can set to customize your container. For this exercise, I just used the defaults.

DB_SID – Changes the SID of the database. Default is ORCLCDB  
DB_PDB – Name of the PDB. Default is ORCLPDB1  
DB_MEMORY – default is 2 GB  
DB_DOMAIN – default is localdomain.

I was planning on creating a new user and schema, creating multiple tables, and populating those tables with my test data. To
reuse the data, I mounted a volume to the /ORCL directory in the container. In my case, I used a host system directory for my
volume. I started my container using the following.
```
docker run -d -it --name &lt;container name&gt; -v &lt;host system directory&gt;:/ORCL -p 1521:1521 -p 5500:5500 store/oracle/database-enterprise:12.2.0.1
```
After running `docker container ls` to check the status of the database, once I saw the status say "healthy," my database and
container were ready for use.

To create the user/schema that I would use to create my tables and test data, I got to the command prompt of my container by
running `docker exec -it &lt;container name&gt; bash`. From there, I navigated to the /home/oracle directory and ran `source .bashrc`.
The .bashrc script set all the environment variables that I would need for the remainder of the task.
```
[oracle@085729ed492c ~]$ cat .bashrc
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
 . /etc/bashrc
fi


# Uncomment the following line if you don't like systemctl's auto-paging feature:
# export SYSTEMD_PAGER=


# User specific aliases and functions
export ORACLE_HOME=/u01/app/oracle/product/12.2.0/dbhome_1
export OH=/u01/app/oracle/product/12.2.0/dbhome_1
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/u01/app/oracle/product/12.2.0/dbhome_1/bin
export TNS_ADMIN=/u01/app/oracle/product/12.2.0/dbhome_1/admin/ORCLCDB
export ORACLE_SID=ORCLCDB;
[oracle@085729ed492c ~]$ 
```
I then ran `sqlplus / as sysdba and alter session set "_ORACLE_SCRIPT"=true;`. Then I was able to create my user and grant the
privileges needed to create the tables and data.
```
[oracle@085729ed492c ~]$ sqlplus / as sysdba
   
   SQL*Plus: Release 12.2.0.1.0 Production on Tue Mar 10 14:56:26 2020
   
   Copyright (c) 1982, 2016, Oracle.  All rights reserved.
   
   
   Connected to:
   Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
   
   SQL&gt; alter session set "_ORACLE_SCRIPT"=true;
   
   Session altered.
   
   SQL&gt; CREATE USER testuser IDENTIFIED BY testpwd;
   
   User created.
   
   SQL&gt; GRANT RESOURCE TO testuser;
   
   Grant succeeded.
   
   SQL&gt; GRANT CONNECT TO testuser;
   
   Grant succeeded.
   
   SQL&gt; GRANT CREATE SESSION TO testuser;
   
   Grant succeeded.
   
   SQL&gt; grant create table to testuser;
   
   Grant succeeded.

   SQL&gt; grant SELECT ANY TABLE to testuser;

   Grant succeeded.

   SQL&gt; grant UPDATE ANY TABLE to testuser;

   Grant succeeded.

   SQL&gt; grant DELETE ANY TABLE to testuser;

   Grant succeeded.

   SQL&gt; grant INSERT ANY TABLE to testuser;

   Grant succeeded.
   
   SQL&gt; grant unlimited tablespace to testuser;
   
   Grant succeeded.
   
   SQL&gt; select value from v$parameter where name='service_names';
   
   VALUE
   --------------------------------------------------------------------------------
   ORCLCDB.localdomain
   
   SQL&gt; exit
   Disconnected from Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
   [oracle@085729ed492c ~]$ 
```
I was then ready to connect to my Oracle database using [DBeaver](https://dbeaver.io/) (or whatever client you prefer) and create my tables and data.
Connecting via JDBC, I used the connection string `jdbc:oracle:thin:@//localhost:1521/ORCLCDB.localdomain`. I was able to validate
the service name my looking at the tnsnames.ora file in the TNS_ADMIN directory (/u01/app/oracle/product/12.2.0/dbhome_1/admin/ORCLCDB).
```
[oracle@085729ed492c ORCLCDB]$ cat tnsnames.ora
ORCLCDB =   (DESCRIPTION =     (ADDRESS = (PROTOCOL = TCP)(HOST = 0.0.0.0)(PORT = 1521))     (CONNECT_DATA =       (SERVER = DEDICATED)       (SERVICE_NAME = ORCLCDB.localdomain)     )   ) 
ORCLPDB1 =   (DESCRIPTION =     (ADDRESS = (PROTOCOL = TCP)(HOST = 0.0.0.0)(PORT = 1521))     (CONNECT_DATA =       (SERVER = DEDICATED)       (SERVICE_NAME = ORCLPDB1.localdomain)     )   ) 
[oracle@085729ed492c ORCLCDB]$
```
With mysql, you can create a docker file that starts with a [mysql base image](https://hub.docker.com/_/mysql) and copy a .sql file to /docker-entrypoint-initdb.d
and create an image that will create a container with all your test data which is very portable. I wanted to do something similar
with this oracle database, so that I can validate my application would work properly with a variety of database vendors. It was
great that I could persist and share the data by mounting a directory on my host system to the /ORCL directory of the container.
Trying to do something like create an image that would copy the contents of that directory into a new image would have been
impractical; the size of that directory is massive. I was able to use Oracle's data pump export utility to create a backup of
that database. In the near future, I'd like to try to utilize the data pump import and create an image that includes my test
user, tables, and data, is a reasonable size, and is very portable. But in the meanwhile, the following are the steps I used to
create a directory in the /ORCL directory and grant the proper privileges (both these steps done from within sqlplus). Then from
the command line, I ran the export which created a backup that was just over 1 MB uncompressed, or less than 500 KB compressed.
The following example included the arguments for compression.
```
[oracle@3fa67e802f81 ~]$ sqlplus  /  as  sysdba

SQL*Plus: Release 12.2.0.1.0 Production on Tue Mar 10 20:18:17 2020

Copyright (c) 1982, 2016, Oracle.  All rights reserved.


Connected to:
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production

SQL&gt; CREATE DIRECTORY  exp_schema  AS '/ORCL/dump';

Directory created.

SQL&gt; GRANT  read,  write  ON  DIRECTORY  exp_schema  TO testuser;

Grant succeeded.

SQL&gt; GRANT  DATAPUMP_EXP_FULL_DATABASE  TO testuser;

Grant succeeded.

SQL&gt; exit
Disconnected from Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
[oracle@3fa67e802f81 ~]$ expdp testuser/testpwd@ORCLCDB schemas=testuser directory=exp_schema DUMPFILE=exp_schm_test.dmp LOGFILE=testdb_exp.log COMPRESSION_ALGORITHM=BASIC COMPRESSION=ALL
```
