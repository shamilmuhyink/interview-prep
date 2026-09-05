# Simple DSA Questions for Java Developers

This document contains **50** common, fundamental Data Structures and Algorithms (DSA) questions frequently asked in Java developer interviews (service & product companies), along with optimized answers and Java code snippets.
**Questions are sorted by frequency and probability of being asked in a real interview, starting from the most commonly asked.**

---

## 🔴 Tier 1: Extremely High Frequency (Must-Know)

### Q1. Two Sum — Find two numbers that add up to a target.
⚡ **O(N)** Time | **O(N)** Space
```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement))
            return new int[] { map.get(complement), i };
        map.put(nums[i], i);
    }
    return new int[]{};
}
```

### Q2. Valid Parentheses — Check if brackets are balanced using a Stack.
⚡ **O(N)** Time | **O(N)** Space
```java
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        if (c == '(') stack.push(')');
        else if (c == '{') stack.push('}');
        else if (c == '[') stack.push(']');
        else if (stack.isEmpty() || stack.pop() != c) return false;
    }
    return stack.isEmpty();
}
```

### Q3. Reverse a Singly Linked List.
⚡ **O(N)** Time | **O(1)** Space
```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null, current = head;
    while (current != null) {
        ListNode next = current.next;
        current.next = prev;
        prev = current;
        current = next;
    }
    return prev;
}
```

### Q4. Detect a Cycle in a Linked List (Floyd's Algorithm).
⚡ **O(N)** Time | **O(1)** Space
```java
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

### Q5. Maximum Subarray Sum (Kadane's Algorithm).
⚡ **O(N)** Time | **O(1)** Space
```java
public int maxSubArray(int[] nums) {
    int maxSum = nums[0], currentSum = 0;
    for (int num : nums) {
        if (currentSum < 0) currentSum = 0;
        currentSum += num;
        maxSum = Math.max(maxSum, currentSum);
    }
    return maxSum;
}
```

### Q6. Best Time to Buy and Sell Stock — One transaction for max profit.
⚡ **O(N)** Time | **O(1)** Space
```java
public int maxProfit(int[] prices) {
    int minPrice = Integer.MAX_VALUE, maxProfit = 0;
    for (int price : prices) {
        minPrice = Math.min(minPrice, price);
        maxProfit = Math.max(maxProfit, price - minPrice);
    }
    return maxProfit;
}
```

### Q7. Merge Two Sorted Linked Lists.
⚡ **O(N + M)** Time | **O(1)** Space
```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0), current = dummy;
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) { current.next = l1; l1 = l1.next; }
        else { current.next = l2; l2 = l2.next; }
        current = current.next;
    }
    current.next = (l1 != null) ? l1 : l2;
    return dummy.next;
}
```

---

## 🟠 Tier 2: Very High Frequency

### Q8. Reverse a String (without built-in reverse).
⚡ **O(N)** Time | **O(N)** Space
```java
public String reverseString(String s) {
    char[] chars = s.toCharArray();
    int left = 0, right = chars.length - 1;
    while (left < right) {
        char temp = chars[left];
        chars[left++] = chars[right];
        chars[right--] = temp;
    }
    return new String(chars);
}
```

### Q9. Palindrome Check — Is a string the same forwards and backwards?
⚡ **O(N)** Time | **O(1)** Space
```java
public boolean isPalindrome(String s) {
    int left = 0, right = s.length() - 1;
    while (left < right) {
        while (left < right && !Character.isLetterOrDigit(s.charAt(left))) left++;
        while (left < right && !Character.isLetterOrDigit(s.charAt(right))) right--;
        if (Character.toLowerCase(s.charAt(left++)) != Character.toLowerCase(s.charAt(right--)))
            return false;
    }
    return true;
}
```

### Q10. Anagram Check — Do two strings have the same character frequencies?
⚡ **O(N)** Time | **O(1)** Space (fixed 26-char array)
```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    int[] counts = new int[26];
    for (int i = 0; i < s.length(); i++) {
        counts[s.charAt(i) - 'a']++;
        counts[t.charAt(i) - 'a']--;
    }
    for (int c : counts) if (c != 0) return false;
    return true;
}
```

### Q11. First Non-Repeating Character in a String.
⚡ **O(N)** Time | **O(1)** Space (fixed 26-char array)
```java
public int firstUniqChar(String s) {
    int[] counts = new int[26];
    for (char c : s.toCharArray()) counts[c - 'a']++;
    for (int i = 0; i < s.length(); i++)
        if (counts[s.charAt(i) - 'a'] == 1) return i;
    return -1;
}
```

### Q12. Find the Middle of a Linked List (One Pass).
⚡ **O(N)** Time | **O(1)** Space
```java
public ListNode middleNode(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}
```

### Q13. Binary Search in a Sorted Array.
⚡ **O(log N)** Time | **O(1)** Space
```java
public int binarySearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
```

### Q14. Longest Substring Without Repeating Characters (Sliding Window).
⚡ **O(N)** Time | **O(min(N, M))** Space — M is charset size
```java
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> map = new HashMap<>();
    int maxLen = 0;
    for (int j = 0, i = 0; j < s.length(); j++) {
        if (map.containsKey(s.charAt(j)))
            i = Math.max(map.get(s.charAt(j)) + 1, i);
        maxLen = Math.max(maxLen, j - i + 1);
        map.put(s.charAt(j), j);
    }
    return maxLen;
}
```

---

## 🟡 Tier 3: High Frequency

### Q15. Move Zeroes to the End of an Array (in-place).
⚡ **O(N)** Time | **O(1)** Space
```java
public void moveZeroes(int[] nums) {
    int idx = 0;
    for (int num : nums)
        if (num != 0) nums[idx++] = num;
    while (idx < nums.length) nums[idx++] = 0;
}
```

### Q16. Find the Missing Number in [0..N].
⚡ **O(N)** Time | **O(1)** Space — Uses Gauss formula
```java
public int missingNumber(int[] nums) {
    int n = nums.length;
    int expectedSum = n * (n + 1) / 2;
    int actualSum = 0;
    for (int num : nums) actualSum += num;
    return expectedSum - actualSum;
}
```

### Q17. Merge Two Sorted Arrays into One.
⚡ **O(N + M)** Time | **O(N + M)** Space
```java
public int[] mergeArrays(int[] a, int[] b) {
    int[] result = new int[a.length + b.length];
    int i = 0, j = 0, k = 0;
    while (i < a.length && j < b.length)
        result[k++] = (a[i] <= b[j]) ? a[i++] : b[j++];
    while (i < a.length) result[k++] = a[i++];
    while (j < b.length) result[k++] = b[j++];
    return result;
}
```

### Q18. Maximum Depth (Height) of a Binary Tree.
⚡ **O(N)** Time | **O(H)** Space — H = height of tree
```java
public int maxDepth(TreeNode root) {
    if (root == null) return 0;
    return Math.max(maxDepth(root.left), maxDepth(root.right)) + 1;
}
```

### Q19. Remove Duplicates from a Sorted Array (in-place).
⚡ **O(N)** Time | **O(1)** Space
```java
public int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;
    int i = 0;
    for (int j = 1; j < nums.length; j++)
        if (nums[j] != nums[i]) nums[++i] = nums[j];
    return i + 1;
}
```

### Q20. Intersection of Two Arrays.
⚡ **O(N + M)** Time | **O(N)** Space
```java
public int[] intersection(int[] nums1, int[] nums2) {
    Set<Integer> set = new HashSet<>();
    for (int n : nums1) set.add(n);
    Set<Integer> result = new HashSet<>();
    for (int n : nums2) if (set.contains(n)) result.add(n);
    return result.stream().mapToInt(Integer::intValue).toArray();
}
```

### Q21. Fibonacci Number (Iterative — O(1) Space).
⚡ **O(N)** Time | **O(1)** Space
```java
public int fib(int n) {
    if (n <= 1) return n;
    int a = 0, b = 1;
    for (int i = 2; i <= n; i++) {
        int temp = a + b;
        a = b;
        b = temp;
    }
    return b;
}
```

### Q22. Invert (Mirror) a Binary Tree.
⚡ **O(N)** Time | **O(H)** Space
```java
public TreeNode invertTree(TreeNode root) {
    if (root == null) return null;
    TreeNode temp = root.left;
    root.left = invertTree(root.right);
    root.right = invertTree(temp);
    return root;
}
```

### Q23. Binary Tree Level Order Traversal (BFS).
⚡ **O(N)** Time | **O(N)** Space
```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int size = queue.size();
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add(level);
    }
    return result;
}
```

### Q24. Validate a Binary Search Tree.
⚡ **O(N)** Time | **O(H)** Space
```java
public boolean isValidBST(TreeNode root) {
    return validate(root, Long.MIN_VALUE, Long.MAX_VALUE);
}
private boolean validate(TreeNode node, long min, long max) {
    if (node == null) return true;
    if (node.val <= min || node.val >= max) return false;
    return validate(node.left, min, node.val) && validate(node.right, node.val, max);
}
```

---

## 🟢 Tier 4: Medium Frequency

### Q25. Find Max and Min in an Array (Single Pass).
⚡ **O(N)** Time | **O(1)** Space
```java
public int[] findMinMax(int[] arr) {
    int min = arr[0], max = arr[0];
    for (int i = 1; i < arr.length; i++) {
        if (arr[i] > max) max = arr[i];
        if (arr[i] < min) min = arr[i];
    }
    return new int[] { min, max };
}
```

### Q26. Factorial of a Number (Iterative).
⚡ **O(N)** Time | **O(1)** Space
```java
public long factorial(int n) {
    long result = 1;
    for (int i = 2; i <= n; i++) result *= i;
    return result;
}
```

### Q27. Rotate an Array by K Positions.
⚡ **O(N)** Time | **O(1)** Space — Three-reversal trick
```java
public void rotate(int[] nums, int k) {
    k %= nums.length;
    reverse(nums, 0, nums.length - 1);
    reverse(nums, 0, k - 1);
    reverse(nums, k, nums.length - 1);
}
private void reverse(int[] nums, int l, int r) {
    while (l < r) {
        int temp = nums[l]; nums[l++] = nums[r]; nums[r--] = temp;
    }
}
```

### Q28. Sort an Array of 0s, 1s, and 2s (Dutch National Flag).
⚡ **O(N)** Time | **O(1)** Space
```java
public void sortColors(int[] nums) {
    int low = 0, mid = 0, high = nums.length - 1;
    while (mid <= high) {
        if (nums[mid] == 0) { swap(nums, low++, mid++); }
        else if (nums[mid] == 1) { mid++; }
        else { swap(nums, mid, high--); }
    }
}
private void swap(int[] a, int i, int j) {
    int t = a[i]; a[i] = a[j]; a[j] = t;
}
```

### Q29. Contains Duplicate — Check if any value appears twice.
⚡ **O(N)** Time | **O(N)** Space
```java
public boolean containsDuplicate(int[] nums) {
    Set<Integer> seen = new HashSet<>();
    for (int num : nums)
        if (!seen.add(num)) return true;
    return false;
}
```

### Q30. Implement Stack using Two Queues.
⚡ Push: **O(N)** | Pop: **O(1)**
```java
class MyStack {
    private Queue<Integer> queue = new LinkedList<>();

    public void push(int x) {
        queue.add(x);
        for (int i = 1; i < queue.size(); i++)
            queue.add(queue.remove());
    }
    public int pop() { return queue.remove(); }
    public int top() { return queue.peek(); }
    public boolean empty() { return queue.isEmpty(); }
}
```

### Q31. Implement Queue using Two Stacks.
⚡ Amortised **O(1)** per operation
```java
class MyQueue {
    private Deque<Integer> in = new ArrayDeque<>(), out = new ArrayDeque<>();

    public void push(int x) { in.push(x); }
    public int pop() { shift(); return out.pop(); }
    public int peek() { shift(); return out.peek(); }
    public boolean empty() { return in.isEmpty() && out.isEmpty(); }

    private void shift() {
        if (out.isEmpty()) while (!in.isEmpty()) out.push(in.pop());
    }
}
```

### Q32. Merge Overlapping Intervals.
⚡ **O(N log N)** Time | **O(N)** Space
```java
public int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
    LinkedList<int[]> merged = new LinkedList<>();
    for (int[] i : intervals) {
        if (merged.isEmpty() || merged.getLast()[1] < i[0])
            merged.add(i);
        else
            merged.getLast()[1] = Math.max(merged.getLast()[1], i[1]);
    }
    return merged.toArray(new int[0][]);
}
```

### Q33. Product of Array Except Self (without division).
⚡ **O(N)** Time | **O(1)** Space (output array doesn't count)
```java
public int[] productExceptSelf(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    result[0] = 1;
    for (int i = 1; i < n; i++)
        result[i] = result[i - 1] * nums[i - 1];  // left prefix products
    int right = 1;
    for (int i = n - 1; i >= 0; i--) {
        result[i] *= right;
        right *= nums[i];                           // right suffix products
    }
    return result;
}
```

### Q34. Reverse Words in a String.
⚡ **O(N)** Time | **O(N)** Space
```java
public String reverseWords(String s) {
    String[] words = s.trim().split("\\s+");
    StringBuilder sb = new StringBuilder();
    for (int i = words.length - 1; i >= 0; i--) {
        sb.append(words[i]);
        if (i > 0) sb.append(' ');
    }
    return sb.toString();
}
```

### Q35. Remove Nth Node From End of Linked List (One Pass).
⚡ **O(N)** Time | **O(1)** Space
```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0, head);
    ListNode fast = dummy, slow = dummy;
    for (int i = 0; i <= n; i++) fast = fast.next;
    while (fast != null) { fast = fast.next; slow = slow.next; }
    slow.next = slow.next.next;
    return dummy.next;
}
```

### Q36. Lowest Common Ancestor of a Binary Search Tree.
⚡ **O(H)** Time | **O(1)** Space (iterative)
```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    while (root != null) {
        if (p.val < root.val && q.val < root.val) root = root.left;
        else if (p.val > root.val && q.val > root.val) root = root.right;
        else return root;
    }
    return null;
}
```

---

## 🔵 Tier 5: Good to Know

### Q37. Check if a Number is Prime.
⚡ **O(√N)** Time | **O(1)** Space
```java
public boolean isPrime(int n) {
    if (n < 2) return false;
    if (n < 4) return true;
    if (n % 2 == 0 || n % 3 == 0) return false;
    for (int i = 5; i * i <= n; i += 6)
        if (n % i == 0 || n % (i + 2) == 0) return false;
    return true;
}
```

### Q38. Reverse an Integer.
⚡ **O(log N)** Time | **O(1)** Space
```java
public int reverse(int x) {
    long reversed = 0;
    while (x != 0) {
        reversed = reversed * 10 + x % 10;
        x /= 10;
    }
    return (reversed > Integer.MAX_VALUE || reversed < Integer.MIN_VALUE) ? 0 : (int) reversed;
}
```

### Q39. Check if a Linked List is a Palindrome.
⚡ **O(N)** Time | **O(1)** Space — Reverse second half, then compare
```java
public boolean isPalindrome(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) { slow = slow.next; fast = fast.next.next; }
    // Reverse second half
    ListNode prev = null;
    while (slow != null) { ListNode next = slow.next; slow.next = prev; prev = slow; slow = next; }
    // Compare
    ListNode left = head, right = prev;
    while (right != null) {
        if (left.val != right.val) return false;
        left = left.next; right = right.next;
    }
    return true;
}
```

### Q40. String Compression — "aabcccaaa" → "a2b1c3a3".
⚡ **O(N)** Time | **O(N)** Space
```java
public String compress(String s) {
    StringBuilder sb = new StringBuilder();
    int i = 0;
    while (i < s.length()) {
        char ch = s.charAt(i);
        int count = 0;
        while (i < s.length() && s.charAt(i) == ch) { i++; count++; }
        sb.append(ch).append(count);
    }
    return sb.length() < s.length() ? sb.toString() : s;
}
```

### Q41. Climbing Stairs — How many distinct ways to reach the top?
⚡ **O(N)** Time | **O(1)** Space — Same recurrence as Fibonacci
```java
public int climbStairs(int n) {
    if (n <= 2) return n;
    int a = 1, b = 2;
    for (int i = 3; i <= n; i++) {
        int temp = a + b;
        a = b;
        b = temp;
    }
    return b;
}
```

### Q42. Kth Largest Element in an Array.
⚡ **O(N log K)** Time | **O(K)** Space — Min-Heap approach
```java
public int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    for (int num : nums) {
        minHeap.offer(num);
        if (minHeap.size() > k) minHeap.poll();
    }
    return minHeap.peek();
}
```

### Q43. Search in Rotated Sorted Array.
⚡ **O(log N)** Time | **O(1)** Space
```java
public int search(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) return mid;
        if (nums[left] <= nums[mid]) { // left half sorted
            if (target >= nums[left] && target < nums[mid]) right = mid - 1;
            else left = mid + 1;
        } else { // right half sorted
            if (target > nums[mid] && target <= nums[right]) left = mid + 1;
            else right = mid - 1;
        }
    }
    return -1;
}
```

### Q44. Number of Islands (BFS/DFS on a Grid).
⚡ **O(M × N)** Time | **O(M × N)** Space
```java
public int numIslands(char[][] grid) {
    int count = 0;
    for (int i = 0; i < grid.length; i++)
        for (int j = 0; j < grid[0].length; j++)
            if (grid[i][j] == '1') { dfs(grid, i, j); count++; }
    return count;
}
private void dfs(char[][] grid, int i, int j) {
    if (i < 0 || j < 0 || i >= grid.length || j >= grid[0].length || grid[i][j] != '1') return;
    grid[i][j] = '0';
    dfs(grid, i + 1, j); dfs(grid, i - 1, j);
    dfs(grid, i, j + 1); dfs(grid, i, j - 1);
}
```

### Q45. Coin Change — Minimum coins to make a target amount.
⚡ **O(amount × N)** Time | **O(amount)** Space — DP
```java
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);
    dp[0] = 0;
    for (int i = 1; i <= amount; i++)
        for (int coin : coins)
            if (coin <= i) dp[i] = Math.min(dp[i], dp[i - coin] + 1);
    return dp[amount] > amount ? -1 : dp[amount];
}
```

### Q46. Longest Consecutive Sequence.
⚡ **O(N)** Time | **O(N)** Space
```java
public int longestConsecutive(int[] nums) {
    Set<Integer> set = new HashSet<>();
    for (int n : nums) set.add(n);
    int longest = 0;
    for (int n : set) {
        if (!set.contains(n - 1)) { // start of a sequence
            int len = 1;
            while (set.contains(n + len)) len++;
            longest = Math.max(longest, len);
        }
    }
    return longest;
}
```

### Q47. Group Anagrams.
⚡ **O(N × K log K)** Time | **O(N × K)** Space — K = max string length
```java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();
    for (String s : strs) {
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars);
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(map.values());
}
```

### Q48. Find the Second Largest Element in an Array.
⚡ **O(N)** Time | **O(1)** Space
```java
public int secondLargest(int[] arr) {
    int first = Integer.MIN_VALUE, second = Integer.MIN_VALUE;
    for (int num : arr) {
        if (num > first) { second = first; first = num; }
        else if (num > second && num != first) second = num;
    }
    return second; // Integer.MIN_VALUE if no second largest exists
}
```

### Q49. Trapping Rain Water.
⚡ **O(N)** Time | **O(1)** Space — Two-pointer approach
```java
public int trap(int[] height) {
    int left = 0, right = height.length - 1;
    int leftMax = 0, rightMax = 0, water = 0;
    while (left < right) {
        if (height[left] < height[right]) {
            leftMax = Math.max(leftMax, height[left]);
            water += leftMax - height[left++];
        } else {
            rightMax = Math.max(rightMax, height[right]);
            water += rightMax - height[right--];
        }
    }
    return water;
}
```

### Q50. Longest Palindromic Substring (Expand Around Center).
⚡ **O(N²)** Time | **O(1)** Space
```java
public String longestPalindrome(String s) {
    int start = 0, end = 0;
    for (int i = 0; i < s.length(); i++) {
        int len1 = expand(s, i, i);     // odd length
        int len2 = expand(s, i, i + 1); // even length
        int len = Math.max(len1, len2);
        if (len > end - start) {
            start = i - (len - 1) / 2;
            end = i + len / 2;
        }
    }
    return s.substring(start, end + 1);
}
private int expand(String s, int l, int r) {
    while (l >= 0 && r < s.length() && s.charAt(l) == s.charAt(r)) { l--; r++; }
    return r - l - 1;
}
```
