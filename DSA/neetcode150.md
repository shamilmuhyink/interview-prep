# NeetCode 150 - Comprehensive Java Guide

This document contains all 150 NeetCode questions, complete with their exact LeetCode descriptions, algorithmic approaches, and full Java solutions.

> **Note**: Due to the massive size of 150 comprehensive solutions, this document is being populated in batches. This is **Batch 1 (Arrays & Hashing)**.

---

## 1. Arrays & Hashing

### 217. Contains Duplicate
**Description:**
Given an integer array `nums`, return `true` if any value appears at least twice in the array, and return `false` if every element is distinct.

**Algorithm:**
Use a Hash Set to keep track of the numbers seen so far. Iterate through the array; if the current number is already in the set, a duplicate exists (return `true`). Otherwise, add the number to the set. If the loop completes without finding duplicates, return `false`.
*   **Time Complexity:** O(N)
*   **Space Complexity:** O(N)

**Java Code:**
```java
class Solution {
    public boolean containsDuplicate(int[] nums) {
        HashSet<Integer> seen = new HashSet<>();
        for (int num : nums) {
            if (seen.contains(num)) {
                return true;
            }
            seen.add(num);
        }
        return false;
    }
}
```

---

### 242. Valid Anagram
**Description:**
Given two strings `s` and `t`, return `true` if `t` is an anagram of `s`, and `false` otherwise. 
An Anagram is a word or phrase formed by rearranging the letters of a different word or phrase, typically using all the original letters exactly once.

**Algorithm:**
If the lengths of the two strings are different, they cannot be anagrams (return `false`). Create an integer array of size 26 to store character frequencies. Iterate through both strings simultaneously: increment the count for characters in `s` and decrement the count for characters in `t`. Finally, iterate through the frequency array; if any count is non-zero, return `false`.
*   **Time Complexity:** O(N) where N is the string length.
*   **Space Complexity:** O(1) because the array is always size 26.

**Java Code:**
```java
class Solution {
    public boolean isAnagram(String s, String t) {
        if (s.length() != t.length()) return false;
        
        int[] count = new int[26];
        for (int i = 0; i < s.length(); i++) {
            count[s.charAt(i) - 'a']++;
            count[t.charAt(i) - 'a']--;
        }
        
        for (int n : count) {
            if (n != 0) return false;
        }
        return true;
    }
}
```

---

### 1. Two Sum
**Description:**
Given an array of integers `nums` and an integer `target`, return indices of the two numbers such that they add up to `target`.
You may assume that each input would have exactly one solution, and you may not use the same element twice. You can return the answer in any order.

**Algorithm:**
Use a Hash Map to store numbers and their corresponding indices as you iterate through the array. For each number, calculate its `complement` (`target - current_number`). Check if this complement already exists in the map. If it does, you have found the pair; return their indices. If not, add the current number and its index to the map.
*   **Time Complexity:** O(N)
*   **Space Complexity:** O(N)

**Java Code:**
```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        HashMap<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            if (map.containsKey(complement)) {
                return new int[] { map.get(complement), i };
            }
            map.put(nums[i], i);
        }
        return new int[0]; // Should not be reached per problem statement
    }
}
```

---

### 49. Group Anagrams
**Description:**
Given an array of strings `strs`, group the anagrams together. You can return the answer in any order.

**Algorithm:**
Use a Hash Map where the key is a unique signature for a group of anagrams, and the value is a list of strings that match that signature. The signature can be created by sorting the string or by creating a frequency string (e.g., `#1#0#0...`). Iterate through the strings, compute the signature for each, and append it to the corresponding list in the map.
*   **Time Complexity:** O(N * K log K) where N is the number of strings and K is the maximum length of a string (due to sorting).
*   **Space Complexity:** O(N * K) to store the hash map.

**Java Code:**
```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map = new HashMap<>();
        for (String s : strs) {
            char[] charArray = s.toCharArray();
            Arrays.sort(charArray);
            String key = String.valueOf(charArray);
            
            map.putIfAbsent(key, new ArrayList<>());
            map.get(key).add(s);
        }
        return new ArrayList<>(map.values());
    }
}
```

---

### 347. Top K Frequent Elements
**Description:**
Given an integer array `nums` and an integer `k`, return the `k` most frequent elements. You may return the answer in any order.

**Algorithm:**
1. Use a Hash Map to count the frequency of each number in the array.
2. Use Bucket Sort: create an array of lists `bucket` where the index represents the frequency, and the list at that index contains the numbers with that frequency.
3. Iterate through the `bucket` array from the end (highest frequency) to the beginning, collecting numbers until you have `k` elements.
*   **Time Complexity:** O(N)
*   **Space Complexity:** O(N)

**Java Code:**
```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        Map<Integer, Integer> countMap = new HashMap<>();
        for (int n : nums) {
            countMap.put(n, countMap.getOrDefault(n, 0) + 1);
        }
        
        List<Integer>[] bucket = new List[nums.length + 1];
        for (int key : countMap.keySet()) {
            int frequency = countMap.get(key);
            if (bucket[frequency] == null) {
                bucket[frequency] = new ArrayList<>();
            }
            bucket[frequency].add(key);
        }
        
        int[] res = new int[k];
        int counter = 0;
        for (int i = bucket.length - 1; i >= 0 && counter < k; i--) {
            if (bucket[i] != null) {
                for (int num : bucket[i]) {
                    res[counter++] = num;
                    if (counter == k) return res;
                }
            }
        }
        return res;
    }
}
```

---

### 238. Product of Array Except Self
**Description:**
Given an integer array `nums`, return an array `answer` such that `answer[i]` is equal to the product of all the elements of `nums` except `nums[i]`.
The product of any prefix or suffix of `nums` is guaranteed to fit in a 32-bit integer.
You must write an algorithm that runs in `O(n)` time and without using the division operation.

**Algorithm:**
Create a result array. Make a first pass from left to right: at each index `i`, store the product of all elements to the left of `i`. Maintain a running `prefix` variable for this.
Make a second pass from right to left: maintain a running `postfix` variable representing the product of all elements to the right of `i`. Multiply the current value in the result array by the `postfix` variable.
*   **Time Complexity:** O(N)
*   **Space Complexity:** O(1) (excluding the output array).

**Java Code:**
```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int[] res = new int[nums.length];
        
        int prefix = 1;
        for (int i = 0; i < nums.length; i++) {
            res[i] = prefix;
            prefix *= nums[i];
        }
        
        int postfix = 1;
        for (int i = nums.length - 1; i >= 0; i--) {
            res[i] *= postfix;
            postfix *= nums[i];
        }
        
        return res;
    }
}
```

---

### 36. Valid Sudoku
**Description:**
Determine if a `9 x 9` Sudoku board is valid. Only the filled cells need to be validated according to the following rules:
1. Each row must contain the digits `1-9` without repetition.
2. Each column must contain the digits `1-9` without repetition.
3. Each of the nine `3 x 3` sub-boxes of the grid must contain the digits `1-9` without repetition.

**Algorithm:**
Iterate through the 9x9 grid. For every filled cell (not `'.'`), attempt to add its value to a Hash Set for its specific row, specific column, and specific 3x3 block. We can encode these as strings like `"5 in row 0"`, `"5 in col 2"`, and `"5 in block 0-0"`. If `HashSet.add()` returns `false`, a collision occurred, meaning the board is invalid.
*   **Time Complexity:** O(1) (Since the board is always 9x9, it takes a constant 81 iterations).
*   **Space Complexity:** O(1) (At most 81 elements in the HashSet).

**Java Code:**
```java
class Solution {
    public boolean isValidSudoku(char[][] board) {
        Set<String> seen = new HashSet<>();
        for (int i = 0; i < 9; ++i) {
            for (int j = 0; j < 9; ++j) {
                char number = board[i][j];
                if (number != '.') {
                    if (!seen.add(number + " in row " + i) ||
                        !seen.add(number + " in column " + j) ||
                        !seen.add(number + " in block " + i/3 + "-" + j/3)) {
                        return false;
                    }
                }
            }
        }
        return true;
    }
}
```

---

### 271. Encode and Decode Strings
**Description:**
Design an algorithm to encode a list of strings to a string. The encoded string is then sent over the network and is decoded back to the original list of strings.

**Algorithm:**
*   **Encode:** For each string in the list, prepend its length followed by a delimiter character (e.g., `#`). So `"hello"` becomes `"5#hello"`. This ensures even if the string itself contains numbers or `#`, we know exactly how many characters to read.
*   **Decode:** Read the integer length until the delimiter `#`. Then read exactly that many characters to extract the string. Move the pointer forward and repeat until the end of the encoded string is reached.
*   **Time Complexity:** O(N) where N is the total length of all characters.
*   **Space Complexity:** O(N)

**Java Code:**
```java
public class Codec {
    // Encodes a list of strings to a single string.
    public String encode(List<String> strs) {
        StringBuilder sb = new StringBuilder();
        for (String s : strs) {
            sb.append(s.length()).append('#').append(s);
        }
        return sb.toString();
    }

    // Decodes a single string to a list of strings.
    public List<String> decode(String s) {
        List<String> res = new ArrayList<>();
        int i = 0;
        while (i < s.length()) {
            int j = i;
            while (s.charAt(j) != '#') j++;
            int length = Integer.parseInt(s.substring(i, j));
            
            i = j + 1 + length;
            res.add(s.substring(j + 1, i));
        }
        return res;
    }
}
```

---

### 128. Longest Consecutive Sequence
**Description:**
Given an unsorted array of integers `nums`, return the length of the longest consecutive elements sequence.
You must write an algorithm that runs in `O(n)` time.

**Algorithm:**
Insert all numbers into a Hash Set to allow O(1) lookups. Iterate through the set. For each number, check if it's the start of a sequence by verifying that `num - 1` does NOT exist in the set. If it is the start, continuously check for `num + 1`, `num + 2`, etc., in the set to find the length of the sequence. Keep track of the maximum length found.
*   **Time Complexity:** O(N). Although there is a nested loop, we only initiate the inner loop if a number is the start of a sequence. Thus, each number is visited at most twice.
*   **Space Complexity:** O(N)

**Java Code:**
```java
class Solution {
    public int longestConsecutive(int[] nums) {
        if (nums == null || nums.length == 0) return 0;
        
        Set<Integer> numSet = new HashSet<>();
        for (int num : nums) {
            numSet.add(num);
        }
        
        int longestStreak = 0;
        
        for (int num : numSet) {
            // Check if this is the start of a sequence
            if (!numSet.contains(num - 1)) {
                int currentNum = num;
                int currentStreak = 1;
                
                while (numSet.contains(currentNum + 1)) {
                    currentNum += 1;
                    currentStreak += 1;
                }
                
                longestStreak = Math.max(longestStreak, currentStreak);
            }
        }
        
        return longestStreak;
    }
}
```
