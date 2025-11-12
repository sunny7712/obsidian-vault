### Hive table creation

```SQL
DROP TABLE IF EXISTS invest_hive.apex.nse_historical_data_cm_equities_hive;

CREATE TABLE invest_hive.apex.nse_historical_data_cm_equities_hive (
  first_column VARCHAR,
  trade_id VARCHAR,
  name VARCHAR,
  series VARCHAR,
  trade_time VARCHAR,
  price VARCHAR,
  quantity VARCHAR,
  trade_date VARCHAR
)
WITH (
    external_location = 'gs://gw-apex-prod-bucket/NSE_HISTORICAL_DATA/CM/EQUITIES/', 
    format = 'CSV',
    csv_separator = '|',
    partitioned_by = ARRAY['trade_date']
);

CALL invest_hive.system.sync_partition_metadata(schema_name => 'apex', table_name => 'nse_historical_data_cm_equities_hive', mode => 'ADD');

SELECT * FROM invest_hive.apex.nse_historical_data_cm_equities_hive LIMIT 10;
```

---
