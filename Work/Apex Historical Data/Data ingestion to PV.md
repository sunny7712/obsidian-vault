- The connector interfaces has return type `List<Blob>` and `Blob` is GCP specific. Ideally the interface must be generic.
- Factory design pattern can be used based on a enum like `Candles` or `Instruments` where each implementation has 
  1. it's own downloading from source logic
  2. it's own duckb ingestion query