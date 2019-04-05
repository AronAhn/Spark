# Chapter 7. Aggregations
- Aggregation is used to summarize (numerical) data usually by means of some grouping
- The grouping types in Spark
  - Simplest grouping: Jusst summarize a complete DF by performing an aggregation in a select statement
  - *Group by*: Specify one or more keys as well as one or more aggregation functions to trasfrom the value columnss
  - *Window*: Its the same to group by, except that the rows input to the fuction are some how related to the current row
  - *Grouping set*: Aggregate at multiple different levels.
    - Grouping sets are available as a primitive in SQL and via rollups and cube in DFs
  - *Rollup*:The groupby is summarized hierarchically
  - *Cube*: The groupby is summarized across all combinations of columns
- When performing calculations over big data, its often much cheaper to simply request an approximate to a reasonable degree of accuracy
- Sample code
```
df = spark.read.format("csv")\
.option("header", "true")\
.option("inferSchema", "true")\
.load("/data/retail-data/all/*.csv")\
.coalesce(5)
df.cache()
df.createOrReplaceTempView("dfTable")
df.count() # note that the count function is evaluated instead of a lazy transformation
```

## Aggregation Functions
- count: To perform it as a transformation instead of an actions, we can do one of two thins
  - Specify a specific column to count
  - All the columns by using count(*) or count(1) to represent that we want to count every row as the literal one, as shown in this example
  	- Note that when performing a count(*), Spark will count null values
  	-When counting an individual column, Spark will not count the null values
- countDistinct: count the number of unique group
- approx_count_distinct: Approximation to a certain degree of accuracy
  - For rsd < 0.01, it is more efficient to use countDistinct()
- first and last: Get the first and last values from a DF
- min, max, sum, sumDistinct, average, skewness and kurtosis are listed
 - variance and standrad deviaiton: Spark has both the formula for the var and sd of sample and the population
   - By default, Spark performs the formula for the sample sd or var
	- Sample code
```
# in Python
from pyspark.sql.functions import var_pop, stddev_pop
from pyspark.sql.functions import var_samp, stddev_samp
df.select(var_pop("Quantity"), var_samp("Quantity"),
stddev_pop("Quantity"), stddev_samp("Quantity")).show()
```
- Covariance and Correlation
  - Some functions compare the interactions of the values in two difference columns together
  - (From Spark doc) Currently only supports “pearson”
  - (From Spark doc) Compute the correlation using Pearson's method. Enter "spearman" for Spearman's method. (With series)
  - Sample code
```
from pyspark.mllib.stat import Statistics
seriesX = sc.parallelize([1.0, 2.0, 3.0, 3.0, 5.0])  # a series
# seriesY must have the same number of partitions and cardinality as seriesX
seriesY = sc.parallelize([11.0, 22.0, 33.0, 33.0, 555.0])
print("Correlation is: " + str(Statistics.corr(seriesX, seriesY, method="pearson")))
```

## Aggregating to Complex Types
- In Spark, user can perform aggregations not just of numerical values using formulas, user can also perform them on complex types
- Grouping: group data on one column and perform some calculations on the other columns that end up in that group
- Grouping with Expressions
  - Rather than passing functions as an expression into a select statement, specify them within agg
  - Sample code
```
from pyspark.sql.functions import count
df.groupBy("InvoiceNo").agg(
    count("Quantity").alias("quan"),
    expr("count(Quantity)")).show()
```

- Grouping with Maps
  - Sometimes, it can be easier to specify the transformations as a series of *Maps* for which the key is the column, and the value is the aggregation function (as a string)
  - Sample code
```
df.groupBy("InvoiceNo").agg(expr("avg(Quantity)"),expr("stddev_pop(Quantity)")).show()
```

- Window Functions: It carries out some unique aggregations by either computing some aggregations on a specific "window" of data
  - A window function calculates a return value for every input row of a table based on a group of rows, calle a frame
  - A common use case is to take a look at a rolling average of some value for which each row represents one day
  - Defining frames will be covered a little later
  - Sample codes
```
from pyspark.sql.functions import col, to_date, desc, max, dense_rank, rank
from pyspark.sql.window import Window


dfWithDate = df.withColumn("date", to_date(col("InvoiceDate"), "MM/d/yyyy H:mm"))
dfWithDate.createOrReplaceTempView("dfWithDate")

windowSpec = (Window
	.partitionBy("CustomerId", "date") # note that the partitions by is unrelated to the partitioning scheme concept that covered thus far
	.orderBy(desc("Quantity"))
	.rowsBetween(Window.unboundedPreceding, Window.currentRow))

maxPurchaseQuantity = max(col("Quantity")).over(windowSpec)

purchaseDenseRank = dense_rank().over(windowSpec)
purchaseRank = rank().over(windowSpec)
```

- Grouping Sets: An aggregation across multiple groups
  - Grouping sets are a low-level tool for combining sets of aggregations together
  - Grouping sets depend on null values for aggregation levels
  - The *GROUPING SETS* operator is only available in SQL. 
  	- To perform the same in DFs, use the *rollup* and *cube* operators, which gives the same results
  - Sample codes
```
dfNoNull = dfWithDate.drop()
dfNoNull.createOrReplaceTempView("dfNoNull")
SELECT CustomerId, stockCode, sum(Quantity) FROM dfNoNull
GROUP BY customerId, stockCode GROUPING SETS((customerId, stockCode),())
ORDER BY CustomerId DESC, stockCode DESC
```

- Rollups: A multidimensional aggregation that performs a variety of group-by style calculations
  - Note that a *null* in both rollup columns specifies the grand total across both of those columns
  - Sample code
```
rolledUpDF = dfNoNull.rollup("Date", "Country").agg(sum("Quantity"))\
.selectExpr("Date", "Country", "`sum(Quantity)` as total_quantity")\
.orderBy("Date")
rolledUpDF.show()
```

- Cube: It takes the rollup to a level deeper
  - It can make a table that includes the folloings
    - The total across all dates and countires
    - The total for each date across all countries
    - The total for each country on each date
    - The total for each country across all dates
  - Sample code
```
from pyspark.sql.functions import sum
dfNoNull.cube("Date", "Country").agg(sum(col("Quantity")))\
  .select("Date", "Country", "sum(Quantity)").orderBy("Date").show()
```

- Grouping Metadata
  - When query the aggregation levels so that user can easily filter them dwon accordingly

- Pivot: It makes it possible to convert a row into a column
  - Sample code
```
pivoted = dfWithDate.groupBy("date").pivot("Country").sum()
```

- User-Defined Aggregation Functions
  - It is a way for users to define their own aggregation functiosn based on custom formulae or business rules
  - Spark maintains a single *AggregationsBuffer* to store intermediate result for every group of input data
  - This text books only handles UDAFs in Scala so passed the details 
  - Refer the following pages to handle UDAFs in Pyspark
    - https://changhsinlee.com/pyspark-udf/
    - https://danvatterott.com/blog/2018/09/06/python-aggregate-udfs-in-pyspark/
  - Pandas UDF example
```    
@pandas_udf("str_col string, bool_col boolean, long_col long", PandasUDFType.GROUPED_MAP)
def pandas_udf_function(group):
    # Some procedures on group
    return group
```
