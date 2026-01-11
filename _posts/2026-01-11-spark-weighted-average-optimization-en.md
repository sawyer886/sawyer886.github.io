---
layout: post
title: Spark Weighted Average Optimization - A Performance Optimization Guide
categories:
  - Big Data Engineering
  - Performance Optimization
  - Apache Spark
tags:
  - Spark
  - Data Processing
  - Performance Optimization
  - SQL Optimization
lang: en
---

## Introduction

In Spark data processing, we frequently need to handle complex data aggregation and computation tasks. This article shares how to optimize weighted average calculations for arrays in Spark, comparing multiple approaches to identify the most efficient implementation.

## Problem Statement

Given device data with user applications, we need to calculate the weighted average embedding vector for all applications on each device. Data structure:

```
- deviceId: Device ID
- appId: Application ID
- appWeight: Application weight
- appEmbedding: Application embedding vector (1024-dimensional double array)
```

The expected output is the weighted average embedding vector for all applications on each device.

## Solution Comparison

### Solution 1: Explode + Aggregation (Baseline Approach)

**Approach**: Explode embedding vector elements individually, calculate weighted averages, then recombine.

**Drawbacks**:
- explode generates 300*1024=307,200 intermediate records
- Creates massive shuffle operations
- Severely impacts performance

```sql
SELECT 
  deviceId,
  concat_ws(',', emb) as avgEmbedding
FROM (
  SELECT 
    deviceId,
    index,
    avg(value) as emb
  FROM tmp.deviceWeightExplode t1
  LATERAL VIEW posexplode(appIdWeightArr) tmplv AS index, value
  GROUP BY deviceId, index
) t2
```

### Solution 2: Spark High-Order Array Functions Optimization (Recommended)

**Approach**: Utilize Spark 3.x's array operation functions (transformArray, aggregateArray, zipWith) for vectorized computation.

**Advantages**:
- Direct array operations on Executors
- Minimized shuffle, avoiding massive intermediate records
- 10x+ performance improvement

```sql
WITH agg AS (
  SELECT 
    deviceId,
    aggregate(
      collect_list(transform(appIdWeightArr, x -> CAST(x AS DOUBLE))),
      array_repeat(CAST(0.0 as double), 1024),
      (acc, v) -> zipwith(acc, v, (x, y) -> x + y)
    ) as sumEmbedding,
    COUNT(1) as appCnt
  FROM tmp.deviceWeightExplode
  GROUP BY deviceId
),
avgemb AS (
  SELECT 
    deviceId,
    transform(sumEmbedding, x -> x / appCnt) as avgEmbedding
  FROM agg
)
SELECT 
  deviceId,
  concat_ws(',', emb) as embedding_values,
  idx + 1 as dimension_index,
  value
FROM avgemb
LATERAL VIEW posexplode(avgEmbedding) tmplv AS idx, value
```

## Core Optimization Techniques

### 1. Data Type Conversion

**Challenge**: `appIdWeightArr` is of type `ARRAY<DECIMAL(1,1)>`, but lambda functions process `ARRAY<DOUBLE>` type.

**Solution**:

```sql
-- Use transform function for type conversion
transform(appIdWeightArr, x -> CAST(x AS DOUBLE))

-- Initialize accumulator array with matching type
array_repeat(CAST(0.0 as double), 1024)
```

### 2. Minimize Shuffle Operations

- Use `collect_list` to aggregate data at Executor level
- Leverage `aggregate` for vectorized computation
- Only explode when necessary

### 3. Memory Optimization

- 1024-dimensional double-precision floating-point array ≈ 8KB
- 300 applications * 1024 dimensions = 3.072 million records (before explosion)
- Direct processing VS. post-explode processing memory difference: **At least 10x**

## Performance Comparison

| Approach | Processing Time | Intermediate Records | Shuffle Volume | Characteristics |
|----------|------------------|----------------------|-----------------|------------------|
| Solution 1 (explode) | 3 hours | 3 million+ | Massive | Basic but inefficient |
| Solution 2 (high-order functions) | 18 minutes | Minimal | Minimal | **Recommended** |
| Performance Gain | **10x** | - | - | - |

## Implementation Recommendations

1. **Upgrade Spark Version**: Ensure Spark 3.3+ to support all high-order array functions

2. **Memory Tuning**:
   ```
   spark.executor.memory = 8g
   spark.driver.memory = 4g
   spark.memory.fraction = 0.8
   ```

3. **Parallelism Optimization**:
   ```
   spark.default.parallelism = 200
   spark.sql.shuffle.partitions = 200
   ```

4. **Caching Strategy**:
   ```sql
   CACHE TABLE tmp.deviceWeightExplode;
   ```

## Summary

By leveraging Spark 3.x's high-order array functions, we can significantly improve large-scale data aggregation performance. Key points include:

- ✅ Replace explode with `transform`, `aggregate`, `zipWith`
- ✅ Avoid unnecessary shuffle operations
- ✅ Ensure data type conversion consistency
- ✅ Properly configure executor memory and parallelism

This solution has been validated in production environments with stable performance improvements of **10x or more**.

---

**Related References**:
- [Spark SQL Functions Documentation](https://spark.apache.org/docs/latest/sql-ref-functions.html)
- [Spark Performance Tuning Guide](https://spark.apache.org/docs/latest/tuning.html)
