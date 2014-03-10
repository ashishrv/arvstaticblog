.. title: Ingest data from database into Hadoop with Sqoop #2
.. slug: ingest-data-from-database-into-hadoop-with-sqoop-2
.. date: 2012/06/08 23:22:38
.. tags: BigData
.. link: 
.. description: 
.. type: text

Here, I explore few other variations for importing data from database into HDFS. 
This is a continuation of :doc:`previous article <ingest-data-from-database-into-hadoop-with-sqoop-1>`.

Previous sqoop command listed were good for one time fetch when you want to import all the current data for a table in database.

A more practical workflow is to fetch data regularly and incrementally into HDFS for analysis. You do not want to skip any previously imported data. For this you have to mark a column for incremental import and also provide an initial value. This column mostly happens to be time-stamp. 


::

	sqoop import 
	             --connect jdbc:oracle:thin:@//HOST:PORT/DB
	             --username DBA_USER 
	             -P
	             --table TABLENAME
	             --columns "column1,column2,column3,.."
	             --as-textfile
	             --target-dir /target/directory/in/hdfs
	             -m 1
	             --check-column COLUMN3
	             --incremental  lastmodified
	             --last-value "LAST VALUE"


At the end of the execution, you will have to note down the lastmodified value from the output ::

	--last-value xxxxxxxxx

and use it in next execution of the command. You can avoid noting down this value and changing your command for subsequent execution by saving your command as "Sqoop Job". 

*Save sqoop commands as jobs*

::

	sqoop job
	             --create JOBNAME
	             -- import
	             --connect jdbc:oracle:thin:@//HOST:PORT/DB
	             --username DBA_USER 
	             -P
	             --table TABLENAME
	             --columns "column1,column2,column3,.."
	             --as-textfile
	             --target-dir /target/directory/in/hdfs
	             -m 1
	             --check-column COLUMN3
	             --incremental  lastmodified
	             --last-value "LAST VALUE"

*List and delete sqoop jobs*

Now it is pretty easy to list sqoop jobs ::

	sqoop job --list
	sqoop job --delete JOB_NAME

*Execute sqoop jobs*

There is one caveat though while executing sqoop jobs. 
It will prompt for password input and if you are thinking of automation, 
you will have to make provisions for that::

	sqoop job --exec JOB_NAME

*Other useful configurations*

There are few other useful command configuration which comes handy during data import. 
Some of the columns in the table might be a clob data which could contain free running text including the default delimiter and field separator. 
To make sure that I can parse the data in HDFS unambiguously, 
I try to provide field separator and also escape value.::

	sqoop import 
	             --fields-terminated-by '\001' 
	             --escaped-by \\
	             --connect jdbc:oracle:thin:@//HOST:PORT/DB
	             --username DBA_USER 
	             -P
	             --table TABLENAME
	             --columns "column1,column2,column3,.."
	             --as-textfile
	             --target-dir /target/directory/in/hdfs
	             -m 1
	             --check-column COLUMN3
	             --incremental  lastmodified
	             --last-value "LAST VALUE"

*Output data format*

So far I have imported data as text files as it is easy to process via several means. 
How ever there are other formats available for use. ::

	--as-avrodatafile  Imports data to Avro Data Files 
	--as-sequencefile  Imports data to SequenceFiles 
	--as-textfile      Imports data as plain text (default)

Happy sqooping! 










