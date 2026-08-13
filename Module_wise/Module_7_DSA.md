# Module 7: Data Structures & Algorithms

> **Scope:** Core Algorithms, Complexity Analysis, Custom DS Design, Memory Efficiency, JVM Cache Locality
> **Questions:** 20 | **Critical:** 5 | **Coverage:** Product & Service-Based Companies | Sorted by interview frequency (descending)

---

## 🔴 CRITICAL / MUST-KNOW (Top 5)

---

### Q1. 🔴 🏢 How do you implement a thread-safe Least Recently Used (LRU) cache with $O(1)$ lookup and eviction time? Explain the internal mechanics.

**An LRU Cache is implemented using a combination of a `HashMap` for $O(1)$ key-based lookups and a custom doubly linked list to track the access order in $O(1)$ time, protected by a reentrant lock or synchronized blocks to ensure thread safety.**

**Internal Mechanics:**
1. **Doubly Linked List (DLL):** Maintains the recency of access. The head represents the most recently used (MRU) node, and the tail represents the least recently used (LRU) node.
2. **HashMap:** Maps keys to node references in the DLL. This bypasses the $O(N)$ list search, enabling instant node access.
3. **Eviction:** When the cache exceeds capacity, the node at the tail of the DLL is removed from both the list and the map in $O(1)$ time.
4. **Access Update:** When a key is read or updated, its node is moved to the head of the DLL in $O(1)$ time.

```java
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class LRUCache<K, V> {
    private class Node {
        K key;
        V value;
        Node prev;
        Node next;
        Node(K key, V value) {
            this.key = key;
            this.value = value;
        }
    }

    private final int capacity;
    private final Map<K, Node> map;
    private final Node head;
    private final Node tail;
    private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.map = new HashMap<>(capacity);
        this.head = new Node(null, null);
        this.tail = new Node(null, null);
        head.next = tail;
        tail.prev = head;
    }

    public V get(K key) {
        rwLock.writeLock().lock(); // Write lock because we modify the DLL structure on access
        try {
            Node node = map.get(key);
            if (node == null) return null;
            moveToHead(node);
            return node.value;
        } finally {
            rwLock.writeLock().unlock();
        }
    }

    public void put(K key, V value) {
        rwLock.writeLock().lock();
        try {
            Node node = map.get(key);
            if (node != null) {
                node.value = value;
                moveToHead(node);
            } else {
                if (map.size() >= capacity) {
                    Node lru = tail.prev;
                    removeNode(lru);
                    map.remove(lru.key);
                }
                Node newNode = new Node(key, value);
                addNode(newNode);
                map.put(key, newNode);
            }
        } finally {
            rwLock.writeLock().unlock();
        }
    }

    private void addNode(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }

    private void removeNode(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void moveToHead(Node node) {
        removeNode(node);
        addNode(node);
    }
}
```

**⚠️ Pitfalls & Edge Cases:**
- **ReentrantReadWriteLock Lock Upgrades:** In Java, you cannot upgrade a read lock to a write lock directly. That is why `get` uses the `writeLock` because updating the DLL order is a write operation.
- **Java's `LinkedHashMap` Alternative:** You can implement this quickly using `LinkedHashMap` by overriding `removeEldestEntry()`, but you must synchronize it externally (e.g., via `Collections.synchronizedMap`).

---

### Q2. 🔴 🏢 How do you merge $k$ sorted lists using a Priority Queue? Analyze the time and space complexity.

**Merging $k$ sorted lists is optimally achieved by utilizing a Min-Heap (implemented via Java's `PriorityQueue`) containing the first element of each list, iteratively extracting the minimum element, and adding the next element from the corresponding list.**

**Algorithm Steps:**
1. Insert the head node of all $k$ non-empty lists into a Min-Heap.
2. Initialize a dummy node to build the result list.
3. While the Min-Heap is not empty:
   - Extract the node with the minimum value.
   - Append it to the result list.
   - If the extracted node has a next node, insert that next node into the heap.

**Complexity Analysis:**
- **Time Complexity:** $O(N \log k)$, where $N$ is the total number of elements across all lists and $k$ is the number of lists. Each insertion and deletion in the Min-Heap of size $k$ takes $O(\log k)$ time.
- **Space Complexity:** $O(k)$ auxiliary space to store the elements in the priority queue.

```java
import java.util.PriorityQueue;

public class MergeKSortedLists {
    public static class ListNode {
        int val;
        ListNode next;
        ListNode(int val) { this.val = val; }
    }

    public ListNode mergeKLists(ListNode[] lists) {
        if (lists == null || lists.length == 0) return null;
        
        PriorityQueue<ListNode> minHeap = new PriorityQueue<>(
            lists.length, (a, b) -> Integer.compare(a.val, b.val)
        );

        // Populate initial heap
        for (ListNode node : lists) {
            if (node != null) {
                minHeap.offer(node);
            }
        }

        ListNode dummy = new ListNode(0);
        ListNode current = dummy;

        while (!minHeap.isEmpty()) {
            ListNode smallest = minHeap.poll();
            current.next = smallest;
            current = current.next;

            if (smallest.next != null) {
                minHeap.offer(smallest.next);
            }
        }

        return dummy.next;
    }
}
```

**⚠️ Pitfalls & Edge Cases:**
- **Null Pointer Exceptions:** Ensure you handle empty lists (`null` inside the array) and empty input arrays.
- **Garbage Collection (GC) Pressure:** Instantiating a `PriorityQueue` wrapper or comparator lamdas inside hot paths can generate object allocations. In highly optimized environments, arrays can be used to manually manage a min-heap structure.

---

### Q3. 🔴 🌐 Analyze the memory footprint and CPU cache locality of a Java `ArrayList` versus a `LinkedList`.

**`ArrayList` utilizes a contiguous array of object references which benefits from CPU cache line prefetching but suffers from overhead due to object wrappers, while `LinkedList` disperses nodes across the heap, completely destroying cache locality and wasting massive amounts of memory due to object headers and pointer fields.**

**Memory Layout Comparison:**

| Feature | `ArrayList<Integer>` | `LinkedList<Integer>` |
|---------|-----------------------|------------------------|
| **Memory Layout** | Contiguous array of references to `Integer` objects. | Scattered node objects linked by pointers. |
| **Node Overhead** | No node wrapper. Just array elements. | Node object contains `item`, `next`, and `prev` references. |
| **Object Header** | 12 or 16 bytes for the array object itself. | 12 or 16 bytes *per node* (plus padding). |
| **Cache Locality** | **High** for reference traversal; **Maximum** if using primitive arrays. | **Extremely Low** due to pointer chasing across heap pages. |

**Why Cache Locality Matters:**
- CPUs fetch data from memory into L1/L2 caches in chunks called **Cache Lines** (typically 64 bytes).
- Traverse a primitive `int[]` or `ArrayList`: The adjacent elements are loaded into the cache ahead of time (spatial locality).
- Traverse a `LinkedList`: Each node dereference (`node.next`) likely fetches a different memory address, triggering a CPU cache miss and waiting on main memory.

**⚠️ Pitfalls:**
- **Autoboxing Overhead:** `ArrayList<Integer>` still incurs cache misses when dereferencing the primitive wrapper. If memory and latency are critical, use primitive collections (e.g., Trove, Fastutil) or direct arrays.

---

### Q4. 🔴 🏢 Explain how to detect cycles and find the topological order in a directed graph using Kahn’s Algorithm.

**Kahn's Algorithm determines topological order by iteratively removing nodes with an in-degree of 0 (no dependencies) and updating their neighbors; if the resulting ordered list is smaller than the total vertex count, a cycle exists.**

**Mechanics:**
1. Compute the **in-degree** (number of incoming edges) for every vertex.
2. Push all vertices with an in-degree of 0 to a queue.
3. Track the number of visited nodes.
4. While the queue is not empty:
   - Poll a vertex $u$ and append it to the topological order list.
   - For each outgoing edge from $u$ to $v$, decrement the in-degree of $v$ by 1.
   - If the in-degree of $v$ drops to 0, push it to the queue.
5. If the count of visited nodes is not equal to the total vertices, the graph contains a cycle (cannot be fully ordered).

```java
import java.util.*;

public class TopologicalSortKahn {
    public List<Integer> findOrder(int numCourses, int[][] prerequisites) {
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
        
        int[] inDegree = new int[numCourses];
        for (int[] edge : prerequisites) {
            adj.get(edge[1]).add(edge[0]);
            inDegree[edge[0]]++;
        }

        Queue<Integer> queue = new LinkedList<>();
        for (int i = 0; i < numCourses; i++) {
            if (inDegree[i] == 0) {
                queue.offer(i);
            }
        }

        List<Integer> order = new ArrayList<>();
        while (!queue.isEmpty()) {
            int curr = queue.poll();
            order.add(curr);

            for (int neighbor : adj.get(curr)) {
                inDegree[neighbor]--;
                if (inDegree[neighbor] == 0) {
                    queue.offer(neighbor);
                }
            }
        }

        if (order.size() != numCourses) {
            return new ArrayList<>(); // Cycle detected, order is invalid
        }
        return order;
    }
}
```

**⚠️ Pitfalls:**
- **Graph Representation:** Using matrix representations for sparse graphs leads to $O(V^2)$ complexity. Always use an adjacency list for $O(V + E)$ time complexity.

---

### Q5. 🔴 🏢 How do you find the sliding window maximum of an array in linear $O(N)$ time? Explain the monotonic queue concept.

**The sliding window maximum is solved in $O(N)$ time using a Monotonic Double-Ended Queue (Deque) that stores array indices, maintaining elements in decreasing order so the maximum is always at the head.**

**Mechanics:**
1. As the window slides, for each new element $A[i]$:
   - Remove indices from the tail of the Deque if the corresponding array elements are less than or equal to $A[i]$ (since they can never be the maximum of this or future windows).
   - Add the current index $i$ to the tail.
2. Remove indices from the head of the Deque if they fall outside the current window range ($< i - k + 1$).
3. The element corresponding to the index at the head of the Deque is the maximum of the current window.

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class SlidingWindowMax {
    public int[] maxSlidingWindow(int[] nums, int k) {
        if (nums == null || nums.length == 0) return new int[0];
        
        int n = nums.length;
        int[] result = new int[n - k + 1];
        Deque<Integer> deque = new ArrayDeque<>(); // Stores indices

        for (int i = 0; i < n; i++) {
            // Remove indices out of current window boundaries
            if (!deque.isEmpty() && deque.peekFirst() < i - k + 1) {
                deque.pollFirst();
            }

            // Maintain monotonic decreasing order in deque
            while (!deque.isEmpty() && nums[deque.peekLast()] <= nums[i]) {
                deque.pollLast();
            }

            deque.offerLast(i);

            // Record maximum once first window is fully covered
            if (i >= k - 1) {
                result[i - k + 1] = nums[deque.peekFirst()];
            }
        }
        return result;
    }
}
```

**⚠️ Pitfalls:**
- **Primitive ArrayDeque Efficiency:** Standard `ArrayDeque` in Java wraps primitives, but because we only store indexes up to $N$, this is highly optimized. Do not use standard `LinkedList` as a queue here due to node allocation overhead.

---

## 🟡 HIGH FREQUENCY (Questions 6–12)

---

### Q6. 🟡 🏢 How do you search for a target element in a rotated sorted array in $O(\log N)$ time?

**Searching a rotated sorted array is achieved via binary search by identifying which half of the array is sorted at each step, and checking if the target lies within the boundaries of that sorted half.**

```java
public class RotatedSortedArray {
    public int search(int[] nums, int target) {
        int left = 0, right = nums.length - 1;

        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] == target) return mid;

            // Check if left half is sorted
            if (nums[left] <= nums[mid]) {
                if (target >= nums[left] && target < nums[mid]) {
                    right = mid - 1;
                } else {
                    left = mid + 1;
                }
            } 
            // Otherwise, right half must be sorted
            else {
                if (target > nums[mid] && target <= nums[right]) {
                    left = mid + 1;
                } else {
                    right = mid - 1;
                }
            }
        }
        return -1;
    }
}
```

**⚠️ Pitfalls:**
- **Duplicate Elements:** If duplicates are allowed (e.g., `[1,0,1,1,1]`), worst-case time complexity degrades to $O(N)$ because we cannot reliably determine which half is sorted when `nums[left] == nums[mid] == nums[right]`.

---

### Q7. 🟡 🏢 Explain the $O(N \log N)$ algorithm for the Longest Increasing Subsequence (LIS) problem.

**The $O(N \log N)$ LIS algorithm uses dynamic programming combined with binary search (Patience Sorting), maintaining a list of active tail elements of all increasing subsequences found so far.**

```java
import java.util.Arrays;

public class LISBinarySearch {
    public int lengthOfLIS(int[] nums) {
        if (nums == null || nums.length == 0) return 0;
        
        int[] tails = new int[nums.length];
        int size = 0;

        for (int x : nums) {
            int i = Arrays.binarySearch(tails, 0, size, x);
            if (i < 0) {
                i = -(i + 1); // Get insertion point
            }
            tails[i] = x;
            if (i == size) {
                size++;
            }
        }
        return size;
    }
}
```

**⚠️ Pitfalls:**
- **Subsequence Extraction:** This algorithm efficiently calculates the *length* of the LIS, but the elements inside the `tails` array at the end do *not* necessarily represent the actual LIS path.

---

### Q8. 🟡 🏢 How do you solve the "Number of Islands" problem efficiently? Compare DFS and BFS in terms of spatial overhead.

**The "Number of Islands" problem uses DFS or BFS to traverse connected land cells (`'1'`), marking them as visited (or turning them to `'0'`) to avoid duplicate processing.**

**Spatial Overhead Comparison:**
- **DFS:** Relies on the execution call stack. In the worst-case (a grid full of land), the recursion depth can reach $O(M \times N)$, potentially causing a `StackOverflowError` on large grids.
- **BFS:** Uses an explicit queue. The maximum size of the queue corresponds to the perimeter/breadth of the grid traversal, which in the worst case is $O(\min(M, N))$, making it much safer for memory consumption on huge grids.

---

### Q9. 🟡 🌐 How do you implement a custom HashTable from scratch? Explain handling collisions via chaining and open addressing.

**A custom HashTable requires an array of buckets, a hash function to map keys to bucket indices, and a resolution strategy like Chaining (linked nodes per bucket) or Open Addressing (probing empty cells) to resolve hash collisions.**

```java
public class CustomHashMap<K, V> {
    private static class Entry<K, V> {
        final K key;
        V value;
        Entry<K, V> next;
        Entry(K key, V value) {
            this.key = key;
            this.value = value;
        }
    }

    private Entry<K, V>[] buckets;
    private int capacity = 16;
    private int size = 0;
    private static final float LOAD_FACTOR = 0.75f;

    @SuppressWarnings("unchecked")
    public CustomHashMap() {
        buckets = new Entry[capacity];
    }

    public void put(K key, V value) {
        int index = getBucketIndex(key);
        Entry<K, V> head = buckets[index];
        while (head != null) {
            if (head.key.equals(key)) {
                head.value = value;
                return;
            }
            head = head.next;
        }
        Entry<K, V> newEntry = new Entry<>(key, value);
        newEntry.next = buckets[index];
        buckets[index] = newEntry;
        size++;

        if ((float) size / capacity >= LOAD_FACTOR) {
            resize();
        }
    }

    public V get(K key) {
        int index = getBucketIndex(key);
        Entry<K, V> head = buckets[index];
        while (head != null) {
            if (head.key.equals(key)) return head.value;
            head = head.next;
        }
        return null;
    }

    private int getBucketIndex(K key) {
        if (key == null) return 0;
        return Math.abs(key.hashCode()) % capacity;
    }

    @SuppressWarnings("unchecked")
    private void resize() {
        capacity *= 2;
        Entry<K, V>[] oldBuckets = buckets;
        buckets = new Entry[capacity];
        size = 0;

        for (Entry<K, V> head : oldBuckets) {
            while (head != null) {
                put(head.key, head.value);
                head = head.next;
            }
        }
    }
}
```

---

### Q10. 🟡 🏢 Write a thread-safe implementation of a Token Bucket rate-limiting algorithm in Java.

**The Token Bucket algorithm regulates request traffic by generating tokens at a constant rate up to a max capacity, letting requests pass only if they can consume a token, which is implemented in Java using synchronization or atomic updates.**

```java
import java.util.concurrent.atomic.AtomicLong;

public class TokenBucketRateLimiter {
    private final long capacity;
    private final long refillRatePerMs;
    private final AtomicLong tokens = new AtomicLong(0);
    private final AtomicLong lastRefillTimestamp = new AtomicLong(0);

    public TokenBucketRateLimiter(long capacity, long refillRatePerSecond) {
        this.capacity = capacity;
        this.refillRatePerMs = refillRatePerSecond / 1000;
        this.lastRefillTimestamp.set(System.currentTimeMillis());
        this.tokens.set(capacity);
    }

    public boolean allowRequest() {
        refill();
        while (true) {
            long currentTokens = tokens.get();
            if (currentTokens <= 0) {
                return false;
            }
            if (tokens.compareAndSet(currentTokens, currentTokens - 1)) {
                return true;
            }
        }
    }

    private void refill() {
        long now = System.currentTimeMillis();
        long last = lastRefillTimestamp.get();
        long elapsed = now - last;
        if (elapsed <= 0) return;

        long tokensToAdd = elapsed * refillRatePerMs;
        if (tokensToAdd > 0) {
            if (lastRefillTimestamp.compareAndSet(last, now)) {
                tokens.updateAndGet(current -> Math.min(capacity, current + tokensToAdd));
            }
        }
    }
}
```

---

### Q11. 🟡 🏢 Explain how to merge overlapping intervals in $O(N \log N)$ time.

**Merging overlapping intervals is achieved by sorting the intervals by their start times and then iterating through them, merging the current interval with the last interval in our result list if they overlap.**

```java
import java.util.Arrays;
import java.util.LinkedList;

public class MergeIntervals {
    public int[][] merge(int[][] intervals) {
        if (intervals.length <= 1) return intervals;

        // Sort by start times
        Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));

        LinkedList<int[]> merged = new LinkedList<>();
        for (int[] interval : intervals) {
            if (merged.isEmpty() || merged.getLast()[1] < interval[0]) {
                merged.add(interval);
            } else {
                // Overlap exists; merge by updating the end time
                merged.getLast()[1] = Math.max(merged.getLast()[1], interval[1]);
            }
        }
        return merged.toArray(new int[merged.size()][]);
    }
}
```

---

### Q12. 🟡 🏢 How do you implement a Trie (Prefix Tree) data structure for efficient search and auto-complete?

**A Trie organizes dictionary keys in a tree-like hierarchy of nodes representing characters, enabling prefix searches and word insertions in $O(L)$ time where $L$ is the length of the word.**

```java
public class Trie {
    private static class TrieNode {
        private final TrieNode[] children = new TrieNode[26];
        private boolean isWord;
    }

    private final TrieNode root = new TrieNode();

    public void insert(String word) {
        TrieNode curr = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (curr.children[idx] == null) {
                curr.children[idx] = new TrieNode();
            }
            curr = curr.children[idx];
        }
        curr.isWord = true;
    }

    public boolean search(String word) {
        TrieNode node = findNode(word);
        return node != null && node.isWord;
    }

    public boolean startsWith(String prefix) {
        return findNode(prefix) != null;
    }

    private TrieNode findNode(String str) {
        TrieNode curr = root;
        for (char c : str.toCharArray()) {
            int idx = c - 'a';
            if (curr.children[idx] == null) return null;
            curr = curr.children[idx];
        }
        return curr;
    }
}
```

---

## 🟢 GOOD TO KNOW (Questions 13–20)

---

### Q13. 🟢 🌐 What sorting algorithms does Java's `Arrays.sort()` use internally for primitive arrays vs. object arrays, and why?

**Java uses Dual-Pivot Quicksort for sorting primitives (prioritizing raw performance and saving memory since stability is irrelevant), and Timsort for objects to guarantee stable sorting ($O(N \log N)$ worst-case) while exploiting pre-sorted patterns in real-world data.**

- **Primitives (Dual-Pivot Quicksort):** Primitives do not require stable sorting (e.g., one `5` is identical to another `5`). Quicksort is in-place ($O(1)$ extra space) and fast.
- **Objects (Timsort):** Object sorting must be stable (preserving original order of equal elements). Timsort combines Merge Sort and Insertion Sort. It has a guaranteed $O(N \log N)$ time complexity and is highly adaptive.

---

### Q14. 🟢 🏢 How do you find the longest palindromic substring in $O(N^2)$ time and $O(1)$ space?

**The longest palindromic substring is solved in $O(1)$ space by expanding outward from each character (and between each character pair) acting as potential centers of a palindrome.**

```java
public class LongestPalindromeSubstring {
    public String longestPalindrome(String s) {
        if (s == null || s.length() < 1) return "";
        int start = 0, end = 0;

        for (int i = 0; i < s.length(); i++) {
            int len1 = expandAroundCenter(s, i, i);     // Odd palindromes
            int len2 = expandAroundCenter(s, i, i + 1); // Even palindromes
            int len = Math.max(len1, len2);

            if (len > end - start) {
                start = i - (len - 1) / 2;
                end = i + len / 2;
            }
        }
        return s.substring(start, end + 1);
    }

    private int expandAroundCenter(String s, int left, int right) {
        while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
            left--;
            right++;
        }
        return right - left - 1;
    }
}
```

---

### Q15. 🟢 🏢 Find the top $K$ frequent elements in an array. Optimize the time complexity to $O(N \log K)$.

**Finding the top $K$ frequent elements is optimized using a Map to count frequencies, and a Min-Heap of size $K$ to retain only the highest-frequency elements.**

```java
import java.util.*;

public class TopKFrequentElements {
    public int[] topKFrequent(int[] nums, int k) {
        Map<Integer, Integer> counts = new HashMap<>();
        for (int num : nums) {
            counts.put(num, counts.getOrDefault(num, 0) + 1);
        }

        PriorityQueue<Integer> heap = new PriorityQueue<>(
            (a, b) -> Integer.compare(counts.get(a), counts.get(b))
        );

        for (int num : counts.keySet()) {
            heap.add(num);
            if (heap.size() > k) {
                heap.poll();
            }
        }

        int[] result = new int[k];
        for (int i = k - 1; i >= 0; i--) {
            result[i] = heap.poll();
        }
        return result;
    }
}
```

---

### Q16. 🟢 🏬 Write a recursive and an iterative DFS method to find the Lowest Common Ancestor (LCA) of two nodes in a Binary Tree.

**LCA is found recursively by traversing the tree to see if nodes reside in left/right subtrees, or iteratively by tracking node-to-parent associations in a Map and tracing path intersections.**

```java
public class LowestCommonAncestor {
    public static class TreeNode {
        int val;
        TreeNode left, right;
        TreeNode(int x) { val = x; }
    }

    // Recursive Approach
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null || root == p || root == q) return root;

        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);

        if (left != null && right != null) return root; // LCA found
        return left != null ? left : right;
    }
}
```

---

### Q17. 🟢 🏢 How do you solve the Edit Distance (Levenshtein Distance) problem using dynamic programming?

**Edit Distance uses a 2D DP matrix to calculate operations needed (insert, delete, replace) to transform one string to another, optimizing space to $O(\text{min}(M, N))$ by storing only the previous row.**

```java
public class EditDistance {
    public int minDistance(String word1, String word2) {
        int m = word1.length(), n = word2.length();
        int[] dp = new int[n + 1];

        for (int j = 0; j <= n; j++) dp[j] = j;

        for (int i = 1; i <= m; i++) {
            int prev = dp[0];
            dp[0] = i;
            for (int j = 1; j <= n; j++) {
                int temp = dp[j];
                if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                    dp[j] = prev;
                } else {
                    dp[j] = 1 + Math.min(prev, Math.min(dp[j - 1], dp[j]));
                }
                prev = temp;
            }
        }
        return dp[n];
    }
}
```

---

### Q18. 🟢 🏢 Explain how to find the Median of Two Sorted Arrays in $O(\log(\min(M, N)))$ time complexity.

**The median of two sorted arrays is calculated by performing binary search on the partition split of the smaller array, ensuring that left elements are less than or equal to right elements.**

```java
public class MedianSortedArrays {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        if (nums1.length > nums2.length) {
            return findMedianSortedArrays(nums2, nums1); // Ensure nums1 is smaller
        }
        int x = nums1.length, y = nums2.length;
        int low = 0, high = x;

        while (low <= high) {
            int partitionX = (low + high) / 2;
            int partitionY = (x + y + 1) / 2 - partitionX;

            int maxLeftX = (partitionX == 0) ? Integer.MIN_VALUE : nums1[partitionX - 1];
            int minRightX = (partitionX == x) ? Integer.MAX_VALUE : nums1[partitionX];

            int maxLeftY = (partitionY == 0) ? Integer.MIN_VALUE : nums2[partitionY - 1];
            int minRightY = (partitionY == y) ? Integer.MAX_VALUE : nums2[partitionY];

            if (maxLeftX <= minRightY && maxLeftY <= minRightX) {
                if ((x + y) % 2 == 0) {
                    return ((double) Math.max(maxLeftX, maxLeftY) + Math.min(minRightX, minRightY)) / 2;
                } else {
                    return Math.max(maxLeftX, maxLeftY);
                }
            } else if (maxLeftX > minRightY) {
                high = partitionX - 1;
            } else {
                low = partitionX + 1;
            }
        }
        throw new IllegalArgumentException("Input arrays are not sorted.");
    }
}
```

---

### Q19. 🟢 🏢 What is a Segment Tree? Implement a Segment Tree for Range Sum Query with mutable updates in $O(\log N)$ time.

**A Segment Tree is a binary tree storing precomputed interval results, allowing range queries and element updates in logarithmic time.**

```java
public class SegmentTree {
    private final int[] tree;
    private final int n;

    public SegmentTree(int[] nums) {
        n = nums.length;
        tree = new int[2 * n];
        buildTree(nums);
    }

    private void buildTree(int[] nums) {
        for (int i = 0; i < n; i++) {
            tree[n + i] = nums[i];
        }
        for (int i = n - 1; i > 0; --i) {
            tree[i] = tree[2 * i] + tree[2 * i + 1];
        }
    }

    public void update(int index, int val) {
        index += n;
        tree[index] = val;
        while (index > 0) {
            int left = index;
            int right = index;
            if (index % 2 == 0) {
                right = index + 1;
            } else {
                left = index - 1;
            }
            tree[index / 2] = tree[left] + tree[right];
            index /= 2;
        }
    }

    public int sumRange(int left, int right) {
        left += n;
        right += n;
        int sum = 0;
        while (left <= right) {
            if ((left % 2) == 1) {
                sum += tree[left];
                left++;
            }
            if ((right % 2) == 0) {
                sum += tree[right];
                right--;
            }
            left /= 2;
            right /= 2;
        }
        return sum;
    }
}
```

---

### Q20. 🟢 🏬 Explain how to use Bitmasks and Bitwise Operations to represent sets and generate combinations.

**Bitmasks represent subset selections where each bit position denotes element presence/absence, enabling constant-time union, intersection, and traversal operations.**

```java
import java.util.ArrayList;
import java.util.List;

public class BitmaskSubsets {
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        int n = nums.length;
        int totalSubsets = 1 << n; // 2^n combinations

        for (int mask = 0; mask < totalSubsets; mask++) {
            List<Integer> subset = new ArrayList<>();
            for (int i = 0; i < n; i++) {
                // If the i-th bit is set, include nums[i] in this subset
                if ((mask & (1 << i)) != 0) {
                    subset.add(nums[i]);
                }
            }
            result.add(subset);
        }
        return result;
    }
}
```
