---
title: "Capability Map"
---

# Algorithm Interview Capability Map & Assessment

> 基于本次完整算法摸底整理。目标不是记录"做过哪些题"，而是记录当前真实能力、暴露的问题、需要形成的模板，以及
> Live Coding 前的复习优先级。

## 1. 总体结论

  -------------------------------------------------------------------------------------
  Level                   当前能力                     Topics
  ----------------------- ---------------------------- --------------------------------
  **A**                   面试中基本可以直接做         HashMap、Heap / Priority
                                                       Queue、Binary Search、Linked
                                                       List

  **A-/B+**               会做，少量细节可能出错       LRU Cache、Two Pointers、Sliding
                                                       Window、Prefix Sum、Intervals

  **B**                   理解核心，需要形成稳定模板   Monotonic Stack、Basic DP、Tree
                                                       BFS/DFS、Graph BFS/DFS、Rate
                                                       Limiter

  **C**                   Pattern recognition /        Topological Sort、0/1
                          implementation 不稳定        Knapsack、Backtracking、Greedy

  **D**                   基本没系统学过               Union Find
  -------------------------------------------------------------------------------------

### 当前最高 ROI

Live Coding 前优先：

1.  **Graph**
2.  **Tree**
3.  **Backtracking**
4.  **0/1 Knapsack**
5.  **Rate Limiter / LRU implementation**

HashMap、Heap、Binary Search、Linked List 不需要再投入大量时间刷基础题。

## 2. HashMap --- A

明显强项。看到 frequency、lookup、duplicate、mapping、Two Sum、character
count，基本可以自然想到 HashMap。

复杂度：

``` text
lookup: expected O(1)
insert: expected O(1)
delete: expected O(1)
```

**结论：不需要重点复习。**

## 3. Heap / Priority Queue --- A

Pattern recognition 比较稳定。看到 Top K、K-th
largest/smallest、continuously get min/max、merge K sorted
lists，应优先想到 Heap。

Go 重点保证 `container/heap` implementation 熟练。

## 4. Binary Search --- A

Lower bound 模板：

``` go
left, right := 0, len(nums)
for left < right {
    mid := left + (right-left)/2
    if nums[mid] >= target {
        right = mid
    } else {
        left = mid + 1
    }
}
```

目标：find the first index where `nums[i] >= target`。

## 5. Linked List --- A

Reverse Linked List mental model：

``` text
prev
current
next
```

核心：

``` go
next := curr.Next
curr.Next = prev
prev = curr
curr = next
```

## 6. LRU Cache --- A-/B+

核心结构：

``` text
HashMap<key, Node>
+
Doubly Linked List

head ⇄ MRU ⇄ ... ⇄ LRU ⇄ tail
```

Node：

``` go
type Node struct {
    key, value int
    prev, next *Node
}
```

删除：

``` go
node.prev.next = node.next
node.next.prev = node.prev
```

插入 head：

``` go
node.prev = head
node.next = head.next
head.next.prev = node
head.next = node
```

操作：

``` text
get → lookup → moveToHead
put existing → update → moveToHead
put new → create → map insert → addToHead
over capacity → remove tail.prev → delete map[lru.key]
```

**结论：不用重新学，mock 前完整手写一次。**

## 7. Two Pointers / Sliding Window --- A-/B+

Two Sum Sorted：

``` text
sum < target → left++
sum > target → right--
```

3Sum：

``` text
sort → fix i → left/right
```

需要加强 duplicate handling：

``` go
if i > 0 && nums[i] == nums[i-1] {
    continue
}
```

Longest Substring Without Repeating Characters：

``` text
HashMap/Frequency + Sliding Window
```

Invariant：window 内不能存在 frequency \> 1 的 character。

## 8. Prefix Sum --- A-

统一使用 `n+1`：

``` text
prefix[i] = sum(nums[0...i-1])
```

``` go
prefix := make([]int, len(nums)+1)
for i := 0; i < len(nums); i++ {
    prefix[i+1] = prefix[i] + nums[i]
}

sum := prefix[right+1] - prefix[left]
```

复杂度：preprocessing O(n)，query O(1)，space O(n)。

## 9. Monotonic Stack --- B

条件反射：

> 对每个元素寻找右侧第一个 greater/smaller element → Monotonic Stack。

通常存 index：

``` go
stack := []int{}
for i := 0; i < len(nums); i++ {
    for len(stack) > 0 && nums[i] > nums[stack[len(stack)-1]] {
        j := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        result[j] = i - j
    }
    stack = append(stack, i)
}
```

每个 index 最多 push/pop 一次，所以 O(n)。

## 10. Tree --- B-/C+

需要系统补基础。

Traversal：

``` text
Preorder:  Root → Left → Right
Inorder:   Left → Root → Right
Postorder: Left → Right → Root
```

Maximum Depth：

``` text
depth(node) = 1 + max(depth(left), depth(right))
```

复杂度：

``` text
Time: O(n)
Space: O(h)
balanced: O(log n)
skewed worst case: O(n)
```

Level-order BFS 使用 Queue，每轮先：

``` go
levelSize := len(queue)
```

## 11. Graph --- B-/C+

当前最高优先级模块之一。

### Node vs Edge

``` text
edges = [[0,1], [1,2], [3,4]]
```

Nodes 是 `0,1,2,3,4`；数组中的 pair 才是 edges。

Graph node 有任意多个 neighbors，不是 binary tree 的 left/right。

### Adjacency List

``` go
graph := make([][]int, n)
for _, edge := range edges {
    a, b := edge[0], edge[1]
    graph[a] = append(graph[a], b)
    graph[b] = append(graph[b], a)
}
```

### DFS

``` go
func dfs(graph [][]int, node int, visited []bool) {
    if visited[node] {
        return
    }
    visited[node] = true
    for _, next := range graph[node] {
        dfs(graph, next, visited)
    }
}
```

### BFS

``` go
visited[start] = true
queue = append(queue, start)

for len(queue) > 0 {
    node := queue[0]
    queue = queue[1:]

    for _, next := range graph[node] {
        if visited[next] {
            continue
        }
        visited[next] = true
        queue = append(queue, next)
    }
}
```

关键：**mark visited when enqueued, not when dequeued**。

### Connected Components

``` text
for each node:
    if visited → continue
    components++
    BFS/DFS(node)
```

### Grid BFS / Shortest Path

四方向：

``` text
[i-1][j]
[i+1][j]
[i][j-1]
[i][j+1]
```

条件反射：

> **Unweighted graph + minimum steps/hops → BFS.**

## 12. Topological Sort --- C+/B-

`indegree` = number of incoming edges。

Dependency：

``` text
A depends on B
→ B → A
→ indegree[A]++
```

Kahn's Algorithm：

``` text
1. Build adjacency list
2. Calculate indegree
3. Enqueue indegree == 0
4. Pop node
5. Decrease neighbors' indegree
6. Neighbor becomes 0 → enqueue
```

``` text
processed == numberOfNodes → no cycle
processed < numberOfNodes → cycle
```

当前主要错误：edge direction 和 decrement 对象容易写反。

## 13. Basic DP --- B

House Robber：

``` text
dp[i] = maximum money considering houses 0...i

don't rob i → dp[i-1]
rob i       → dp[i-2] + nums[i]

dp[i] = max(dp[i-1], dp[i-2] + nums[i])
```

Base：

``` text
dp[0] = nums[0]
dp[1] = max(nums[0], nums[1])
```

DP 面试固定流程：

``` text
Define state
→ choices
→ transition
→ base cases
→ complexity
→ space optimization
```

## 14. 0/1 Knapsack --- B-/C+

``` text
dp[w] = capacity w 下能获得的 maximum value
```

Transition：

``` go
dp[w] = max(dp[w], dp[w-weight]+value)
```

最重要：

``` text
0/1 Knapsack
→ item only once
→ capacity 大 → 小

Complete Knapsack
→ item unlimited
→ capacity 小 → 大
```

0/1 必须倒序，否则同一个 item 会在本轮被重复使用。

## 15. Backtracking --- B-/C+

统一 mental model：

``` text
current state
→ choices
→ choose
→ recursive explore
→ unchoose
```

模板：

``` go
cur = append(cur, choice)
dfs(...)
cur = cur[:len(cur)-1]
```

核心 invariant：

> 进入一个 branch 前是什么状态，从 branch 返回后就恢复成什么状态。

Subsets 对每个 `nums[i]`：

``` text
don't take
take
```

保存 Go slice result 时需要 copy：

``` go
result = append(result, append([]int{}, cur...))
```

需要补：Generate Parentheses constraint pruning、Permutations、pattern
recognition。

## 16. Greedy --- TBD，约 B-/C+

Stock 题最初主要是漏看 `buy once / sell once`。理解后能得到：

``` text
minPrice
maxProfit
```

扫描时：

``` text
profit if selling today = price - minPrice
```

维护 `minPrice` 与 `maxProfit`。

Greedy 更准确的理解：

> Make a locally optimal choice that can be proven not to hurt the
> global optimum.

需要再用 1--2 道题确认真实水平。

## 17. Union Find / DSU --- D

目前没系统使用过。

Mental model：

``` text
find(x)
→ x 属于哪个 connected component

union(a,b)
→ merge two components
```

新增 `[a,b]`：

``` text
find(a) == find(b)
→ already connected
→ adding edge creates cycle
```

后续再学 path compression、union by rank/size。当前不是最高优先级。

## 18. Rate Limiter --- B

### Fixed Window --- B+

``` text
userID → windowStart + count
```

窗口对齐。主要问题是 boundary burst。

### Sliding Window Log --- B+/A-

``` text
userID → []timestamp
```

对于 `(t-60, t]`：

``` text
remove timestamp <= t-60
keep timestamp > t-60
```

可以 binary search upper bound，也可以 deque 从 head 清理。

### Concurrency

相同 timestamp 本身完全合法。

真正需要 atomic 的是：

``` text
cleanup → check quota → update state
```

单机可 per-user mutex；distributed 可以使用 Redis atomic operation /
Lua。

### Sliding Window Counter

保存 previous/current fixed-window counts，通过 previous window 的
weighted portion 近似 sliding window。

优点 O(1) state/user；缺点是 approximation。

### Token Bucket --- C+/B-

状态：

``` text
tokens
lastRefillTime
```

不是周期性 reset。

``` text
refill = elapsed * refillRate
tokens = min(capacity, tokens + refill)
```

``` text
capacity → maximum burst
refill rate → long-term rate
```

## 19. Clarification：跨题核心能力

有时不是不会算法，而是 requirement 没完全读清楚。Stock 题漏掉 `once`
是典型例子。

Live Coding 固定流程：

``` text
1. Restate the problem
2. Clarify constraints
3. Give example / edge case
4. Explain approach
5. Analyze complexity
6. Code
7. Test
```

特别确认：

``` text
duplicates?
sorted?
negative numbers?
empty input?
one solution or multiple?
can reuse elements?
once or unlimited?
directed or undirected?
weighted or unweighted?
input size?
```

## 20. 最终能力地图

  Topic                                      Level     Priority
  ----------------------------- ------------------ ------------
  HashMap                                    **A**          Low
  Heap / Priority Queue                      **A**          Low
  Binary Search                              **A**          Low
  Linked List                                **A**          Low
  LRU Cache                              **A-/B+**       Medium
  Two Pointers                              **A-**          Low
  Sliding Window                         **A-/B+**       Medium
  Prefix Sum                                **A-**          Low
  Intervals                              **A-/B+**          Low
  Monotonic Stack                            **B**       Medium
  Basic DP                                   **B**       Medium
  Tree                                   **B-/C+**     **High**
  Graph BFS/DFS                          **B-/C+**     **High**
  Grid BFS                                  **B-**     **High**
  Topological Sort                       **C+/B-**     **High**
  0/1 Knapsack                           **B-/C+**     **High**
  Backtracking                           **B-/C+**     **High**
  Greedy                          **TBD \~ B-/C+**       Medium
  Union Find                                 **D**   Medium-Low
  Fixed Window Rate Limiter                 **B+**       Medium
  Sliding Window Rate Limiter            **B+/A-**       Medium
  Token Bucket                           **C+/B-**       Medium

## 21. 接下来两天复习优先级

### Priority 1

``` text
Graph
├── adjacency list
├── DFS
├── BFS
├── connected components
├── grid BFS / shortest path
└── topological sort

Tree
├── preorder / inorder / postorder
├── recursive DFS
├── level-order BFS
└── common tree recursion

Backtracking
├── subsets
├── permutations
├── generate parentheses
└── choose → explore → unchoose

DP
├── House Robber
└── 0/1 Knapsack
```

### Priority 2

``` text
LRU Cache → 完整手写一次
Rate Limiter → 完整手写一次
Monotonic Stack → 完整手写一次
```

### Priority 3

进入 Live Coding Mock。重点测试：

``` text
clarify
→ identify pattern
→ explain
→ code
→ test
→ handle follow-up
```

## 22. Jacksite 推荐结构

``` text
Interview Preparation
│
├── Algorithm Capability Map
├── Algorithm Patterns
│   ├── HashMap
│   ├── Sliding Window
│   ├── Binary Search
│   ├── Tree
│   ├── Graph
│   ├── Backtracking
│   ├── DP
│   ├── Knapsack
│   ├── Rate Limiter
│   └── LRU
├── Live Coding Retrospectives
├── System Design
│   ├── Asset Price Watching
│   ├── Order Routing
│   └── Account Opening
└── Interview Question Bank
```

Capability Map 应动态维护：

``` text
Graph = C+
→ 练习
→ Graph = B
→ Mock 中独立完成
→ Graph = B+
```

## 23. 本次摸底最值得保留的错误清单

1.  Tree traversal：Preorder / Inorder 一开始混淆。
2.  Tree complexity：Maximum Depth 一开始误判为 O(log n) time / O(1)
    space。
3.  Graph representation：一开始混淆 node 与 edge，不熟悉 adjacency
    list。
4.  Graph BFS：enqueue neighbor 时漏掉 `visited = true`。
5.  Topological Sort：dependency edge direction 写反。
6.  Topological Sort：decrement 了错误 node 的 indegree。
7.  0/1 Knapsack：没有稳定记住 capacity 必须倒序。
8.  Generate Parentheses：一开始识别成 Valid Parentheses / Stack。
9.  Backtracking：开始不理解 recursion 返回后为什么必须 unchoose。
10. Token Bucket：一开始理解成周期性 reset，而不是 elapsed-time refill。
11. Stock / Greedy：漏读 `buy once / sell once`，说明 requirement
    clarification 需要刻意执行。
12. Sliding Window Rate Limiter：主动识别 concurrency 问题；需记住问题是
    atomic check-and-update，而不是相同 timestamp 本身。

## 24. 面试前最终原则

不要追求"所有算法都学过"。

更重要的是：

> **强项稳定拿分，弱项补到能识别 pattern，并把 Live Coding
> 的完整执行流程练熟。**

每一道题强制按照：

``` text
Clarify
→ Define
→ Approach
→ Invariant / State
→ Complexity
→ Code
→ Test
→ Follow-up
```

执行。
