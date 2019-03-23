# Part I. Gentle Overview of Big Data and Spark.md

# Chapter 1. What is Apache Spark?
## Apache Spark's Philosophy
### Unified: 하나에 다 담았다!
  - Spark is designed to support a wide range of data analytics tasks, ranging from simple data loading and SQL queries to machine learning and streaming computation, over the same computing engine and with a consistent set of APIs
### Computing engine: 저장 영역과 분리함으로써 db 없이도 임의의 저장소(s3 등)와 쉽게 결합
  - Spark limits its scope to a computing engine
  - By this, Spark handles loading data from storage systems and performing computation on it, not permanent storage as the end itself
  - Data is expensive to move so Spark focuses on performing computations over the data, no matter where it resides.
  - Although Spark runs well on Hadoop storage, it is also used broadly in environments for which the Hadoop architecture does not make sense, such as the *public cloud*
### Libries: core는 둔 채로 library를 통해 다양한 API 제공
  - Spark's standard libraries are actually the bulk of the open source project
  - Spark core engine itself has changed little since it was first released

## History of Spark
  - MapReduce engine made it both challenging and inefficient to build large applications: in MapReduce, each pass had to be written as a separate MapReduce job
  - To address this problem, Spark is designed based on *functional programming*


# Chapter 2. A Gentle Introduction to Spark
## Spark Applications
  - spark applications consist of a *driver process* and a set of *executor processes*
    - Driver process: Maintaining information, responding to a user's program or input, analyzing, distributing, and scheduling work across the executors
    - Executors: responsible for actually carrying out the work that the driver assigns them
## Spark's Language APIs
  - Scala: Spark is primarily written in Scala, making it Spark's *default* language.
  - Python: Python supports nearly all constructs that Scala supports.
## Partitions
  - To allow every executor to perform work in paralle, Spark breaks up the data into chunks called *partitions*
  - An important thing to note is that with DataFrames users do not (for the most part) manipulate partitions manually or individually. Users simply specify high-level transformations of data in the physical partitions, and Spark determins how this work will actually execute on the cluster
## Transformations: 실행되지 않는(lazy) 변환들
  - In Spark, the core data structures are *immutable*, meaning they cannot be changed after they're created
  - To *change* a df, you need to instruct Spark how you would like to modify it to do what you want: *Transformation*
  - Narrow dependencies: Each input partition will contribute to only one output partition: no dependency among partitions!
  - Wide dependency: input partitions contributing to many output partitions: shuffle needed!
    - With narrows tran, Spark will automatically perform an operation called pipelining
## Lazy Evaluation: execution을 최대한 미룬 뒤, action 시에 쌓인 execution들을 효율적으로 재정리
  - Spark will wait until the very last moment(*action*) to execute the graph of computation instructions
  - In Spark, instead modifying the data immediately when user express some operation, user buil up a plan(*lineage*) of transformations
  - Spark compiles this plan as efficiently as possible across the cluster
  - Predicate pushdown: filtering이 뒤에 달림 -> 앞에 것들 쓸모없는 처리가 아까움 -> Spark가 알아서 filtering을 앞으로 옮김 -> 중간에 쓸모없는 처리가 사라짐
## Actions: 쌓여온 transformation들의 실제 실행
  - There are three kinds of actions; viewing data / colleting data to object in the respective language / write to output
  
# Chapter 3. A Tour of Spark's Toolset
- spark-submit: A built-in command-line tool. It offers several controls with which you can specify the resources your application needs as well as how it should be run and its command-line arguments

## Datasets: Type-Safe Structure APIs
  - It write statically typed code in Java and Scala
  - *Not available in Python and R*, because those languages and dynamically typed
  - It  gives users the ability to assign a Java/Scala class to the records within a DataFrame and manipulate it as collection of typed objects

## Structure Streaming
  - A high-level API for stream procesing that became production-ready in Spark 2.2
  - It allows user to rapidly and quickly extract value out of streaming systems with virtually *no code changes*
  - Write a batch job as a way to prototype it and then convert it to a streaming job
  - Code
```
# Static version
from pyspark.sql.functions import window, column, desc, col
staticDataFrame = (
spark.read.forma("csv")
```
