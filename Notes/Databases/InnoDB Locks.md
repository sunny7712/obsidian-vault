#databases #innodb #locks

#### 1. Shared and Exclusive Locks

`InnoDB` implements standard row-level locking where there are two types of locks, [shared (`S`) locks](https://dev.mysql.com/doc/refman/8.4/en/glossary.html#glos_shared_lock "shared lock") and [exclusive (`X`) locks](https://dev.mysql.com/doc/refman/8.4/en/glossary.html#glos_exclusive_lock "exclusive lock").

- A [shared (`S`) lock](https://dev.mysql.com/doc/refman/8.4/en/glossary.html#glos_shared_lock "shared lock") permits the transaction that holds the lock to read a row.
- An [exclusive (`X`) lock](https://dev.mysql.com/doc/refman/8.4/en/glossary.html#glos_exclusive_lock "exclusive lock") permits the transaction that holds the lock to update or delete a row.    

If transaction `T1` holds a shared (`S`) lock on row `r`, then requests from some distinct transaction `T2` for a lock on row `r` are handled as follows:

- A request by `T2` for an `S` lock can be granted immediately. As a result, both `T1` and `T2` hold an `S` lock on `r`.
- A request by `T2` for an `X` lock cannot be granted immediately.

If a transaction `T1` holds an exclusive (`X`) lock on row `r`, a request from some distinct transaction `T2` for a lock of either type on `r` cannot be granted immediately. Instead, transaction `T2` has to wait for transaction `T1` to release its lock on row `r`.

#### 2. Record Locks

A record lock is a lock on an index record. For example, `SELECT c1 FROM t WHERE c1 = 10 FOR UPDATE;` prevents any other transaction from inserting, updating, or deleting rows where the value of `t.c1` is `10`.

Record locks always lock index records, even if a table is defined with no indexes. For such cases, `InnoDB` creates a hidden clustered index and uses this index for record locking. See [Section 17.6.2.1, “Clustered and Secondary Indexes”](https://dev.mysql.com/doc/refman/8.4/en/innodb-index-types.html "17.6.2.1 Clustered and Secondary Indexes").

Transaction data for a record lock appears similar to the following in [`SHOW ENGINE INNODB STATUS`](https://dev.mysql.com/doc/refman/8.4/en/show-engine.html "15.7.7.16 SHOW ENGINE Statement") and [InnoDB monitor](https://dev.mysql.com/doc/refman/8.4/en/innodb-standard-monitor.html "17.17.3 InnoDB Standard Monitor and Lock Monitor Output") output:

```terminal
RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t`
trx id 10078 lock_mode X locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 6; hex 00000000274f; asc     'O;;
 2: len 7; hex b60000019d0110; asc        ;;
```

