---
date: 2026-08-11T00:00:00+08:00
tags: ["consistency", "concurrency", "tier1"]
title: "Pessimistic Lock Toolbox"
weight: 10
---

## Interview Trigger

```
High Contention / Inventory / Wallet / Stock / Seat Booking / Bank Transfer
```

## Optimistic vs Pessimistic

|  | Optimistic | Pessimistic |
| --- | --- | --- |
| 默认假设 | 冲突少 | 冲突多 |
| 是否提前加锁 | ❌ | ✅ |
| 是否需要 Retry | ✅ | 一般不用 |
| 吞吐量 | 高 | 较低 |
| Deadlock 风险 | 无 | 有 |

---

## MySQL 实现: SELECT ... FOR UPDATE

```sql
BEGIN;
SELECT * FROM account WHERE account_id='A' FOR UPDATE;  -- Row Lock
UPDATE account SET balance = balance - 80 WHERE account_id='A';
COMMIT;
```

---

## Deadlock

Pessimistic Lock 100% Follow-up。

```
Tx1: Lock A → Waiting B
Tx2: Lock B → Waiting A
→ Deadlock → 数据库 Kill 其中一个 Transaction
```

### How to avoid deadlock?

**方法一:** 固定 Lock 顺序。永远 Smaller Account ID → Larger Account ID。

**方法二:** 减少 Transaction 时间。不要 Lock 拿太久（尤其不要在 Transaction 中 Call External API）。

**方法三:** Timeout。Wait 3 Seconds → Fail。

---

## Coinbase 场景

Transfer A → B，需要同时锁两个 Account。推荐：Always lock smaller account id first。

---

## Interview Follow-up

**Q1: 什么时候用 Optimistic?**

> When conflicts are rare and I want higher throughput.

**Q2: 什么时候用 Pessimistic?**

> When conflicts are frequent and retries would be expensive.

**Q3: 为什么不用 Redis Lock?**

> I'd prefer database row locks because they naturally participate in the same transaction and avoid keeping distributed locks in sync with the database.

Redis Lock 用于保护非数据库资源的并发写，或者跨数据库的原子问题。

---

## 真实例子

演唱会最后一张票，10000 用户同时抢：

- **Optimistic:** 9999 Retry → 疯狂冲突
- **Pessimistic:** Lock → One User → Success → Others → Sold Out

---

## 选择原则

```
Concurrency Control
├── Atomic SQL       (一条 SQL 能解决时优先)
├── Optimistic Lock  (冲突少)
└── Pessimistic Lock (冲突多)
```

> It depends on the contention level. If conflicts are rare, such as users updating their own wallets, optimistic locking provides better throughput. If many requests compete for the same resource, such as ticket booking, pessimistic locking is more appropriate because it prevents excessive retries.

---

## DB 锁 vs Redis 锁

```
Concurrency Control
├── Database
│     ├── Optimistic Lock
│     ├── SELECT ... FOR UPDATE
│     └── Atomic SQL
└── Distributed Systems
      └── Redis Distributed Lock
```

**Redis Lock 不是用来替代数据库锁，而是当资源已经超出单个数据库时才需要。**

---

## Transaction vs Lock

Transaction 和 Lock 是两个独立的工具，经常一起使用但谁也不包含谁。

**只需要 Transaction (无并发):**

```sql
BEGIN;
UPDATE account SET balance = balance - 100 WHERE id = 'A';
UPDATE account SET balance = balance + 100 WHERE id = 'B';
COMMIT;
```

**Transaction + Lock (有并发):**

```sql
BEGIN;
SELECT ... FOR UPDATE;     -- Lock: 解决 Concurrency
UPDATE A; UPDATE B;        -- Transaction: 解决 Atomicity
COMMIT;
```
