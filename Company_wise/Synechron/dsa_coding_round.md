# Synechron Interview Questions - DSA Coding Round

> **Source:** Aggregated from LeetCode, Glassdoor, and Interview Experiences.
> **Focus:** Core Data Structures, Algorithms, Problem Solving, and Java Streams.
> **Format:** Questions are ordered by frequency based on recent interview experiences.

---

### Q1. 🔴 [🌐] How do you find the first non-repeating character in a String using Java 8 Streams?
**Answer:**
- **Complexity:** ⚡ Time: **O(N)** | Space: **O(N)**
- **Approach:** Map characters to their frequencies using `Collectors.groupingBy` with a `LinkedHashMap` (to preserve insertion order). Then filter for frequency == 1 and get the first element.

💻 **Production-quality code snippet:**
```java
public Character firstNonRepeating(String input) {
    return input.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(Function.identity(), LinkedHashMap::new, Collectors.counting()))
        .entrySet().stream()
        .filter(e -> e.getValue() == 1L)
        .map(Map.Entry::getKey)
        .findFirst()
        .orElse(null);
}
```

### Q2. 🔴 [🌐] Two Sum Problem.
**Answer:**
- **Complexity:** ⚡ Time: **O(N)** | Space: **O(N)**
- **Approach:** Use a `HashMap` to store numbers and indices. Check if `target - current` exists in the map to find the pair in O(1) time.

💻 **Code Snippet (Optimized):**
```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) return new int[] { map.get(complement), i };
        map.put(nums[i], i);
    }
    throw new IllegalArgumentException("No solution");
}
```

### Q3. 🔴 [🌐] How do you implement a custom LRU (Least Recently Used) Cache?
**Answer:**
- **Complexity:** ⚡ Time: **O(1)** | Space: **O(N)**
- **Approach:** Use a `HashMap` combined with a `Doubly Linked List`. The HashMap provides O(1) access, while the DLL manages the most/least recently used items via O(1) removals and additions at the ends.

💻 **Production-quality code snippet:**
```java
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    public LRUCache(int capacity) {
        super(capacity, 0.75f, true); // true for access-order
        this.capacity = capacity;
    }
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}
```

### Q4. 🟠 [🌐] Reverse a Linked List.
**Answer:**
- **Complexity:** ⚡ Time: **O(N)** | Space: **O(1)**
- **Approach:** Iterate through the list using three pointers (`prev`, `current`, `next`). In each step, change the `next` pointer of the current node to point to the previous node.

💻 **Code Snippet:**
```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode current = head;
    while (current != null) {
        ListNode nextTemp = current.next;
        current.next = prev;
        prev = current;
        current = nextTemp;
    }
    return prev;
}
```

### Q5. 🟠 [🌐] Check if a string is a Palindrome.
**Answer:**
- **Complexity:** ⚡ Time: **O(N)** | Space: **O(1)**
- **Approach:** Use two pointers, one at the start and one at the end of the string. Compare the characters and move the pointers towards the center. Ignore non-alphanumeric characters and case.

💻 **Code Snippet:**
```java
public boolean isPalindrome(String s) {
    int left = 0, right = s.length() - 1;
    while (left < right) {
        while (left < right && !Character.isLetterOrDigit(s.charAt(left))) left++;
        while (left < right && !Character.isLetterOrDigit(s.charAt(right))) right--;
        
        if (Character.toLowerCase(s.charAt(left)) != Character.toLowerCase(s.charAt(right))) {
            return false;
        }
        left++;
        right--;
    }
    return true;
}
```

### Q6. 🟡 [🌐] Implement a custom HashMap from scratch.
**Answer:**
- **Complexity:** ⚡ Time: **O(1)** average for put/get | Space: **O(N)**
- **Approach:** Create an array of buckets (Linked List nodes). Use a hash function to compute the bucket index. Handle collisions by chaining (adding to the linked list in the bucket).

💻 **Code Snippet (Simplified):**
```java
public class CustomHashMap<K, V> {
    private static class Entry<K, V> {
        final K key;
        V value;
        Entry<K, V> next;
        Entry(K key, V value) { this.key = key; this.value = value; }
    }
    private Entry<K, V>[] buckets = new Entry[16];

    public void put(K key, V value) {
        int index = Math.abs(key.hashCode()) % buckets.length;
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
    }

    public V get(K key) {
        int index = Math.abs(key.hashCode()) % buckets.length;
        Entry<K, V> head = buckets[index];
        while (head != null) {
            if (head.key.equals(key)) return head.value;
            head = head.next;
        }
        return null;
    }
}
```

### Q7. 🟢 [🌐] Reverse a String without using built-in methods.
**Answer:**
- **Complexity:** ⚡ Time: **O(N)** | Space: **O(N)** (due to character array)
- **Approach:** Convert the string to a character array and swap the characters from both ends moving towards the center.

💻 **Code Snippet:**
```java
public String reverseString(String s) {
    if (s == null || s.isEmpty()) return s;
    char[] chars = s.toCharArray();
    int left = 0, right = chars.length - 1;
    while (left < right) {
        char temp = chars[left];
        chars[left] = chars[right];
        chars[right] = temp;
        left++;
        right--;
    }
    return new String(chars);
}
```

### Q8. 🟡 [🌐] Binary Tree Level Order Traversal (BFS).
**Answer:**
- **Complexity:** ⚡ Time: **O(N)** | Space: **O(N)** (queue can contain at most N/2 nodes at the lowest level)
- **Approach:** Use a Queue to perform Breadth-First Search. Keep track of the number of nodes at the current level to group them in the output list.

💻 **Code Snippet:**
```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    
    while (!queue.isEmpty()) {
        int levelSize = queue.size();
        List<Integer> currentLevel = new ArrayList<>();
        
        for (int i = 0; i < levelSize; i++) {
            TreeNode current = queue.poll();
            currentLevel.add(current.val);
            if (current.left != null) queue.offer(current.left);
            if (current.right != null) queue.offer(current.right);
        }
        result.add(currentLevel);
    }
    return result;
}
```

### Q9. 🟡 [🌐] Longest Substring Without Repeating Characters.
**Answer:**
- **Complexity:** ⚡ Time: **O(N)** | Space: **O(min(N, M))** (where M is the charset size)
- **Approach:** Use the sliding window technique with a `HashSet` or `HashMap`. Expand the window by adding characters, and if a duplicate is found, shrink the window from the left until the duplicate is removed.

💻 **Code Snippet (Optimized using HashMap):**
```java
public int lengthOfLongestSubstring(String s) {
    int n = s.length(), maxLen = 0;
    Map<Character, Integer> map = new HashMap<>(); // current index of character
    
    for (int j = 0, i = 0; j < n; j++) {
        if (map.containsKey(s.charAt(j))) {
            i = Math.max(map.get(s.charAt(j)), i);
        }
        maxLen = Math.max(maxLen, j - i + 1);
        map.put(s.charAt(j), j + 1);
    }
    return maxLen;
}
```

### Q10. 🟢 [🌐] Implement Binary Search in a sorted array.
**Answer:**
- **Complexity:** ⚡ Time: **O(log N)** | Space: **O(1)**
- **Approach:** Maintain two pointers (`left` and `right`). Check the middle element. If it is the target, return its index. If the target is smaller, search the left half; if larger, search the right half.

💻 **Code Snippet:**
```java
public int binarySearch(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2; // Prevents integer overflow
        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return -1;
}
```
