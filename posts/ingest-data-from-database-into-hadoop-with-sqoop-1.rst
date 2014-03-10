.. title: Ingest data from database into Hadoop with Sqoop #1
.. slug: ingest-data-from-database-into-hadoop-with-sqoop-1
.. date: 2012/06/07 21:15:29
.. tags: BigData
.. link: 
.. description: 
.. type: text

Sqoop is an easy tool to import data from databases to HDFS and export data from Hadoop/Hive tables to Databases. Databases has been de-facto standard for storing structured data. Running complex queries on large data in databases can be detrimental to their performance.
It is some times useful to import the data into Hadoop for ad hoc analysis. Tools like hive, raw map-reduce can provide tremendous flexibility in performing various kinds of analysis.
This becomes particularly useful when database has been used mostly as storage device (Ex: Storing XML or unstructured string data as clob data).

Sqoop is very simple on it's face. Internally, it uses map-reduce in parallel data import from Database and utilizes JDBC connection for the purpose.

I am jumping straight into using sqoop with oracle database and will leave installation for some other post.

Sqoop commands are executed from command lines using following structure:: 

	sqoop COMMAND [ARGS]

All available sqoop commands can be listed with: sqoop help

Article focuses on importing from database specifically Oracle DB. All commands displayed multiline has to be run as a single command.

*List database schema present on Oracle server*

User credentials should have have sufficient permission to list the databases::

	sqoop list-databases 
	             --connect jdbc:oracle:thin:@//HOST:PORT
	             --username DBA_USER 
	             --password password

Providing password on command line is insecure and we should be using "-P" instead. Sqoop will prompt for the password when you execute the command. 

*List tables present in a Oracle DB* ::

	sqoop list-tables 
	             --connect jdbc:oracle:thin:@//HOST:PORT/DB
	             --username DBA_USER 
	             -P

*Run arbitrary sql command using Sqoop* ::

	sqoop eval 
	             --connect jdbc:oracle:thin:@//HOST:PORT/DB
	             --username DBA_USER 
	             -P
	             --query "SQL QUERY"

*List columns for table in Oracle DB* ::


	sqoop eval 
	             --connect jdbc:oracle:thin:@//HOST:PORT/DB
	             --username DBA_USER 
	             -P
	             --query "SELECT  t.*  FROM  TABLENAME  t WHERE 1=0"


*Importing tables from Oracle DB*

There are many destinations from importing data from Oracle DB. You can import data to :

- HDFS as raw files
- Hive table existing or newly created
- HBase tables

I have not explored all of them but I definitely prefer storing them as raw files over hive tables. This makes it easy to define different hive table schema using external tables. You also need to ensure that the target directory in hdfs does not exist prior to running the sqoop command.
Following will fetch all the data from the table and store it as text file in target directory in HDFS.::

	hadoop fs -rmr /target/directory/in/hdfs

	sqoop import 
	             --connect jdbc:oracle:thin:@//HOST:PORT/DB
	             --username DBA_USER 
	             -P
	             --table TABLENAME
	             --as-textfile
	             --target-dir /target/directory/in/hdfs


The above tries to execute a query to extract column names from the table. If this is not possible then it will error out with ::

	ERROR tool.ImportTool: Imported Failed: Attempted to generate class with no columns!

In that case, we have to specify the columns manually in the command. ::

	sqoop import 
	             --connect jdbc:oracle:thin:@//HOST:PORT/DB
	             --username DBA_USER 
	             -P
	             --table TABLENAME
	             --columns "column1,column2,column3,.."
	             --as-textfile
	             --target-dir /target/directory/in/hdfs

The above command works well when primary key is defined for the table. If this is not specified, you will have to perform sequential import by forcing a single mapper::

	sqoop import 
	             --connect jdbc:oracle:thin:@//HOST:PORT/DB
	             --username DBA_USER 
	             -P
	             --table TABLENAME
	             --columns "column1,column2,column3,.."
	             --as-textfile
	             --target-dir /target/directory/in/hdfs
	             -m 1

or split by column in the table ::

	sqoop import 
	             --connect jdbc:oracle:thin:@//HOST:PORT/DB
	             --username DBA_USER 
	             -P
	             --table TABLENAME
	             --columns "column1,column2,column3,.."
	             --as-textfile
	             --target-dir /target/directory/in/hdfs
	             --split-by COLUMNNAME

I will explore few other nuances in importing data into HDFS in subsequent article. 





















