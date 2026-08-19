# LeetCode Top Interview 150 - Java Solutions

This document contains the Top Interview 150 questions, complete with their exact LeetCode descriptions, algorithmic approaches, and full Java solutions.

> **Note**: This document is being populated in batches. This is **Batch 1 (Array/String - Part 1)**.

---

## 1. Array / String

### 88. Merge Sorted Array
**Description:**
You are given two integer arrays `nums1` and `nums2`, sorted in non-decreasing order, and two integers `m` and `n`, representing the number of elements in `nums1` and `nums2` respectively.
Merge `nums1` and `nums2` into a single array sorted in non-decreasing order.
The final sorted array should not be returned by the function, but instead be stored inside the array `nums1`. To accommodate this, `nums1` has a length of `m + n`, where the first `m` elements denote the elements that should be merged, and the last `n` elements are set to `0` and should be ignored. `nums2` has a length of `n`.

**Algorithm:**
Since `nums1` has extra space at the end, we can use three pointers starting from the back. `p1` points to the last valid element in `nums1` (index `m - 1`), `p2` points to the last element in `nums2` (index `n - 1`), and `p` points to the last position in `nums1` (index `m + n - 1`). Compare elements at `p1` and `p2`, place the larger one at `p`, and decrement the respective pointers.
*   **Time Complexity:** O(m + n)
*   **Space Complexity:** O(1)

**Java Code:**
```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int p1 = m - 1;
        int p2 = n - 1;
        int p = m + n - 1;
        
        while (p1 >= 0 && p2 >= 0) {
            if (nums1[p1] > nums2[p2]) {
                nums1[p] = nums1[p1];
                p1--;
            } else {
                nums1[p] = nums2[p2];
                p2--;
            }
            p--;
        }
        
        // If there are remaining elements in nums2, copy them
        while (p2 >= 0) {
            nums1[p] = nums2[p2];
            p2--;
            p--;
        }
    }
}
```

---

### 27. Remove Element
**Description:**
Given an integer array `nums` and an integer `val`, remove all occurrences of `val` in `nums` in-place. The order of the elements may be changed. Then return the number of elements in `nums` which are not equal to `val`.
Consider the number of elements in `nums` which are not equal to `val` be `k`, to get accepted, you need to do the following things:
- Change the array `nums` such that the first `k` elements of `nums` contain the elements which are not equal to `val`. The remaining elements of `nums` are not important as well as the size of `nums`.
- Return `k`.

**Algorithm:**
Use a two-pointer approach. Maintain a slow pointer `k` initialized to `0`. Iterate through the array with a fast pointer `i`. If `nums[i]` is not equal to `val`, copy `nums[i]` to `nums[k]` and increment `k`. 
*   **Time Complexity:** O(n)
*   **Space Complexity:** O(1)

**Java Code:**
```java
class Solution {
    public int removeElement(int[] nums, int val) {
        int k = 0;
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] != val) {
                nums[k] = nums[i];
                k++;
            }
        }
        return k;
    }
}
```

---

### 26. Remove Duplicates from Sorted Array
**Description:**
Given an integer array `nums` sorted in non-decreasing order, remove the duplicates in-place such that each unique element appears only once. The relative order of the elements should be kept the same. Then return the number of unique elements in `nums`.
Consider the number of unique elements of `nums` to be `k`, to get accepted, you need to do the following things:
- Change the array `nums` such that the first `k` elements of `nums` contain the unique elements in the order they were present in `nums` initially. The remaining elements of `nums` are not important as well as the size of `nums`.
- Return `k`.

**Algorithm:**
Since the array is sorted, duplicates will be adjacent. Use a slow pointer `k` starting at `1` (since the first element is always unique). Iterate through the array with a fast pointer `i` starting at `1`. If `nums[i]` is different from `nums[i - 1]`, it's a new unique element. Copy it to `nums[k]` and increment `k`.
*   **Time Complexity:** O(n)
*   **Space Complexity:** O(1)

**Java Code:**
```java
class Solution {
    public int removeDuplicates(int[] nums) {
        if (nums.length == 0) return 0;
        int k = 1;
        for (int i = 1; i < nums.length; i++) {
            if (nums[i] != nums[i - 1]) {
                nums[k] = nums[i];
                k++;
            }
        }
        return k;
    }
}
```

---

### 80. Remove Duplicates from Sorted Array II
**Description:**
Given an integer array `nums` sorted in non-decreasing order, remove some duplicates in-place such that each unique element appears at most twice. The relative order of the elements should be kept the same.
Since it is impossible to change the length of the array in some languages, you must instead have the result be placed in the first part of the array `nums`. More formally, if there are `k` elements after removing the duplicates, then the first `k` elements of `nums` should hold the final result. It does not matter what you leave beyond the first `k` elements.
Return `k` after placing the final result in the first `k` slots of `nums`.
Do not allocate extra space for another array. You must do this by modifying the input array in-place with O(1) extra memory.

**Algorithm:**
Use a pointer `k` to represent the position where the next valid element should be placed. Initialize `k = 2`, since the first two elements can always be kept. Iterate from index `2`. Check if the current element `nums[i]` is different from `nums[k - 2]`. If it is, it means we haven't seen this element more than twice. Place it at `nums[k]` and increment `k`.
*   **Time Complexity:** O(n)
*   **Space Complexity:** O(1)

**Java Code:**
```java
class Solution {
    public int removeDuplicates(int[] nums) {
        if (nums.length <= 2) return nums.length;
        int k = 2;
        for (int i = 2; i < nums.length; i++) {
            if (nums[i] != nums[k - 2]) {
                nums[k] = nums[i];
                k++;
            }
        }
        return k;
    }
}
```

---

### 169. Majority Element
**Description:**
Given an array `nums` of size `n`, return the majority element.
The majority element is the element that appears more than `⌊n / 2⌋` times. You may assume that the majority element always exists in the array.

**Algorithm:**
Boyer-Moore Voting Algorithm: Maintain a `candidate` and a `count`. Iterate through the array. If `count` is `0`, pick the current element as the new `candidate`. Then, if the current element equals the `candidate`, increment `count`, otherwise decrement `count`. Because the majority element appears more than n/2 times, its count will outnumber all other elements combined.
*   **Time Complexity:** O(n)
*   **Space Complexity:** O(1)

**Java Code:**
```java
class Solution {
    public int majorityElement(int[] nums) {
        int count = 0;
        Integer candidate = null;

        for (int num : nums) {
            if (count == 0) {
                candidate = num;
            }
            count += (num == candidate) ? 1 : -1;
        }

        return candidate;
    }
}
```

---

### 189. Rotate Array
**Description:**
Given an integer array `nums`, rotate the array to the right by `k` steps, where `k` is non-negative.

**Algorithm:**
Reverse the entire array. Then, reverse the first `k` elements. Finally, reverse the remaining `n - k` elements. (Note: `k` should be modulo `nums.length` to handle cases where `k` is greater than the array size).
*   **Time Complexity:** O(n)
*   **Space Complexity:** O(1)

**Java Code:**
```java
class Solution {
    public void rotate(int[] nums, int k) {
        k = k % nums.length;
        reverse(nums, 0, nums.length - 1);
        reverse(nums, 0, k - 1);
        reverse(nums, k, nums.length - 1);
    }
    
    private void reverse(int[] nums, int start, int end) {
        while (start < end) {
            int temp = nums[start];
            nums[start] = nums[end];
            nums[end] = temp;
            start++;
            end--;
        }
    }
}
```

---

### 121. Best Time to Buy and Sell Stock
**Description:**
You are given an array `prices` where `prices[i]` is the price of a given stock on the `ith` day.
You want to maximize your profit by choosing a single day to buy one stock and choosing a different day in the future to sell that stock.
Return the maximum profit you can achieve from this transaction. If you cannot achieve any profit, return 0.

**Algorithm:**
Maintain two variables: `minPrice` (initialized to infinity) and `maxProfit` (initialized to 0). Iterate through the prices. If the current price is less than `minPrice`, update `minPrice`. Otherwise, calculate the profit if sold today (`price - minPrice`) and update `maxProfit` if this profit is greater than the current `maxProfit`.
*   **Time Complexity:** O(n)
*   **Space Complexity:** O(1)

**Java Code:**
```java
class Solution {
    public int maxProfit(int[] prices) {
        int minPrice = Integer.MAX_VALUE;
        int maxProfit = 0;
        
        for (int price : prices) {
            if (price < minPrice) {
                minPrice = price;
            } else if (price - minPrice > maxProfit) {
                maxProfit = price - minPrice;
            }
        }
        
        return maxProfit;
    }
}
```
