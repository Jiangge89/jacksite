---
date: 2026-08-11T00:00:00+08:00
tags: ["api", "tier3"]
title: "SSE Toolbox"
weight: 18
---

## SSE (Server Sent Event)

### Trigger

```
Notification / Dashboard / Stock Price / One-way Push
```

### What

只有 Server → Client，不能 Client Push。

### 什么时候比 WebSocket 更适合？

```
Price Dashboard / Notification / AI Streaming
```

只需要单向推送时，SSE 比 WebSocket 更简单。
