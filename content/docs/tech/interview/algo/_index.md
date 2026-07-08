---
title: "Algo 算法"
weight: 1
bookCollapseSection: true
---

## Pattern 1: HashMap

**When:** O(1) lookup, counting, grouping, finding pairs

```go
seen := map[keyType]valueType{}
for _, item := range items {
    if val, ok := seen[complement]; ok {
        // found match
    }
    seen[item] = value
}
```

**Sub-patterns:**

1. **Complement/Pair Lookup** — Store one, look up the other
2. **Frequency Counting** — Count occurrences, use counts for decisions
3. **Existence Set** — `map[int]bool{}` for O(1) "is it there?" checks

**Problems:** [Two Sum](two-sum/), [Group Anagrams](group-anagrams/), [Top K Frequent Elements](top-k-frequent-elements/), [Longest Consecutive Sequence](longest-consecutive-sequence/), [Clone Graph](clone-graph/)

---

## Pattern 2: Sliding Window

**When:** Find min/max/count of a subarray/substring satisfying a condition

```go
left := 0
for right := 0; right < len(s); right++ {
    // expand: add s[right] to window state

    for windowCondition() {
        // shrink: remove s[left] from window state
        left++
    }

    // update answer
}
```

**Two flavors:**

```go
// Flavor A: shrink when VALID → find minimum window
for formed == required {
    record minimum
    shrink left
}

// Flavor B: shrink when INVALID → find longest window
for windowIsInvalid() {
    shrink left
}
record maximum
```

**Problems:** [Longest Substring Without Repeating Characters](longest-substring-without-repeating/), [Minimum Window Substring](minimum-window-substring/)

---

## Pattern 3: Binary Search

**When:** Sorted data, O(log n) needed, "find boundary" problems

```go
left, right := 0, len(nums)-1
for left <= right {
    mid := left + (right-left)/2
    if nums[mid] == target {
        return mid
    } else if nums[mid] < target {
        left = mid + 1
    } else {
        right = mid - 1
    }
}
```

**Rotated array variant — determine which half is sorted first:**

```go
if nums[left] <= nums[mid] {  // left half sorted
    if target >= nums[left] && target < nums[mid] {
        right = mid - 1
    } else {
        left = mid + 1
    }
} else {  // right half sorted
    if target > nums[mid] && target <= nums[right] {
        left = mid + 1
    } else {
        right = mid - 1
    }
}
```

**Problems:** [Binary Search](binary-search/), [Search in Rotated Sorted Array](search-in-rotated-sorted-array/)

---

## Pattern 4: Heap

**When:** Kth largest/smallest, repeatedly extract min/max, priority-based processing

**Template (min heap of size k):**

```go
h := &MinHeap{}
heap.Init(h)
for _, item := range items {
    heap.Push(h, item)
    if h.Len() > k {
        heap.Pop(h)  // evict smallest, keep k largest
    }
}
// root = kth largest
```

**Go heap interface (memorize this):**

```go
type MinHeap []int
func (h MinHeap) Len() int            { return len(h) }
func (h MinHeap) Less(i, j int) bool  { return h[i] < h[j] }
func (h MinHeap) Swap(i, j int)       { h[i], h[j] = h[j], h[i] }
func (h *MinHeap) Push(x interface{}) { *h = append(*h, x.(int)) }
func (h *MinHeap) Pop() interface{} {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[0 : n-1]
    return x
}
```

**Key rules:**

- Your `Push` appends, your `Pop` removes last — the `heap` package handles ordering
- Always call `heap.Push` / `heap.Pop`, never `h.Push` / `h.Pop` directly
- Min heap: `h[i] < h[j]`, Max heap: `h[i] > h[j]`

**Problems:** [Kth Largest Element](kth-largest-element/), [Top K Frequent Elements](top-k-frequent-elements/), [Meeting Rooms II](meeting-rooms-ii/)

---

## Pattern 5: DFS (Depth-First Search)

**When:** Explore all paths, connected components, trees, cycle detection

**Template — Grid:**

```go
func dfs(grid [][]byte, i, j int) {
    if i < 0 || i >= len(grid) || j < 0 || j >= len(grid[0]) {
        return
    }
    if grid[i][j] == '0' {
        return
    }
    grid[i][j] = '0'
    dfs(grid, i+1, j)
    dfs(grid, i-1, j)
    dfs(grid, i, j+1)
    dfs(grid, i, j-1)
}
```

**Template — Graph (cycle detection, 3 states):**

```go
// 0=unvisited, 1=visiting, 2=visited
func hasCycle(graph [][]int, state []int, node int) bool {
    state[node] = 1
    for _, neighbor := range graph[node] {
        if state[neighbor] == 1 { return true }
        if state[neighbor] == 0 {
            if hasCycle(graph, state, neighbor) { return true }
        }
    }
    state[node] = 2
    return false
}
```

**Template — Tree:**

```go
func dfs(root *TreeNode) *TreeNode {
    if root == nil { return nil }
    left := dfs(root.Left)
    right := dfs(root.Right)
    // combine left and right results
    return result
}
```

**Problems:** [Number of Islands](number-of-islands/) (grid), [Course Schedule](course-schedule/) (cycle detection), [Lowest Common Ancestor](lowest-common-ancestor/) (tree), [Clone Graph](clone-graph/)

---

## Pattern 6: BFS (Breadth-First Search)

**When:** Shortest path, level-by-level traversal, nearest neighbor

```go
queue := []T{start}
visited := map[T]bool{start: true}

for len(queue) > 0 {
    levelSize := len(queue)
    for i := 0; i < levelSize; i++ {
        node := queue[0]
        queue = queue[1:]

        // process node

        for _, neighbor := range getNeighbors(node) {
            if !visited[neighbor] {
                visited[neighbor] = true
                queue = append(queue, neighbor)
            }
        }
    }
}
```

**DFS vs BFS:**

| | DFS | BFS |
|---|-----|-----|
| Data structure | Stack (recursion) | Queue |
| Use when | Explore all paths, detect cycles | Shortest path, level order |
| Space | O(depth) | O(width) |

**Problems:** [Binary Tree Level Order Traversal](binary-tree-level-order-traversal/), [Number of Islands](number-of-islands/) (BFS variant)

---

## Pattern 7: DP (1D Dynamic Programming)

**When:** Optimization (min/max) or counting with overlapping subproblems

```go
dp := make([]int, n)
dp[0] = baseCase

for i := 1; i < n; i++ {
    dp[i] = transition(dp[i-1], dp[i-2], ...)
}
return dp[n-1]
```

**Three types — know which formula to use:**

```go
// Counting (how many ways?) → ADD
dp[i] = dp[i-1] + dp[i-2]                     // Climbing Stairs

// Optimization (min cost?) → MIN/MAX
dp[i] = min(dp[i-coin] + 1, dp[i])            // Coin Change
dp[i] = max(dp[i-1], dp[i-2] + nums[i])       // House Robber

// Longest sequence → MAX with condition
dp[i] = max(dp[j]+1) for j < i where nums[j] < nums[i]  // LIS
```

**2D DP:**

```go
dp[i][j] = dp[i-1][j] + dp[i][j-1]           // Unique Paths
```

**Key questions to ask:**

1. What does `dp[i]` represent?
2. What's the base case?
3. What's the transition? (Add for counting, min/max for optimization)

**Problems:** [Climbing Stairs](climbing-stairs/), [Coin Change](coin-change/), [House Robber](house-robber/), [Unique Paths](unique-paths/), [Longest Increasing Subsequence](longest-increasing-subsequence/), [Maximum Subarray](maximum-subarray/)

---

## Pattern 8: Prefix Sum

**When:** Range sum queries, cumulative calculations, "product except self"

```go
// Build
prefix := make([]int, n+1)
for i := 0; i < n; i++ {
    prefix[i+1] = prefix[i] + nums[i]
}

// Query [left, right] in O(1)
sum := prefix[right+1] - prefix[left]
```

**Prefix/Suffix variant:**

```go
// Left pass
left[0] = base
for i := 1; i < n; i++ {
    left[i] = combine(left[i-1], nums[i-1])
}

// Right pass
right[n-1] = base
for i := n-2; i >= 0; i-- {
    right[i] = combine(right[i+1], nums[i+1])
}

// Combine
for i := 0; i < n; i++ {
    result[i] = merge(left[i], right[i])
}
```

**Problems:** [Range Sum Query](prefix-sums/), [Product of Array Except Self](product-of-array-except-self/), [Trapping Rain Water](trapping-rain-water/), [Count Divisors](count-divisors/)

---

## Quick Reference: Problem → Pattern

| Problem | Pattern(s) |
|---------|-----------|
| [Two Sum](two-sum/) | HashMap |
| [Product of Array Except Self](product-of-array-except-self/) | Prefix/Suffix |
| [Maximum Subarray](maximum-subarray/) | DP (Kadane's) |
| [Merge Intervals](merge-intervals/) | Sort + Sweep |
| [Longest Substring Without Repeating](longest-substring-without-repeating/) | Sliding Window + HashMap |
| [Container With Most Water](container-with-most-water/) | Two Pointers |
| [Trapping Rain Water](trapping-rain-water/) | Prefix/Suffix or Two Pointers |
| [Coin Change](coin-change/) | DP |
| [Climbing Stairs](climbing-stairs/) | DP |
| [House Robber](house-robber/) | DP |
| [Unique Paths](unique-paths/) | 2D DP |
| [Longest Increasing Subsequence](longest-increasing-subsequence/) | DP |
| [Range Sum Query](prefix-sums/) | Prefix Sum |
| [Count Divisors](count-divisors/) | Math (O(1)) |
| [MaxCounters](max-counters/) | Lazy Propagation |
| [Binary Search](binary-search/) | Binary Search |
| [Search in Rotated Sorted Array](search-in-rotated-sorted-array/) | Binary Search |
| [Kth Largest Element](kth-largest-element/) | Heap |
| [Number of Islands](number-of-islands/) | DFS/BFS |
| [Course Schedule](course-schedule/) | DFS + Topological Sort |
| [Lowest Common Ancestor](lowest-common-ancestor/) | DFS (Tree) |
| [Clone Graph](clone-graph/) | DFS + HashMap |
| [Group Anagrams](group-anagrams/) | HashMap |
| [Top K Frequent Elements](top-k-frequent-elements/) | HashMap + Heap |
| [Longest Consecutive Sequence](longest-consecutive-sequence/) | HashMap (Set) |
| [Minimum Window Substring](minimum-window-substring/) | Sliding Window + HashMap |
| [Binary Tree Level Order Traversal](binary-tree-level-order-traversal/) | BFS |
| [Meeting Rooms II](meeting-rooms-ii/) | Sort + Heap |
