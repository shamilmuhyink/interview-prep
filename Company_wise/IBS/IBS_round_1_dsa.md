# IBS Software Services: Round 1 (Coding & Technical) Interview Questions

This document strictly contains coding challenges and algorithmic questions ranked by their frequency of being asked in IBS Software Services technical rounds. IBS technical rounds heavily emphasize **Java 8 Streams API** and **String/Array manipulation** over generic algorithms.

## Tier 1: Java 8 Streams API (Most Frequent)

### Q1: How do you find the maximum and minimum numbers in a list using Java 8 Streams?
**Answer:**
```java
public void findMinMax(List<Integer> list) {
    int max = list.stream().max(Integer::compare).orElseThrow();
    int min = list.stream().min(Integer::compare).orElseThrow();
    System.out.println("Max: " + max + ", Min: " + min);
}
```

### Q2: How do you count the occurrences of each character in a String using Streams?
**Answer:**
```java
public Map<String, Long> countCharacters(String str) {
    return Arrays.stream(str.split(""))
                 .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
}
```

### Q3: How do you count the occurrences of each word in a String using Streams?
**Answer:**
```java
public Map<String, Long> countWords(String str) {
    return Arrays.stream(str.split("\\s+"))
                 .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
}
```

### Q4: How do you find duplicate elements in a List using Streams?
**Answer:**
```java
public Set<Integer> findDuplicates(List<Integer> list) {
    Set<Integer> items = new HashSet<>();
    return list.stream()
               .filter(n -> !items.add(n)) // Set.add() returns false if already present
               .collect(Collectors.toSet());
}
```

### Q5: How do you group a list of objects (e.g. Employees) by a property (e.g. Department) using Streams?
**Answer:**
```java
public Map<String, List<Employee>> groupEmployeesByDepartment(List<Employee> employees) {
    return employees.stream()
                    .collect(Collectors.groupingBy(Employee::getDepartment));
}
```

### Q6: How do you find the employee with the highest salary using Streams?
**Answer:**
```java
public Employee findHighestPaidEmployee(List<Employee> employees) {
    return employees.stream()
                    .max(Comparator.comparingDouble(Employee::getSalary))
                    .orElseThrow(() -> new RuntimeException("No employees found"));
}
```

### Q7: How do you sort a list of employees by age and then by salary using Streams?
**Answer:**
```java
public List<Employee> sortEmployeesByAgeAndSalary(List<Employee> employees) {
    return employees.stream()
                    .sorted(Comparator.comparingInt(Employee::getAge)
                                      .thenComparingDouble(Employee::getSalary))
                    .collect(Collectors.toList());
}
```

---
## Tier 2: String Manipulations

### Q8: Write logic to check if a String is a Palindrome.
**Answer:**
```java
public boolean isPalindrome(String str) {
    int left = 0, right = str.length() - 1;
    while(left < right) {
        if(str.charAt(left++) != str.charAt(right--)) return false;
    }
    return true;
}
```

### Q9: Check if two strings are Anagrams.
**Answer:**
Use an integer array to count the frequencies of characters (Time: O(N), Space: O(1)).
```java
public boolean isAnagram(String str1, String str2) {
    if (str1.length() != str2.length()) return false;
    int[] counts = new int[26];
    for (int i = 0; i < str1.length(); i++) {
        counts[str1.charAt(i) - 'a']++;
        counts[str2.charAt(i) - 'a']--;
    }
    for (int count : counts) {
        if (count != 0) return false;
    }
    return true;
}
```

### Q10: Find the first non-repeating character in a String.
**Answer:**
Use a `LinkedHashMap<Character, Integer>` to store the character counts while maintaining insertion order, then iterate and return the first char with count 1.
```java
public Character firstNonRepeatingChar(String str) {
    Map<Character, Integer> counts = new LinkedHashMap<>();
    for (char c : str.toCharArray()) {
        counts.put(c, counts.getOrDefault(c, 0) + 1);
    }
    for (Map.Entry<Character, Integer> entry : counts.entrySet()) {
        if (entry.getValue() == 1) {
            return entry.getKey();
        }
    }
    return null;
}
```

### Q11: Write a program to reverse a String without using built-in methods.
**Answer:**
```java
public String reverse(String str) {
    char[] chars = str.toCharArray();
    String reversed = "";
    for(int i = chars.length - 1; i >= 0; i--) reversed += chars[i];
    return reversed;
}
```

### Q12: Remove all whitespaces from a String without using the `replace()` method.
**Answer:**
```java
public String removeWhitespaces(String str) {
    StringBuilder sb = new StringBuilder();
    for (char c : str.toCharArray()) {
        if (c != ' ' && c != '\t' && c != '\n') {
            sb.append(c);
        }
    }
    return sb.toString();
}
```

### Q13: Count the number of vowels and consonants in a String.
**Answer:**
```java
public void countVowelsAndConsonants(String str) {
    int vowels = 0, consonants = 0;
    str = str.toLowerCase();
    for (char c : str.toCharArray()) {
        if (c >= 'a' && c <= 'z') {
            if (c == 'a' || c == 'e' || c == 'i' || c == 'o' || c == 'u') {
                vowels++;
            } else {
                consonants++;
            }
        }
    }
    System.out.println("Vowels: " + vowels + ", Consonants: " + consonants);
}
```

---
## Tier 3: Arrays & Linked Lists

### Q14: How do you find the second highest number in an array?
**Answer:**
```java
public int secondHighest(int[] arr) {
    int highest = Integer.MIN_VALUE, secondHighest = Integer.MIN_VALUE;
    for(int num : arr) {
        if(num > highest) {
            secondHighest = highest;
            highest = num;
        } else if (num > secondHighest && num != highest) {
            secondHighest = num;
        }
    }
    return secondHighest;
}
```

### Q15: How do you remove duplicate elements from an Array?
**Answer:**
Using a HashSet (O(N) time complexity):
```java
public Integer[] removeDuplicates(int[] arr) {
    Set<Integer> set = new LinkedHashSet<>();
    for(int num : arr) set.add(num);
    return set.toArray(new Integer[0]);
}
```

### Q16: Write a program to swap two numbers without using a third/temporary variable.
**Answer:**
```java
public void swap(int a, int b) {
    a = a + b;
    b = a - b;
    a = a - b;
}
```

### Q17: Reverse an Array in-place (without using a new array).
**Answer:**
```java
public void reverseArray(int[] arr) {
    int left = 0, right = arr.length - 1;
    while (left < right) {
        int temp = arr[left];
        arr[left] = arr[right];
        arr[right] = temp;
        left++;
        right--;
    }
}
```

### Q18: Find the intersection of two arrays.
**Answer:**
```java
public Integer[] findIntersection(int[] arr1, int[] arr2) {
    Set<Integer> set1 = new HashSet<>();
    Set<Integer> intersection = new HashSet<>();
    for (int num : arr1) set1.add(num);
    for (int num : arr2) {
        if (set1.contains(num)) {
            intersection.add(num);
        }
    }
    return intersection.toArray(new Integer[0]);
}
```

### Q19: Merge and sort two arrays into a single array.
**Answer:**
```java
public int[] mergeAndSort(int[] arr1, int[] arr2) {
    int[] merged = new int[arr1.length + arr2.length];
    System.arraycopy(arr1, 0, merged, 0, arr1.length);
    System.arraycopy(arr2, 0, merged, arr1.length, arr2.length);
    Arrays.sort(merged);
    return merged;
}
```

### Q20: How to find if a Linked List has a cycle?
**Answer:**
Use Floyd’s Cycle-Finding Algorithm (Tortoise and Hare). Move a `slow` pointer by 1 step and a `fast` pointer by 2 steps. If they meet, there is a cycle.
```java
class ListNode {
    int val;
    ListNode next;
    ListNode(int x) { val = x; next = null; }
}

public boolean hasCycle(ListNode head) {
    if (head == null || head.next == null) return false;
    ListNode slow = head;
    ListNode fast = head;
    
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {
            return true;
        }
    }
    return false;
}
```

---
## Tier 4: Logic Patterns & Matrices

### Q21: Write optimized logic to check if a number is Prime.
**Answer:**
Iterate up to the square root of the number to optimize performance. Time complexity is O(√N).
```java
public boolean isPrime(int n) {
    if (n <= 1) return false;
    if (n == 2 || n == 3) return true;
    if (n % 2 == 0 || n % 3 == 0) return false;
    
    for (int i = 5; i * i <= n; i += 6) {
        if (n % i == 0 || n % (i + 2) == 0) return false;
    }
    return true;
}
```

### Q22: Write a program to print a pyramid pattern of stars.
**Answer:**
```java
public void printPyramid(int rows) {
    for (int i = 1; i <= rows; i++) {
        for (int j = rows - i; j > 0; j--) System.out.print(" ");
        for (int k = 1; k <= (2 * i - 1); k++) System.out.print("*");
        System.out.println();
    }
}
```

### Q23: How do you rotate a square matrix by 90 degrees in-place?
**Answer:**
To rotate an N x N matrix by 90 degrees clockwise in-place, you first transpose the matrix (swap `matrix[i][j]` with `matrix[j][i]`), and then reverse each row.
```java
public void rotateMatrix(int[][] matrix) {
    int n = matrix.length;
    // Step 1: Transpose the matrix
    for (int i = 0; i < n; i++) {
        for (int j = i; j < n; j++) {
            int temp = matrix[i][j];
            matrix[i][j] = matrix[j][i];
            matrix[j][i] = temp;
        }
    }
    // Step 2: Reverse each row
    for (int i = 0; i < n; i++) {
        int left = 0, right = n - 1;
        while (left < right) {
            int temp = matrix[i][left];
            matrix[i][left] = matrix[i][right];
            matrix[i][right] = temp;
            left++;
            right--;
        }
    }
}
```

### Q24: How do you print a 2D matrix in a spiral order?
**Answer:**
You maintain four boundaries (`top`, `bottom`, `left`, `right`) and iterate through the matrix, adjusting the boundaries as you complete each row/column.
```java
public List<Integer> spiralOrder(int[][] matrix) {
    List<Integer> result = new ArrayList<>();
    if (matrix == null || matrix.length == 0) return result;
    
    int top = 0, bottom = matrix.length - 1;
    int left = 0, right = matrix[0].length - 1;
    
    while (top <= bottom && left <= right) {
        // Traverse top row
        for (int i = left; i <= right; i++) result.add(matrix[top][i]);
        top++;
        
        // Traverse right column
        for (int i = top; i <= bottom; i++) result.add(matrix[i][right]);
        right--;
        
        if (top <= bottom) {
            // Traverse bottom row
            for (int i = right; i >= left; i--) result.add(matrix[bottom][i]);
            bottom--;
        }
        
        if (left <= right) {
            // Traverse left column
            for (int i = bottom; i >= top; i--) result.add(matrix[i][left]);
            left++;
        }
    }
    return result;
}
```

### Q25: Write a function to check if a 3x3 matrix is a valid Magic Square.
**Answer:**
A magic square is a matrix where the sum of every row, column, and both diagonals is the same. For a 3x3 matrix containing numbers 1-9, the sum is always 15.
```java
public boolean isMagicSquare(int[][] grid) {
    int expectedSum = 15;
    
    // Check diagonals
    int diag1 = grid[0][0] + grid[1][1] + grid[2][2];
    int diag2 = grid[0][2] + grid[1][1] + grid[2][0];
    if (diag1 != expectedSum || diag2 != expectedSum) return false;
    
    // Check rows and columns
    for (int i = 0; i < 3; i++) {
        int rowSum = grid[i][0] + grid[i][1] + grid[i][2];
        int colSum = grid[0][i] + grid[1][i] + grid[2][i];
        if (rowSum != expectedSum || colSum != expectedSum) return false;
    }
    
    // Valid 3x3 Magic Square
    return true;
}
```

---
## Tier 5: Two Pointers & Sliding Window Patterns

### Q26: Two Sum
**Question:** Given an array of integers `nums` and an integer `target`, return indices of the two numbers such that they add up to `target`.
**Answer:**
Use a `HashMap` to store the numbers and their indices as you iterate. Time complexity is O(N).
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
    throw new IllegalArgumentException("No two sum solution");
}
```

### Q27: 3Sum
**Question:** Find all unique triplets in an array which gives the sum of zero.
**Answer:**
Sort the array first, then use a `for` loop combined with two pointers (left and right) to find the triplets. Time complexity is O(N^2).
```java
public List<List<Integer>> threeSum(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> res = new ArrayList<>();
    for (int i = 0; i < nums.length - 2; i++) {
        if (i == 0 || (i > 0 && nums[i] != nums[i - 1])) {
            int lo = i + 1, hi = nums.length - 1, sum = 0 - nums[i];
            while (lo < hi) {
                if (nums[lo] + nums[hi] == sum) {
                    res.add(Arrays.asList(nums[i], nums[lo], nums[hi]));
                    while (lo < hi && nums[lo] == nums[lo + 1]) lo++;
                    while (lo < hi && nums[hi] == nums[hi - 1]) hi--;
                    lo++; hi--;
                } else if (nums[lo] + nums[hi] < sum) {
                    lo++;
                } else {
                    hi--;
                }
            }
        }
    }
    return res;
}
```

### Q28: Longest Substring Without Repeating Characters
**Question:** Find the length of the longest substring without repeating characters.
**Answer:**
Use the Sliding Window technique with a `HashSet` to track characters in the current window.
```java
public int lengthOfLongestSubstring(String s) {
    Set<Character> set = new HashSet<>();
    int left = 0, maxLen = 0;
    
    for (int right = 0; right < s.length(); right++) {
        while (set.contains(s.charAt(right))) {
            set.remove(s.charAt(left));
            left++;
        }
        set.add(s.charAt(right));
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

### Q29: Maximum Sum Subarray of Size K
**Question:** Given an array of integers and a number `k`, find the maximum sum of a contiguous subarray of size `k`.
**Answer:**
Use a fixed-size Sliding Window. Add the next element and remove the first element of the previous window.
```java
public int maxSumSubarray(int[] arr, int k) {
    int maxSum = 0, windowSum = 0;
    for (int i = 0; i < k; i++) {
        windowSum += arr[i];
    }
    maxSum = windowSum;
    
    for (int i = k; i < arr.length; i++) {
        windowSum += arr[i] - arr[i - k];
        maxSum = Math.max(maxSum, windowSum);
    }
    return maxSum;
}
```

### Q30: Container With Most Water
**Question:** Given an integer array `height` representing vertical lines on a graph, find two lines that together with the x-axis form a container that holds the most water.
**Answer:**
Use Two Pointers starting at the extremes of the array, moving the pointer pointing to the shorter line inward.
```java
public int maxArea(int[] height) {
    int maxArea = 0;
    int left = 0, right = height.length - 1;
    
    while (left < right) {
        int width = right - left;
        int minHeight = Math.min(height[left], height[right]);
        maxArea = Math.max(maxArea, width * minHeight);
        
        if (height[left] < height[right]) {
            left++;
        } else {
            right--;
        }
    }
    return maxArea;
}
```
