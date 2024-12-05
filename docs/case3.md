![many windows](windows-small.jpg)

# Case Study 3: Window Functions as a Bottleneck in Data Aggregation

In this case study, we are focusing on the processing task of our Databricks job. This task is the most CPU 
intensive and time consuming component in the data pipeline and our underlying infrastructure becomes more 
vulnerable when being exposed to increasing data volumes in result. We will focus on how a shift from 
traditional row-based processing methods to more advanced array-based operations has helped to mitigate 
bottlenecks in data partitioning and processing large datasets. The improvements introduced have shifted 
execution to leverage SQL transformations on arrays rather than relying on computationally expensive window 
functions.

## Problem

In our previous pipeline, we encountered several bottlenecks and inefficiencies in an aggregation step. Upon 
inspecting the job cluster metrics, we noticed excessive disk spilling and memory usage. These inefficiencies 
stemmed from heavy reliance on row-based window functions and poor data partitioning and the challenges to be 
solved included:

* Expensive window functions: A window function evaluates each row in a table by referencing a specified set 
of rows, known as the frame, which can vary for each row. The pipeline extensively used an unbounded window 
function to calculate values across large datasets. These operations, scanning the whole DataFrame row by row, 
caused significant slowdowns, especially since our data were poorly partitioned, leading to excessive 
shuffling across nodes.

```
inc_window = Window.partitionBy("id").orderBy("subId")`
full_window = inc_window.rowsBetween(Window.unboundedPreceding, Window.unboundedFollowing)
```

* Inefficient partitioning and shuffling: Spark's default shuffling mechanisms often led to suboptimal data 
distribution across workers. Without explicit repartitioning, the data were poorly balanced, leading to 
increased resource consumption and slower processing.

* Inefficient joins and union operations: We perform semi-joins followed by union operations between different 
DataFrames and this exacerbated these issues, particularly when combining window functions and the lack of 
partitioning. These operations were computationally expensive and added to the overhead.

## Challenges & Solutions

Window functions are a powerful feature in Apache Spark that allows for the computation of values across 
partitions. However, when used improperly, they can become a significant performance bottleneck. Our 
transition from a window-based processing approach to an array-based strategy, alongside improvements in data 
partitioning, addressed several key performance challenges that we previously faced. Below is a breakdown of 
these challenges and the corresponding solutions.

(1) Heavy reliance on window functions:

In our previous approach, we extensively used window functions to generate new attributes for each row across 
large datasets, which then served as intermediate variables for further processing. Our computations required 
a full unbounded window and were particularly costly, as they required scanning entire partitions, leading to 
performance degradation.

We replaced most window functions with array-based operations. Data were grouped by matching unique IDs into 
arrays using `collect_list`, and then sorted and filtered through array-based transformations (`array_sort`, 
`filter`, etc.). This allowed multiple rows to be processed in a single pass, minimizing computational overhead 
and improving performance.

(2) Inefficient data partitioning and shuffling:

Without explicit repartitioning, the data were poorly distributed across Spark workers, leading to excessive 
shuffling. This inefficiency slowed down operations such as window functions, joins, and aggregations. As a 
result of the subsequent grouping, we explicitly repartitioned the data by their matching unique IDs before 
any significant transformations or joins were performed. This ensured that data were evenly distributed across 
nodes, reducing the need for costly shuffling and improving the locality of operations. Although 
repartitioning introduces some shuffling time, overall, the result was a meaningful reduction in execution 
time and better use of cluster resources.

(3) Performance overheads in joins and unions:

As a nice side effect, our new approach with the explicit partitioning also helped streamlining the process, 
leading to more efficient joins and unions with less resource consumption, which would otherwise increase 
latency with data frequently shuffled across nodes.

(4) Redundant row-based computations:

The array-based processing performs operations like sorting, filtering, etc. on groups of data, rather than 
repeatedly computing values for different rows within the same partition. The application of these 
calculations on the array level avoids redundancy and minimizes the number of required computations, which 
positively impacted memory consumption.

However, when using Spark SQL window functions with unbounded ranges on large DataFrames, the impact on memory 
can be significant. Unbounded windows (e.g., ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) require Spark 
to hold all relevant rows within the partition in memory simultaneously to perform calculations. This became 
problematic for us with large datasets, as the memory consumption grew substantially, leading to out-of-memory 
errors or excessive disk spilling.

Such memory pressure not only increases the risk of job failures but also slows down execution due to frequent 
garbage collection processes and the need for Spark to spill data to disk. To avoid these inefficiencies, it 
is crucial to either use bounded windows where appropriate or to pre-aggregate data into smaller, manageable 
chunks before applying window functions, ensuring that memory usage remains within acceptable limits and 
performance is optimized.

(5) Complicated schema fixing and column selection:

At the end of this task, we had to enforce a certain DataFrame schema and had to select and fix specific 
columns after the window operations, leading to potential schema mismatches and increased complexity. While 
these operations remained similar in both approaches, the reduced complexity in the data preparation steps and 
additional transformation flow ensures fewer schema issues and less overall overhead.

## Quantitative Impact

The transition from row-based processing using window functions to array-based groupBy operations resulted in 
substantial improvements in shuffle read and write volumes, as well as a reduction in memory and disk spills. 
The following tables summarize the key metrics and provide a comparison between the two approaches.

On to the next [Case Study 4: Transitioning from User Defined Functions to Scalable Spark-Based Solutions](case4.md).
