PlantUML

```
@startuml
actor User
Participant "apex-cf-worker" as Worker
participant "apex-api-trading-krakend" as Krakend
participant "groww-apex-stocks-adapter" as Adapter
database "apex-cache" as Cache
participant "fno-greeks-service"
participant "stocks-derivatives-core" as Dcore

User -> Worker: Option chain API Request
Activate Worker
Worker -> Krakend: Fetch Active contracts Request
note right: Authentication

Krakend -> Adapter: Fetch Active contracts request
Activate Adapter

Adapter -> Cache: lookup(key(underlying, exchange, expiry))
alt cache hit
 Cache --> Adapter: cached active contracts (TTL valid)
 note right of Cache: cached response (TTL=1hour)
else cache miss
 Adapter -> Dcore: Fetch active contracts
 Activate Dcore
 Dcore --> Adapter: Active contracts response
 Deactivate Dcore
 
 Adapter -> Cache: store(key(underlying, exchange, expiry) -> active_contracts)
 Cache --> Adapter: ack
end

Adapter --> Worker: Active contracts response
Deactivate Adapter
Worker --> User: Option Chain Response
note right 
Calling workers of 
latest prices batch 
and greeks and 
aggregating the response
end note
Deactivate Worker


User -> Krakend: Greeks API Request
note right: Authentication
Krakend -> Adapter: Greeks API Request
activate Adapter

Adapter -> Cache: lookup(key(trading_symbol, exchange, expiry))
alt cache hit
  Cache --> Adapter: cached greeks (TTL valid)
  note right of Cache: cached response (TTL=3m)
else cache miss
  Adapter -> "fno-greeks-service": Fetch greeks for contract
  activate "fno-greeks-service"
  "fno-greeks-service" --> Adapter: Greeks response for a contract
  deactivate "fno-greeks-service"

  Adapter -> Cache: store(key(trading_symbol, exchange, expiry) -> greeks)
  Cache --> Adapter: ack
end

Adapter --> User: Greeks API Response
deactivate Adapter
@enduml
```

