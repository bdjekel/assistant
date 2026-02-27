# SQL Patterns

## Query Optimization

### Use CTEs for Readability
```sql
WITH active_users AS (
    SELECT user_id, email
    FROM users
    WHERE status = 'active'
),
recent_orders AS (
    SELECT user_id, COUNT(*) as order_count
    FROM orders
    WHERE created_at > CURRENT_DATE - INTERVAL '30 days'
    GROUP BY user_id
)
SELECT 
    au.email,
    COALESCE(ro.order_count, 0) as order_count
FROM active_users au
LEFT JOIN recent_orders ro ON au.user_id = ro.user_id;
```

### Avoid SELECT *
```sql
-- Bad
SELECT * FROM orders;

-- Good
SELECT order_id, user_id, amount, created_at FROM orders;
```

### Filter Early
```sql
-- Filter in subquery, not after join
SELECT o.*, u.email
FROM (
    SELECT * FROM orders WHERE status = 'completed'
) o
JOIN users u ON o.user_id = u.user_id;
```

## Window Functions

```sql
-- Row number for deduplication
SELECT *
FROM (
    SELECT *,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) as rn
    FROM events
) t
WHERE rn = 1;

-- Running totals
SELECT 
    date,
    amount,
    SUM(amount) OVER (ORDER BY date) as running_total
FROM transactions;

-- Lag/Lead for comparisons
SELECT 
    date,
    revenue,
    LAG(revenue, 1) OVER (ORDER BY date) as prev_revenue,
    revenue - LAG(revenue, 1) OVER (ORDER BY date) as change
FROM daily_metrics;
```

## Data Modeling

### Fact Tables
- Contain measurable events (orders, clicks, transactions)
- Include foreign keys to dimension tables
- Include timestamps and numeric measures

### Dimension Tables
- Contain descriptive attributes (users, products, locations)
- Include surrogate keys
- Support slowly changing dimensions (SCD Type 2)

## Anti-Patterns

- Avoid `DISTINCT` to fix duplicate issues — find the root cause
- Avoid correlated subqueries — use JOINs instead
- Avoid `OR` in JOIN conditions — use UNION instead
- Avoid implicit type conversions in WHERE clauses
