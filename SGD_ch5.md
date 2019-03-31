# Chapter 5. Basic Structured Operations

## Schemas
- A schema defines the column names and types of a DataFrame, We can either let a data source define the schema (called *schema-on-read*) or we can defin it explicitly ourselves
  - For ad hoc analysis, schema-on-read usually works just fine although it may be slow with plain-text like csv or json
  - Howevere it can also lead to precision issues like a *long* type incorrectly set as an integer when readin in a file
  - When using Spark for production ETL, it is often a good idea to define your schemas manually, especially when working with untyped data sources like CSV or JSON
  - Schema inference can vary depending on the type of data that you read in
  - 요약: production 라인에 올릴 땐 schema를 infer하지않는 게 안전. 특히 텍스트 데이터들의 경우에 더더욱
- A schema is a *StructType* made up of a number of fields, *StructFields* that have a name, type, a Boolean flag which specifies whether that column can contain missing or *null* values
- Users can optionally specify associated metadata with that column; The metadata is a way of storing information about this column (Spark uses this in its machine learning library)
- Sample codes
```
# in Python
from pyspark.sql.types import StructField, StructType, StringType, LongType
myManualSchema = StructType([
StructField("DEST_COUNTRY_NAME", StringType(), True),
StructField("ORIGIN_COUNTRY_NAME", StringType(), True),
StructField("count", LongType(), False, metadata={"hello":"world"})
])
df = spark.read.format("json").schema(myManualSchema)\
.load("/data/flight-data/json/2015-summary.json")
```

## Columns and Expressions
- Columns in Spark are siilar to columns in a spreadsheet, R df, or pd df.
- The operations on these are represented as *expressions*
- To Spark, columns are logical constructions that simply represent a value computed on a per-record basis by means of an expression
  - Users cannot manipulate an individual column outside the context of a DF

### columns
- *col* method on the specific DF is used to refer to a specific DF's column

### Expressions
- An *Expression* is a set of transformations on one or more values in a record in a DF
- In the simplest case, expr("somlCel") is equivalent to col("someCol")
- Sample code
```
from pyspark.sql.functions import expr
expr("(((someCol + 5) * 200) - 6) < otherCol")
```

## Records and Rows
- In Spark, each row in a DF is a single record, which is represented as an object of type *Row*

### Creating Rows
- Rows can be created by manually instantiating a Row object with the values that belong in each column
- Sample code
```
from pyspark.sql import Row
myRow = Row("Hello", None, 1, False)
```

## DataFrame Transformations
### Creating DFs
- Sample code
```
from pyspark.sql import Row
from pyspark.sql.types import StructField, StructType, StringType, Longtype
myManualSchema = StructType([
  StructField("some", StringType(), True),
  StructField("col", StringType(), True),
  StructField("names", LongType(), True)
])
myRow = Row("Hello", None, 1)
myDf = spark.createDataFrame([myRow], myManualSchema)
myDf.show()

# or, simply
df = sqlContext.createDataFrame([
  ("a", "Alice", 34),
  ("b", "Bob", 36),
  ("c", "Charlie", 30),
  ("d", "David", 29),
  ("e", "Esther", 32),
  ("f", "Fanny", 36),
  ("g", "Gabby", 60)], ["id", "name", "age"])
```


### select and selectExpr
- *as* or *alias* method is useful
- Sample code
```
df.select(expr("DEST_COUNTRY_NAME AS destination")).show()
df.select(expr("DEST_COUNTRY_NAME").alias("destination")).show()
```
- *selectExpr* is a simple way to build up complex expressions that create new DFs
- Sample code
```
df.selectExpr(
  "*", # all original columns
  "(DEST_COUNTRY_NAME = ORIGIN_COUNTRY_NAME) as withinCountry").show()
```

### Converting to Spark Types(Literals)
- Assign a constant value column using *lit*
- Sample code
```
from pyspark.sql.functions import lit
df.select(expr("*"), lit(1).alias("One")).show()
```

### Adding Columns: using *withColumn*
- Sample code
```
df.withColumn("numberOne", lit(1)).show()
```

### Renamng columns
```
df.withColumnRenamed("DEST_COUNTRY_NAME", "dest").columns
```

### Reserved Characters and Keywords
- To reserve characters like spaces or dashes in column names: Use backtick(`)
- Sample code
```
dfWithLongColName = df.withColumn("This Long Column-Name", expr("ORIGIN_COUNTRY_NAME")) 
# above code works well, but when referencing a column in an expression, backticks are required
dfWithLongColName.selectExpr(
  "`This Long Column-Name`",
  "`This Long Column-Name` as `enw col`").show()
```

### Case Sensitivity
- By default, Spark is case insensitive
- Following make Spark case sensitive by setting the configuration
```
-- in SQL
set spark.sql.caseSensitive true
```

### Removing Columns: *drop*
- Sample code
```df.drop("ORIGIN_COUNTRY_NAME")```

### Changing a Column's Type(cast)
- Convert columns from one type to another by casting the column from one type to another; *cast*
- Sample code
```
df.withColumn("count2" col("count").cast("Long"))
```

### Filtering Rows
- There are two exactly same methods to perform filter operation: *where* and *filter*
- Sample code
```
df.filter(col("count")>2).show()
df.where("count < 2").show
```
- Spark automatically performs all filtering operations at the same time regardless of the filtering ordering
  - This means that when multiple filters are needed just chain them sequntially and let Spark handle the rest

### Getting Unique Rows: *distinct*
```df.select("ORIGIN_COUNTRY_NAME", "DEST_COUNTRY_NAME").distinct().count()```

### Random Samples
- Note that only the probabilty that each row is sampled is specified, not the number of samples!
- The three arguments are used: replacment as boolean, fraction as float, seed as integer
- Sample code: ```df.sample(False, 0.5, 85)```

### Random Split
- This is often usd with machine learning algorithms to create trainig, validation, and test sets
- Note that the fraction is not requiared to be sum to one but should be floats
- Sample code: ```dataFrames = df.randomSplit([1.0, 3.0]), 85)```

### Concatenating and Appending Rows (Union)
- DFs are immutable: they cannot be appended
- To union two DFs, they must be sure to have the same schema and number of columns: otherwise, the union will fail
- Sample codes
```
from pyspark.sql import Row
schema = df.schema
newRows= [
  Row("New Country", "Other Country", Integer),
  Row("New Country 2", "Other Country 3", )
  ]
