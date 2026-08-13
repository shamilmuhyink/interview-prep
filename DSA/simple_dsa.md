# Simple DSA Questions for Java Developers

This document contains 20 common, fundamental Data Structures and Algorithms (DSA) questions frequently asked in Java developer interviews, along with simple answers and Java code snippets. 
**The questions have been sorted by their frequency and probability of being asked in a real interview, starting from the most commonly asked.**

---

## 🔴 Tier 1: Extremely High Frequency (Must-Know)

### Q1. Explain the "Two Sum" problem and provide an O(N) solution.

**Answer:** Given an array and a target, find two numbers that add up to the target. We use a `HashMap` to store numbers as we iterate, checking if the required difference already exists.

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[] { map.get(complement), i };
        }
        map.put(nums[i], i);
    }
    return new int[]{};
}
```

### Q2. How do you check for valid parentheses using a Stack?

**Answer:** Push opening brackets onto the stack. For closing brackets, check if they match the top of the stack.

```java
public boolean isValid(String s) {
    Stack<Character> stack = new Stack<>();
    for (char c : s.toCharArray()) {
        if (c == '(' || c == '{' || c == '[') {
            stack.push(c);
        } else {
            if (stack.isEmpty()) return false;
            char top = stack.pop();
            if ((c == ')' && top != '(') ||
                (c == '}' && top != '{') ||
                (c == ']' && top != '[')) return false;
        }
    }
    return stack.isEmpty();
}
```

### Q3. How do you reverse a singly linked list?

**Answer:** Iterate through the list, changing the `next` pointer of the current node to point to the previous node.

```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null, current = head;
    while (current != null) {
        ListNode nextNode = current.next;
        current.next = prev;
        prev = current;
        current = nextNode;
    }
    return prev;
}
```

### Q4. How do you detect a cycle (loop) in a linked list?

**Answer:** Use Floyd’s Cycle-Finding Algorithm (Tortoise and Hare). A slow pointer moves one step, a fast pointer moves two. If they meet, there is a cycle.

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

### Q5. Find the maximum subarray sum (Kadane's Algorithm).

**Answer:** Iterate through the array keeping a running sum. If the running sum becomes negative, reset it to zero. Track the maximum sum encountered.

```java
public int maxSubArray(int[] nums) {
    int maxSum = nums[0];
    int currentSum = 0;
    for (int num : nums) {
        if (currentSum < 0) currentSum = 0;
        currentSum += num;
        maxSum = Math.max(maxSum, currentSum);
    }
    return maxSum;
}
```

---

## 🟠 Tier 2: Very High Frequency

### Q6. How do you reverse a String in Java?

**Answer:** The easiest and most efficient way is to use the built-in `StringBuilder.reverse()` method, as Strings in Java are immutable.

```java
public String reverse(String str) {
    return new StringBuilder(str).reverse().toString();
}
```

### Q7. How do you check if a String is a palindrome?

**Answer:** A palindrome reads the same forwards and backwards. We can use a two-pointer approach comparing characters from the outside in.

```java
public boolean isPalindrome(String str) {
    int left = 0, right = str.length() - 1;
    while (left < right) {
        if (str.charAt(left++) != str.charAt(right--)) return false;
    }
    return true;
}
```

### Q8. Check if two strings are anagrams of each other.

**Answer:** Two strings are anagrams if they contain the same characters in the exact same frequencies. We can count character frequencies using an integer array.

```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    int[] counts = new int[26];
    for (int i = 0; i < s.length(); i++) {
        counts[s.charAt(i) - 'a']++;
        counts[t.charAt(i) - 'a']--;
    }
    for (int count : counts) {
        if (count != 0) return false;
    }
    return true;
}
```

### Q9. Find the first non-repeating character in a string.

**Answer:** Use a frequency map (or array) to count character occurrences, then iterate through the string again to find the first character with a count of 1.

```java
public int firstUniqChar(String s) {
    int[] counts = new int[26];
    for (char c : s.toCharArray()) counts[c - 'a']++;
    for (int i = 0; i < s.length(); i++) {
        if (counts[s.charAt(i) - 'a'] == 1) return i;
    }
    return -1;
}
```

### Q10. How do you find the middle element of a linked list in one pass?

**Answer:** Use the fast and slow pointer technique. The slow pointer moves one step, while the fast pointer moves two. When the fast pointer reaches the end, the slow pointer is at the middle.

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

---

## 🟡 Tier 3: High Frequency

### Q11. Write a Binary Search implementation.

**Answer:** Binary Search finds a target in a _sorted_ array by repeatedly dividing the search interval in half. Time complexity is O(log N).

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

### Q12. Move all zeros to the end of an array.

**Answer:** Use a pointer to keep track of the position for the next non-zero element. Iterate through the array and move non-zeros to the front, then fill the rest with zeros.

```java
public void moveZeroes(int[] nums) {
    int index = 0;
    for (int num : nums) {
        if (num != 0) {
            nums[index++] = num;
        }
    }
    while (index < nums.length) {
        nums[index++] = 0;
    }
}
```

### Q13. How do you find the missing number in a given integer array of 1 to N?

**Answer:** Calculate the expected sum using the formula `N * (N + 1) / 2` and subtract the actual sum of the array elements.

```java
public int findMissingNumber(int[] arr, int n) {
    int expectedSum = n * (n + 1) / 2;
    int actualSum = 0;
    for (int num : arr) {
        actualSum += num;
    }
    return expectedSum - actualSum;
}
```

### Q14. Merge two sorted arrays into one sorted array.

**Answer:** Use two pointers starting from the beginning of both arrays. Compare elements and add the smaller one to the result array.

```java
public int[] mergeArrays(int[] arr1, int[] arr2) {
    int[] result = new int[arr1.length + arr2.length];
    int i = 0, j = 0, k = 0;
    while (i < arr1.length && j < arr2.length) {
        if (arr1[i] < arr2[j]) result[k++] = arr1[i++];
        else result[k++] = arr2[j++];
    }
    while (i < arr1.length) result[k++] = arr1[i++];
    while (j < arr2.length) result[k++] = arr2[j++];
    return result;
}
```

### Q15. Calculate the maximum depth (height) of a Binary Tree.

**Answer:** Use recursion. The depth of a node is 1 plus the maximum depth of its left and right subtrees.

```java
public int maxDepth(TreeNode root) {
    if (root == null) return 0;
    int leftDepth = maxDepth(root.left);
    int rightDepth = maxDepth(root.right);
    return Math.max(leftDepth, rightDepth) + 1;
}
```

---

## 🟢 Tier 4: Medium Frequency

### Q16. How do you remove duplicates from a sorted array in place?

**Answer:** Use a two-pointer approach where one pointer iterates and the other tracks the position of the unique elements.

```java
public int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;
    int i = 0;
    for (int j = 1; j < nums.length; j++) {
        if (nums[j] != nums[i]) {
            i++;
            nums[i] = nums[j];
        }
    }
    return i + 1; // Length of unique array
}
```

### Q17. Find the intersection of two arrays.

**Answer:** Use a `HashSet` to store the elements of the first array, then iterate through the second array and check if the element exists in the set.

```java
public int[] intersection(int[] nums1, int[] nums2) {
    Set<Integer> set1 = new HashSet<>();
    for (int n : nums1) set1.add(n);

    Set<Integer> intersect = new HashSet<>();
    for (int n : nums2) {
        if (set1.contains(n)) intersect.add(n);
    }
    return intersect.stream().mapToInt(Number::intValue).toArray();
}
```

### Q18. Write a method to print the nth number in the Fibonacci sequence.

**Answer:** We can iterate from 2 to `n`, keeping track of the previous two numbers to calculate the next one.

```java
public int fibonacci(int n) {
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

---

## 🔵 Tier 5: Fundamental / Warm-up

### Q19. How do you find the maximum and minimum elements in an array?

**Answer:** Initialize `min` and `max` with the first element, then iterate through the rest of the array to update them.

```java
public void findMinMax(int[] arr) {
    if (arr == null || arr.length == 0) return;
    int min = arr[0], max = arr[0];
    for (int i = 1; i < arr.length; i++) {
        if (arr[i] > max) max = arr[i];
        if (arr[i] < min) min = arr[i];
    }
    System.out.println("Min: " + min + ", Max: " + max);
}
```

### Q20. Write a method to calculate the factorial of a number.

**Answer:** Factorial can be solved iteratively or recursively. The iterative approach avoids stack overflow for large numbers.

```java
public int factorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; i++) {
        result *= i;
    }
    return result;
}
```
