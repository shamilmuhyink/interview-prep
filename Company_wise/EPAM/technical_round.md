# EPAM Technical Round: Live Coding & Rapid Fire (Java)

> **Interview Format Note:** The **First Round (L1)** is typically a high-paced technical screening focusing heavily on **live coding (DSA/Programming)** and **rapid-fire conceptual questions** across the Core Java stack. Candidates are expected to write code (often without IDE assistance) and answer quick-fire theory questions back-to-back.
> **Skill Set:** Core Java, Streams API, Multithreading, Data Structures, HashMaps, Strings.
> **Sources:** AmbitionBox, Glassdoor, LeetCode, Candidate Experiences.
> **Note:** Questions are ranked strictly by interview frequency (descending) based on aggregated data for EPAM Java roles.

---

## 💻 Section 1: Live Coding (Data Structures & Algorithms)

### 1. 🌐 DSA: Group Anagrams from an array of strings.
**Answer:**
- **Problem:** Given an array of strings, group the anagrams together.
- **Optimized Solution:** Use a HashMap where the key is the sorted version of the string (or character count array) and the value is a list of anagrams.

```java
import java.util.*;

public class GroupAnagrams {
    public List<List<String>> groupAnagrams(String[] strs) {
        if (strs == null || strs.length == 0) return new ArrayList<>();
        Map<String, List<String>> map = new HashMap<>();
        
        for (String s : strs) {
            char[] ca = s.toCharArray();
            Arrays.sort(ca);
            String key = String.valueOf(ca);
            
            if (!map.containsKey(key)) {
                map.put(key, new ArrayList<>());
            }
            map.get(key).add(s);
        }
        return new ArrayList<>(map.values());
    }
}
```
**Time Complexity:** O(N * K log K) where N is strings count and K is max length. | **Space Complexity:** O(N * K)

---

### 2. 🌐 DSA: Find the Longest Substring Without Repeating Characters.
**Answer:**
- **Problem:** Given a string `s`, find the length of the longest substring without repeating characters.
- **Optimized Solution:** Use the Sliding Window technique with a HashSet or an integer array (for ASCII) to track character occurrences.

```java
public class LongestSubstring {
    public int lengthOfLongestSubstring(String s) {
        int n = s.length(), ans = 0;
        int[] index = new int[128]; // Current index of character
        
        // Try to extend the range [i, j]
        for (int j = 0, i = 0; j < n; j++) {
            i = Math.max(index[s.charAt(j)], i);
            ans = Math.max(ans, j - i + 1);
            index[s.charAt(j)] = j + 1;
        }
        return ans;
    }
}
```
**Time Complexity:** O(N) | **Space Complexity:** O(1) (fixed size array)

---

### 3. 🌐 DSA: Two Sum Problem. Write an optimized solution.
**Answer:**
- **Problem:** Given an array of integers and a target sum, return the indices of the two numbers that add up to the target.
- **Optimized Solution:** Use a HashMap to store the difference needed to reach the target.

```java
import java.util.HashMap;
import java.util.Map;

public class TwoSum {
    public int[] findTwoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();
        
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            
            if (map.containsKey(complement)) {
                return new int[] { map.get(complement), i };
            }
            map.put(nums[i], i);
        }
        throw new IllegalArgumentException("No two sum solution");
    }
}
```
**Time Complexity:** O(N) | **Space Complexity:** O(N)

---

### 4. 🌐 DSA: How do you merge two sorted arrays into a single sorted array in-place?
**Answer:**
- **Problem:** Given two sorted integer arrays `nums1` and `nums2`, merge `nums2` into `nums1` as one sorted array. Assume that `nums1` has a size equal to $m + n$.
- **Optimized Solution:** Start from the end of both arrays and place the larger element at the end of `nums1`.

```java
public class MergeSortedArrays {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int i = m - 1; 
        int j = n - 1; 
        int k = m + n - 1; 

        while (j >= 0) {
            if (i >= 0 && nums1[i] > nums2[j]) {
                nums1[k] = nums1[i];
                i--;
            } else {
                nums1[k] = nums2[j];
                j--;
            }
            k--;
        }
    }
}
```
**Time Complexity:** O(M + N) | **Space Complexity:** O(1)

---

### 5. 🌐 DSA: Write an optimized program to determine if a string of parentheses is valid.
**Answer:**
- **Problem:** Given a string containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`, determine if the input string is valid.
- **Optimized Solution:** Use a Stack or an `ArrayDeque` to match incoming closing brackets.

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class ValidParentheses {
    public boolean isValid(String s) {
        Deque<Character> stack = new ArrayDeque<>();
        for (char c : s.toCharArray()) {
            if (c == '(' || c == '{' || c == '[') {
                stack.push(c);
            } else {
                if (stack.isEmpty()) return false;
                char top = stack.pop();
                if ((c == ')' && top != '(') ||
                    (c == '}' && top != '{') ||
                    (c == ']' && top != '[')) {
                    return false;
                }
            }
        }
        return stack.isEmpty();
    }
}
```
**Time Complexity:** O(N) | **Space Complexity:** O(N)

---

## ⚡ Section 2: Rapid-Fire Core Java (Theory & Small Snippets)

### 6. 🌐 Java 8 Streams: How do you count the frequency of characters in a string using Streams?
**Answer:**
- A common rapid-fire question to test Streams API fluency.
```java
import java.util.Map;
import java.util.function.Function;
import java.util.stream.Collectors;

public class FrequencyCount {
    public static void main(String[] args) {
        String input = "epam systems";
        Map<Character, Long> frequency = input.chars()
            .mapToObj(c -> (char) c)
            .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
        System.out.println(frequency);
    }
}
```

---

### 7. 🌐 Java 8 Streams: Write a pipeline to filter even numbers, square them, and collect to a list.
**Answer:**
```java
import java.util.List;
import java.util.stream.Collectors;

public class StreamExample {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8);
        List<Integer> result = numbers.stream()
                .filter(n -> n % 2 == 0) // Filter even
                .map(n -> n * n)         // Square them
                .collect(Collectors.toList());
        System.out.println(result); // [4, 16, 36, 64]
    }
}
```

---

### 8. 🏢 Multithreading: Write a program to print Even and Odd numbers using two threads.
**Answer:**
- **Mechanism:** Use standard `wait()` and `notify()` on a shared lock object to alternate execution between threads.
```java
public class EvenOddPrinter {
    private int counter = 1;
    private final int MAX = 10;
    private final Object lock = new Object();

    public void printOdd() {
        synchronized (lock) {
            while (counter <= MAX) {
                if (counter % 2 == 0) {
                    try { lock.wait(); } catch (InterruptedException e) {}
                } else {
                    System.out.println("Odd: " + counter++);
                    lock.notify();
                }
            }
        }
    }

    public void printEven() {
        synchronized (lock) {
            while (counter <= MAX) {
                if (counter % 2 != 0) {
                    try { lock.wait(); } catch (InterruptedException e) {}
                } else {
                    System.out.println("Even: " + counter++);
                    lock.notify();
                }
            }
        }
    }
}
```

---

### 9. 🌐 Core Java: How does a HashMap work internally? What happens on collision?
**Answer:**
- **Internals:** `HashMap` uses an array of `Node` (bucket) objects. On `put(K,V)`, it calculates the hash, derives the bucket index, and places the node.
- **Collisions:** If multiple keys map to the same index, they form a Linked List.
- **Java 8 Optimization (Treeification):** If a bucket's linked list grows beyond 8 elements (`TREEIFY_THRESHOLD`), it transforms into a Red-Black tree to improve worst-case search time from $O(N)$ to $O(\log N)$.

---

### 10. 🏢 Core Java: What is the difference between HashMap and ConcurrentHashMap?
**Answer:**
- **HashMap:** Not thread-safe. Concurrent modifications can lead to a `ConcurrentModificationException` or an infinite loop (pre-Java 8).
- **ConcurrentHashMap:** Thread-safe, designed for high concurrency.
- **Internal Mechanism (Java 8+):** It abandons segment-level locking and uses **CAS (Compare-And-Swap)** operations and `synchronized` blocks at the **bucket level** (Node level). Multiple threads can write to different buckets simultaneously.
- **Null Keys/Values:** Neither `null` keys nor `null` values are allowed in `ConcurrentHashMap`.
