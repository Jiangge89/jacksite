---
date: 2026-08-11T00:00:00+08:00
tags: ["consistency", "distributed", "tier3"]
title: "Saga Toolbox"
weight: 13
---

## What is Saga?

> **Saga is a pattern for maintaining consistency across multiple services using local transactions and compensation instead of one global transaction.**

## Interview Trigger

```
Multiple Services / Multiple Databases / Distributed Workflow
Payment / Order / Wallet / Inventory / Shipping
```

每个 Service 有自己的 DB，不能跨 DB 做一个 Transaction → Saga。

---

## 为什么需要 Saga？

买 BTC 流程：

```
Create Order → Deduct Wallet → Execute Trade → Reward User
```

如果 Trade 失败，Order 和 Wallet 已经 Commit，不能 Rollback。只能补偿：`Refund Wallet → Cancel Order`。

---

## Compensation

不是 Database Rollback，而是 Business Undo：

| Forward | Compensation |
| --- | --- |
| Reserve Balance | Release Balance |
| Withdraw | Deposit |
| Create Order | Cancel Order |
| Reserve Inventory | Release Inventory |

有些操作没有真正的 Undo（例如 Send Email），只能发补偿通知。

---

## 两种实现

### Choreography

没有 Coordinator，完全 Event Driven：

```
Order Service → OrderCreated
  → Wallet Service → BalanceReserved
    → Trade Service → TradeCompleted
```

失败时 Trade 发 `TradeFailed`，Wallet 监听后 Refund，Order 监听后 Cancel。

优点：松耦合。缺点：Service 增加后 Event Flow 越来越复杂。

### Orchestration (推荐)

增加一个 Saga Coordinator：

```
Coordinator → Call Order → Call Wallet → Call Trade → Call Reward
```

失败时 Coordinator 告诉 Wallet Refund、告诉 Order Cancel。

优点：Flow 清晰 / Retry 容易 / Monitoring 容易。缺点：Coordinator 需要维护 State。

---

## Saga 和 Outbox 的关系

不是二选一：

- **单个 Service 内部:** `Update Wallet → Publish WalletUpdated` → 需要 **Outbox**
- **整个业务流程:** `Transfer → Reward → Notification` → 需要 **Saga**

实际上：

```
Service 内部: Transaction → Outbox → Kafka
整个系统: Saga
```

---

## Failure 怎么处理？

Trade 失败 → Coordinator 开始 Compensation → Wallet Refund。

如果 Refund 又失败？→ Retry。

所以 Compensation 必须 **Idempotent**。

---

## Saga vs 2PC

|  | 2PC | Saga |
| --- | --- | --- |
| Consistency | Strong | Eventual |
| Performance | Lower | Higher |
| Blocking | Yes | No |
| Availability | Lower | Higher |
| Rollback | Transaction | Compensation |

现代互联网一般用 Saga。

---

## Saga 最大的限制

每一步都必须能 Compensate。例如 Withdraw 可以 Deposit，但 Email Sent 没有真正 Undo。

---

## 面试标准回答

> How would you implement a purchase workflow across multiple services?

> I'd use the Saga pattern. Each service executes its own local transaction. If a later step fails, previously completed services execute compensating actions. I prefer orchestration for financial workflows because it provides better visibility, retries, and monitoring.

---

## 金融系统一致性架构全链路

```
Transaction
  → Update Business Data
    → Outbox Table
      → CDC (Binlog)
        → Kafka
          → Idempotent Consumers
            → Multiple Services (Saga)
```

这是 Coinbase、Stripe、Uber、DoorDash 等事件驱动微服务最经典的链路。
