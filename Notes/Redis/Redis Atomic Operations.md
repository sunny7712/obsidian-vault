#redis #transactions #lua_scripts

Redis offers several mechanisms to ensure atomicity, meaning operations are executed as a single, indivisible unit, preventing race conditions and ensuring data consistency.

---
### 1. Single Commands:
Many basic Redis commands are inherently atomic because Redis is single-threaded. For example, `SET`, `GET`, `INCR`, `DECR`, `LPUSH`, and `RPUSH` are atomic operations on individual keys. This means that when a client executes `INCR mykey`, the increment operation is guaranteed to complete before any other client can modify `mykey`.

### 2. Transactions (MULTI/EXEC/WATCH):
https://redis.io/docs/latest/develop/using-commands/transactions/
All the commands in a transaction are serialized and executed sequentially. A request sent by another client will never be served **in the middle** of the execution of a Redis Transaction. This guarantees that the commands are executed as a single isolated operation.

Redis transactions allow you to group multiple commands into a single atomic block using `MULTI` and `EXEC`. 

- `MULTI`: Initiates a transaction, *queuing* subsequent commands. From redis 2.6.5, If there are any errors while queuing, the whole transaction is rejected and running `EXEC` will return an error. In effect, nothing in the queue is executed.
- `EXEC`: Atomically executes all queued commands. It's important to note that **even when a command fails, all the other commands in the queue are processed**
- Calling [`DISCARD`](https://redis.io/docs/latest/commands/discard/) instead will flush the transaction queue and will exit the transaction.
- `WATCH`: Provides optimistic locking. You can `WATCH` one or more keys before a transaction. If any watched key is modified by another client before `EXEC`, the transaction is aborted, preventing inconsistencies.

Example of a transaction with WATCH:
*Code*
```
WATCH mykey
GET mykey
# Perform some client-side logic based on the value
MULTI
SET mykey new_value
EXEC
```

If `mykey` is modified between `WATCH` and `EXEC` by another client, the `EXEC` command will fail, and the transaction will not be applied.

### 3. Lua Scripting:
Lua scripts executed in Redis are guaranteed to be atomic. The entire script runs as a single, indivisible operation, and no other commands or scripts can run concurrently while a Lua script is executing. This provides strong transactional semantics and allows for complex, multi-step operations to be performed atomically without the overhead of multiple client-server round trips.

Example of an atomic Lua script:
*Code*
```Lua
local current_value = redis.call('GET', KEYS[1])
if tonumber(current_value) > tonumber(ARGV[1]) then 
	 redis.call('SET', KEYS[1], ARGV[2])    
	 return 1
else    
	return 0
end
```

This script atomically checks a value and conditionally sets it, ensuring consistency.

---

In summary:

- **Single commands:** Atomic by nature due to Redis's single-threaded model.
- **Transactions (MULTI/EXEC/WATCH):** Group commands and provide optimistic locking for multi-key operations.
- **Lua Scripting:** Execute complex, multi-command logic atomically on the server side, ensuring complete isolation and consistency.