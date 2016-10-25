# Document Oriented Table (DOT) on HBase #

["Project Panthera"](https://github.com/intel-hadoop/project-panthera) is our open source efforts to enable efficient support of standard SQL features on Hadoop. In Hadoop based data warehousing systems (e.g., Hive or [ASE](https://github.com/intel-hadoop/project-panthera-ase)), users store their data as tables in HDFS and explore the data using a high level query language; the queries are then automatically converted into a series of MapReduce jobs for query processing. By bringing SQL to the Hadoop ecosystem, these systems make MapReduce much more accessible to mainstream users. 

On the other hand, high-update rate SQL query workloads cannot be directly supported in these systems. While HBase can be used to support such workloads, query processing on HBase can incur significant overheads as the it completely ignores the SQL relational model:

* *Space overheads*. To provide flexible schema support, physically HBase stores its table as a multi-dimensional map, where each cell (except the row key) is stored on disk as a key-value pair: *(row_id, family:column, timestamp) -> cell*. On the other hand, a Hive table has a fixed relational model, and consequently HBase can introduce large space overheads (sometimes as large as 3x) compared to storing the same table in HDFS.

* *Query performance*. Query processing on HBase can be much (sometimes 3~5x) slower than that on HDFS due to various reasons. One of the reason is related to how HBase handles data accesses – HBase provides very good support for highly concurrent read/write accesses; consequently, one needs to pay some amount of overheads (e.g., concurrency control) for each column read. On the other hand, data accesses in analytical query processing are predominantly read (with some append), and should preferably avoid the column read overheads.

Therefore, under "Project Panthera", we are building DOT, document oriented table on HBase, which provides an efficient storage engine for relational SQL query with high-update rate by leveraging the SQL relation mondel to provide document semantics, and is currently implemented as an HBase co-processor application. The figure below illustrates the data model of the document store.

<img src="http://cloud.github.com/downloads/intel-hadoop/hbase-0.94-panthera/datamodel.jpg" alt="DOT Data Model" width="436" height="180" />

In the document store, a table can be declared as a *document-oriented table* (DOT) at the table creation time. Each row in DOT contains, in addition to the *row key*, a collection of documents (doc), and each document contains a collection of *fields*; in query processing, each column in a relational table is simply mapped to a field in some document.

Physically, each document is encoded using a serialization framework (such as Avro or Protocol Buffers), and its schema is stored separately (just once); consequently, the storage overheads can be greatly reduced. In addition, each document is mapped to an HBase column and is the unit for update; consequently the associated read overheads can be amortized across different fields in a document.
 
When creating a DOT, the user is required to specify the schema and serializer (e.g., Avro) for each document in the table; the schema information is stored in table metadata by the `preCreateTable` co-processor. The users of DOT (e.g., a Hive query) can access individual fields in the document-oriented table in the same way as they access individual columns in a conventional HBase table – just specifying “doc.field” in place of “column qualifier” in `Get`/`Scan`/`Put`/`Delete` and `Filter` objects; the associated co-processors are responsible for dynamically handling the mapping of fields, documents and HBase columns. 

Please refer to INSTALL.txt on the detailed build and install instructions.
