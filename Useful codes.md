## Source
- Official documents http://spark.apache.org/docs/latest/api/python/
- From Udemy lecture
- Google(link missed)


## codes
- importing library
```
from pyspark.sql.types import StructField, StringType, IntegerType, StructType, LongType, FloatType
import pyspark.sql.functions as F
```

- set schema
```
data_schema = [StructField('age', IntegerType(), True),
				StructField('name', StringType(), True)]	
final_struc = StructType(field=data_schema)
#set schema while reading a file
df = spark.read.json('people.json', schema=final_struc)
#get the schema
df.printSchema()
```

- column
```
#assign a scolumn
df.withColumn('double_age', df['age']*2)
#rename a column
df.withColumnRenamed('age', 'my_new_age')
#transform the type of a column
changedTypedf = df.withColumn("age", df["age"].cast(StringType()))
```

- sql
```
#register a dataframe as a sql temporary vie
wdf.createOrReplaceTempView('people')
result = spark.sql("SELECT * FROM people")
```

- groupby
```
#group by and aggregate operation
df.groupBy('Company').count().show()
df.agg({'Sales':'max'}).show()

group_data = df.groupBy('Company')
group_data.agg({'Sales':'max'})

#group by  collect
(df
  .groupby("id")
  .agg(F.collect_set("code"),
       F.collect_list("name"))
  .show())
# to melt the array column into multiple rows: F.explode("ids")
#or
df.select(
        "num",
        f.split("letters", ", ").alias("letters"),
        f.posexplode(f.split("letters", ", ")).alias("pos", "val")
    )\
    .show()

```

- ordering
```
df.orderBy("Sales").show()
df.orderBy(df['Sales'].desc()).show()
```

- generate index
```
#generate id
#note that there might be jumps between ids
#to avoid it, set the number of partition as 1, if possible (for small dataframe)
url_df.withColumn('dummy', F.monotonically_increasing_id())
```

- row concatenate
```
#row concatenate
from functools import reduce  # For Python 3.x
from pyspark.sql import DataFrame

surnames_part = reduce(DataFrame.unionAll, [rankers, small_surnames])
```

- find duplicates
```
(df
   .groupBy(df.columns[1:])
   .agg(collect_list("id").alias("ids"))
   .where(size("ids") > 1))
(paper_df
   .groupBy('paper_title')
   .agg(F.concat_ws(",", F.collect_list('paper_id')).alias('ids'))
   .filter("ids like '%,%'"))
```

- fill na
```
paper.fillna({'citation_count_filter':0})
```


### udfs
- https://databricks.com/blog/2017/10/30/introducing-vectorized-udfs-for-pyspark.html
- https://danvatterott.com/blog/2018/09/06/python-aggregate-udfs-in-pyspark/
- https://florianwilhelm.info/2017/10/efficient_udfs_with_pyspark/
