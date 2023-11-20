---
layout: post
title:  "Database systems VI： Transaction & NoSQL"
date:   2023-11-20 14:03 +1100
categories: database
layout: math
---
# Database systems VI： Transaction & NoSQL

## Key points

Transaction:
- Transaction
- Concurrency control
- Recovery

NoSQL:
- NoSQL concept
- Different data model
- Key-Value, Document, Column-family, Graph

## Transaction

A transaction is a unit of program execution that accesses and possibly updates various data items. To preserve the integrity of data, the database system must ensure the ACID property.

### Transaction states

- **Active:** the initial state; the transaction stays in this state while it is executing.
- **Partially committed:** after the final statement has been executed.
- **Failed:** after the discovery that normal execution can no longer proceed.
- **Aborted:** after the transaction has been rolled back using log and the database restored to its state prior to the start of the transaction. Two options after it has been aborted:
  - Restart the transaction
  - Kill the transaction
- **Committed:** after successful completion.

![](/assets/images/transaction_states.png)

### ACID

- **Atomicity:** Either all operations of the transaction are properly reflected in the database, or none are.
- **Consistency:** Every transaction sees a consistent database.
- **Isolation:** Although multiple transactions may execute concurrently, each transaction must be unaware of other concurrently executing transactions. Intermediate transaction results must be hidden from other concurrently executed transactions.
  > That is, for every pair of transactions Ti and Tj, it must appear to Ti that either Tj, finished execution before Ti started, or Tj started execution after Ti finished.
- **Durability:** After a transaction completes successfully, the changes it has made to the database persist, even if there are system failures.

## Schedule

A sequence that specifies the order in which instructions of concurrent transactions are executed.

### Serial schedule

A schedule S is serial if for every transaction T in the schedule, all (the operations of) T are executed consecutively in the schedule.

**Example:** a serial schedule in which T1
is followed by T2.

### Serializability

A (possibly concurrent) schedule S of n transactions is serializable if it is **equivalent** to some serial schedule of the same n transactions.

## Conflict

### Conflicting Instructions

Two operations O1 and O2 are conflicting if
- they are in different transactions,
- they access the same data item,
- at least one of them must be a write.

### Conflict Equivalence

Two schedules are conflict equivalent if:
- involve the same actions of the same transactions;
- every pair of conflicting actions is ordered the same way.

### Conflict Serializability

A schedule S to be conflict serializable if it is (conflict) equivalent to some serial schedule S. 

If a schedule S can be transformed into a schedule S' by *a series of swaps of non-conflicting instructions,* we say that S and S' are conflict equivalent.

The concept of serializability of schedules is used to identify which schedules are **correct** when transaction executions have interleaving of their operations in the schedules.

>  Conflict serializable schedules are correct.

### Precedence Graph

Consider some schedule of a set of transactions 𝑇1, 𝑇2, ..., 𝑇𝑛.

Precedence graph: a directed graph 𝐺 = (𝑉, 𝐸), where the vertices (𝑉) are the transactions.

We draw an arc from 𝑇𝑖 to 𝑇𝑗 (𝑇𝑖 → 𝑇𝑗), if the two transactions are conflict, and 𝑇𝑖 accessed the data item earlier.
- 𝑇𝑖 executes write(Q) before 𝑇𝑗 executes read(Q)
- 𝑇𝑖 executes read(Q) before 𝑇𝑗 executes write(Q)
- 𝑇𝑖 executes write(Q) before 𝑇𝑗 executes write(Q)

We may label the arc by the item that was accessed.

### Testing Conflict Serializability

1. Construct a precedence graph.
2. Check if the graph is cyclic:
   - Cyclic: non-serializable.
   - Acyclic: serializable.

If the precedence graph is acyclic, the serializability order can be obtained by a **topological sorting** of the graph.

## Concurrency control

Concurrency-control protocols allow concurrent schedules, ensure that the schedules are serializable.

### Locks

Data items can be locked in two modes:
1. Exclusive (X) mode.
  - The data item can be both read as well as written.
  - Also known as a Write Lock.
2. Shared (S) mode.
  - The data item can only be read.
  - Also known as a Read Lock

Lock-compatibility matrix:

![](/assets/images/lock_compatibility.png)

A transaction may be granted a lock on an item if the requested lock is **compatible** with locks already held on the item by other transactions.

This means:
- any number of transactions can hold shared locks on an item;
- but if a transaction holds an exclusive on an item, then no other transaction may hold a lock on the item.

### Locking rules

1. If T has only one operation (read/write) manipulating an item X:
  - obtain a read lock on X before reading it,
  - obtain a write lock on X before writing it,
  - unlock X when done with it.
2. If T has several operations manipulating X:
  - obtain one proper lock only on X:
    - a read lock if all operations on X are reads;
    - a write lock if one of these operations on X is a write.
  - unlock X after the last operation on X in T has been executed.

### Two Phase Locking (2PL)

1. Growing Phase
  - Transaction obtains locks
  - Transaction does not release any locks
2. Shrinking Phase
  - Transaction releases locks
  - Transaction does not obtain new locks

This protocol ensures serializability: and produces conflictserializable schedules. It can be proved that the transactions can be serialized in the order of their lock points (i.e., the point where a transaction acquired its final lock).

> This locking protocol is sufficient to guarantee serializability, but introduces **deadlocks.**

### Testing for deadlocks

Create a wait-for graph for currently active transactions:
- create a vertex for each transaction; and
- an arc from Ti to Tj if Ti is waiting for an item locked by Tj.

If the graph has a cycle, then a deadlock has occurred.

## Database recovery after transaction failure

### System log

Log records:
1. **[start transaction, T]:** Start transaction marker, records that transaction T has started execution.
2. **[read item, T, X]:** Records that transaction T has read the value of database item X.
3. **[write item, T, X, old value, new value]:** Records that T has changed the value of database item X from old value to new value.
4. **[commit, T]:** Commit transaction marker, records that transaction T has completed successfully, and confirms that its effect can be committed (recorded permanently) to the database.
5. **[abort, T]:** Records that transaction T has been aborted.

### Write-ahead log strategy

1. Must force the log record for an update before corresponding data page gets to the disk.
2. Must force all log records for a transaction before commit.

### Undo & Redo

Procedure UNDO (WRITE_OP):
- Undoing a write_item operation WRITE_OP.
- Consists of examining its log entry [write_item, T, X, old_value, new_value] and setting the value of item X in the database to old_value.
- Undoing a number of write operations from one or more transactions from the log must proceed in the <u>**reverse order**</u> in which the operations were written in the log.

> Undo helps guarantee atomicity in ACID.

Procedure REDO (WRITE_OP):
- Redoing a write_item operation WRITE_OP.
- Consists of examining its log entry [write_item, T, X, old_value, new_value] and setting the value of item X in the database to new_value.

> Redo guarantees durability in ACID.

### Checkpoint

Taking a checkpoint consists of the following actions:
1. Suspend execution of transactions temporarily.
2. Force-write all main memory buffers that have been modified to disk.
3. Write a \[checkpoint\] record to the log, and force-write the log to disk.
4. Resume executing transactions.

> Without checkpoints, memory buffers are force-written to disk when the buffer pool is full and a `request_block` operation triggers a buffer replacement policy.

### Database recovery

If there is no checkpoint,
- undo transactions that haven't been committed yet;
- redo transactions that have been committed.

If there is a checkpoint,
- undo transcations that haven't been committed yet;
- redo transcations that have been committed after the checkpoint;
- do nothing to transactions that have been committed before the checkpoint.

## NoSQL

Not only SQL.

### NoSQL Properties

1. Flexible scalability
  - **horizontal scalability** instead of vertical
2. Dynamic schema of data
  - different levels of flexibility for different types of DB
3. Efficient reading
  - spend more time storing the data, but read fast
  - keep relevant information together
4. Cost saving
  - designed to run on commodity hardware
  - typically open-source (with a support from a company)

### CAP Theorem

At most two of the following three can be
maximized at one time
- Consistency
  > Each client has the same view of the data
- Availability
  > Each client can always read and write
- Partition Tolerance
  > System works well across distributed physical networks

![](/assets/images/cap.png)

### Different data model

- Key-Value
- Document
- Column-family
- Graph
