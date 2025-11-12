Mapping schema
- identifier
- Corporate action announcement timestamp
- Corporate action implementation timestamp
- Type of corporate action
- Corporate Action context - JSON

Implementation logic
- for symbol change logic, just check the mapping for new symbol, if it exists, take the old symbol of the corresponding row and do it recursively.

## SQL queries to apply Corporate Action before storing in PV

### Case 1: Symbol Change

- Since it's partitioned on date, fetch all the corporate actions greater than or equal to that trade date and apply

```sql
CREATE TEMPORARY VIEW source_data AS
SELECT
  start_time,
  open,
  high,
  low,
  close,
  total_volume,
  volume,
  CASE
    WHEN identifier IN ('identifier1', 'identifier2')
    THEN 'new_identifier'
    ELSE identifier
  END AS identifier,
  CAST(start_time AS DATE) AS date
FROM read_parquet('%s');

COPY (
  SELECT start_time, open, high, low, close,
         total_volume, volume, identifier, date
  FROM source_data
)
TO '%s'
(FORMAT 'parquet', PARTITION_BY (identifier, date), OVERWRITE_OR_IGNORE);
```

## Case 2: Candle normalization

```sql
CREATE TEMPORARY VIEW source_data AS
SELECT
  start_time,
  open,
  high,
  low,
  close,
  total_volume,
  volume,
  CASE
    WHEN identifier = 
    
    THEN 'new_identifier'
    ELSE identifier
  END AS identifier,
  CAST(start_time AS DATE) AS date
FROM read_parquet('%s');

COPY (
  SELECT start_time, open, high, low, close,
         total_volume, volume, identifier, date
  FROM source_data
)
TO '%s'
(FORMAT 'parquet', PARTITION_BY (identifier, date), OVERWRITE_OR_IGNORE);
```


## Corporate Action context for different events

- Split - splitRatioNumerator, splitRatioDenominator
- Dividend - no context needed
- Reverse split - reverseSplitRatioNumerator, reverseSplitRatioDenominator
- bonus - bonusRatioNumerator, bonusRatioDenominator
- rights - rightsRatioNumerator, rightsRatioDenominator, faceValue, premium
- nse symbol change - new symbol, old symbol`
- bse scrip code - old scrip code, new scrip code