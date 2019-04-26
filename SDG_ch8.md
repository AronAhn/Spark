# Chapter 8. Joins

## Join Expressions
- A *join* brings together two sets of data, the *left* and the *right*, by comparing the value of one or more *keys* of the left and right
- The most common join expression is *equi-join*
  - It compares whether the specified keys in both left and right datasets are equal

## Join Types
- Inner, outer, left outer, right outer
- Left semi joins: keep the rows in the left, and only the left, dataset where the key appears in the right dataset
- Left anti joins: keep the rows in the left, and only the left, dataset where they do not appear in the right datset
- Natural join: perform a join by implicitly matching the columns between the two datasets with the same name
- Cross(or Cartesian) join: match every row in the left dataset with every row in the right datset
  - It may cause an absolute explosion in the number of rows contained in the resulting DF


## Challenges When Using Joins


### Join on Complex Types
- Just do it!
- Sample code
```
person = spark.createDataFrame([
(0, "Bill Chambers", 0, [100]),
(1, "Matei Zaharia", 1, [500, 250, 100]),
(2, "Michael Armbrust", 1, [250, 100])])\
.toDF("id", "name", "graduate_program", "spark_status")
graduateProgram = spark.createDataFrame([
(0, "Masters", "School of Information", "UC Berkeley"),
(2, "Masters", "EECS", "UC Berkeley"),
(1, "Ph.D.", "EECS", "UC Berkeley")])\
.toDF("id", "degree", "department", "school")
sparkStatus = spark.createDataFrame([
(500, "Vice President"),
(250, "PMC Member"),
(100, "Contributor")])\
.toDF("id", "status")
person.withColumnRenamed('id', 'personId').join(sparkStatus, F.expr("array_contains(spark_status, id)")).show()
```

### Handling Duplicate Column Names
- Use "on" like pandas merge: ```df.join(df2, 'col_to_join')```
- Drop or rename the columns

### How Spark Performs Join
- To understand how Spark performs joins, understanding the two core resources are needed: *node-to-node communication stategy* and *per node computation strategy*
- Big table-to big table: shuffle join
  - Every node talks to every other node and they share data according to which node has a certain key or set of keys
  - These joins are expensive 
- Big table-to-small table: broadcast join
  - When the table is small enough to fit into the memory of a single worker node
- With the DF API, we can also explicitly give the optimizer a *hint*
- Note that correctly partition the data prior to a join
