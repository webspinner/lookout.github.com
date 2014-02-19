---
layout: post
title: "Move Rows from MySQL to Hadoop with Pipefish"
author: rothrock
tags:
- Hadoop
- MySQL
- HDFS
- Hive
- Impala
- JNI
- C
---

Move Rows from Mysql to Hadoop with [Pipefish](https://github.com/lookout/pipefish)
==============================================

Have you ever wanted to send your MySQL query results directly to a file in [HDFS](http://hadoop.apache.org/docs/stable1/hdfs_design.html)? Well, now you can!

Why does this exist?
--------------------
Our Analytics team needs daily table snapshots from our production databases.
Not such a big deal, except that the analysts use tools like [Impala](http://www.cloudera.com/content/cloudera/en/products-and-services/cdh/impala.html) and [Hive](http://hive.apache.org) to crunch their numbers.
That means we need to ship these rows from MySQL into the Hadoop filesystem (HDFS).

First pass
----------
My first solution was to simply script the export and import using the mysql and hive CLI tools.
That worked out to three distinct steps:

1. Read from MySQL, write to a tmp file.
2. Read from the tmp file, write to HDFS.
3. Remove the tmp file.

This was OK for few small tables, but the size (>50GB) and the number (dozens) of tables grew.
Two problems quickly cropped up:

- Writing so much tmp data often filled up the filesystem.
- Intermediate storage in tmp files slowed the process significantly.

Second pass
-----------
Decomposing the tables into smaller, bite-size chunks offered a way to smooth out the tmp file problem, but it brought other problems:

- The process needed monotonically increasing primary key with no gaps. Why?
  - A primary key for fast selection of a chunk of rows.
  - No (or few) gaps so that each chunk was about the same size.
- Some place to track what chunk we were on.
- No point-in-time consistency. The script can\'t lock the source table indefinitely so we trade off a little accuracy.
- Disk intermediation still not solved.

Third pass
----------
What next?
Could I invent a way to disintermediate the tmp file?
Could I do it in a way that was fun and interesting?
Yes, because:

- Both Hive and Impala read delimited text files.
- I like writing in C. This would be the fun part.
- I knew MySQL had well-documented client libraries in C, but what about Hadoop?  Wasn\'t it all Java?
- After a bit of research I found a [C api for HDFS](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/LibHdfs.html)

Usage
-----

Less than 200 lines of C later, I had my utility:
```
  $ pf --defaults_file=~/passwd --hdfs_path="/user/hive/warehouse/my.db/table.txt" --overwrite --sql="select * from my.table"
```
The passwd file is formatted as a MySQL config file:
```
  $ cat ~/passwd
  [client]
  host     = my_production_server
  user     = copy_user
  password = showmesomethinggood
  database = production
```
The `--overwrite` flag does what you might expect. Without the flag, the utility attempts to append, so a file must already exist.

The `--sql` parameter takes a valid SQL query.
It need not be simply `select * from..`.
Use this flexibility to transform data as you like.

The usage in summary:
```
  $ pf --help
  Using JAVA_HOME=/usr/lib/jvm/java-6-openjdk-amd64
  args:
   --defaults_file='/path/to/my.cnf'
   --user='user'
   --host='host'
   --db='db name'
   --password='password'
   --hdfs_path='/path/to/file'
   --sql='sql statement'
   [--overwrite]
  user, host, db, and password settings override those specified in defaults_file.
```

The JNI and libhdfs
-------------------
The HDFS C library uses [JNI](http://docs.oracle.com/javase/6/docs/technotes/guides/jni/) so there must be a working JAVA_HOME.
The `pf` command is actually a bash script wrapper that:

1. Tries to find JAVA_HOME,
2. Builds up a very long Java CLASSPATH,
3. Executes the pipefish binary.

To do
-----
1. Handle HDFS write failures more gracefully.
2. Implement some sense of rollback on failure.
3. Optionally partition tables.
4. Option for different delimiters.
5. Winnow the CLASSPATH.

So why pipefish? I enjoy husbanding marine creatures, and I like fish-based names of software. The pipefish just seemed appropriate.

*- [Joseph (Beau) Rothrock](https://github.com/rothrock)*
