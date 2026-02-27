# Databricks Patterns

## Unity Catalog

### Three-Level Namespace
```
catalog.schema.table
```

### Creating Objects
```sql
-- Create catalog
CREATE CATALOG IF NOT EXISTS my_catalog;

-- Create schema
CREATE SCHEMA IF NOT EXISTS my_catalog.my_schema;

-- Create table
CREATE TABLE my_catalog.my_schema.my_table (
    id BIGINT,
    name STRING,
    created_at TIMESTAMP
) USING DELTA;
```

### Grants
```sql
GRANT USE CATALOG ON CATALOG my_catalog TO `data-engineers`;
GRANT USE SCHEMA ON SCHEMA my_catalog.my_schema TO `data-engineers`;
GRANT SELECT ON TABLE my_catalog.my_schema.my_table TO `data-analysts`;
```

## Delta Lake

### MERGE (Upsert)
```sql
MERGE INTO target t
USING source s
ON t.id = s.id
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;
```

### Time Travel
```sql
-- Query historical version
SELECT * FROM my_table VERSION AS OF 5;
SELECT * FROM my_table TIMESTAMP AS OF '2024-01-01';

-- Restore to previous version
RESTORE TABLE my_table TO VERSION AS OF 5;
```

### Optimize
```sql
-- Compact small files
OPTIMIZE my_table;

-- Z-order for query performance
OPTIMIZE my_table ZORDER BY (date, user_id);

-- Vacuum old files (default 7 days retention)
VACUUM my_table;
```

## Jobs

### Job Configuration (YAML)
```yaml
resources:
  jobs:
    etl_pipeline:
      name: "ETL Pipeline"
      schedule:
        quartz_cron_expression: "0 0 6 * * ?"
        timezone_id: "America/New_York"
      tasks:
        - task_key: "extract"
          notebook_task:
            notebook_path: "/Repos/project/notebooks/extract"
        - task_key: "transform"
          depends_on:
            - task_key: "extract"
          notebook_task:
            notebook_path: "/Repos/project/notebooks/transform"
```

## Notebooks

### Widget Parameters
```python
dbutils.widgets.text("date", "2024-01-01", "Processing Date")
date = dbutils.widgets.get("date")
```

### Secrets
```python
secret = dbutils.secrets.get(scope="my-scope", key="api-key")
```

### Exit with Value
```python
dbutils.notebook.exit("SUCCESS")
```

## Best Practices

1. **Use Unity Catalog** for governance
2. **Partition by date** for time-series data
3. **Z-order on filter columns** for query performance
4. **Schedule OPTIMIZE** weekly for large tables
5. **Use Repos** for version control
6. **Parameterize notebooks** with widgets
