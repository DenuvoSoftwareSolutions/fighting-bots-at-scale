![loading ships](loading-small.jpg)

# Case Study 2: Optimizing Data Loading Workflows 

After dealing with the performance interferences in the data loading and management procedure, we determined 
through experiments that our current Databricks job workflow design also caused performance bottlenecks when 
dealing with ever-increasing volumes of data. In this case study, we examine how we transitioned from a 
traditional, resource-intensive pipeline to a more efficient and structured workflow in Databricks. This new 
approach improved data processing, storage, and I/O performance by addressing specific bottlenecks encountered 
with large-scale data operations.

## Problem 

Previously, the data pipeline consisted of a single task combining multiple processing steps of varying complexity. The task involved reading JSON data from Amazon S3, carrying out very resource-intensive computations on the data, and uploading the results as Parquet files back to S3. While functional, this approach introduced inefficiencies and performance bottlenecks, primarily due to the unstructured nature of JSON and the overhead of repeatedly interacting with S3 throughout the workflow. To address these issues, we restructured the job workflow into distinct stages: 

1. Data Ingestion: Reading the JSON data from S3 and storing them as a table in Databricks 

2. Data Processing: Passing this table to the processing task 

3. Data Storage: Storing the processed data as a table 

4. Data Export: Uploading the final table result in Parquet format 

This structured job approach streamlined operations, reduced resource usage and improved overall performance, which we will now review. Each stage is defined clearly by its computation tasks and is tied directly to its purpose, making the overall workflow and the subsequent review easier to understand. 

## Challenges & Solutions 

(1) I/O bottlenecks (reading and writing from S3): 

In PySpark, the Data Source API includes interfaces and classes that allow developers to read from and write 
to various data sources like HDFS, HBase, Cassandra, JSON, CSV, and Parquet. This API provides a consistent 
approach for working with data, regardless of the underlying format or storage technology. The JSON data in 
our case were nested, and handling these nested fields required computational effort due to Spark needing to 
resolve the full hierarchy of the data structure. Spark SQL can automatically infer the schema, including the 
nested fields. These nested fields are represented as complex data types in our DataFrame, such as structs 
(similar to objects), arrays, or maps. While this allows Spark to retain the structure of the data, accessing 
these nested fields and performing operations such as select, explode, or withColumn further increased 
computational complexity and processing overhead:

a.	Nested field access: Accessing nested fields requires Spark to resolve the full hierarchy of the data. Each time we accessed a nested field, Spark had to traverse the struct (or array/map) to reach the desired field. This traversal added significant computational cost compared to accessing flat fields. 

b.	Transformations on nested data: To make matters worse, our intensive operations on these nested fields and additional functions such as select, explode, or withColumn, only further increased the complexity of the transformations. Spark had to generate extra steps in the execution plan, leading to more processing overhead. 

c.	Shuffling and joins: Under certain conditions, the nested data needed to be first transformed or joined with other datasets, the nested structure led to more complex shuffle and join operations, even further degrading our computational efficiency. 

To acquire insights into these bottlenecks, we utilized Databricks Spark UI and the compute cluster metrics. 
The Spark UI provided visibility into job stages, task durations, and shuffle operations, helping us identify 
performance bottlenecks related to nested data access and transformations. The cluster metrics dashboard 
showed I/O and memory usage patterns, confirming inefficiencies in the JSON processing.

To minimize I/O and memory overhead, we converted JSON data into a structured table format in Databricks at 
the earliest stage of the workflow. This change reduced the need for repeated pulls from S3. By keeping the 
data in an efficient format, we significantly cut down on unnecessary reads and writes, improving processing 
speed.

(2) Serialization, memory usage, and CPU overhead: 

The JSON data were converted to Parquet at every stage within our workflow and Spark had to deserialize the 
JSON, essentially wasting memory and CPU resources. Since our data contained complex and deeply nested 
structures, this process was very slow at querying the data and large datasets put immense pressure on memory, 
causing shuffling and disk spilling.

We monitored these inefficiencies using tools like Databricks Ganglia Metrics and Spark UI, tracking memory 
usage, shuffle writes, and CPU utilization. These tools allowed us to pinpoint where memory pressure was 
highest and when disk spilling occurred.

Now, with the data reading designed as its own dedictated task, we load the data once from JSON and convert it 
to a table format. We were able to avoid repeated serialization and deserialization when performing any 
transformations on the table.

(3) Coordination and dependency overhead: 

Running tasks such as reading, processing, and uploading in a monolithic job created coordination overhead, 
resulting in inefficient task scheduling and resource contention. Even with Databricks’ optimized resource 
handling, a single cluster configuration could not efficiently meet the necessary requirements at every stage. 
Thus, we decoupled the workflow into distinct tasks (data loading, processing, and result uploading). This 
allowed each job to be optimized individually from a code perspective as well as cluster configuration on 
Databricks, reducing dependencies and enabling parallelization where possible. This also ensured better 
resource allocation at each stage, improving overall efficiency.

a.	Data loading: Since this is mainly an I/O-bound task, we allocated nodes with higher I/O throughput to pull data from S3 quickly. This ensures that the ingestion task completes faster, reducing delays before the processing stage starts. 

b.	Data processing: The transformation stage is more CPU-intensive. We allocated larger clusters with more CPUs and memory to speed up this step. Since the other tasks are not running concurrently, we could maximize resource usage during this stage to improve processing performance. 

c.	Data storage: After processing, the storage stage writes data back to S3. This is less CPU-heavy but requires good I/O performance. By using I/O-optimized nodes for this stage, we minimized delays when writing large processed datasets back to S3. 

Additionally, by ensuring that each task was decoupled from the previous one, we can now scale individual 
tasks as needed without reconfiguring the entire pipeline. For instance, when our dataset size increases, we 
scale the data processing stage by adding more compute resources to handle the increased volume.

(4) Concurrency and scalability: 

We ensure that each task runs as efficiently as possible while avoiding unnecessary waiting times between 
stages. Each task is triggered as soon as the previous one successfully completes. We automated the transition 
between tasks to avoid any unnecessary idle time between stages. For example, as soon as the data ingestion 
task finishes pulling data from S3, the transformation task kicks off immediately. The handover process is 
also optimized, since we ensure that the output of one task is stored in a columnar optimized format that can 
be efficiently consumed by the next task without requiring additional serialization or format conversion 
steps. This avoids the need for manual intervention or any unnecessary waiting, keeping the pipeline running 
as smoothly as possible.

## Quantitative Impact 

We were able to improve performance and achieve immense cost reductions through the optimized Databricks 
workflow. By transitioning from the previous, monolithic pipeline to the distinct staged and 
resource-efficient approach, we observed substantial gains across varying data volumes: 10 million (20 GB) and 
100 million (200 GB) daily rows to be processed.

Table 1: Performance and cost analysis for 10 millions rows

|Metric|Old workflow|New workflow|Change|
|:---|---:|---:|---:|
|Runtime|6h 8m 57s|42m 19s|88.53%|
|Databricks costs|$44.37|$6.26|85.89%|
|AWS costs|$25.02|$10.88|56.51%|
|Total costs|$69.39|$17.14|75.30%|

In the case of processing 10 million daily rows, the new workflow reduced runtime by nearly 89% and total costs
by over 75%, demonstrating the effectiveness of minimizing unnecessary reads/writes and optimizing resource
allocation.

Table 2: Performance and cost analysis for 100 million rows

|Metric|Old workflow|New workflow|Change|
|:---|---:|---:|---:|
|Runtime|12h 52m 41s|2h 22m 11s|81.60%|
|Databricks costs|$361.40|$34.61|90.42%|
|AWS costs|$166.47|$35.94|78.41%|
|Total costs|$527.87|$70.55|86.63%|

For the larger dataset of 100 million daily rows, the optimized workflow reduced runtime by approximately 82% 
and overall costs by nearly 87%. These results emphasize the benefits of decoupling tasks and utilizing 
cluster resources tailored to specific stages of the pipeline suporting future growth and high data volumes.

On to the next [Case Study 3: Window Functions as a Bottleneck in Data Aggregation](case3.md).
