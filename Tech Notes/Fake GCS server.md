#gcs #docker

1. Starting a container
```
docker run --rm -it -p 4443:4443 fsouza/fake-gcs-server -scheme http
```

2. Creating a container
```cURL
curl -X POST \
  -H "Content-Type: application/json" \
  "http://localhost:4443/storage/v1/b" \
  -d '{"name":"my-test-bucket"}'
```

3. Confirm if bucket exists
```cURL
curl -k http://localhost:4443/storage/v1/b
```

4. Listing objects in bucket
```cURL
curl -k http://localhost:4443/storage/v1/b/gw-apex-stage-bucket/o
```

5. Uploading a file to a bucket
```cURL
curl -X POST \
  -H "Content-Type: application/octet-stream" \
  --data-binary @/Users/vamsik/Downloads/HISTORICAL_DATA_ICEBERG_MINUTE_CANDLES_trade_date=2025-01-01_20250620_074447_66584_anrj4_252402c6-b1ba-47a1-a1ca-7b4196cf0550.gz \
  "http://localhost:4443/upload/storage/v1/b/gw-apex-stage-bucket/o?uploadType=media&name=HISTORICAL_DATA/ICEBERG/MINUTE_CANDLES/trade_date=2025-01-01/20250620_074447_66584_anrj4_252402c6-b1ba-47a1-a1ca-7b4196cf0550.gz"
```

