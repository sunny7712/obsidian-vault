## 1 min candles across multiple time ranges (1 day, 1 week, 1 month, 3 months and 6 months)

### CRDB Grafana Dashboard

- https://iamgrafana.growwinfra.in/d/crdb-console-hardware/crdb-console3a-hardware?var-rate_interval=30s&var-DS_PROMETHEUS=uyCTeBOVk&var-cluster=apex-charting-data-primary&var-node=$__all&from=2025-08-19T11:43:00.000Z&to=2025-08-19T11:50:59.000Z&timezone=Asia%2FKolkata&tab=queries


###  Olly metrics
- https://observability.growwinfra.in/olly/services/load-testing?tnt=invest&env=prod&team=apex&sp=apex-candle-data-manager&pod=apex-candle-data-manager-5df58cf6bd-pdcln&t=2025-08-19T17%3A13%3A00%2F2025-08-19T17%3A25%3A00

- https://observability.growwinfra.in/olly/services/routes?tnt=invest&env=prod&team=apex&sp=apex-candle-data-manager&pod=apex-candle-data-manager-5df58cf6bd-pdcln&t=2025-08-19T17%3A13%3A00%2F2025-08-19T17%3A25%3A00#GET%20/api/v1/candles/data

- https://observability.growwinfra.in/olly/services/jvm?tnt=invest&env=prod&team=apex&sp=apex-candle-data-manager&pod=apex-candle-data-manager-5df58cf6bd-pdcln&t=2025-08-19T17%3A13%3A00%2F2025-08-19T17%3A25%3A00

### Locust results
- ![[Pasted image 20250819181510.png]]

## INTERVALS_MINUTES = [1, 3, 5, 10, 60, 240, 1440, 10080] candles across multiple time ranges (1 day, 1 week, 1 month, 3 months and 6 months)

![[Screenshot 2025-08-19 at 6.18.39 PM.png]]