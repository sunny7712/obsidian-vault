#redis #transactions #lua_scripts 

Use **WATCH + MULTI/EXEC** for _simple, optimistic_ multi-command sequences you can retry client-side; use **Lua scripts (EVAL/EVALSHA)** when you need _true server-side atomicity + complex logic + fewer round-trips_ and can accept the risk of blocking the Redis server. Both run single-threaded on the server and have important tradeoffs (error/rollback semantics, cluster constraints, debugging, performance impact). Below I give the full practical comparison, examples, pitfalls, and a decision checklist.

# What each actually guarantees

### MULTI / EXEC (with optional WATCH)

- MULTI begins a transaction; commands are **queued**; EXEC executes the queued commands **atomically** (no interleaving with other clients).
- **WATCH** provides optimistic locking: if any watched key is modified by another client between WATCH and EXEC, **EXEC will abort** and return a nil/empty reply — client must retry.
- **No rollback**: if one command inside EXEC fails at execution time (type error, wrong args), Redis still executes the remaining commands; the failing command returns an error in the EXEC reply but others still run.
- Validation of some errors happens only at EXEC time (not at queue time).

### Lua scripts (EVAL / EVALSHA)

- The whole script runs **atomically** and **synchronously** inside the Redis server (single-threaded) — no other commands interleave.
- You can read, compute, branch, and write multiple keys in one run; you get an all-or-nothing logical operation _as implemented in the script_.
- Scripts cannot do external I/O; they run within Redis process and block the server for their duration.
- Scripts can be cached and run by EVALSHA (faster after load).

# Strengths — when to prefer which

### Prefer MULTI/EXEC (+ WATCH) when:

- Your sequence is small/simple (a handful of commands).    
- You can afford a client-side retry strategy when a WATCH aborts under contention.
- You want clearer debugging/tracing of individual commands from client logs.
- You prefer to keep logic in application code (easier to test, version, CI).
- You want to avoid long server-side blocking.

### Prefer Lua scripts when:
- You need complex read-modify-write logic that must be atomic (no retry logic).
- You want to reduce network round-trips — one EVAL vs multiple commands and back-and-forth retries.
- You need to enforce invariants server-side (e.g., check & decrement, conditional set-with-computation).
- You want better performance under high concurrency (less retry thrash) as long as scripts are short.

# Important limitations & gotchas

### Both

- Both are **single-node**: operations must be on keys in the same Redis node. In a Redis Cluster, **all keys a transaction or script touches must hash to the same slot** (use hash tags `{...}` if needed). Otherwise they won't work across slots.
- Both run under Redis’ single-threaded event loop; heavy work => blocks all other Redis clients.

### MULTI/EXEC specifics

- **No rollback** of already-applied commands on error — partial failure possible.    
- WATCH requires careful retry logic; under high contention retries can cause wasted CPU and latency.
- If you pipeline MULTI + many commands + EXEC you can reduce RTT, but it still doesn’t solve race conditions without WATCH.

### Lua-specific specifics
- Long-running scripts block Redis; there’s a server config for script-time thresholds (watch for slow scripts). If a script blocks too long it can degrade the whole system.
- Debugging is harder: script sources in the server, stack traces less convenient than application logs.
- Scripts are persistent in Redis’ script cache but managing versions (updating scripts) requires care (SCRIPT LOAD, EVALSHA).
- If script crashes or contains bugs, it can corrupt logic — test heavily.
- Some advanced Redis features or commands may behave differently inside Lua (rare) — check command availability.
- Memory and CPU of Redis are used — heavy computations belong outside Redis.

# Performance considerations

- **Round-trips**: Lua wins (single RPC). MULTI+commands + EXEC can be pipelined to approximate single round-trip, but WATCH/conditional retries re-introduce extra trips.
- **Under contention**: WATCH causes aborts; many retries => worse latency. Script eliminates that class of race conditions.
- **Server blocking**: even if Lua reduces network latency, long scripts block the entire server — trade-off.
- **CPU usage**: move CPU-heavy calculations out of Redis. Redis should mainly be used for fast data ops.

# Error handling & retries

- MULTI/EXEC: client must interpret EXEC result. If EXEC returns nil because of WATCH, client retries. If EXEC returns a list with some error replies, client must handle them (no rollback).
- Lua: either returns your result or throws an error. Within the script you can check conditions and return error codes. There’s no automatic retry — implement logic in the script if needed.

# Persistence, AOF, and replication notes (practical)

- Scripts and transactions execute on the master and are replicated to slaves. With AOF, the commands are recorded; for scripts Redis logs the script invocation so replication/AOF is consistent.
- For safety, ensure your scripts produce deterministic behavior so replicas replay the same state correctly.

# Cluster considerations (very important)

- In Redis Cluster, you **must target keys in the same hash slot** for MULTI/EXEC and for scripts. If not, the command will fail with CROSSSLOT error.
- Use hash tags `{tag}` when you must co-locate keys across different logical names.

# Security & operational concerns

- Avoid running untrusted Lua code on the server.
- Keep scripts small and predictable.
- Monitor script durations and configure alerts for long-running scripts.
- Have a plan to update/replace scripts — store script source in your app repo and use CI to manage versioning (load by SHA).