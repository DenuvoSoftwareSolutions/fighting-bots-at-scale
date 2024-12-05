![pipes](pipes-small.jpg)

# Case Study 5: Reduction in DataFrame Operations 

As our organization increasingly depends on Databricks for large-scale data processing, identifying and 
eliminating performance bottlenecks became essential to run efficient workloads. Databricks excels in 
distributed computing and scalability. However, by overlooking inefficient operations we suffered major 
performance degradation. We will showcase a specific scenario in our daily operational workflow, where we 
eliminated a small, but critical common pitfall and were able to reduce our task’s processing time by half.

## Problem 

In distributed systems like Spark, bottlenecks often arise due to carelessly employing inefficient I/O 
operations, redundant data transformations, or poor resource management. When working with Spark DataFrames, 
the following operations can introduce significant bottlenecks:

* Multiple DataFrame transformations: Redundant filtering or transformation actions can result in multiple scans of the same data, leading to wasted computation. 

* Inefficient writing strategies: Writing DataFrames to storage multiple times without optimizing for partitioning or indexing leads to unnecessary I/O and increased processing latency. 

* Skewed partitioning: Imbalanced partitions in DataFrames can cause certain nodes to process significantly more data than others, leading to delays. 

To illustrate the principles of optimizing Spark DataFrame operations, we review our daily data pipeline that 
processes and writes large volumes of data. The original implementation of the processing task contained 
several inefficiencies, which we address step by step to demonstrate the performance gains achieved by 
applying best practices.

After performing heavy processing tasks, we separated the results into two categories by filtering the 
DataFrame:

* df_ok for valid results. 

`df_ok = df.filter("erroy_type is None")`

* df_failed for failed results. 

`df_failed = df.filter("error_type is None") `

These operations caused multiple filters on the same DataFrame, resulting in multiple data passes. 
Additionally, writing each DataFrame separately increased the number of I/O operations, exacerbating 
performance issues. Therefore, a simple, redundant operation became an increasingly significant performance 
bottleneck as the processing logic grew more complex.

## Challenges & Solutions 

To address these issues, we redesigned the filtering and post-filtering stage in our pipeline: 

* Added a new column to flag whether the processing failed. 

`df = df.withColumn("failed", col("error_type").isNull())`

* Partitioned data based on this flag during the write phase, reducing the need for multiple filters and leveraging Spark’s partitioning capabilities. 

`df = save_and_read(df, output_table, partition_by=["failed"])`

This refactoring writes the DataFrame once and uses partitioning. Partitioning the DataFrame based on the 
failed column allows Spark to efficiently segregate the data into partitions where there were errors or no 
errors. This reduces the number of write operations and leverages Spark's optimized partitioning mechanisms by 
skipping irrelevant partitions.

(1) Minimize redundant DataFrame transformations: 

Our refactoring combined multiple operations into a single step, and by adding flags we eliminated redundant 
filtering. The trick was to declare a conditional column for the filter and prevent multiple passes over the 
data, halving the computational load.

(2) Leverage partitioning for large datasets: 

Since failed is a boolean, partitioning on this column results in two partitions. This is generally efficient, 
but data skew has to be kept in mind.

(3) Efficient data access: 

Another favourful consequence of the partitioning is the more compact data management, while still allowing 
Spark to read only the relevant partitions when executing queries, leading to faster read times and reduced 
computational overhead.

Overall, our suboptimal DataFrame operations sabotaged Databricks’ and Spark’s advantages at processing large 
datasets. By following simple best practices like reducing redundant operations, using partitioning, and 
optimizing I/O, performance and scalability was significantly enhanced.

Let's wrap it up with some [conclusions](conclusion.md).
