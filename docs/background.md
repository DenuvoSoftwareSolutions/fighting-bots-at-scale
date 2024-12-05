![spark](spark-small.jpg)

# Background: Databricks and Apache Spark 

[Apache Spark](https://spark.apache.org), created in 2009 at U.C. Berkeley, is an open-source distributed processing framework for big 
data analytics and machine learning. It is well-known for its capabilities to process big data 
volumes in a fault-tolerant, scalable and flexible fashion and achieves this by distributing data processing 
tasks across multiple computers. These features make Apache Spark the number one distributed computing system 
for data analytics and processing used by data science groups in companies, universities, research 
laboratories and data engineers worldwide. [Databricks](https://www.databricks.com), founded in 2013 by the original creators of Apache 
Spark, is a cloud-based platform designed to provide an enhanced environment for big data analytics and 
machine learning. While Apache Spark itself is an open-source distributed processing engine, Databricks offers 
a unified platform that simplifies the use of Spark and extends its capabilities with additional tools and 
services tailored for large-scale data processing.

## Streamlining Big Data Analytics

Although Databricks and Apache Spark are linked, they are inherently different products, and each serves 
different needs within the big data and analytics space. Spark is a cluster computing framework that provides 
speed, ease of use and breadth of use benefits with fast in-memory caching and other modern tools that can 
handle complex analytical queries within:

* Data integration and Extract, Transform, Load (ETL) tasks
* Interactive analytics 
* Machine learning and advanced analytics 
* Real-time data processing 

It should also be noted that deploying and managing a Spark application requires manual setup and management 
of clusters. Users must handle infrastructure, scaling, and configuration.

On the other hand, Databricks is a company and provides a commercial product built on top of Spark, which adds 
additional service features. At the heart of Databricks is an enhanced version of Spark, called the 
“Databricks Runtime”, which delivers superior performance compared to a standard Spark cluster. The platform 
also offers a comprehensive set of data management and developer tools, built-in visualization features, and 
several convenience enhancements for users. In contrast to Apache Spark, it provides a fully managed service, 
abstracting away the complexities of infrastructure management. It is a cloud-based service. Thus, Databricks 
can be deployed across multiple cloud providers ([Azure](https://azure.microsoft.com), [AWS](https://aws.amazon.com) or [Google Cloud](https://cloud.google.com)), and its interface allows for 
quick and easy cluster creation and scaling.

## Understanding Common Bottlenecks 

Apache Spark, the engine behind Databricks, supports large-scale distributed data processing with built-in 
parallelism and fault tolerance. However, as workflows and data volumes scale up, the project structure around 
multiple components can introduce performance bottlenecks. These are often attributed to various aspects of 
Spark’s architecture and the way it interacts with the underlying infrastructure in Databricks.

### Spark Architecture Basic Overview

* Driver program: The driver program is the central controller that coordinates the execution of Spark jobs. This program calls the main program of an application and is the code written by the user. This is where the SparkContext that provides access to the functionalities within Spark is created. Together with the cluster manager, the driver program converts user-defined transformations and actions into tasks and schedules them for execution on the worker nodes. 
* Cluster manager: This component manages resources across the Spark cluster and assigns the tasks to the executors. Databricks typically uses its own managed clusters, but other Spark clusters can be managed using Hadoop YARN, Apache Mesos, or Kubernetes. 
* Executors: These are the worker processes that run on cluster nodes and execute the tasks assigned to them. Each executor is responsible for managing its own memory and disk space to process the data. Typically, an executor’s lifetime is the same as that of the Spark application and the executor is available to accept and process tasks while the spark application is running. In case cluster autoscaling features are used in Databricks, adding and removing worker nodes works dynamically and an executor could be terminated or started on demand. This autoscaling process is handled and coordinated via the Databricks Runtime engine. 
* Tasks: A task is the smallest unit of work in Spark, which is executed by executors. Tasks are derived from Spark’s transformations on Resilient Distributed Datasets (RDDs) or DataFrames. 
* Stages: A job is broken down into stages, each representing a group of tasks that can be executed in parallel. The boundaries of stages are determined by shuffling (e.g., between operations like joins or groupBys). 
* RDDs/DataFrames: RDDs are immutable, distributed collections of objects. Once an RDD is created in a SparkContext, it can be allocated across the different worker nodes and features local caching. DataFrames are built on RDDs but offer optimizations and abstractions for tabular data. They are the primary API for data processing in Spark and demonstrate better performance and scalability than RDDs. 

Let's look at [Spark's performace features](perf.md) next.
