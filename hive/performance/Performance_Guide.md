
# Hive Optimization app level

# Set the Execution Engine to Tez :
Hive>Set hive.execution.engine=tez;

# Hive – Compress Query Output
hive> set hive.exec.compress.output=true;

# Hive – Auto Map Join
 Joining a big table to a small table.The default value of hive.smalltable.filesize is 25 MB.
  set hive.auto.convert.join=true;
  set hive.smalltable.filesize=<filezie_threadhold>; (optional)

# Hive – Compress Intermediate Output
  set hive.exec.compress.intermediate=true;
  set mapred.output.compression.type=BLOCK;

# Cost based query optimization
CBO is to generate efficient execution plans by examining the tables and conditions specified in the query, ultimately cutting down on query execution time and reducing resource utilization. Calcite has an efficient plan pruner that can select the cheapest query plan. All SQL queries are converted by Hive to a physical operator tree, optimized and converted to Tez/MapReduce jobs, then executed on the Hadoop cluster. This conversion includes SQL parsing and transforming, as well as operator-tree optimization.
  set hive.cbo.enable=true;
  set hive.compute.query.using.stats=true;
  set hive.stats.fetch.column.stats=true;
  set hive.stats.fetch.partition.stats=true;


# Use Vectorization
A parallel processing technique in which operations are applied on multiple rows of data rather than single row. 
In Hive, vectorization works with a column of data (the vector) and applies a single operation to the entire column.
Access to data in columnar format is crucial for vectorization, so it only works with ORC formatted tables.
  set hive.vectorized.execution.enabled = true;
  set hive.vectorized.execution.reduce.enabled = true;

# SORT Buffers:
The sort buffers can be tweaked to make it faster by making those allocations lazy (i.e allocating 1800mb contiguously on a 4Gb container will cause a 500-700ms gc pause, even if there are 100 rows to be processed). This can be used in conjunction with vectorization.
Set tez.runtime.pipelined.sorter.lazy-allocate.memory=true;
set hive.vectorized.execution.reduce.groupby.enabled = true;

# Adjust Configuration for uneven Distribution:
By default Hive puts the data with the same group-by keys to the same reducer.
If the distinct value of the group-by columns has data skew, one reducer may get most of the shuffled data.Setting the below parameter will trigger an additional MapReduce job whose map output will randomly distribute to the reducer to avoid data skew
Set hive.groupby.skewindata=true;


# Small Data Sets:
Setting Hive in "AUTO/LOCAL MODE" is helpful dealing with small amount of datasets.
Set hive.exec.mode.local.auto=true;

Are you getting a shuffle join and not a map join for smaller dimension tables?
hive.auto.convert.join.noconditionaltask.size determines whether a table is broadcasted or shuffled for a join. If the small table size is larger than hive.auto.convert.join.noconditonaltask.size a shuffle join is used. For accurate size accounting by the compiler, run ANALYZE TABLE [table_name] COMPUTE STATISTICS for COLUMNS. Then enable hive.stats.fetch.column.stats. This enables the Hive physical optimizer to use more accurate per-column statistics instead of the uncompressed file size in HDFS.

# Use ORC File Format :
Using ORC File format can improve the Hive query performance.
A new hive table can be created with ORC file format by just adding STORED AS ORC clause to CREATE TABLE command in hive. The compression techniques can be provided in the TBLPROPERTIES clause.
Existing tables can also be convereted into ORC formatted tables.

# Tez Auto Parallelism
When enabled, Hive will estimate data sizes and set  parallelism estimates. Tez will sample source vertices' output sizes and adjust the estimates at runtime as necessary.
set hive.tez.auto.reducer.parallelism;
set hive.tez.auto.reducer.parallelism = true;
set tez.runtime.report.partition.stats=true;

# Control the resources at Independent Job Level
set Number of Reducers in Hive using below command.
set mapreduce.job.reduces=XX

# set XX number of reducers for all parts of the query.
set hive.exec.reducers.bytes.per.reducer=xx
default value = 256 MB
example : if the input size is 1 GB then 4 reducers will be used
set hive.exec.reducers.max=xx
Default Value: 1009

# maximum number of reducers that will be used.
If you want to set the number or parallel threads to run the job
set hive.exec.parallel=true
set hive.exec.parallel.thread.number=xx

# Implement a Combiner – Job Level
You can run the application without a combiner, but including a combiner can improve performance, Because the combiner is not used, the values for Combine input records and Combine output records are zero, and the number for Reduce input records is the same as for Map output records or higher.

The combiner function is used as an optimization for the MapReduce job. The combiner function runs on the output of the Map phase, and is used as a filtering or aggregating step to lessen the number of intermediate keys

The Combiner class is used in between the Map class and the Reduce class to reduce the volume of data transfer between Map and Reduce
Combiners can only be used in specific cases which are going to be job dependent. 

1 One constraint that a Combiner will have, unlike a Reducer, is that the input/output key and value types must match the output types of your Mapper.
  2 Combiners can only be used on the functions that are commutative(a.b = b.a) and associative {a.(b.c) = (a.b).c} . This also means that combiners may operate only on a subset of your keys and values or may not execute at all, still you want the output of the program to remain same.
  3 Reducers can get data from multiple Mappers as part of the partitioning process. Combiners can only get its input fr
