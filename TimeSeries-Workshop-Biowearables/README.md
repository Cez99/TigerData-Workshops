# TimeSeries Workshop - Biowearables

## Overview

This workshop demonstrates advanced time-series data analysis using PostgreSQL and TigerData for Biowearables,  
eg track health metrics from wearable devices including:

- Heart Rate (BPM)
- Blood Pressure (Systolic/Diastolic)
- Blood Oxygen Saturation (SpO2)
- Body Temperature
- Steps Count
- Sleep Quality Score

Use case: Remote patient monitoring, fitness tracking, and health analytics


## What You'll Learn

- **Hypertables**: Convert regular PostgreSQL tables into time-series optimized hypertables

- **Working with Bioweareables Data**: Work with sample generated bioweareables data

- **Bioweareables Data Analysis**: Generate daily health summaries & other health monitoring queries

- **Columnar Compression**: Achieve 10x storage reduction while improving query performance

- **Continuous Aggregates**: Pre-compute aggregations for lightning-fast analytics

- **Real-time Updates**: Build materialized views that update automatically as new data arrives

## Contents

- **`analyze-bioweareables-data-psql.sql`**: Complete workshop for psql command-line interface

## Prerequisites

- Timescale Cloud account (get free 30 days at <https://console.cloud.timescale.com/signup>)

- Optional, but recommended - install psql CLI https://www.tigerdata.com/blog/how-to-install-psql-on-mac-ubuntu-debian-windows

- Basic knowledge of SQL and relational data concepts

## Data Structure

### Health Data Table

```sql
CREATE TABLE health_data (
   time TIMESTAMPTZ NOT NULL,
   device_id INTEGER,
   heart_rate INTEGER,  -- BPM
   blood_pressure_systolic INTEGER,  -- mmHg
   blood_pressure_diastolic INTEGER,  -- mmHg
   spo2 DOUBLE PRECISION,  -- Blood oxygen saturation %
   body_temperature DOUBLE PRECISION,  -- Celsius
   steps_count INTEGER,  -- Daily cumulative steps
   sleep_quality_score DOUBLE PRECISION,  -- 0-100 scale
   activity_level TEXT  -- sedentary, light, moderate, vigorous
) WITH (
   tsdb.hypertable,
   tsdb.partition_column='time',
   tsdb.segmentby='device_id',
   tsdb.orderby='time DESC'
);

```

### Users Table

```sql
CREATE TABLE users(
   id SERIAL PRIMARY KEY,
   name VARCHAR(100), 
   age INTEGER,
   gender VARCHAR(10),
   medical_conditions TEXT
);
```

### Wearable Devices Table

```sql
CREATE TABLE wearable_devices(
   id SERIAL PRIMARY KEY,
   user_id INTEGER,
   device_type VARCHAR(50),  -- smartwatch, fitness_band, patch, ring
   device_model VARCHAR(100),
   firmware_version VARCHAR(20)
);

```

## Key Features Demonstrated

### 1. Hypertable Creation

Convert regular tables into time-series optimized hypertables:

```sql
SELECT create_hypertable('health_data', by_range('time'));
```

### 2. Data Generation

Create data for daily health analysis

```sql
SELECT
   time_bucket('1 day', time) AS day,
   device_id,
   AVG(heart_rate) AS avg_heart_rate,
   MIN(heart_rate) AS min_heart_rate,
   MAX(heart_rate) AS max_heart_rate,
   AVG(blood_pressure_systolic) AS avg_bp_systolic,
   AVG(blood_pressure_diastolic) AS avg_bp_diastolic,
   AVG(spo2) AS avg_spo2,
   AVG(body_temperature) AS avg_temperature,
   MAX(steps_count) AS total_steps,
   AVG(sleep_quality_score) AS avg_sleep_quality,
   COUNT(*) AS readings_count
FROM health_data
WHERE health_data.time >= NOW() - INTERVAL '14 days'
GROUP BY day, device_id;
```

### 3. Columnar Compression 

Enable ~10x storage compression with improved query performance:

```sql
ALTER TABLE health_data 
SET (
    timescaledb.enable_columnstore = true, 
    timescaledb.segmentby = 'device_id',
    timescaledb.compress_orderby = 'time DESC'
);
```

### 4. Real Time Continuous Aggregates

Create self-updating materialized views for instant analytics:

```sql
CREATE MATERIALIZED VIEW daily_health_summary
WITH (
   timescaledb.continuous,
   timescaledb.materialized_only = false
) AS
SELECT
   time_bucket('1 day', time) AS day,
   device_id,
   AVG(heart_rate) AS avg_heart_rate,
   MIN(heart_rate) AS min_heart_rate,
   MAX(heart_rate) AS max_heart_rate,
   AVG(blood_pressure_systolic) AS avg_bp_systolic,
   AVG(blood_pressure_diastolic) AS avg_bp_diastolic,
   AVG(spo2) AS avg_spo2,
   AVG(body_temperature) AS avg_temperature,
   MAX(steps_count) AS total_steps,
   AVG(sleep_quality_score) AS avg_sleep_quality,
   COUNT(*) AS readings_count
FROM health_data
GROUP BY day, device_id;
```




## Getting Started

### Using psql Command Line

1. Follow the instructions in `analyze-bioweareables-data-psql.sql`

2. The script will generate sample data and guide you through each step

3. Includes timing comparisons to demonstrate performance improvements

## Workshop Highlights

- **Bioweareables Data**: Work with sample generated bioweareables data 
- **Performance Optimization**: Compare query times before and after compression
- **Storage Efficiency**: See 10x storage reduction with columnar compression
- **Automatic Updates**: Demonstrate real-time continuous aggregate updates
- **Production Ready**: Learn policies for automatic compression and aggregate refreshing

## Performance Benefits

- **Hypertables**: Automatic partitioning for optimal time-series queries
- **Columnar Storage**: 90% storage reduction with faster analytical queries
- **Continuous Aggregates**: Sub-second response times for complex aggregations
- **Automatic Policies**: Set-and-forget data lifecycle management

## License
