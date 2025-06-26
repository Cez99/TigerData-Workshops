# TimeSeries Workshop - Financial Data Analysis

## Overview

This workshop demonstrates advanced time-series data analysis using PostgreSQL and TimescaleDB for financial applications. You'll learn how to work with cryptocurrency tick data, create candlestick charts from OHLCV (Open, High, Low, Close, Volume) data, and implement performance optimization techniques including columnar compression and continuous aggregates.

![candlestick data](https://i.imgur.com/DNUzIPD.png)

## What You'll Learn

- **Hypertables**: Convert regular PostgreSQL tables into time-series optimized hypertables

- **Real-time Financial Data**: Work with actual cryptocurrency tick data from Twelve Data

- **OHLCV Analysis**: Generate candlestick chart data for financial asset visualization

- **Columnar Compression**: Achieve 10x storage reduction while improving query performance

- **Continuous Aggregates**: Pre-compute aggregations for lightning-fast analytics

- **Real-time Updates**: Build materialized views that update automatically as new data arrives

## Contents

- **`analyze-financial-data-psql.sql`**: Complete workshop for psql command-line interface

- **`analyze-financial-data-UI.sql`**: Workshop version optimized for Timescale Cloud Console UI

## Prerequisites

- Timescale Cloud account (get free 30 days at <https://console.cloud.timescale.com/signup>)

- Optional, but recommended - install psql CLI https://www.tigerdata.com/blog/how-to-install-psql-on-mac-ubuntu-debian-windows

- Basic knowledge of SQL and financial data concepts## Data StructureThe workshop uses two main datasets:

### Crypto Ticks Table

```sql
CREATE TABLE crypto_ticks (
    "time" TIMESTAMPTZ NOT NULL,
    symbol TEXT,
    price DOUBLE PRECISION,
    day_volume NUMERIC
);
```

### Crypto Assets Table

```sql
CREATE TABLE crypto_assets (
    symbol TEXT UNIQUE,
    "name" TEXT
);
```

## Key Features Demonstrated

### 1. Hypertable Creation

Convert regular tables into time-series optimized hypertables:
```sql
SELECT create_hypertable('crypto_ticks', by_range('time'));
```

### 2. OHLCV Candlestick Data Generation

Create candlestick data for financial analysis:

```sql
SELECT
    time_bucket('1 day', time) AS bucket,
    symbol,
    FIRST(price, time) AS "open",
    MAX(price) AS high,
    MIN(price) AS low,
    LAST(price, time) AS "close",
    LAST(day_volume, time) AS day_volume
FROM crypto_ticks
WHERE symbol = 'BTC/USD'
GROUP BY bucket, symbol
ORDER BY bucket;
```

### 3. Columnar Compression 

Enable ~10x storage compression with improved query performance:

```sql
ALTER TABLE crypto_ticks 
SET (
    timescaledb.enable_columnstore = true, 
    timescaledb.segmentby = 'symbol',
    timescaledb.compress_orderby = 'time DESC'
);
```

### 4. Continuous Aggregates

Create self-updating materialized views for instant analytics:

```sql
CREATE MATERIALIZED VIEW one_day_candle
WITH (timescaledb.continuous, timescaledb.materialized_only = false) AS
SELECT
    time_bucket('1 day', time) AS bucket,
    symbol,
    FIRST(price, time) AS "open",
    MAX(price) AS high,
    MIN(price) AS low,
    LAST(price, time) AS "close",
    LAST(day_volume, time) AS day_volume
FROM crypto_ticks
GROUP BY bucket, symbol;
```

## Getting Started

### Option 1: Using psql Command Line

1. Follow the instructions in `analyze-financial-data-psql.sql`

2. The script will automatically download sample data and guide you through each step3. Includes timing comparisons to demonstrate performance improvements

### Option 2: Using Timescale Cloud Console

1. Follow the instructions in `analyze-financial-data-UI.sql`

2. Use the S3 import feature to load data directly from Timescale's demo datasets3. Execute queries directly in the web-based query editor

## Workshop Highlights

- **Real Data**: Uses actual cryptocurrency market data from Twelve Data
- **Performance Optimization**: Compare query times before and after compression
- **Storage Efficiency**: See 10x storage reduction with columnar compression
- **Automatic Updates**: Demonstrate real-time continuous aggregate updates
- **Production Ready**: Learn policies for automatic compression and aggregate refreshing

## Data Sources

The workshop uses sample cryptocurrency data including:

- Bitcoin (BTC/USD) tick data with timestamps, prices, and volumes
- Multiple cryptocurrency symbols for comprehensive analysis
- Real market data suitable for production-style financial applications

## About OHLCV and Candlestick Charts

Financial candlestick charts visualize price movements using:

- **Open**: Opening price for the time period
- **High**: Highest price during the period  
- **Low**: Lowest price during the period
- **Close**: Closing price for the period
- **Volume**: Total trading volume

This data format is essential for technical analysis and algorithmic trading applications.

## Performance Benefits

- **Hypertables**: Automatic partitioning for optimal time-series queries
- **Columnar Storage**: 90% storage reduction with faster analytical queries
- **Continuous Aggregates**: Sub-second response times for complex aggregations
- **Automatic Policies**: Set-and-forget data lifecycle management

## License

MIT License

## Acknowledgments

This workshop was created by Anton Umnikov/Timescale and is based on the official TimescaleDB financial data tutorial.
