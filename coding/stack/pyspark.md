# PySpark Patterns

## DataFrame Operations

### Column Selection
```python
df.select("col1", "col2")
df.select(col("col1"), col("col2").alias("renamed"))
```

### Filtering
```python
df.filter(col("status") == "active")
df.filter((col("age") > 18) & (col("country") == "US"))
```

### Aggregations
```python
df.groupBy("category").agg(
    count("*").alias("count"),
    sum("amount").alias("total"),
    avg("amount").alias("average")
)
```

## Performance Tuning

### Broadcast Joins
Use for small dimension tables (< 10MB):
```python
from pyspark.sql.functions import broadcast

result = large_df.join(broadcast(small_df), "key")
```

### Partitioning
```python
# Repartition for parallelism
df.repartition(200, "partition_key")

# Coalesce to reduce partitions (no shuffle)
df.coalesce(10)
```

### Caching
```python
df.cache()  # Memory only
df.persist(StorageLevel.MEMORY_AND_DISK)
```

## UDFs

Prefer built-in functions. When UDFs are necessary:

```python
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType

@udf(returnType=StringType())
def clean_text(text):
    return text.strip().lower() if text else None

df.withColumn("cleaned", clean_text(col("raw_text")))
```

## Window Functions

```python
from pyspark.sql.window import Window

window = Window.partitionBy("user_id").orderBy("timestamp")

df.withColumn("row_num", row_number().over(window))
df.withColumn("running_total", sum("amount").over(window))
```

## Schema Definition

```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

schema = StructType([
    StructField("id", IntegerType(), nullable=False),
    StructField("name", StringType(), nullable=True),
])

df = spark.read.schema(schema).json("path/to/data")
```
