`HSET` writes fields into a Redis hash.  
A Redis _hash_ is just a small **key–value map** stored under a single Redis key.

### Syntax:
`HSET key field value [field value ...]`

What it actually does:
1. **If the field does NOT exist** → it creates the field and sets the value.  
    Return value increments for each newly created field.
2. **If the field ALREADY exists** → it overwrites the value.  
    Return value does _not_ increment for overwritten fields.
3. **Multiple fields** in one call are atomic — Redis handles the entire command as one operation.

### Example:
`HSET user:123 name "Vamsi" age "25"`

Creates a hash stored at `user:123` with fields `{name: "Vamsi", age: "25"}`.

`HSET session:789 user_id 52 expires_at 1737032912 device "android"

```json
{
  "session:789": {
        "user_id": "52",
        "expires_at": "1737032912",
        "device": "android"
  }
}
```

Key points you must remember:
- It does **not** support nested structures — everything is string.
- It _replaces_ values for existing fields, no merge logic.
- It returns the count of **new** fields added.
- Hashes are efficient for storing many small fields under one key.

## ❓ So why do Redis hashes exist?

Because they let you store **structured data under one key** without serializing/deserializing the whole blob.

### Benefits:

### **1. Partial updates**

You can update one field without rewriting the whole object.

`HSET user:1 name "Sunny"`

Instead of doing:

`GET user:1   → parse JSON → modify → serialize → SET user:1`

This is faster _and_ atomic.

---

### **2. Lower memory usage**

Redis uses an internal optimized encoding (“ziplist” / “listpack”) for small hashes.  
This is much more compact than storing multiple keys like:

`SET user:1:name "Sunny" SET user:1:age 26 SET user:1:city "BLR"`

Hashes compress very well.

---

### **3. Fewer top-level keys**

Redis struggles when you create millions of top-level keys (memory fragmentation, eviction issues, higher overhead).

Using hashes groups related fields under a single key → far less overhead.

---

### **4. Cleaner semantics**

Your data becomes logically grouped:

```
user:1   
	├─ name   
	├─ email   
	├─ age   
	└─ last_login

```

That’s better than spamming the database with thousands of separate flat keys.

---

## ❓ When should you _not_ use Redis hashes?

Use hashes when:
- You want a small object with many small fields.
- You want partial reads/updates.
- You want atomic field-level manipulation.

Don’t use hashes when:
- You need nested structures (use `RedisJSON`).
- You want expiration per field (Redis hashes don’t support per-field TTL).
- You’re storing large blobs (store blob directly with `SET`).

---
### Related commands
- `HMGET`
- `HGET`
- `HINCRBY`
https://redis.io/docs/latest/commands/hincrby/