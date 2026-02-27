# AWS Patterns

## S3

### Bucket Naming
- Use lowercase, hyphens, no underscores
- Include environment and purpose: `company-data-lake-prod`

### Key Structure
```
s3://bucket/
├── raw/
│   └── source_name/
│       └── year=2024/month=01/day=15/
├── processed/
│   └── domain/
│       └── table_name/
└── curated/
    └── domain/
        └── table_name/
```

### Python (boto3)
```python
import boto3

s3 = boto3.client('s3')

# Upload
s3.upload_file('local.csv', 'bucket', 'key/file.csv')

# Download
s3.download_file('bucket', 'key/file.csv', 'local.csv')

# List objects
paginator = s3.get_paginator('list_objects_v2')
for page in paginator.paginate(Bucket='bucket', Prefix='prefix/'):
    for obj in page.get('Contents', []):
        print(obj['Key'])
```

## Glue

### Crawler Configuration
- Schedule crawlers for new partitions
- Use exclusion patterns for temp files
- Set table prefix for organization

### ETL Job (PySpark)
```python
from awsglue.context import GlueContext
from pyspark.context import SparkContext

sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session

# Read from catalog
df = glueContext.create_dynamic_frame.from_catalog(
    database="my_database",
    table_name="my_table"
).toDF()

# Write to S3
df.write.mode("overwrite").parquet("s3://bucket/output/")
```

## Lambda

### Handler Pattern
```python
import json
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def handler(event, context):
    logger.info(f"Event: {json.dumps(event)}")
    
    try:
        # Process event
        result = process(event)
        return {
            'statusCode': 200,
            'body': json.dumps(result)
        }
    except Exception as e:
        logger.error(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }
```

## Step Functions

### State Machine Pattern
```json
{
  "StartAt": "Extract",
  "States": {
    "Extract": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:extract",
      "Next": "Transform"
    },
    "Transform": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:transform",
      "Next": "Load"
    },
    "Load": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:load",
      "End": true
    }
  }
}
```

## IAM

### Least Privilege
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::bucket/prefix/*"
    }
  ]
}
```

### Role Trust Policy
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```
