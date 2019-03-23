# Chapter 4. Structured API Overview

- *Structured APIs* are a tool for manipulation all sorts of data, from unstructured log files to semi-structured CSV files and highly structure Parquet files
- The core types of distributed collection APIs
  - Datasets
  - DataFrame
  - SQL tables and views
- The majority of the Structured APIs apply to both *batch* and *streaming*

## DataFrames and DataSets
  - DataFrames and DataSets are distributed table-like collections with well-defined rows and columns
  - Each columns has type information that must be consistent for every row in the collection.
  - To Spark, DF and DS represent immutable, lazily evaluated plans that specify what operations to apply to data residing at a location to generate some output
  
## Overview of Structured Spark Types
- Internally, Spark uses an engine called *Catalyst* that maintains its own type information through the planning and processing of work
- Even if we use Spark's Structured APIs from python or R, the majority of our manipulations will operate strictly on Spark types, not Python types
  - python이든 scala든 무슨 언어로 돌려도 결국은 Spark types이란 걸로 명령은 돌아간다

## DFs vs DSs
- In essence, within the Structured APIs, there are two more APIs, the "untyped" DF and the "typed" DSs
- the untyped DFs actually have types, but Spark maintains them completely and only check whether those types line up to those specified in the scema at *runtime*
- On the other hand, DSs check whether types conform to the specification at *compile time*
- To Spark, DFs are simply Datasets of Tye "Row"
  - The "Row" type is Spark's internal representation of its optimized in-memory format for computation
- To spark in python, there is no such thing as a Dataset: everything is a DF and therefore we always operate on that optimized format
- 이하 요약: 위의 개념들은 이해하기가 좀 버거울 수 있고 대부분 경우 깊이 알 필요가 없어 자세한 설명은 생략. 대강 DF만 잘 써먹으면 된다

## Columns
- Columns represent a *simple type* like an integer or string, a *complex type* like an array or map or a *null value*

## Rows
- A row is nothing more than a recod of data. Each record in a DF must be of type Row, as we can see when we collect the following DFs

## Spark Types
- Type reference is listed
- Note that JVM only support signed types, when converting unsigned types, keep in mind that it required 1 more bit when stored as singed types

## Overview of Structured API Execution
- An overview of the steps:
  1. Write DF/DS/SQL code
  2. If valied code, Spark converts this to a *Logical Plan*
  3. Spark transforms this *Logical Plan* to *Physical Plan*, checking for optimizations along the way
  4. Sparkthen executes this *Physical Plan*(RDD manipulations) on the cluster
<img src ="https://www.oreilly.com/library/view/spark-the-definitive/9781491912201/assets/spdg_0401.png"> </img>

### Logical Planning 
<img src ="https://www.oreilly.com/library/view/spark-the-definitive/9781491912201/assets/spdg_0402.png"> </img>
- Logical plan only represents a set of abstract transformations that do not refer to executors or drivers, it's purely to convert the user's set of expressions into the most optimized version: *unresolved logical plan*
- This plan is unresolved because although the code might be valid, the tables or columns that it refrers to might or might not exist

- Spark uses the *catalog*, a repository of all table and DF information, to resolve columns and tables in the *analyzer*
- The analyzer might reject the unresolved logical plan if the required table or column name does not exist in catalog
- If the analyzer can resolve it, the result is passed through the *Catalyst Optimizer*
- Catalyst Optimizer is a collection of rules that attempt to optimize the the logical plan by pushing down predicates or seletions
- 요약
  - unresolved logical plan: 문법적으로 올바른지 검사만 해서 최적화시키는 단계
  - analyzer는 테이블 정보가 담긴 catalog를 통해 위의 plan이 올바르게 작동 가능한지 확인
  - Catalyst Optimizer는 다시 push down과 selection까지 해서 plan을 또 optimize

### Physical Planning
<img src ="https://www.oreilly.com/library/view/spark-the-definitive/9781491912201/assets/spdg_0403.png"> </img>
- The *physical plan*, often called a Spark plan, specifies how the logical plan will execute on the cluster by generating different physical execution strategies and comparing them through a cost model, as depicted above
- Physical planning results in a series of RDDs and transformations

### Execution
- Upon selecting a physical plan, Spark runs all of this code over RDDs, the lower-level programming interface of Spark

### Conclusion
