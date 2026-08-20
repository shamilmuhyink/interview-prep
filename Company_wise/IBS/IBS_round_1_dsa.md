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

### Q19: How to find if a Linked List has a cycle?
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

### Q20: Write a program to print a pyramid pattern of stars.
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

### Q21: How do you rotate a square matrix by 90 degrees in-place?
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

### Q22: How do you print a 2D matrix in a spiral order?
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
