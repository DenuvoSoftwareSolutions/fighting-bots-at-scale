![footwear](footwear-small.jpg)

# Case Study 4: Transitioning from User Defined Functions to Scalable Spark-Based Solutions

## Problem 

Previously, one of our data aggregation logics was implemented as a monolithic Python script within a Spark 
User-Defined Function (UDF). This UDF processed observations in a sequential manner, applying multiple 
decision algorithms (multiple detection logics and utility calculations) within a single execution flow. While 
effective for smaller datasets, this approach created several performance bottlenecks and scalability issues 
as our observation volumes grew. The use of a UDF meant that each observation was processed individually, and 
our entire analysis was performed in-memory, leading to long execution times and high memory consumption, 
making it impractical for large-scale, real-time data processing.

## Challenges & Solutions 

Our aim was to do a refactoring and translate the aggregation logic to leverage Spark’s native operations and 
distributed computing capabilities, replacing the UDF-based approach. The new design breaks down the 
monolithic script into distinct processing stages, utilizing Spark’s built-in functions and window operations 
to manage and optimize each stage efficiently. This refactoring allows our logic to aggregate larger datasets 
and higher observation volumes in a structured and scalable manner.

* UDF overhead and memory consumption: The use of Spark UDFs caused significant overhead because Spark had to serialize and deserialize each row, leading to inefficient execution. Additionally, processing all observations in-memory created huge memory pressure. Our approach to this problem was the refactoring of the code to use Spark DataFrames and window functions, eliminating the need for UDFs. The logic can then be applied in parallel across multiple clusters. This reduces memory pressure and distributes the computation load efficiently, allowing the system to scale with larger datasets. 

* Code modularity and scalability: Our previous code, tightly coupled and enclosed within the UDFs, made it challenging to scale or modify specific parts without affecting our entire system. We decomposed the aggregation logic into distinct phases: detection, utility property calculation, and final decision logic. We implemented each phase as a modular Spark function, which can be optimized and scaled independently. This modularity enables flexibility and allows us to run each phase on clusters optimized for the specific task, such as memory-optimized nodes for ingestion or CPU-optimized nodes for processing. 

* Concurrency and task scheduling: The old, UDF-based approach had limited concurrency capabilities, resulting in inefficient resource usage and task scheduling. By makeing use of the DataFrame native functions, we enabled parallel task execution. Additionally, using Spark’s scheduling capabilities, tasks are triggered based on data availability, reducing idle time and improving the overall efficiency of the pipeline. 

## Quantitative Impact 

Our transition from a sequential UDF-based approach to a modular, Spark-based solution significantly optimized 
our performance, surpassing our initial expectations. Below is again a detailed comparison for different data 
volumes experiments:

Table 6: Load test experiments

|Data volume|Metric|Old logic|New logic|Improvement|
|:---|:---|---:|---:|---:|
|10 million|Total task runtime|15m 7s|8m 33s|43.45%|
|	|Logic runtime|7m 9s|57s|86.71%
|	|Databricks costs|$1.43|$0.76|46.85%|
|	|AWS costs|$1.08|$0.21|80.56%|
|100 million|Total task runtime|55m 46s|10m 31s|81.15%|
|	|Logic runtime|29m 54s|1m 11s|96.04%|
|	|Databricks costs|$8.79|$0.83|90.56%|
|	|AWS costs|$9.13|$3.29|63.98%|
|1 billion|Total task runtime|7h 24m|14m 32s|96.73%|
|	|Logic runtime|4h 50m|3m 10s|98.91%|
|	|Databricks costs|$119.41|$1.43|98.80%|
|	|AWS costs|$101.69|$4.38|95.69%|

These results demonstrate the significant performance and cost improvements we achieved through the refactored 
approach. The new modular and distributed design not only reduces the logic runtime for larger data volumes by 
up to 96% and 99% but also lowered operational costs by as much as 90% and 98% respectively. We were 
astonished by the structured approach providing such a scalable and efficient solution for processing large 
datasets. This timely and cost-effective behaviour supports our future growth in processing increasing data 
volumes.

On to the next [Case Study 5: Reduction in DataFrame Operations](case5.md).
