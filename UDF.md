# User-Defined Functions (UDFs)
- From the Spark Apache docs:
"Use the higher-level standard Column-based functions with Dataset operators whenever possible before reverting to using your own custom UDF functions since UDFs are a blackbox for Spark and so it does not even try to optimize them.”



## From textbook
In general, avoiding UDFs is a good optimization opportunity. UDFs are expensive because they force representing data as objects in the JVM and sometimes do this multiple times per record in a query. You should try to use the Structured APIs as much as possible to perform your manipulations simply because they are going to perform the transformations in a much more efficient manner than you can do in a high-level language. There is also ongoing work to make data available to UDFs in batches, such as the Vectorized UDF extension for Python that gives your code multiple records at once using a Pandas data frame. We discussed UDFs and their costs in Chapter 18.



## Performance Consideration
- Python UDFs result in data being serialized between the executor JVM and the Python interpreter running the UDF logic
- It significantly reduces performance as compared to UDF implementations in Java or Scala
- Potential solutions to alleviate this serialization bottleneck include
	- Accessing a Hive UDF from PySpark
		- Note that this approach only provides access to the UDF from the Apache Spark’s SQL query language.
		- Making use of the approach to access UDFs implemented in Java or Scala from PySpark




## Serialization
- Serialization is used for performance tuning on Apache Spark.
- All data that is sent over the network or written to the disk or persisted in the memory should be serialized. 

### MarshalSerializer
- Serializes objects using Python’s Marshal Serializer. This serializer is faster than PickleSerializer, but supports fewer datatypes.

### PickleSerializer
- This serializer supports nearly any Python object, but may not be as fast as more specialized serializers.


## Simple UDF

- Create a python udf function, and register it using *udf* from pyspark.sql.function
- Notice that specifying output type is important. It may not required, but recommended
- Output can be integer, float, or even a array
	- Array type contains only single type
	- For mixed type, use the *StructType*
- To use the registered function, using *select* is convenient
- You MUST name the output column using alias
- One of the parameters is *Nullable*, whose default is True
```
from pyspark.sql.functions import udf
from pyspark.sql.types import IntegerType, ArrayType


# Integer type output
def square(x):
    return x**2
square_udf_int = udf(lambda z: square(z), IntegerType())

df.select('integers', square_udf_int('integers').alias('int_squared'))


# Array type output
def square_list(x):
    return [float(val)**2 for val in x]

square_list_udf = udf(lambda y: square_list(y), ArrayType(FloatType()))

df.select('integer_arrays', square_list_udf('integer_arrays')).show()
```



## Whats NEW: Pandas UDFs (a.k.a. Vectorized UDFs)
- Apache Spark 2.3 release that substantially improves the performance and usability of user-defined functions (UDFs) in Python.
- In python API in version 0.7, user-defined functions operated one-row-at-a-time, and thus suffered from high serialization and invocation overhead.
- Pandas UDFs built on top of Apache Arrow bring you the best of both worlds—the ability to define low-overhead, high-performance UDFs entirely in Python.
- In Spark 2.3, there are two types of Pandas UDFs: scalar and grouped map


## Pandas UDF
- Note that using UDF always requires serialization
- PySpark UDFs work in a similar way as the pandas .map() and .apply() methods for pandas series and dataframes.
- The only difference is that with PySpark UDFs I have to specify the output data type.

### Scalar Pandas UDFs
- Scalar Pandas UDFs are used for vectorizing scalar operations
- To define a scalar Pandas UDF, simply use @pandas_udf to annotate a Python function that takes in pandas.Series as arguments and returns another pandas.Series of the same size.
- Note that there are two important requirements when using scalar pandas UDFs:
	- The input and output series must have the same size.
	- How a column is split into multiple pandas.Series is internal to Spark, and therefore the result of user-defined function must be independent of the splitting.

```
from pyspark.sql.functions import pandas_udf, PandasUDFType

# Use pandas_udf to define a Pandas UDF
@pandas_udf('double', PandasUDFType.SCALAR)
# Input/output are both a pandas.Series of doubles

def pandas_plus_one(v):
    return v + 1

df.withColumn('v2', pandas_plus_one(df.v))
```

### Grouped Map Pandas UDFs
- Grouped map Pandas UDFs
	- 1. first splits a Spark DataFrame into groups based on the conditions specified in the groupby operator
	- 2. applies a user-defined function (pandas.DataFrame -> pandas.DataFrame) to each group
	- 3. combines and returns the results as a new Spark DataFrame.
- Difference between scalar and grouped map:
	- Input: Sereis VS DF
	- Output: Series VS DF
	- Grouping semantics: No grouping semantics VS Defined by “groupby” clause
	- Output size: Same as input size VS *Any size*
	- Return types: A datatype VS A *StructType*
```
@pandas_udf(df.schema, PandasUDFType.GROUPED_MAP)
# Input/output are both a pandas.DataFrame
def subtract_mean(pdf):
    return pdf.assign(v=pdf.v - pdf.v.mean())

df.groupby('id').apply(subtract_mean)

## another example of schema
### @pandas_udf("paper_id long, dupl_index long, sci_names string, sci_ids string, matched_ids string", PandasUDFType.GROUPED_MAP)
```


### References
- https://blog.cloudera.com/blog/2017/02/working-with-udfs-in-apache-spark/
- https://hackernoon.com/apache-spark-tips-and-tricks-for-better-performance-cf2397cac11
- https://www.tutorialspoint.com/pyspark/pyspark_serializers.htm
- https://changhsinlee.com/pyspark-udf/
- https://databricks.com/blog/2017/10/30/introducing-vectorized-udfs-for-pyspark.html
