![bottleneck](bottle-small.jpg)

# Case Study 1: Bottlenecks in Traditional Data Partition Management 

With the proliferation of large datasets in cloud storage solutions such as [Amazon S3](https://aws.amazon.com/s3), ensuring efficient data 
loading and processing becomes critical for maintaining performance and scalability in modern data platforms 
like Databricks. A common challenge arises when working with partitioned data over long periods, as managing 
file system paths for specific date-hour ranges or other dimensions can introduce significant overhead and 
bottlenecks.

For example, consider a scenario where JSON data are stored in the following S3 path structure:

`s3://bucket/prefix/customer={customer}/dt={date}/hr={hour}/`

Here, our datetime sensitive data are partitioned by:

* Customer: Data are organized and partitioned customer-wise.

* Date (`dt`): Data are stored in separate folders for each day, such as `dt=2024-01-01`, `dt=2024-01-02`, etc.

* Hour (`hr`): Each day’s data are further divided into hourly subfolders, such as `hr=00`, `hr=01`, up to `hr=23`.

This partitioning enables us efficient data retrieval in later pipeline stages for specific customers and 
timeframes. For instance, to retrieve customer data from January 1, 2024, at 14:00, we would access the file 
path:

`s3://bucket/prefix/customer=some_customer/dt=2024-01-01/hr=14`

However, as datasets grow, manually managing these file system paths for date-hour partitions can lead to 
performance degradation. We recognized certain inefficiencies in the existing data loading practices, such as 
manual file path construction and partition management, and undefined or schema-less JSON data handling, that 
slowed down our data pipeline and increasing the operational costs of the Databricks job. Ultimately, we were 
able to streamline the whole data loading process more effectively by leveraging Databricks' and PySpark’s 
internal optimizations and by defining explicit schemas for our data.

## Problem 

The data pipeline used to load and process events from millions of user activities and metadata stemming from 
smartphone touch interactions. These events are partitioned by customer-specific fields, date and hour from 
S3-stored JSON data. When dealing with large, partitioned datasets, we knew that we could make use of the 
partitioning to enable efficient querying and retrieval. However, we added performance degradations and 
bottlenecks when managing these partitions manually and when processing schema-less JSON data. In our case, by 
employing the wrong data loading strategy, we added the computational overhead of:

* Explicit file path construction: With our approach, we wanted to make use of Spark’s lazy loading feature, by explicitly constructing file system paths based on date, hour, customer-specific fields, etc., and iterating through folder hierarchies, manually collecting all date-hour paths that fulfill certain conditions.  

* Manual folder existence checks: The manual collection of date-hour folders for a specific condition required pre-determining the exact paths to load, leading to inefficiencies, especially when scanning large datasets across many partitions. As the number of partitions grews, the system slowed down, increasing the latency of every query. 

* Inefficient partition pruning: Our approach builds explicit file system paths for customers and partition columns using nested hierarchies. This implied that every query had to handle folder existence checks and file path constructions before loading data, adding to the overhead. This manual management of partitions made our system often load irrelevant data, failing to efficiently prune partitions that didn’t meet the query criteria. This led to wasted memory, higher compute costs, and unnecessary data loading.  

* Lack of explicit schema definition for JSON data: Spark can infer schemas dynamically from JSON data, but this approach adds significant overhead during the parsing phase, as it requires multiple passes over the data to infer the structure. The lack of explicit schema often caused errors or inefficiencies. Without a predefined schema, the process was slower, and the risk of incorrect type inference was higher, leading to runtime issues and query failures. 

## Challenges & Solutions 

We managed to address these issues by leveraging Spark optimizations on Databricks to handle large-scale data 
efficiently. Our new approach enhances performance by eliminating manual path management and explicitly 
defining schemas for JSON data during the DataFrame creation process. As a result, in our case PySpark’s 
optimizations such as lazy evaluation, partition pruning, and predicate pushdown could be applied at scale and 
now Spark can dynamically load only the necessary data while optimizing I/O and computation across the 
cluster. The solution now features a data loading process that applies the following best practices and 
optimizations:

(1) Explicit schema definition: 

One of the key optimizations we implemented was the explicit schema definition for the JSON data files we were 
processing. Previously, multiple scans of the data had to be performed in order to infer the schema for the 
Dataframe creation. This involved specifying a detailed and complex schema for the data, which not only 
improved performance by avoiding I/O overhead of schema inference but also brought stability to the data 
pipeline. This is particularly important in production pipelines when inferring the schema could lead to 
incorrect types and subsequent errors during query execution, as the data parsing is now more stable and 
predictable.

In our case, the schema was highly nested, reflecting the complexity of the underlying data. It included 
structured metadata of different device platforms, input devices, version information, and event lists, with 
fields ranging from simple data types (like strings and integers) to more complex ones (like arrays, maps, and 
nested structures). Here’s a generalized view of the schema we used:

a. Nested structures: 

The schema included several nested levels, representing different categories of metadata. For example, a 
nested structure for platform related metadata contained subfields like BUILD, SCREEN, and VERSION, each of 
which had its own internal fields (e.g., BUILD had fields for BRAND, MODEL, PRODUCT, etc.).

b. Complex data types: 

The schema also defined complex data types, such as: 

i. Arrays: For handling lists of values, like event data, we used arrays of arrays for numeric data types (e.g., `ArrayType(ArrayType(DoubleType()))` for event lists). 

ii. Maps: We defined maps to handle key-value pairs, such as input device mappings, where the key was a string and the value was another nested structure. 

c. Event data: 

Event-related data fields were particularly complex, consisting of multiple arrays representing various event 
types. Each event type had a predefined array structure, which allowed us to efficiently store and retrieve 
event information without the overhead of dynamic type inference during runtime.

d. Predefined counters and labels: 

We also predefined schema fields for specific counters and labels, such as user session information, 
timestamps, and triggers. These fields were essential for maintaining the integrity of the data processing 
pipeline, as they ensured consistency in the interpretation of key metrics.

(2) Lazy evaluation:

The optimized data loading code does not build paths explicitly for each partition and hour. Instead, it 
relies on loading the whole global folder and applies a filter on the date and hour after the data has been 
loaded into PySpark. The filtering is done dynamically via PySpark's DataFrame operations, which allows it to 
utilize Spark’s internal optimizations, particularly lazy evaluation. Spark can then apply several 
transformations at once and produce a single, optimized query plan.

(3) Predicate pushdown:

By making use of the predicate pushdown feature, Spark can apply filters early in the data reading process 
before loading data into memory. When our data are partitioned (e.g., by date and hour), PySpark automatically 
pushes filter predicates (e.g., `WHERE date >= '2024-01-01' AND hour = 12`) down to the file source level. This 
prevents irrelevant partitions from being read, minimizing I/O overhead.

(4) Partition pruning:

Since our datasets are already stored in a partitioned way on S3, we can simply skip reading certain 
partitions that don’t meet the filter criteria. We had to ensure that the data are partitioned on frequently 
queried columns, such as time (date/hour), and Spark could then automatically prune partitions using its 
native partitioning mechanisms. We now let Spark’s query planner dynamically choose which partitions to load 
and ensure balanced partition sizes, avoiding data skew as well as lowering both the query latency and the 
amount of data shuffled across the network.

(5) [Spark Catalyst](https://www.databricks.com/glossary/catalyst-optimizer) optimizer:

In the end, when applying all the above-named best practices and solutions, we ended up with an optimized 
physical plan, originating from our initial logical plans. This is thanks to Spark’s Catalyst optimizer. Our 
augmented usage of DataFrame operations delegated partition and file management to the Catalyst optimizer, 
which decides the most efficient data access path. Thus, data shuffling is minimized, reducing not only I/O 
operations, but memory likewise.

## Quantitative Impact 

The performance improvements achieved by transitioning to the new data loading strategy were significant, 
already for small datasets. In our experiments, using data that consisted of 3 419 676 rows spread across 30 
000 S3 files and accounting for 6.7 GB file storage on S3, we observed substantial reductions in processing 
time.

Initially, with the old data parsing strategy and without an explicit schema, processing our JSON data took 
585 seconds. By defining an explicit schema, we were able to reduce this time to 315 seconds—a 46% 
improvement.

However, the overall greatest improvements were seen after transitioning to the new data reading approach. 
With the new approach, which leverages both the optimized schema definition and Spark’s internal 
optimizations, the processing time dropped to 288 seconds. This represents an additional 8.6% reduction in 
processing time compared to the old data reading process with schema and a 50.8% improvement overall from the 
initial 585 seconds without schema.

On to the next [Case Study 2: Optimizing Data Loading Workflows](case2.md).
